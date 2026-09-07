# Current Status

Fuente: resultados verificados en `verification.md`, `engineering-findings.md` y ejecución real de tests (2026-09-07).

## Qué está implementado y funcionando

- **API backend completa y testeada**: 8 endpoints en [backend/app/routes.py](../backend/app/routes.py), con 15 tests en [backend/tests/test_routes.py](../backend/tests/test_routes.py) — todos pasan (`pytest -q` → `15 passed`).
- **Dashboard frontend funcional**: KPIs + 2 gráficos, consumiendo `/api/metrics` en tiempo real vía `fetch`. 5 tests unitarios en [frontend/src/lib/financial-utils.test.ts](../frontend/src/lib/financial-utils.test.ts) — todos pasan (`npm run test` → `5 passed`).
- **Entorno de desarrollo reproducible**: `docker compose up --build` levanta ambos servicios con hot-reload, verificado end-to-end.
- **Manejo de estado de carga y error en UI**: `App.tsx` maneja `loading`/`error` con mensaje visible si falla el fetch.

## Limitaciones y problemas conocidos (con evidencia)

1. **Backend con más capacidad que la que usa el frontend**: 7 de 8 endpoints (`facets`, `summary`, `categories/top`, `comparison`, `alerts`, `b2b`, `b2c`) no tienen ningún consumidor en el frontend actual — ver [memory-bank/product-overview.md](./product-overview.md).
2. **Semilla mock duplicada 8 veces** (`seed=42` literal en `routes.py`) sin constante compartida — ver `.agents/rules/mock-data-seed-constant.md`.
3. **Archivo `frontend/src/lib/mock-data.ts` es código muerto** (sin ninguna importación en el proyecto) — ver `.agents/rules/remove-unused-mock-data.md`.
4. **Sin formateador de código (Prettier) en el frontend**: coexisten dos estilos de comillas/`;` sin que ESLint los detecte — ver `.agents/rules/frontend-code-style-quotes-semicolons.md`.
5. **CORS configurado de forma permisiva**: `allow_origins=["*"]` + `allow_credentials=True`; verificado que Starlette refleja cualquier `Origin` recibido, aceptando peticiones autenticadas desde cualquier origen — ver `.agents/rules/cors-configuration-security.md`.
6. **Puerto de debug remoto (`5678`) siempre expuesto**, sin distinción dev/prod en el `Dockerfile` — ver `.agents/rules/debug-port-environment-guard.md`.
7. **Dependencia dura del hostname `backend`** en el proxy de Vite: correr el frontend fuera de Docker Compose falla con `ENOTFOUND backend` a menos que se configure `VITE_API_BASE_URL` — ver `.agents/rules/local-dev-docker-dependency.md`.
8. **Sin persistencia real**: todos los datos son aleatorios en memoria; no hay base de datos, por lo que las métricas no reflejan información financiera real de ningún negocio.
9. **Sin CI/CD**: no existe `.github/workflows/` ni pipeline automatizado que corra `pytest`/`vitest` en cada cambio; la verificación de tests depende de ejecución manual.
10. **Sin autenticación**: cualquier persona con acceso de red al backend puede consultar todos los endpoints sin restricción.

## Próximas prioridades reales (derivadas de las limitaciones, no inventadas)

Basado estrictamente en las brechas detectadas arriba, las prioridades con mayor soporte en evidencia son:

1. Decidir si los 7 endpoints no consumidos (`summary`, `comparison`, `alerts`, `categories/top`, `b2b`, `b2c`, `facets`) se integran a la UI o se documentan como API pública independiente — hoy son código sin consumidor conocido.
2. Resolver la configuración de CORS antes de exponer el backend fuera de un entorno de desarrollo local (riesgo de seguridad más alto de lo esperado, confirmado con `curl`).
3. Introducir un formateador (Prettier) o reglas ESLint de estilo (`quotes`, `semi`) para detener la divergencia de estilo ya presente en 8+ archivos.
4. Agregar un pipeline de CI que ejecute `pytest` y `vitest` en cada push/PR, dado que hoy no existe automatización de esa verificación.
