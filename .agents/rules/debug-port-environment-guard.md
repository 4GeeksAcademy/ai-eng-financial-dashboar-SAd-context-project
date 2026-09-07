# Regla: El puerto de debug remoto (`5678`) no debe activarse fuera de desarrollo

## Propósito
Evitar que el backend exponga un puerto de debug remoto sin autenticación (`debugpy`) en un entorno que no sea desarrollo local aislado.

## Alcance (Scope)
`backend/Dockerfile` y `docker-compose.yml`.

## Justificación basada en el código base
[backend/Dockerfile](../../backend/Dockerfile) línea 13 define:
```
CMD ["python", "-m", "debugpy", "--listen", "0.0.0.0:5678", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```
Este `CMD` arranca `debugpy` incondicionalmente, sin distinción por entorno (dev/staging/prod). [docker-compose.yml](../../docker-compose.yml) líneas 18-20 exponen el puerto `5678` al host en todo momento que el contenedor esté corriendo.

## Guía específica y accionable
- Mantener este `CMD` tal cual **solo mientras el proyecto se use exclusivamente en desarrollo local/Codespaces**.
- Si se crea un `Dockerfile` o `docker-compose.override.yml` para staging/producción, el `CMD` debe excluir `debugpy` y arrancar directamente con `uvicorn app.main:app --host 0.0.0.0 --port 8000` (sin `--reload` ni `debugpy`).
- No exponer el puerto `5678` en ningún `docker-compose` que no sea el de desarrollo local.

## Prueba de validación propuesta
Tarea concreta: con `docker compose up backend` corriendo, ejecutar `curl -v telnet://localhost:5678` (o intentar adjuntar un debugger remoto) desde otra terminal y confirmar que el puerto responde/acepta conexión, documentando esto como evidencia de que el puerto queda abierto por defecto sin necesidad de iniciar una sesión de debug explícita.
