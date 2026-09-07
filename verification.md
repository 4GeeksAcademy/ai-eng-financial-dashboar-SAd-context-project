## Resumen del proyecto

- **Qué hace:** Dashboard financiero que muestra KPIs, ingresos/egresos mensuales y % de ganancia, segmentados por tipo de negocio (B2B/B2C) y categoría. El backend expone endpoints REST (`/api/metrics`, `/summary`, `/categories/top`, `/comparison`, `/alerts`, `/b2b`, `/b2c`, `/health`) que generan datos financieros **simulados** (mock, semilla fija `42`), sin base de datos ni persistencia.
- **Cómo se conectan los componentes:** El frontend (React 19 + Vite) hace `fetch` a `/api/metrics` (y otras rutas). En desarrollo, Vite actúa como proxy interno redirigiendo `/api` hacia `http://backend:8000`, resolviendo `backend` como hostname de la red interna de Docker Compose. No se requieren variables de entorno adicionales para este flujo por defecto (confirmado en [README.md](README.md#L45-L47)).
- **Cómo se ejecuta localmente:** Vía `docker compose up --build` ([README.md](README.md#L41-L43)), que levanta dos contenedores:
  - `frontend`: puerto host `5173` (Vite dev server).
  - `backend`: puertos host `8000` (API FastAPI/Uvicorn) y `5678` (debug remoto con `debugpy`).

  No hay evidencia de un modo de ejecución nativo (sin Docker) para ambos servicios simultáneamente.

## Mapa de estructura y evidencia

| Componente | Evidencia | Detalle |
|---|---|---|
| Backend | [backend/Dockerfile](backend/Dockerfile), [backend/app/main.py](backend/app/main.py) | FastAPI servido con `uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload`; `debugpy` escuchando en `0.0.0.0:5678` |
| Frontend | [frontend/Dockerfile](frontend/Dockerfile), [frontend/vite.config.ts](frontend/vite.config.ts) | Vite + React 19, `npm run dev -- --host 0.0.0.0 --port 5173` |
| Orquestación | [docker-compose.yml](docker-compose.yml) | Servicios `frontend` (`5173:5173`) y `backend` (`8000:8000`, `5678:5678`), con `depends_on: backend` |
| Proxy dev | [frontend/vite.config.ts](frontend/vite.config.ts#L11-L16) | Vite redirige `/api` → `http://backend:8000` |
| Cliente API | [frontend/src/App.tsx](frontend/src/App.tsx#L13) | `API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? ""` — sin definir, usa rutas relativas dependientes del proxy de Vite |
| Rutas backend | [backend/app/routes.py](backend/app/routes.py) | Datos generados aleatoriamente en cada request (`generate_mock_movements`, semilla fija `42`) |
| UI Kit | [frontend/components.json](frontend/components.json) | Configuración shadcn/ui (`style: new-york`, alias `ui: @/components/ui`) sobre Tailwind CSS v4 |

## Afirmaciones clave verificadas

| # | Afirmación | Estado |
|---|---|---|
| 1 | El backend corre FastAPI vía Uvicorn en el puerto `8000` dentro del contenedor y se expone en `8000` del host. | ✅ Verificado — [docker-compose.yml](docker-compose.yml#L18-L20), [backend/Dockerfile](backend/Dockerfile#L11), confirmado además en [README.md](README.md#L48) |
| 2 | El frontend corre Vite en el puerto `5173`, expuesto igual en el host. | ✅ Verificado — [docker-compose.yml](docker-compose.yml#L6-L7), [frontend/Dockerfile](frontend/Dockerfile#L9), confirmado en [README.md](README.md#L47) |
| 3 | Existe un puerto adicional `5678` habilitado para debug remoto Python (`debugpy`). | ✅ Verificado — [docker-compose.yml](docker-compose.yml#L19), [backend/Dockerfile](backend/Dockerfile#L13) |
| 4 | El frontend usa por defecto el proxy de Vite (`/api` → `http://backend:8000`) y sólo requiere `VITE_API_BASE_URL` si se quiere apuntar a otro origen (vía `frontend/.env.example`). | ✅ Verificado — existe [frontend/.env.example](frontend/.env.example) y el mecanismo está documentado explícitamente en [README.md](README.md#L45-L46) |
| 5 | Todos los datos financieros son generados aleatoriamente en cada request (mock, semilla fija `42`), sin base de datos. | ✅ Verificado — [backend/app/routes.py](backend/app/routes.py#L91-L99); confirmado también por el test `test_generate_mock_movements_returns_full_year_sorted_data` (360 registros) |
| 6 | El proyecto tiene un directorio `.agents/rules` y `.agents/skills` con reglas/skills operativos ya definidos, según indica `AGENTS.md`. | ❌ Erróneo — esos directorios **no existen aún**. El propio [README.md](README.md#L28-L36) los describe como estructura **esperada/a crear** por el estudiante, no como algo ya presente |
| 7 | Existe un `memory-bank` con contexto de proyecto persistente. | ❌ Erróneo — el directorio no existe en el repositorio; `AGENTS.md` lo referencia condicionalmente ("si existe"), y aún no se ha creado |
| 8 | El frontend usa Tailwind CSS v4 + componentes tipo shadcn/ui (`components.json`, carpeta `ui/`). | ✅ Verificado — [frontend/components.json](frontend/components.json) confirma `style: "new-york"`, alias `ui: "@/components/ui"`, y `package.json` incluye `@tailwindcss/vite` |
| 9 | Hay tests automatizados en backend (`pytest`, `backend/tests/`) y frontend (`vitest`, `financial-utils.test.ts`), y ambas suites pasan. | ✅ Verificado — se ejecutó `pytest` (15 tests, todos pasan) y `npm run test` con vitest (5 tests, todos pasan) |
| 10 | El volumen montado (`./backend:/app`, `./frontend:/app`) habilita hot-reload en desarrollo dentro de contenedores. | ✅ Verificado — [docker-compose.yml](docker-compose.yml#L9-L11, #L21), junto con `--reload` en Uvicorn y modo dev de Vite |

## Informe de auditoría (hallazgos)

**Lo que estaba bien:**
- Las afirmaciones de puertos (1, 2, 3), montaje de volúmenes (10) y ausencia de BD (5) estaban correctamente respaldadas por evidencia directa en `docker-compose.yml` y los `Dockerfile`.
- La detección de que `.agents/rules`, `.agents/skills` y `memory-bank` no existen (6, 7) fue correcta; se confirma además que el propio `README.md` los describe como estructura pendiente de crear, no como una alucinación del agente.

**Lo que estaba mal reportado o incompleto:**
- La afirmación 4 estaba marcada como ❓ sin necesidad: existe `frontend/.env.example` y el `README.md` documenta explícitamente el comportamiento por defecto (proxy de Vite) y el mecanismo de override. Se corrigió a ✅ y se reformuló para describir el mecanismo real confirmado.
- La afirmación 8 estaba marcada como ❓ por falta de lectura de `components.json`; al revisar el archivo se confirma el uso de shadcn/ui (`style: new-york`). Se corrigió a ✅.
- La afirmación 9 estaba marcada como ✅ pero solo verificaba la *existencia* de archivos de test, no que realmente pasaran. Se ejecutaron ambas suites (`pytest` → 15 passed; `vitest` → 5 passed) y se actualizó la redacción para reflejar la ejecución real.
- Las citas de líneas en `vite.config.ts` (`#L9-L14`) eran ligeramente imprecisas respecto al bloque real del proxy; se corrigieron a `#L11-L16`.

**Sin alucinaciones detectadas:** ninguna afirmación marcada como ✅ resultó ser falsa tras la auditoría; no se encontraron rutas, puertos o archivos citados que no existieran realmente.

## Notas adicionales

- `AGENTS.md` referencia `./.agents/rules`, `./.agents/skills` y `./memory-bank` como estructura a construir durante el proyecto, no como algo ya presente. Esto es consistente con el `README.md`, que describe la creación de esta estructura como parte de los "Recommended steps".
