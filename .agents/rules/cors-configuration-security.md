# Regla: No replicar la configuración CORS wildcard + credenciales fuera de desarrollo local

## Propósito
Prevenir que la configuración actual de CORS, válida solo para desarrollo local sin autenticación, se copie a un entorno con credenciales reales o se despliegue tal cual a producción.

## Alcance (Scope)
`backend/app/main.py` y cualquier configuración de despliegue derivada de este proyecto.

## Justificación basada en el código base
[backend/app/main.py](../../backend/app/main.py) líneas 7-13 configuran:
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    ...
)
```
**Corrección tras prueba (2026-09-07):** se levantó el backend (`docker compose up backend`) y se ejecutó `curl -i -H "Origin: http://localhost:5173" http://localhost:8000/api/metrics`. La respuesta **no fue bloqueada**; Starlette's `CORSMiddleware` reflejó el origen específico de la petición en vez del literal `*`:
```
access-control-allow-origin: http://localhost:5173
access-control-allow-credentials: true
```
Esto es más riesgoso de lo que sugería la redacción original (que decía "los navegadores rechazan esta combinación"): en realidad `allow_credentials=True` hace que Starlette **automáticamente reflipe cualquier `Origin` recibido** en vez de enviar `*` literal, por lo que el resultado práctico es que **cualquier origen es aceptado junto con credenciales** — el equivalente funcional de un wildcard con credenciales habilitadas, sin que el navegador lo bloquee (OWASP A05 - Security Misconfiguration).

## Guía específica y accionable
- Si el proyecto sigue siendo solo de datos mock sin autenticación, esta configuración puede mantenerse **solo en desarrollo local**, documentando explícitamente por qué (no hay cookies/sesión que proteger).
- Antes de agregar cualquier mecanismo de autenticación (JWT en cookie, sesión, etc.), reemplazar `allow_origins=["*"]` por una lista explícita, por ejemplo `allow_origins=["http://localhost:5173"]`, tomada de una variable de entorno — de lo contrario cualquier sitio externo puede hacer peticiones autenticadas con cookies del usuario.
- No asumir que el navegador protege esta configuración por defecto; la prueba demostró que la respuesta se sirve igual con cualquier `Origin` enviado.

## Prueba de validación propuesta
Tarea concreta (ya ejecutada): con el backend corriendo (`docker compose up backend`), correr `curl -i -H "Origin: http://localhost:5173" http://localhost:8000/api/metrics | grep -i access-control` y confirmar que el header `access-control-allow-origin` refleja el origen enviado (no `*`) junto con `access-control-allow-credentials: true` — evidencia de que cualquier origen sería aceptado igual.
