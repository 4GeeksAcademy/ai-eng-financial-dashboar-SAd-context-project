# Regla: Centralizar la semilla de datos mock del backend

## Propósito
Evitar que la semilla usada para generar datos financieros simulados quede duplicada y desincronizada entre endpoints.

## Alcance (Scope)
`backend/app/routes.py` y cualquier módulo nuevo que llame a `generate_mock_movements(...)`.

## Justificación basada en el código base
`generate_mock_movements(seed=42)` aparece **8 veces literalmente** en [backend/app/routes.py](../../backend/app/routes.py) (líneas 255, 264, 277, 295, 311, 350, 370, 386 al momento de esta auditoría). No existe una constante compartida ni una capa de caché: cada endpoint regenera el año completo de movimientos en cada request.

## Guía específica y accionable
- Definir `MOCK_SEED = 42` como constante a nivel de módulo en `routes.py` (o en un módulo de configuración si se crea uno) y reemplazar las 8 llamadas literales por `generate_mock_movements(seed=MOCK_SEED)`.
- Si se agrega un noveno endpoint que necesite el dataset mock, debe reutilizar la misma constante — nunca escribir el número `42` de nuevo.
- No es obligatorio agregar caché en esta regla; si se decide cachear, debe aplicarse a los 8 endpoints existentes a la vez, no parcialmente.

## Prueba de validación propuesta
Tarea concreta: refactorizar `routes.py` para introducir `MOCK_SEED` y verificar con `grep -n "seed=42" backend/app/routes.py` que ya no quedan literales `42` fuera de la definición de la constante. Ejecutar `pytest` en `backend/` y confirmar que los 15 tests existentes siguen en verde (el comportamiento no debe cambiar, solo la fuente del valor).
