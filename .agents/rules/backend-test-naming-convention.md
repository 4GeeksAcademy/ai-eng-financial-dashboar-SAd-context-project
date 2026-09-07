# Regla: Nombrar tests de backend como oración descriptiva completa

## Propósito
Mantener la legibilidad de la suite de tests del backend, donde el nombre del test ya documenta el comportamiento esperado sin necesidad de leer el cuerpo.

## Alcance (Scope)
`backend/tests/**`.

## Justificación basada en el código base
En [backend/tests/test_routes.py](../../backend/tests/test_routes.py), los 5 tests existentes siguen el patrón `test_<sujeto>_<comportamiento_esperado>`: `test_generate_mock_movements_returns_full_year_sorted_data`, `test_filter_movements_by_date_includes_range_edges`, `test_health_endpoint_returns_ok`, `test_metrics_endpoint_respects_date_filters`, `test_b2b_endpoint_only_returns_b2b_records`. Ninguno usa nombres cortos genéricos como `test_metrics()`.

## Guía específica y accionable
- Todo test nuevo debe nombrarse `test_<sujeto>_<comportamiento_esperado>` en snake_case, describiendo explícitamente qué se espera (no solo qué se ejecuta).
- Evitar nombres como `test_endpoint()` o `test_1()` que no describen el resultado esperado.

## Prueba de validación propuesta
**Resultado de la ejecución (2026-09-07):** `grep -n "comparison" backend/tests/test_routes.py` mostró que el endpoint `/api/metrics/comparison` **ya tiene** un test dedicado: `test_metrics_comparison_returns_delta_fields` (línea 157). La suite real tiene 15 tests, no 5 — corrección respecto a la redacción original, que solo citaba 5 nombres de ejemplo.

Tarea concreta corregida: agregar un test nuevo para un caso límite genuinamente no cubierto, por ejemplo validar que `/api/metrics/categories/top` rechaza `limit=21` con `422` (fuera del rango `ge=1, le=20` definido en [backend/app/routes.py](../../backend/app/routes.py)), nombrado `test_top_categories_rejects_limit_above_twenty`, y ejecutar `pytest -q backend/tests/test_routes.py -k limit` para confirmarlo.
