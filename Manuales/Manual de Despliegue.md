# Manual de Despliegue — C.O.R.T.E.

**Cervecería Cuello Negro**

**Edición:** 1.0 · **Fecha:** 2 de octubre de 2026

**Alcance documentado:** despliegue actual en el servidor del taller INFO282 (CI/CD con GitHub Actions + GHCR, `deploy.sh` y Docker Compose detrás del proxy Caddy del curso) y las recomendaciones para el traspaso a la infraestructura del cliente al término del convenio (Convenio, cláusula 4). Basado en el SRS Técnico v0.3 (§48 variables, §49 entornos, §50 despliegue, §51 infraestructura, §52 backups, §53 recuperación, §54 seguridad) y en el estado del repositorio NEXO a octubre de 2026.

Este manual está dirigido al rol **DevOps** del equipo y al **equipo de TI del cliente**. Describe cómo construir, publicar y levantar el sistema, cómo verificarlo, cómo respaldarlo y qué se requiere para operarlo de forma autónoma después de la entrega. Los valores sensibles (claves, tokens) **nunca** se escriben en este documento ni en el repositorio: se leen de variables de entorno.

## Índice

1. [Visión general y entornos](#vision)
2. [Requisitos previos](#requisitos)
3. [Variables de entorno y secretos](#variables)
4. [Despliegue en el servidor del taller](#despliegue-taller)
5. [Base de datos y datos iniciales](#base-datos)
6. [Verificación posterior al despliegue](#verificacion)
7. [Respaldos y recuperación](#respaldos)
8. [Traspaso a la infraestructura del cliente](#traspaso)
9. [Seguridad técnica](#seguridad)
10. [Resolución de problemas](#problemas)
11. [Anexo: control del documento](#control)

<a id="vision"></a>

## 1. Visión general y entornos

El sistema se despliega como **tres contenedores Docker** detrás de un proxy inverso:

```text
Internet
   │
Caddy (proxy inverso del servidor del taller · red red_taller_software · HTTP)
   ├── grupo2_frontend   (Next.js 14 standalone · puerto 3002 · usuario sin privilegios)
   ├── grupo2_backend    (Express 5 + Prisma · puerto 4002)
   │        │
   └────────┴── grupo2_db   (MySQL 8.0 · puerto 3306 · volumen grupo2_db_data)
```

El flujo de despliegue va desde el código hasta el servidor pasando por **GitHub Actions** (construye las imágenes) y **GHCR** (las publica):

```text
git push a main (NEXO) → GitHub Actions (build.yml) → imágenes corte-frontend / corte-backend
   → GHCR (:latest) → ./deploy.sh (SSH al servidor) → docker-compose pull + up -d → Caddy publica el servicio
```

**Entornos del proyecto:**

| Entorno | Propósito | Cómo se levanta | URL |
|---|---|---|---|
| Desarrollo local (híbrido) | Programar con recarga en caliente | MySQL en Docker; backend `npm run dev`; frontend `pnpm dev` (ver `INSTALACION.md`) | `http://localhost:3000` y `:3001` |
| Prueba local del build | Verificar la imagen del backend | `docker compose up -d --build` en `backend_corte/` | `http://localhost:3001` |
| Producción (taller) | Servicio del proyecto | `deploy.sh` + `docker-compose.server.yml` | `http://grupo2.146.83.216.166.nip.io` |
| Producción definitiva (cliente) | Operación tras el convenio | Por definir con TI del cliente (ver §8) | Por definir, con HTTPS |

> **Nota de alcance:** el servidor del taller está disponible hasta **enero de 2027** (Charter §E). Después, el sistema se traspasa al cliente; la infraestructura definitiva se acuerda con su TI (DPO-032).

**Resultado esperado:** comprendes qué piezas se despliegan, por dónde pasa el tráfico y en qué entorno estás trabajando.

<a id="requisitos"></a>

## 2. Requisitos previos

| Herramienta / acceso | Detalle |
|---|---|
| Acceso SSH al servidor | `grupo2@146.83.216.166` con la clave del equipo DevOps |
| Docker Engine + Docker Compose | En el servidor del taller ya están instalados (Compose v1 vía `docker-compose`) |
| Acceso a GHCR | Token `GH_PAT` con permisos de lectura de paquetes, para `docker-compose pull` |
| Repositorio NEXO clonado en el servidor | En `~/nexo_corte`, con `deploy.sh` y `docker-compose.server.yml` |
| Archivo `.env` en el servidor | En `~/nexo_corte/.env`, con permisos `600` (ver §3) |

> **Importante:** no se necesita instalar MySQL ni Node a mano en el servidor; todo corre dentro de los contenedores.

**Resultado esperado:** tienes acceso SSH, el repositorio en el servidor y las credenciales listas.

<a id="variables"></a>

## 3. Variables de entorno y secretos

Toda la configuración se inyecta por variables de entorno. **Ningún secreto se guarda en el repositorio.** Las plantillas válidas son `.env.server.example`, `Base de datos/.env.example` y `backend_corte/.env.example`.

> **Advertencia (DT-012):** el `.env.example` de la raíz del NEXO es una plantilla genérica del curso (menciona Vite, `grupo0`, puerto 4000) y **no** corresponde a este stack. No la uses como referencia.

Variables principales:

| Variable | Componente | Descripción | Ejemplo | Sensible |
|---|---|---|---|:---:|
| `DATABASE_URL` | Backend | Conexión de Prisma | `mysql://<usuario>:<clave>@grupo2_db:3306/corte_db` | **Sí** |
| `JWT_SECRET` | Backend | Secreto para firmar los tokens | Valor aleatorio ≥ 32 bytes | **Sí** |
| `JWT_EXPIRES_IN` | Backend | Duración del token | `8h` | No |
| `FRONTEND_URL` | Backend | Origen permitido por CORS | el valor de `DOMAIN` | No |
| `NEXT_PUBLIC_API_URL` | Frontend (build) | URL base de la API. **Debe terminar en `/api`** | `http://grupo2.146.83.216.166.nip.io/api` | No |
| `MYSQL_ROOT_PASSWORD` | BD | Clave de root | — | **Sí** |
| `MYSQL_DATABASE` | BD | Nombre de la base | `corte_db` | No |
| `MYSQL_USER` / `MYSQL_PASSWORD` | BD | Usuario de la aplicación | `corte_user` / — | **Sí** |
| `PORT_BACKEND` / `PORT_FRONTEND` | Producción | Puertos internos que usa Caddy | `4002` / `3002` | No |
| `DOMAIN` | Producción | URL pública (CORS y build del frontend) | `http://grupo2.146.83.216.166.nip.io` | No |
| `GH_PAT` | CI | Token para clonar subrepos y publicar/leer en GHCR | — | **Sí** |
| `SERVER` / `REMOTE_DIR` | `deploy.sh` | Destino del despliegue por SSH | `grupo2@146.83.216.166` / `nexo_corte` | No |

> **Deuda técnica (RNF-SEG-005):** hoy `JWT_SECRET` **no figura** en ninguna plantilla ni en los compose, y el código usa un valor embebido si falta. **Hay que definirlo explícitamente** en el `.env` del servidor con un valor aleatorio de ≥ 32 bytes. Generar uno: `openssl rand -base64 48`.

**Resultado esperado:** el archivo `~/nexo_corte/.env` existe, tiene todas las variables obligatorias (incluido `JWT_SECRET`) y permisos `600`.

<a id="despliegue-taller"></a>

## 4. Despliegue en el servidor del taller

### 4.1. Opción A — con CI/CD (recomendada)

1. Haz **push a `main` del repositorio NEXO** (o ejecuta el *workflow* `build.yml` a mano desde GitHub Actions). GitHub Actions:
   1. sincroniza la rama principal de los subrepositorios (`actualizar_repos_local.sh`);
   2. construye las dos imágenes, inyectando `NEXT_PUBLIC_API_URL`;
   3. las publica en GHCR como `ghcr.io/serruchosdevteam/corte-frontend:latest` y `corte-backend:latest`.
2. Conéctate por SSH al servidor y entra al directorio del despliegue:
   ```bash
   ssh grupo2@146.83.216.166
   cd ~/nexo_corte
   ```
3. Ejecuta el script de despliegue:
   ```bash
   ./deploy.sh
   ```
   El script: verifica que exista `~/nexo_corte/.env`, copia `docker-compose.server.yml` como `docker-compose.yml`, ejecuta `docker-compose pull` y `docker-compose up -d`, y muestra el estado de los contenedores.
4. Al arrancar, el backend ejecuta `prisma db push`, que **sincroniza el esquema sin migraciones**.

> **Aviso:** un push a `frontend_corte` o `backend_corte` **no** dispara la CI. Hay que hacer push al NEXO o ejecutar el *workflow* manualmente. Además, `NEXT_PUBLIC_API_URL` queda **fija en la imagen**: cambiarla exige reconstruir.

### 4.2. Opción B — construir en el servidor

Si no se usa GHCR, se puede construir en el propio servidor, en el directorio donde están clonados los subrepositorios:

```bash
cd ~/nexo_corte
docker compose up -d --build
```

**Resultado esperado:** los tres contenedores (`grupo2_db`, `grupo2_backend`, `grupo2_frontend`) quedan en ejecución y Caddy publica el servicio en el dominio del taller.

<a id="base-datos"></a>

## 5. Base de datos y datos iniciales

- La base corre en el contenedor `grupo2_db` (MySQL 8.0), con los datos persistidos en el volumen `grupo2_db_data`.
- El esquema se sincroniza con **`prisma db push`** al arrancar el backend (no hay migraciones versionadas todavía).

> **Riesgo (RSK-012):** el **seed NO debe ejecutarse en producción** — borra todos los datos y crea usuarios de prueba con contraseñas débiles. El seed es solo para desarrollo local y pruebas.

> **Riesgo del esquema:** `prisma db push` puede fallar, o **perder datos** si se fuerza, ante cambios destructivos del esquema. **Respalda antes de cada despliegue** (§7). Se recomienda migrar a migraciones versionadas de Prisma.

**Resultado esperado:** la base existe, el esquema está sincronizado y no se ejecutó el seed en producción.

<a id="verificacion"></a>

## 6. Verificación posterior al despliegue

1. Revisa el estado de los contenedores:
   ```bash
   docker-compose ps      # los tres deben estar "Up"; grupo2_db, "healthy"
   ```
2. Comprueba el endpoint de salud del backend:
   ```bash
   curl http://grupo2.146.83.216.166.nip.io/api/health
   # Esperado: 200 {"success":true,"timestamp":...,"data":{"status":"OK","database":"Connected"}}
   ```
3. Abre el dominio público en el navegador e inicia sesión con una cuenta válida.
4. Verifica que la grilla de la cámara (Bodega 1) carga datos reales.

**Resultado esperado:** `/api/health` responde `200` con `database: "Connected"`, la aplicación carga y el inicio de sesión funciona.

<a id="respaldos"></a>

## 7. Respaldos y recuperación

**Situación actual:** los repositorios no definen respaldos; los datos solo viven en el volumen Docker. Esto debe corregirse antes de la entrega al cliente.

**Respaldo (propuesto):** diario, fuera del horario operativo, y **antes de cada despliegue o migración**, con `mysqldump` desde el contenedor (las credenciales se leen del `.env`):

```bash
docker exec grupo2_db sh -c 'mysqldump --single-transaction -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE"' | gzip > corte_$(date +%F).sql.gz
```

| Aspecto | Definición propuesta |
|---|---|
| Retención | 7 diarios, 4 semanales, 6 mensuales |
| Ubicación | Fuera del servidor (nube o almacenamiento del cliente); nunca solo en el mismo volumen |
| Cifrado | En reposo con AES-256 (`age` o `gpg`), clave custodiada por TI del cliente |
| Restauración | Prueba mensual documentada (tiempo y resultado) |

**Recuperación ante desastres (metas a acordar con el cliente):** RPO 24 h (1 h si se activan los *binary logs*); RTO 4 h hábiles. Procedimiento: preparar servidor con Docker, recuperar el `.env`, desplegar la última versión estable, restaurar el último respaldo válido, verificar `/api/health` e inicio de sesión, y apuntar el DNS/proxy al nuevo servidor.

**Resultado esperado:** existe un respaldo reciente y probado antes de cualquier cambio en producción.

<a id="traspaso"></a>

## 8. Traspaso a la infraestructura del cliente

Al término del convenio el sistema se traspasa al cliente (DPO-032). El despliegue actual depende de recursos del curso (red `red_taller_software`, nombres `grupo2_*`, el Caddy del curso y el dominio `nip.io` sin HTTPS), por lo que **no es portable tal cual**. Para operar de forma autónoma se requiere:

| Elemento | Recomendado para el cliente |
|---|---|
| Servidor | Máquina virtual Linux dedicada (Ubuntu LTS o Debian) |
| CPU / RAM | 2 vCPU · 4 GB (MySQL ~1 GB, cada proceso Node ~512 MB) |
| Almacenamiento | 40 GB SSD + respaldo externo |
| Red | Red Docker interna; solo el puerto 443 expuesto, con firewall |
| Contenedores | Docker Engine 24+ con Compose v2 |
| Proxy y TLS | Caddy o Nginx **propio**, con certificado automático (Let's Encrypt) y el dominio del cliente |

Pasos de portabilidad:

1. Reemplazar la red externa `red_taller_software` por una red Docker propia y renombrar los servicios `grupo2_*` a nombres neutros.
2. Incorporar un proxy inverso propio (Caddy/Nginx) con **HTTPS** y redirección de HTTP (80) a HTTPS (443).
3. Fijar `DOMAIN`, `FRONTEND_URL` y `NEXT_PUBLIC_API_URL` con `https://` y el dominio del cliente (recordar que `NEXT_PUBLIC_API_URL` se hornea en el build del frontend).
4. Definir secretos nuevos y fuertes por entorno (ver §9) y entregarlos formalmente a TI del cliente.
5. Configurar respaldos (§7) y monitoreo de `/api/health`.

**Resultado esperado:** existe una ruta clara para levantar el sistema en infraestructura del cliente sin depender de los recursos del curso.

<a id="seguridad"></a>

## 9. Seguridad técnica

| Tema | Estado actual | Requisito para producción |
|---|---|---|
| HTTPS/TLS | No hay (HTTP, dominio `nip.io`): credenciales y tokens viajan sin cifrar | TLS 1.2+ de extremo a extremo; HSTS ≥ 6 meses; redirección 80 → 443 |
| Secreto JWT | `JWT_SECRET` ausente de plantillas; el código usa un valor embebido si falta | Secreto aleatorio ≥ 32 bytes, distinto por entorno, solo por variable de entorno |
| Gestión de secretos | `.env` fuera de git, pero con valores de ejemplo | `.env` con permisos `600`; rotación semestral y ante salida de un integrante; `GH_PAT` con permisos mínimos y expiración |
| CORS | Restringido a `FRONTEND_URL` + `localhost` | Quitar `localhost` en producción |
| Contraseñas | bcrypt (costo 12; el seed usa 10) | Política única 12–72 caracteres; cambio obligatorio en el primer acceso; bloqueo tras varios intentos |
| Contenedores | Frontend como usuario `nextjs`; backend como `root` | Ejecutar también el backend sin privilegios |

> **Prioridad de seguridad para la entrega:** definir `JWT_SECRET`, habilitar HTTPS y configurar respaldos cifrados son los tres puntos críticos antes del traspaso al cliente.

**Resultado esperado:** los secretos están definidos y protegidos, y hay un plan para habilitar HTTPS en la producción definitiva.

<a id="problemas"></a>

## 10. Resolución de problemas

| Situación o mensaje | Qué hacer |
|---|---|
| `deploy.sh` falla indicando que falta `.env` | Crea `~/nexo_corte/.env` a partir de `.env.server.example` con todas las variables obligatorias (§3). |
| `docker-compose pull` falla por autenticación | Verifica el `GH_PAT` y haz `docker login ghcr.io` con ese token. |
| El backend no arranca: `Can't reach database server` | Revisa que `grupo2_db` esté `healthy` (`docker-compose ps`) y que `DATABASE_URL` apunte a `grupo2_db:3306`. |
| `/api/health` responde pero `database` no es `Connected` | Revisa credenciales de la BD en el `.env` y los logs: `docker logs grupo2_backend`. |
| El frontend pega contra la API equivocada | `NEXT_PUBLIC_API_URL` quedó fija en la imagen; corrígela y **reconstruye** la imagen del frontend. |
| Un push a backend/frontend no desplegó nada | La CI solo se dispara desde el NEXO; haz push al NEXO o ejecuta el *workflow* a mano. |
| Necesitas volver a una versión anterior | Solo existe la etiqueta `latest`; hay que re-etiquetar a mano. Se propone usar etiquetas por versión/commit. |
| Cambios del esquema rompieron datos | Restaura el último respaldo (§7). Nunca fuerces `prisma db push` en producción sin respaldo. |

**Resultado esperado:** identificas la causa y aplicas la corrección sin perder datos.

<a id="control"></a>

## 11. Anexo: control del documento

Este manual se basa en el SRS Técnico v0.3 (§48–§54) y en el estado del repositorio NEXO a octubre de 2026. Refleja el despliegue **vigente** en el servidor del taller y las **recomendaciones** para el traspaso al cliente; los puntos marcados como "propuesto", "deuda técnica" o "requisito para producción" aún no están implementados y deben abordarse antes de la entrega formal.

Observaciones heredadas registradas en el SRS: `JWT_SECRET` ausente de las plantillas (RNF-SEG-005), producción sin HTTPS (RNF-SEG-004), ausencia de respaldos automáticos (§52) y el Manual de Despliegue previamente vacío (DT-020). Esta edición cubre la redacción del manual; la verificación de cada procedimiento contra el servidor queda pendiente de una revisión cruzada del equipo.
