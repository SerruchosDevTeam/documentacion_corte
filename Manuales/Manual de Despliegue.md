# Manual de despliegue — C.O.R.T.E.

**Cervecería Cuello Negro**

**Edición:** 1.1 · **Fecha:** 9 de octubre de 2026

**Versión documentada:** tag `v0.1.0-sprint1` en los tres repositorios: `nexo_corte@bd9d727`, `frontend_corte@0f3cb65` y `backend_corte@9f3fc26`. Las comprobaciones de la sección 8 se ejecutaron contra producción el 09-10-2026.

> **Versión desplegada al 09-10-2026.** La última construcción de imágenes (GitHub Actions, 05-10-2026 10:26) usó `frontend_corte@0f3cb65` (igual al tag) y `backend_corte@d652f07` (mismo contenido que el tag).
>
> **Cuidado:** `main` de `frontend_corte` (`01ca318`) **no** es igual al tag: el rollback del 05/10 revirtió también el despacho desde la Vista de Cámara (PR #12). Si se lanza el workflow sin corregir `main`, ese botón desaparece de producción.

Este manual explica cómo está desplegado C.O.R.T.E. en el servidor del taller, cómo publicar una versión nueva y cómo verificarla. Incluye la arquitectura del backend y el contrato de respuestas de la API, que corresponden a [TECH-02 — Arquitectura de API y respuestas estándar](https://trello.com/c/YLFTH2ls).

Está dirigido al equipo de desarrollo y a quien administre el servidor. Para instalar el entorno de desarrollo en tu computador, usa `INSTALACION.md` del repositorio `nexo_corte`.

## Índice

1. [Arquitectura del despliegue](#arquitectura)
2. [Arquitectura del backend](#backend)
3. [Contrato de respuestas de la API](#contrato)
4. [Variables de entorno](#variables)
5. [Construir las imágenes (GitHub Actions)](#build)
6. [Desplegar en el servidor](#deploy)
7. [Primer despliegue en un servidor nuevo](#primer-despliegue)
8. [Verificar un despliegue](#verificar)
9. [Resolver problemas](#problemas)
10. [Volver a una versión anterior](#rollback)
11. [Respaldos y recuperación](#respaldos)
12. [Seguridad técnica](#seguridad)
13. [Traspaso a la infraestructura del cliente](#traspaso)

<a id="arquitectura"></a>

## 1. Arquitectura del despliegue

| Componente | Detalle |
|---|---|
| Dirección pública | `https://corte-cuellonegro.inf.uach.cl` (certificado Let's Encrypt, TLS 1.2 y 1.3; HTTP redirige a HTTPS) |
| Proxy | Caddy del curso, en la red Docker compartida `red_taller_software`. No lo administra el equipo |
| Frontend | Contenedor `grupo2_frontend`: Next.js 14 (modo *standalone*), puerto interno `PORT_FRONTEND` (3002) |
| Backend | Contenedor `grupo2_backend`: Express 5 + Prisma 5, puerto interno `PORT_BACKEND` (4002) |
| Base de datos | Contenedor `grupo2_db`: MySQL 8.0, volumen `grupo2_db_data` |
| Imágenes | GitHub Container Registry: `ghcr.io/serruchosdevteam/corte-frontend:latest` y `corte-backend:latest` |
| Carpeta en el servidor | `~/nexo_corte`, con `docker-compose.yml` (copia de `docker-compose.server.yml`) y `.env` |

**Enrutamiento de Caddy:**
- `https://corte-cuellonegro.inf.uach.cl/api/*` → `grupo2_backend`, **sin quitar** el prefijo `/api`.
- Cualquier otra ruta → `grupo2_frontend`.

Por eso el navegador llama a la API en el mismo dominio (`/api/...`) y la ruta interna de Next `/api/auth/login` **no se ejecuta en producción**: esa petición la atiende Express.

```
Navegador ──HTTPS──> Caddy ──/api/*──> grupo2_backend (Express) ──> grupo2_db (MySQL)
                           └──resto───> grupo2_frontend (Next.js)
```

Los contenedores no publican puertos al host. Solo Caddy los alcanza, por su nombre, dentro de `red_taller_software`. Los nombres `grupo2_*` son obligatorios: Caddy enruta con ellos y evitan choques con los contenedores de otros grupos.

**Entornos:**

| Entorno | Para qué | Cómo se levanta | URL |
|---|---|---|---|
| Desarrollo local | Programar con recarga en caliente | MySQL en Docker; backend con `npm run dev`; frontend con `pnpm dev` (ver `INSTALACION.md`) | `http://localhost:3000` (frontend) y `http://localhost:3001` (API) |
| Producción (taller) | Servicio del proyecto y demos al cliente | `deploy.sh` con `docker-compose.server.yml` | `https://corte-cuellonegro.inf.uach.cl` |
| Producción definitiva (cliente) | Operación después del convenio | Por definir con TI del cliente (sección 13) | Por definir |

El servidor del taller está disponible hasta enero de 2027 (Project Charter). Fuera de la red de la universidad se accede con la VPN de la UACh.

**Repositorios:**

| Repositorio | Contenido |
|---|---|
| `SerruchosDevTeam/nexo_corte` | Orquestador: workflow de GitHub Actions, `docker-compose*.yml`, `deploy.sh`, plantillas `.env` |
| `SerruchosDevTeam/frontend_corte` | Aplicación Next.js |
| `SerruchosDevTeam/backend_corte` | API Express, esquema y seed de Prisma, pruebas |
| `SerruchosDevTeam/documentacion_corte` | SRS, manuales y documentos del proyecto |

<a id="backend"></a>

## 2. Arquitectura del backend

**Tecnologías:** Node.js 20, Express 5, Prisma 5 (MySQL 8), Zod 4 para validar, `jsonwebtoken` para las sesiones y `bcryptjs` para las contraseñas.

**Estructura de `backend_corte/src`:**

| Carpeta | Responsabilidad |
|---|---|
| `index.ts` | Crea la aplicación, configura CORS y JSON, monta las rutas bajo `/api` y el manejador 404 |
| `routes/` | Define los endpoints de cada recurso y los middlewares que aplican |
| `middlewares/` | `auth.middleware` (exige y valida el JWT), `authorization.middleware` (exige un cargo, por ejemplo Jefe de Planta activo), `validate.middleware` (valida el cuerpo con Zod) |
| `schemas/` | Esquemas Zod de cada recurso (`auth`, `usuarios`, `profile`, `config`, `pallet-operations`) |
| `controllers/` | Reciben la solicitud, llaman al servicio y responden con el formato estándar |
| `services/` | Lógica de negocio y transacciones con Prisma |
| `lib/` | `apiResponse` (formato de respuesta), `prisma` (cliente único), `service-error` (errores con código HTTP), `mappers` |

**Recorrido de una solicitud:**

1. Caddy entrega la solicitud a Express con la ruta completa (`/api/...`).
2. Si la ruta lo exige, `requireAuth` valida el token `Authorization: Bearer <JWT>`. Sin token o con uno inválido responde **401**.
3. Si la operación es de administración, el middleware de autorización consulta en la base de datos el cargo y el estado **actuales** de la cuenta. Si no corresponde responde **403**. Por eso un cambio de cargo se aplica de inmediato, sin esperar a que expire el token.
4. El esquema Zod valida el cuerpo. Si no cumple responde **400**.
5. El servicio ejecuta la operación, en una transacción cuando modifica varios registros (por ejemplo, un despacho múltiple es todo o nada).
6. El controlador responde con `sendSuccess` o `sendError` (sección 3).

**Sesiones:** el JWT incluye identificador, RUT, correo y rol, dura `JWT_EXPIRES_IN` (8 horas por defecto) y se firma con `JWT_SECRET`. El frontend lo guarda en el navegador y restaura la sesión al recargar. Cerrar sesión borra el token del navegador, pero el servidor no lo revoca.

**Arranque del contenedor:** ejecuta `prisma db push` y luego `node dist/index.js`. `db push` sincroniza el esquema con la base de datos sin migraciones versionadas. Si un cambio de esquema implica perder datos, **el comando falla y el contenedor no arranca** (sección 9).

<a id="contrato"></a>

## 3. Contrato de respuestas de la API

Todas las respuestas del backend usan este formato (`src/lib/apiResponse.ts`):

```json
{ "success": true, "data": { }, "timestamp": "2026-09-30T14:07:23.980Z" }
```

```json
{ "success": false, "error": "Credenciales incorrectas", "timestamp": "2026-09-30T14:07:24.381Z" }
```

| Campo | Cuándo aparece | Contenido |
|---|---|---|
| `success` | Siempre | `true` si la operación se completó; `false` si hubo un error |
| `data` | Solo si `success` es `true` | El resultado: un objeto, una lista o `{ "guardado": true }` en operaciones de escritura |
| `error` | Solo si `success` es `false` | Normalmente un texto en español para mostrar al usuario |
| `timestamp` | Siempre | Fecha y hora del servidor en UTC (ISO 8601) |

**Códigos HTTP:**

| Código | Significado |
|---|---|
| 200 / 201 | Operación correcta (201 al crear, por ejemplo un usuario) |
| 400 | Datos inválidos (validación Zod o de negocio) |
| 401 | Falta el token, es inválido o expiró; o credenciales incorrectas en el login |
| 403 | La cuenta no tiene el cargo o no está activa para esa operación |
| 404 | El recurso no existe |
| 409 | Conflicto: RUT o correo duplicado, pallet ya despachado, cambio concurrente, último Jefe de Planta |
| 500 | Error interno. El mensaje es genérico; el detalle queda en los registros del contenedor |
| 503 | `/api/health` no pudo consultar la base de datos |

**Endpoint de salud (`GET /api/health`, sin autenticación):**

```json
{ "success": true, "timestamp": "…", "data": { "status": "OK", "database": "Connected" } }
```

Si la base de datos no responde, devuelve **503** con `"error": { "status": "ERROR", "database": "Disconnected", "message": "…" }`.

**Excepciones conocidas (DT-020 en el SRS):**
- Una ruta inexistente responde 404 con `{ "error": "Ruta no encontrada" }`, sin `success` ni `timestamp`.
- Un cuerpo JSON mal formado responde 400 con la página HTML de error de Express.
- Algunos mensajes de validación de Zod salen en inglés (H-31).

El catálogo completo de endpoints está en el SRS, §24 (API-001 a API-035).

<a id="variables"></a>

## 4. Variables de entorno

### 4.1. En el servidor (`~/nexo_corte/.env`)

Se crea una sola vez a partir de `.env.server.example` y **nunca se sube al repositorio**.

| Variable | Uso | Ejemplo / valor |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | Contraseña de root de MySQL | Clave fuerte y única |
| `MYSQL_DATABASE` | Base de datos de la aplicación | `corte_db` |
| `MYSQL_USER` / `MYSQL_PASSWORD` | Usuario de la aplicación | `corte_user` / clave fuerte |
| `PORT_BACKEND` | Puerto interno del backend | `4002` |
| `PORT_FRONTEND` | Puerto interno del frontend | `3002` |
| `DOMAIN` | URL pública; el backend la usa para CORS | `https://corte-cuellonegro.inf.uach.cl` |
| `JWT_SECRET` | **Clave para firmar las sesiones** | Texto aleatorio de al menos 32 caracteres |
| `JWT_EXPIRES_IN` | Duración de la sesión (opcional) | `8h` |

> **Obligatorio: define `JWT_SECRET`.** Si falta, el backend firma los tokens con un valor por defecto que está escrito en el código público, y cualquiera podría fabricar una sesión de Jefe de Planta. `.env.server.example` todavía no incluye esta variable. Para generar una clave:
>
> ```bash
> openssl rand -base64 48
> ```
>
> Después de agregarla, reinicia el backend. Todas las sesiones abiertas se cerrarán.

### 4.2. Al construir el frontend (`.github/workflows/build.yml`)

| Variable | Valor | Por qué |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | `https://corte-cuellonegro.inf.uach.cl/api` | Next.js la **incrusta en el JavaScript al construir la imagen**. Cambiarla en el `.env` del servidor no tiene efecto |

Debe usar `https://` e incluir `/api`. Con `http://`, el navegador bloquea las llamadas (*Mixed Content*); sin `/api`, las rutas del frontend no coinciden con las de Express.

<a id="build"></a>

## 5. Construir las imágenes (GitHub Actions)

El workflow **Build and Push Docker Images to GHCR** (`.github/workflows/build.yml` de `nexo_corte`):

1. Clona `nexo_corte` y, con el secreto `GH_PAT`, sincroniza `backend_corte`, `frontend_corte` y `documentacion_corte` en su rama `main` (`actualizar_repos_local.sh`).
2. Construye la imagen del backend.
3. Construye la del frontend con `NEXT_PUBLIC_API_URL`.
4. Publica ambas en GHCR con la etiqueta `latest`.

**Cuándo se ejecuta:**
- Automáticamente, con cada push a `main` de **`nexo_corte`**.
- Manualmente, desde GitHub → Actions → *Run workflow*, o con:

```bash
gh workflow run build.yml -R SerruchosDevTeam/nexo_corte --ref main
```

> **Un merge en `frontend_corte` o `backend_corte` no lanza el workflow.** Después de mergear un PR en esos repositorios, lánzalo a mano. Si no, la imagen publicada no tendrá los cambios.

Para seguir la ejecución y confirmar el commit con que se construyó cada imagen:

```bash
gh run watch -R SerruchosDevTeam/nexo_corte
```

En el registro, la línea `"vcs:revision"` de cada imagen muestra el commit de `frontend_corte` o `backend_corte` que se usó.

<a id="deploy"></a>

## 6. Desplegar en el servidor

Espera a que el workflow termine con éxito. Luego, desde tu computador, en la carpeta de `nexo_corte`:

```bash
./deploy.sh
```

El script (usa `SERVER=grupo2@146.83.216.166` y `REMOTE_DIR=nexo_corte` por defecto):

1. Comprueba por SSH que exista `~/nexo_corte/.env`. Si falta, se detiene.
2. Copia `docker-compose.server.yml` como `~/nexo_corte/docker-compose.yml`.
3. Ejecuta `docker-compose pull` y `docker-compose up -d`. Solo se recrean los contenedores cuya imagen cambió.
4. Muestra `docker-compose ps`.

Si `pull` falla, el script se detiene **antes** de `up -d`: producción sigue con las imágenes anteriores y no se cae.

Necesitas acceso SSH al servidor con la clave del grupo.

<a id="primer-despliegue"></a>

## 7. Primer despliegue en un servidor nuevo

1. Comprueba que exista la red del proxy. Si no existe (solo fuera del servidor del taller):

   ```bash
   docker network create red_taller_software
   ```

2. Crea la carpeta y el `.env` a partir de la plantilla, y complétalo (sección 4.1, incluido `JWT_SECRET`):

   ```bash
   scp .env.server.example grupo2@146.83.216.166:nexo_corte/.env
   ```

3. Si las imágenes de GHCR son privadas, inicia sesión en el servidor con un token que tenga permiso `read:packages`:

   ```bash
   ssh grupo2@146.83.216.166 "docker login ghcr.io"
   ```

4. Ejecuta `./deploy.sh`. Al arrancar, el backend crea las tablas con `prisma db push`.
5. La base queda **vacía**: sin cargos, usuarios ni bodegas. Carga los datos iniciales con el procedimiento que defina el responsable. El seed de desarrollo (`npm run db:seed`) **borra todas las tablas** antes de cargar datos de ejemplo: no lo ejecutes contra una base con datos reales.

<a id="verificar"></a>

## 8. Verificar un despliegue

Ejecuta estas comprobaciones sin iniciar sesión. Ninguna modifica datos.

```bash
curl -s https://corte-cuellonegro.inf.uach.cl/api/health
```

Debe responder `{"success":true, …, "data":{"status":"OK","database":"Connected"}}`.

| Comprobación | Resultado esperado |
|---|---|
| `GET /api/health` | 200 con `database: "Connected"` |
| `GET /api/usuarios` sin token | 401 con el formato estándar |
| `POST /api/pallets/despachar-varios` sin token | 401 (confirma que la ruta existe en la imagen desplegada) |
| `POST /api/auth/login` con una contraseña incorrecta | 401 «Credenciales incorrectas o usuario inactivo» |
| `https://corte-cuellonegro.inf.uach.cl/login` | 200 y la pantalla de inicio de sesión |
| `http://corte-cuellonegro.inf.uach.cl/` | 308 que redirige a `https://` |

Después, en el navegador:

1. Recarga con **Ctrl+Shift+R** (o abre una ventana privada) para no usar JavaScript en caché.
2. Inicia sesión con una cuenta de prueba y confirma que llegas al **Panel principal** con el menú de tu perfil.
3. Recarga (F5) y confirma que la sesión se mantiene.
4. En las herramientas de desarrollo → Red, confirma que las llamadas van a `https://corte-cuellonegro.inf.uach.cl/api/...` y que no hay bloqueos *Mixed Content*.

Para confirmar qué versión del frontend se está sirviendo, compara el nombre del archivo `/_next/static/chunks/app/layout-*.js` de la página de login antes y después del despliegue: si no cambió, el servidor sigue con la imagen anterior.

<a id="problemas"></a>

## 9. Resolver problemas

| Síntoma | Causa probable | Qué hacer |
|---|---|---|
| En la consola aparecen llamadas a un dominio antiguo o **Mixed Block** | La imagen del frontend se construyó con otro `NEXT_PUBLIC_API_URL` | Corrige `build.yml` (sección 4.2), vuelve a lanzar el workflow y despliega. Cambiar el `.env` del servidor no basta |
| Después del login quedas en **Inventario sin menú** | Frontend anterior a `40e33be`: leía mal la respuesta del login en producción | Despliega una imagen construida desde `frontend_corte@40e33be` o posterior |
| `deploy.sh` falla con `TLS handshake timeout` hacia `ghcr.io` | Problema momentáneo de red del servidor | Vuelve a ejecutar `./deploy.sh`. Comprueba la conexión con `ssh grupo2@146.83.216.166 "curl -s -o /dev/null -w '%{http_code}' https://ghcr.io/v2/"` (401 es correcto). Si persiste, ver la alternativa de abajo |
| El despliegue terminó, pero no ves los cambios | Caché del navegador, o el workflow no se relanzó después del merge | Ctrl+Shift+R; compara el `layout-*.js` (sección 8); revisa en Actions el `vcs:revision` de la última imagen |
| **502** en `/api/*` o en todo el sitio | El contenedor no responde o se está reiniciando | Revisa el estado y los registros (abajo). Si el backend se reinicia en bucle, busca un error de `prisma db push` |
| 401 en `/api/actividad` en la pantalla de login | Frontend anterior a `30df2db` | Despliega una versión actual; desde esa versión no se consulta la actividad sin sesión |
| 503 en `/api/health` | El backend no alcanza MySQL | Revisa `grupo2_db` y las variables `MYSQL_*` del `.env` |

**Estado y registros de los contenedores:**

```bash
ssh grupo2@146.83.216.166 "cd nexo_corte && docker-compose ps"
```

```bash
ssh grupo2@146.83.216.166 "docker logs --tail 100 grupo2_backend"
```

**Desplegar sin que el servidor descargue de GHCR** (cuando `pull` falla repetidamente). Tu computador debe tener sesión en `ghcr.io`:

```bash
docker pull ghcr.io/serruchosdevteam/corte-frontend:latest
```

```bash
docker pull ghcr.io/serruchosdevteam/corte-backend:latest
```

```bash
docker save ghcr.io/serruchosdevteam/corte-frontend:latest ghcr.io/serruchosdevteam/corte-backend:latest | gzip | ssh grupo2@146.83.216.166 "gunzip | docker load"
```

```bash
ssh grupo2@146.83.216.166 "cd nexo_corte && docker-compose up -d"
```

En este caso usa `up -d` sin `pull`; si vuelves a ejecutar `deploy.sh`, intentará descargar de GHCR otra vez.

<a id="rollback"></a>

## 10. Volver a una versión anterior

Las imágenes solo se publican con la etiqueta `latest` (DT-011 en el SRS), así que no hay una versión anterior lista para volver a desplegar. El workflow siempre construye desde `main`, así que para revertir:

1. Revierte el commit problemático en `main` del repositorio afectado (con un PR de *revert*).
2. Lanza el workflow (sección 5) y despliega (sección 6).

**Versiones entregadas:** cada cierre de sprint se marca con un tag de git en los tres repositorios (`v0.1.0-sprint1` para el Sprint 1). Sirven de referencia para saber qué código corresponde a cada entrega y para comparar con lo desplegado:

```bash
git -C frontend_corte diff --stat v0.1.0-sprint1 <commit desplegado>
```

El commit desplegado se obtiene del `vcs:revision` del registro del workflow (sección 5).

**Recomendación pendiente:** publicar además cada imagen con la etiqueta del commit (`:sha-<commit>`), para poder volver a una versión con `docker-compose` sin reconstruir.

---

<a id="respaldos"></a>

## 11. Respaldos y recuperación

**Situación actual:** no hay respaldos automáticos. Los datos solo viven en el volumen Docker `grupo2_db_data` del servidor. Hay que resolverlo antes de la entrega al cliente.

**Respaldo manual** (antes de cada despliegue que cambie el esquema, y como mínimo una vez por semana). Las credenciales se leen del `.env` dentro del contenedor:

```bash
ssh grupo2@146.83.216.166 "docker exec grupo2_db sh -c 'mysqldump --single-transaction -u\"\$MYSQL_USER\" -p\"\$MYSQL_PASSWORD\" \"\$MYSQL_DATABASE\"'" | gzip > corte_$(date +%F).sql.gz
```

El archivo queda en tu computador, fuera del servidor. No lo subas al repositorio: contiene datos del cliente.

**Restaurar un respaldo:**

```bash
gunzip -c corte_AAAA-MM-DD.sql.gz | ssh grupo2@146.83.216.166 "docker exec -i grupo2_db sh -c 'mysql -u\"\$MYSQL_USER\" -p\"\$MYSQL_PASSWORD\" \"\$MYSQL_DATABASE\"'"
```

Después de restaurar, verifica con la sección 8.

**Propuesta para producción** (a acordar con el cliente):

| Aspecto | Propuesta |
|---|---|
| Frecuencia | Diaria, fuera del horario de operación, y antes de cada despliegue |
| Retención | 7 diarios, 4 semanales y 6 mensuales |
| Ubicación | Fuera del servidor, en almacenamiento del cliente |
| Cifrado | En reposo (`age` o `gpg`); la clave la custodia TI del cliente |
| Prueba de restauración | Mensual, con tiempo y resultado anotados |
| Metas | RPO 24 h y RTO 4 h hábiles |

<a id="seguridad"></a>

## 12. Seguridad técnica

| Tema | Estado actual | Pendiente para la entrega |
|---|---|---|
| HTTPS | Activo: Let's Encrypt, TLS 1.2 y 1.3 (TLS 1.1 rechazado) y redirección 308 de HTTP a HTTPS, a cargo del Caddy del curso | Mantenerlo en la infraestructura del cliente |
| Secreto JWT | El código usa un valor por defecto si falta `JWT_SECRET`, y `.env.server.example` no la incluye | Definirla en el `.env` del servidor (sección 4.1) y agregarla a la plantilla |
| Secretos | `.env` fuera de git | Permisos `600` en el servidor; rotar las claves al terminar el proyecto y cuando un integrante deje el equipo |
| CORS | Permite `DOMAIN` y además `http://localhost:3000` y `:3001` (`backend_corte/src/index.ts`) | Quitar los orígenes `localhost` en producción |
| Contenedores | Sin puertos publicados al host; solo Caddy los alcanza | Ejecutar el backend con un usuario sin privilegios |
| Seed | Borra todas las tablas y crea usuarios de prueba | Nunca ejecutarlo en producción |

<a id="traspaso"></a>

## 13. Traspaso a la infraestructura del cliente

Al terminar el convenio, el sistema se traspasa al cliente. El despliegue actual depende de recursos del curso (red `red_taller_software`, nombres `grupo2_*`, el Caddy del curso y el dominio `inf.uach.cl`), así que no se puede mover tal cual.

| Elemento | Recomendado |
|---|---|
| Servidor | Máquina virtual Linux (Ubuntu LTS o Debian) |
| CPU / RAM | 2 vCPU y 4 GB |
| Disco | 40 GB SSD, más respaldo externo |
| Contenedores | Docker Engine 24 o superior con Compose v2 |
| Proxy y TLS | Caddy o Nginx propio, con certificado automático y el dominio del cliente |

**Pasos:**

1. Reemplazar la red externa `red_taller_software` por una red propia del compose y dar nombres neutros a los contenedores.
2. Agregar un proxy propio con HTTPS que mantenga el enrutamiento de la sección 1 (`/api/*` al backend, el resto al frontend).
3. Cambiar `DOMAIN` y `NEXT_PUBLIC_API_URL` al dominio del cliente y **reconstruir** la imagen del frontend (sección 4.2).
4. Generar secretos nuevos (sección 12) y entregarlos formalmente a TI del cliente.
5. Restaurar el último respaldo (sección 11), configurar los respaldos automáticos y verificar con la sección 8.

---

**Control del documento**

| Edición | Fecha | Cambios |
|---|---|---|
| 1.0 | 30-09-2026 | Primera edición, a partir de `nexo_corte`, `backend_corte` y `frontend_corte` y de los despliegues del 29 y 30-09-2026 |
| 1.1 | 09-10-2026 | Versión del tag `v0.1.0-sprint1` (corregido al commit desplegado del frontend) y aviso sobre `main`; entornos; comprobaciones de la sección 8 repetidas contra producción (sin modificar datos); versiones con tags; respaldos, seguridad técnica y traspaso al cliente |
