# Manual de despliegue — C.O.R.T.E.

**Cervecería Cuello Negro** · **Edición:** 1.1 · **Fecha:** 9 de octubre de 2026

Cómo publicar una versión nueva de C.O.R.T.E. en el servidor del taller. Para instalar el entorno de desarrollo en tu computador, usa `INSTALACION.md` del repositorio `nexo_corte`.

## Flujo de trabajo

```
PR mergeado en main          GitHub Actions             Tu computador             Servidor del taller
(frontend_corte o     ──>    construye las imágenes ──> ./deploy.sh        ──>    descarga las imágenes
 backend_corte)              y las publica en GHCR      (por SSH)                 y reinicia los contenedores
```

Producción queda en `https://corte-cuellonegro.inf.uach.cl`. Fuera de la red de la universidad se accede con la VPN de la UACh.

## Antes de empezar

- Acceso SSH al servidor: `grupo2@146.83.216.166`, con la clave del grupo.
- El repositorio `nexo_corte` clonado en tu computador.
- `gh` (GitHub CLI) con sesión iniciada, o acceso a la pestaña *Actions* de `nexo_corte` en GitHub.

## Pasos

### 1. Mergear los cambios en `main`

Los cambios entran a producción solo desde la rama `main` de `frontend_corte` y `backend_corte`, mediante un PR aprobado.

### 2. Construir las imágenes

Un merge en `frontend_corte` o `backend_corte` **no** lanza la construcción. Hay que lanzarla a mano en `nexo_corte`:

```bash
gh workflow run build.yml -R SerruchosDevTeam/nexo_corte --ref main
```

También se puede hacer desde GitHub → `nexo_corte` → *Actions* → *Build and Push Docker Images to GHCR* → *Run workflow*. Un push a `main` de `nexo_corte` también la lanza.

Espera a que termine con éxito (unos 4 minutos):

```bash
gh run watch -R SerruchosDevTeam/nexo_corte
```

### 3. Desplegar

Desde tu computador, en la carpeta de `nexo_corte`:

```bash
./deploy.sh
```

El script se conecta por SSH al servidor, copia `docker-compose.server.yml`, descarga las imágenes nuevas y reinicia solo los contenedores que cambiaron. Si la descarga falla, se detiene y producción sigue con la versión anterior.

### 4. Verificar

```bash
curl -s https://corte-cuellonegro.inf.uach.cl/api/health
```

Debe responder `"status":"OK","database":"Connected"`. Después, en el navegador:

1. Recarga con **Ctrl+Shift+R** para no ver la versión anterior en caché.
2. Inicia sesión con una cuenta de prueba y revisa que se vea el cambio que desplegaste.

## Si algo falla

| Síntoma | Qué hacer |
|---|---|
| `deploy.sh` dice que falta `.env` | El servidor necesita `~/nexo_corte/.env`. Créalo una sola vez a partir de `.env.server.example` y complétalo (incluida `JWT_SECRET`). |
| `deploy.sh` falla al descargar de `ghcr.io` | Problema momentáneo de red del servidor: vuelve a ejecutarlo. |
| No se ve el cambio | Recarga sin caché. Si sigue igual, revisa que el workflow del paso 2 haya terminado **después** del merge. |
| 502 o el sitio no carga | Revisa los contenedores: `ssh grupo2@146.83.216.166 "cd nexo_corte && docker-compose ps"` y `ssh grupo2@146.83.216.166 "docker logs --tail 100 grupo2_backend"`. |

**Volver atrás:** revierte el PR en `main` (PR de *revert*) y repite los pasos 2 a 4.

## Versión desplegada

Sprint 1: tag `v0.1.0-sprint1` (`frontend_corte@0f3cb65`, `backend_corte@9f3fc26`, `nexo_corte@bd9d727`). Verificado en producción el 09-10-2026.

> **Cuidado:** `main` de `frontend_corte` todavía no incluye el despacho desde la Vista de Cámara (PR #12, revertido el 05/10). Si se construye sin corregirlo, ese botón desaparece de producción.
