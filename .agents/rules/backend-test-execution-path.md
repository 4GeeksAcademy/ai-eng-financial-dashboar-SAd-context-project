# Regla: No mover ni eliminar `conftest.py` sin preservar el ajuste de `sys.path`

## Propósito
Evitar que se rompa la resolución del paquete `app` al ejecutar `pytest`, ya que depende explícitamente de un ajuste manual de `sys.path` en `conftest.py`, no del directorio de trabajo.

## Alcance (Scope)
`backend/tests/conftest.py` y cualquier comando de CI/CD o documentación que invoque `pytest`.

## Justificación basada en el código base
[backend/tests/conftest.py](../../backend/tests/conftest.py) ejecuta `sys.path.insert(0, str(ROOT_DIR))` con `ROOT_DIR = Path(__file__).resolve().parents[1]` (es decir, `backend/`), calculado a partir de `__file__`, no del `cwd`. No existe `backend/pyproject.toml` ni `backend/pytest.ini` en el repositorio — no hay instalación editable (`pip install -e .`) ni configuración declarativa de rootdir que reemplace este mecanismo.

**Corrección tras prueba (2026-09-07):** se ejecutó `python -m pytest backend/tests -q` desde la **raíz del repositorio** (no desde `backend/`) y los 15 tests pasaron igual. La redacción original de esta regla asumía incorrectamente que el working directory importaba — en realidad `conftest.py` resuelve la ruta de forma absoluta vía `__file__`, por lo que `pytest` funciona desde cualquier cwd mientras `conftest.py` exista en su ubicación actual.

## Guía específica y accionable
- `pytest` puede ejecutarse desde la raíz del repo o desde `backend/` indistintamente — el mecanismo que lo permite es `conftest.py`, no el directorio de trabajo.
- No mover `conftest.py` de ubicación sin actualizar `parents[1]`, ya que ese índice asume que el archivo vive en `backend/tests/`.
- Si se agrega un `pyproject.toml` con `[tool.pytest.ini_options]` o se instala el paquete en modo editable, se debe eliminar o simplificar este hack de `sys.path` en el mismo cambio.

## Prueba de validación propuesta
Tarea concreta: ejecutar `python -m pytest backend/tests -q` tanto desde la raíz del repo como desde `backend/` y confirmar que ambos casos terminan en "15 passed" — documentando que la dependencia real es la existencia de `conftest.py`, no el cwd.
