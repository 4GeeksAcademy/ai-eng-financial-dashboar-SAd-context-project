# Product Overview

Fuente: evidencia directa del código en `backend/app/routes.py`, `frontend/src/App.tsx`, `frontend/src/components/dashboard/`, `README.md`.

## Qué entrega el repositorio

Un **dashboard financiero de una sola pantalla** que muestra:
- 4 KPIs principales: ingreso total, egreso total, ganancia (profit) y margen de ganancia (%), calculados en [frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts) (`computeKPIs`).
- Un gráfico de líneas de ingresos vs. egresos por mes ([frontend/src/components/dashboard/income-outcome-chart.tsx](../frontend/src/components/dashboard/income-outcome-chart.tsx)).
- Un gráfico de porcentaje de ganancia por mes ([frontend/src/components/dashboard/profit-percent-chart.tsx](../frontend/src/components/dashboard/profit-percent-chart.tsx)).

Todo esto se renderiza en [frontend/src/App.tsx](../frontend/src/App.tsx), que consume un único endpoint (`/api/metrics`) al cargar la página.

## Para quién

No hay login, roles ni multiusuario en el código: no existen rutas de autenticación, tablas de usuarios ni middleware de sesión en `backend/app/`. El dashboard está diseñado para un **único usuario/vista ejecutiva** que revisa métricas agregadas de un periodo fijo ("2024 - Full Year", hardcodeado en [frontend/src/App.tsx](../frontend/src/App.tsx#L44)).

## Capacidades del backend (más allá de lo que consume hoy el frontend)

El backend expone 8 endpoints en [backend/app/routes.py](../backend/app/routes.py), pero **el frontend solo usa uno** (`/api/metrics`, confirmado por única llamada `fetch` en `App.tsx`):

| Endpoint | Uso actual en frontend |
|---|---|
| `GET /health` | No consumido por el frontend |
| `GET /api/metrics` | ✅ Consumido (`App.tsx`) |
| `GET /api/metrics/facets` | No consumido |
| `GET /api/metrics/summary` | No consumido |
| `GET /api/metrics/categories/top` | No consumido |
| `GET /api/metrics/comparison` | No consumido |
| `GET /api/metrics/alerts` | No consumido |
| `GET /api/metrics/b2b` | No consumido |
| `GET /api/metrics/b2c` | No consumido |

Esto significa que el backend tiene capacidades (comparación de periodos, alertas de gasto anómalo, top de categorías, segmentación B2B/B2C) que **existen y tienen tests**, pero que **no están reflejadas en ninguna pantalla del frontend actual**.

## Qué NO hace (evidencia negativa, no asunción)

- **No persiste datos reales**: todos los movimientos financieros se generan aleatoriamente en memoria con semilla fija (`generate_mock_movements(seed=42)`); no hay ORM, modelos de base de datos, ni cliente de BD en `backend/requirements.txt`.
- **No tiene autenticación ni autorización**: no hay ninguna dependencia de auth (JWT, OAuth, sesiones) en `requirements.txt` ni middleware de auth en `main.py`.
- **No es multi-tenant ni multi-periodo interactivo**: el periodo mostrado en el header es un string fijo, no seleccionable por el usuario en la UI actual.
