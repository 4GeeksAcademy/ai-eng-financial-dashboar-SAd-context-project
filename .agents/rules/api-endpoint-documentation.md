# Regla: Documentación de endpoints basada en tipos, no en docstrings

## Propósito
Mantener consistencia en cómo se documentan los endpoints de la API, evitando que un agente agregue docstrings a un solo endpoint dejando los otros 7 sin documentar.

## Alcance (Scope)
`backend/app/routes.py`.

## Justificación basada en el código base
Una búsqueda de `"""` en [backend/app/routes.py](../../backend/app/routes.py) no arroja resultados: ninguna de las funciones, incluyendo los 8 endpoints `@router.get(...)` (`/health`, `/api/metrics`, `/api/metrics/facets`, `/api/metrics/summary`, `/api/metrics/categories/top`, `/api/metrics/comparison`, `/api/metrics/alerts`, `/api/metrics/b2b`, `/api/metrics/b2c`), tiene docstring. La documentación disponible hoy proviene únicamente de los tipos de Pydantic (`response_model=...`) y nombres de parámetros `Query(...)`, visibles en `/docs` (Swagger autogenerado por FastAPI).

## Guía específica y accionable
- No agregar un docstring a un único endpoint como parte de una tarea no relacionada; si se decide documentar con docstrings, debe hacerse de forma consistente en los 8 endpoints en el mismo cambio.
- Al agregar un endpoint nuevo, seguir el patrón actual: tipar todos los parámetros con `Query(...)` y declarar `response_model` explícito — esa es la documentación real que consume este proyecto.
- Cualquier aclaración de comportamiento no evidente en la firma (p. ej. por qué `/api/metrics/comparison` requiere `start_date`/`end_date` obligatorios sin default) debe ir en un comentario de una línea, no en un docstring largo.

## Prueba de validación propuesta
Tarea concreta: levantar el backend (`docker compose up backend` o `uvicorn app.main:app`) y abrir `http://localhost:8000/docs`, confirmando que los 8 endpoints ya muestran documentación útil (parámetros, tipos, response model) pese a no tener docstrings — validando que la guía es consistente con el comportamiento observable.
