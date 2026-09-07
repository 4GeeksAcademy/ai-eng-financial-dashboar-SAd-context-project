# Hallazgos de Ingeniería y Reglas Propuestas

Cada hallazgo cita evidencia verificable del repositorio (archivo, línea o comportamiento reproducible) y una regla accionable derivada directamente de esa evidencia.

## Arquitectura

### H1 — Constante mágica `seed=42` duplicada 8 veces sin fuente única de verdad
`generate_mock_movements(seed=42)` se repite literalmente 8 veces en [backend/app/routes.py](backend/app/routes.py#L255-L386) (líneas 255, 264, 277, 295, 311, 350, 370, 386). Cada endpoint regenera el dataset completo del año en cada request, sin caché ni capa de datos compartida.
> **Regla:** Extraer `MOCK_SEED = 42` como constante de módulo y/o memoizar `generate_mock_movements()` por request. Un cambio de semilla hoy requiere editar 8 líneas idénticas manualmente.

### H2 — `frontend/src/lib/mock-data.ts` es código muerto conectado a nada
[frontend/src/lib/mock-data.ts](frontend/src/lib/mock-data.ts) exporta `mockMovements`, pero ningún archivo lo importa (confirmado por búsqueda de texto `mock-data` en todo el repo — sin resultados). El dashboard real obtiene datos vía `fetch("/api/metrics")` en [frontend/src/App.tsx](frontend/src/App.tsx#L14-L20). Aun así, `tsconfig.app.json` con `"include": ["src"]` ([frontend/tsconfig.app.json](frontend/tsconfig.app.json#L24)) hace que el archivo se compile igual.
> **Regla:** Antes de asumir que `mock-data.ts` alimenta el dashboard, confirmar con `grep -r "mock-data"` que sigue sin uso. El flujo de datos real es 100% vía API backend.

## Nomenclatura y estilo (Frontend)

### H3 — Dos convenciones de comillas/punto y coma conviven sin Prettier que arbitre
No existe `.prettierrc` ni `eslint-plugin-prettier` en el repo, y [frontend/eslint.config.js](frontend/eslint.config.js) no activa reglas `quotes`/`semi`. Conviven dos estilos:
- Comillas simples, sin `;`: [financial-types.ts](frontend/src/lib/financial-types.ts), [kpi-row.tsx](frontend/src/components/dashboard/kpi-row.tsx), [dashboard-header.tsx](frontend/src/components/dashboard/dashboard-header.tsx), [kpi-card.tsx](frontend/src/components/dashboard/kpi-card.tsx), [income-outcome-chart.tsx](frontend/src/components/dashboard/income-outcome-chart.tsx), [profit-percent-chart.tsx](frontend/src/components/dashboard/profit-percent-chart.tsx), [ui/card.tsx](frontend/src/components/ui/card.tsx), [main.tsx](frontend/src/main.tsx).
- Comillas dobles, con `;`: [App.tsx](frontend/src/App.tsx), [financial-utils.ts](frontend/src/lib/financial-utils.ts), [vite.config.ts](frontend/vite.config.ts).
> **Regla:** No usar un archivo aislado como referencia de estilo; el patrón dominante en `components/` y `lib/` (salvo `App.tsx`/`financial-utils.ts`) es comillas simples sin `;`. Confirmar contra archivos hermanos de la misma carpeta antes de escribir código nuevo.

### H4 — Convención de nombres de archivo kebab-case (a preservar, no es riesgo)
Todos los componentes en `frontend/src/components/dashboard/` y `ui/`, y los módulos de `lib/`, siguen kebab-case (`kpi-card.tsx`, `income-outcome-chart.tsx`, `financial-utils.ts`). Sin excepciones detectadas.
> **Regla:** Nuevos archivos deben nombrarse en kebab-case, replicando el 100% de consistencia observada.

## Testing

### H5 — Nombres de test como oración descriptiva completa (backend)
En [backend/tests/test_routes.py](backend/tests/test_routes.py), todos los tests siguen `test_<sujeto>_<comportamiento_esperado>` (`test_generate_mock_movements_returns_full_year_sorted_data`, `test_b2b_endpoint_only_returns_b2b_records`), sin abreviaciones cortas.
> **Regla:** Tests nuevos deben describir sujeto + comportamiento esperado en snake_case largo, replicando el patrón existente.

### H6 — `conftest.py` inserta la raíz del proyecto en `sys.path` manualmente, sin `pyproject.toml`/`pytest.ini`
[backend/tests/conftest.py](backend/tests/conftest.py) ejecuta `sys.path.insert(0, str(ROOT_DIR))`. Confirmado que no existe `backend/pyproject.toml` ni `backend/pytest.ini` en el repositorio (búsqueda de archivos sin resultados) — no hay instalación editable ni configuración declarativa de rootdir.
> **Regla:** Ejecutar `pytest` siempre desde `backend/` (no desde la raíz del repo); si se introduce un `pyproject.toml`, este hack de `sys.path` debe revisarse o eliminarse.

## Documentación

### H7 — Cero docstrings en 8 endpoints públicos de la API
`grep '"""'` sobre [backend/app/routes.py](backend/app/routes.py) no arroja resultados: ninguna de las funciones (incluyendo los 8 endpoints `@router.get(...)`) tiene docstring. La única documentación disponible es la generada automáticamente por FastAPI a partir de tipos y nombres de parámetros (`/docs`).
> **Regla:** Cualquier endpoint nuevo o modificado debe documentarse dependiendo solo de tipos/nombres (patrón actual), o si se decide agregar docstrings, hacerlo de forma consistente en los 8 endpoints existentes a la vez, no de forma parcial.

## Seguridad

### H8 — CORS configurado con wildcard + credenciales, combinación inválida/riesgosa
[backend/app/main.py](backend/app/main.py#L7-L13) configura `allow_origins=["*"]` junto con `allow_credentials=True`. Esta combinación viola la especificación CORS (los navegadores rechazan `credentials: true` con origen wildcard) y es una configuración de riesgo si se reemplaza ingenuamente el wildcard por un origen reflejado dinámicamente (OWASP A05 - Security Misconfiguration).
> **Regla:** No replicar este bloque de CORS en un entorno con autenticación; fijar una lista explícita de orígenes permitidos (`allow_origins=["http://localhost:5173"]`) antes de cualquier despliegue fuera de desarrollo local.

## Flujo Operativo / DX

### H9 — El flujo local depende exclusivamente de Docker Compose; el proxy usa un hostname no resoluble fuera de él
[README.md](README.md#L41-L43) solo documenta `docker compose up --build`. [frontend/vite.config.ts](frontend/vite.config.ts#L13) apunta el proxy `/api` a `http://backend:8000`, hostname que solo resuelve dentro de la red de Docker Compose.
> **Regla:** Si se corre `npm run dev` fuera de Docker, se debe definir `VITE_API_BASE_URL` (vía `frontend/.env.example`) o el proxy fallará con `ENOTFOUND backend`.

### H10 — Puerto de debug remoto (`5678`) siempre activo, no condicional al entorno
[backend/Dockerfile](backend/Dockerfile#L13) arranca siempre con `python -m debugpy --listen 0.0.0.0:5678 -m uvicorn ...`, sin distinción dev/prod. Cualquier `docker compose up` expone un puerto de debug remoto sin autenticación en `0.0.0.0`.
> **Regla:** Antes de desplegar este backend fuera de un entorno de desarrollo aislado, diferenciar el `CMD` por entorno (con/sin `debugpy`).

