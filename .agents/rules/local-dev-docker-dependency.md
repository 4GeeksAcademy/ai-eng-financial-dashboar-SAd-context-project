# Regla: El proxy de Vite depende del hostname `backend`, solo resoluble dentro de Docker Compose

## Propósito
Evitar que un agente o desarrollador intente correr `npm run dev` fuera de Docker y asuma que el proxy `/api` funcionará sin ajustes.

## Alcance (Scope)
`frontend/vite.config.ts` y la documentación de arranque local (`README.md`).

## Justificación basada en el código base
[frontend/vite.config.ts](../../frontend/vite.config.ts) líneas 11-16 configuran el proxy `/api` apuntando a `http://backend:8000`. El hostname `backend` solo resuelve dentro de la red interna creada por [docker-compose.yml](../../docker-compose.yml). [README.md](../../README.md) líneas 41-43 solo documenta `docker compose up --build` como forma de correr el proyecto; no hay instrucciones para correr frontend y backend de forma nativa (sin contenedores).

## Guía específica y accionable
- Ejecutar el proyecto localmente solo vía `docker compose up --build`, salvo que se ajuste explícitamente la configuración.
- Si se necesita correr el frontend fuera de Docker (`npm run dev` directo), definir `VITE_API_BASE_URL=http://localhost:8000` copiando `frontend/.env.example` a `.env` (mecanismo ya documentado en README.md líneas 45-46), ya que el proxy de Vite no podrá resolver `backend`.
- No hardcodear `http://localhost:8000` dentro de `vite.config.ts` como "solución universal", porque rompería el flujo de Docker Compose donde `backend` sí resuelve pero `localhost:8000` del contenedor frontend no apunta al backend.

## Prueba de validación propuesta
Tarea concreta: detener el contenedor backend (`docker compose stop backend`) y ejecutar `npm run dev` directamente en `frontend/` fuera de Docker, confirmando el error `ENOTFOUND backend` o similar en la consola de Vite al llamar `/api/metrics` — documentando el error real como evidencia de la dependencia.
