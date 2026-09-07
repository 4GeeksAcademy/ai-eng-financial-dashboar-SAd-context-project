# Tech Stack

Fuente: `backend/requirements.txt`, `backend/Dockerfile`, `frontend/package.json`, `frontend/vite.config.ts`, `docker-compose.yml`.

## Backend

| Elemento | Valor verificado | Evidencia |
|---|---|---|
| Lenguaje | Python 3.13 | [backend/Dockerfile](../backend/Dockerfile#L1) (`FROM python:3.13-slim`) |
| Framework web | FastAPI | [backend/requirements.txt](../backend/requirements.txt), [backend/app/main.py](../backend/app/main.py) |
| Servidor ASGI | Uvicorn (modo `--reload` en dev) | [backend/Dockerfile](../backend/Dockerfile#L15) |
| Validación de datos | Pydantic (vía `BaseModel` en `routes.py`) | [backend/app/routes.py](../backend/app/routes.py#L22-L52) |
| Testing | pytest + pytest-cov + httpx (`TestClient`) | [backend/requirements.txt](../backend/requirements.txt), [backend/tests/test_routes.py](../backend/tests/test_routes.py) |
| Debug remoto | `debugpy`, puerto `5678` | [backend/Dockerfile](../backend/Dockerfile#L15) |
| Persistencia | **Ninguna** — no hay driver de BD (`psycopg2`, `sqlalchemy`, etc.) en `requirements.txt` | [backend/requirements.txt](../backend/requirements.txt) |

## Frontend

| Elemento | Valor verificado | Evidencia |
|---|---|---|
| Lenguaje | TypeScript (`~6.0.2`) | [frontend/package.json](../frontend/package.json) |
| Framework UI | React `19.2.4` | [frontend/package.json](../frontend/package.json) |
| Build tool / dev server | Vite `8.0.4` | [frontend/package.json](../frontend/package.json), [frontend/vite.config.ts](../frontend/vite.config.ts) |
| Estilos | Tailwind CSS `4.2.2` vía `@tailwindcss/vite` | [frontend/package.json](../frontend/package.json), [frontend/vite.config.ts](../frontend/vite.config.ts#L3) |
| Sistema de componentes | shadcn/ui (estilo `new-york`) | [frontend/components.json](../frontend/components.json) |
| Gráficos | `recharts` `3.8.1` | [frontend/package.json](../frontend/package.json), usado en `income-outcome-chart.tsx`, `profit-percent-chart.tsx` |
| Iconos | `lucide-react` | [frontend/package.json](../frontend/package.json) |
| Utilidades de clases CSS | `clsx` + `tailwind-merge` (helper `cn()`) | [frontend/src/lib/utils.ts](../frontend/src/lib/utils.ts) |
| Testing | Vitest `4.1.4` + `@vitest/coverage-v8` | [frontend/package.json](../frontend/package.json), [frontend/src/lib/financial-utils.test.ts](../frontend/src/lib/financial-utils.test.ts) |
| Linting | ESLint `9.39.4` (`js.configs.recommended`, `typescript-eslint`, `react-hooks`, `react-refresh`) — **sin Prettier configurado** | [frontend/eslint.config.js](../frontend/eslint.config.js) |

## Arquitectura de ejecución

- **Orquestación**: Docker Compose con 2 servicios (`frontend`, `backend`), definidos en [docker-compose.yml](../docker-compose.yml).
- **Puertos expuestos al host**: `5173` (frontend/Vite), `8000` (backend/API), `5678` (backend/debugpy, siempre activo).
- **Comunicación frontend→backend en desarrollo**: proxy de Vite (`/api` → `http://backend:8000`), definido en [frontend/vite.config.ts](../frontend/vite.config.ts#L11-L16). El hostname `backend` solo resuelve dentro de la red de Docker Compose (confirmado: `npm run dev` fuera de Docker produce `Error: getaddrinfo ENOTFOUND backend`).
- **Hot reload**: volúmenes montados (`./backend:/app`, `./frontend:/app`) + `--reload` en Uvicorn + modo dev de Vite.
- **CORS**: `allow_origins=["*"]` + `allow_credentials=True` en [backend/app/main.py](../backend/app/main.py#L7-L13); verificado que Starlette refleja cualquier `Origin` recibido en vez de bloquear la petición (ver `.agents/rules/cors-configuration-security.md`).

## Ausencias confirmadas (no asumidas)

- Sin `pyproject.toml` ni `pytest.ini` en `backend/` — la resolución de imports en tests depende de `sys.path.insert()` en [backend/tests/conftest.py](../backend/tests/conftest.py).
- Sin `.prettierrc` en `frontend/` — coexisten dos estilos de código (comillas simples sin `;` vs. comillas dobles con `;`).
- Sin CI/CD (`.github/workflows`) detectado en el repositorio al momento de esta documentación.
