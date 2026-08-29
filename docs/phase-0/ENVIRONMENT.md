# Entorno compartido · verificación 0.6

Objetivo: los cuatro integrantes ejecutan frontend y backend sin secretos
commiteados. `DEMO_TOKEN` es un token compartido **solo para la demo**, no una
credencial de producción.

## Variables requeridas

Backend (`Hack-a-ton-end/.env`, creado desde `.env.example`):

```env
DEMO_TOKEN=replace-with-a-shared-demo-token
```

Frontend (`Hack-a-ton-front/.env`, creado desde `.env.example`):

```env
VITE_DEMO_TOKEN=replace-with-a-shared-demo-token
```

Los valores deben coincidir. Las plantillas se commitean; los archivos `.env`
reales no. Al ser una aplicación Vite, `VITE_DEMO_TOKEN` queda visible en el
bundle: solo protege el handshake de la demo y no sustituye autenticación real.

## Backend

Desde `Hack-a-ton-end`:

```bash
cp .env.example .env
docker compose -f docker/docker-compose.yml up --build -d
curl --fail http://localhost:8000/ready
docker compose -f docker/docker-compose.yml down
```

Resultado esperado: HTTP 200 y `{"status":"ready"}`.

## Frontend

Con el backend disponible en `http://localhost:8000`, desde
`Hack-a-ton-front`:

```bash
cp .env.example .env
docker compose up --build -d
curl --fail http://localhost:3000/
docker compose down
```

Resultado esperado: HTTP 200. El compose del frontend levanta solo el frontend;
no usa una imagen placeholder ni administra el ciclo de vida del backend.

## Sign-off del equipo

Cada integrante registra fecha, commit probado y resultado. La verificación
técnica de un entorno no sustituye el sign-off de las otras tres máquinas.

| Integrante | Fecha | Front commit | Back commit | Resultado |
|---|---|---|---|---|
| Lane B/D | 2026-08-29 | `0d91656` + working tree Phase 0 | `6c16e15` + working tree Phase 0 | PASS: build, containers, HTTP 200 y token presente |
| Lane A | — | — | — | Pendiente confirmación |
| Lane C | — | — | — | Pendiente confirmación |
| Cuarto integrante | — | — | — | Pendiente confirmación |

No se marca el DoD humano “los 4 entornos” como verificado hasta completar esta
tabla. El código y los comandos reproducibles sí forman parte de 0.6.

### Evidencia local 2026-08-29

- Backend construido desde el Dockerfile actualizado.
- PostgreSQL y backend reportaron `healthy`.
- `DEMO_TOKEN` estuvo presente dentro del contenedor, sin imprimir su valor.
- `GET /ready` devolvió `{"status":"ready"}`.
- Frontend construido con Docker y servido por Nginx; `/` devolvió HTTP 200.
- Se usó `BACKEND_PORT=18000` porque el entorno integrado existente ya ocupaba
  8000; la configuración default permanece en 8000.
- Los stacks temporales se bajaron con `docker compose down`, preservando el
  volumen de PostgreSQL y sin tocar el stack integrado preexistente.
