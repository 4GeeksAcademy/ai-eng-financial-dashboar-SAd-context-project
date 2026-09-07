# Regla: No asumir que `mock-data.ts` alimenta el dashboard

## Propósito
Evitar que un agente o desarrollador modifique `mock-data.ts` pensando que afecta lo que se ve en pantalla, cuando en realidad el dashboard consume datos exclusivamente de la API backend.

## Alcance (Scope)
`frontend/src/lib/mock-data.ts` y cualquier componente en `frontend/src/components/dashboard/` o `frontend/src/App.tsx`.

## Justificación basada en el código base
`frontend/src/lib/mock-data.ts` exporta `mockMovements`, pero una búsqueda de texto `mock-data` en todo el repositorio no arroja ninguna importación. El flujo real de datos es: [frontend/src/App.tsx](../../frontend/src/App.tsx) línea 14-20 hace `fetch(`${API_BASE_URL}/api/metrics`)` contra el backend. Sin embargo, `frontend/tsconfig.app.json` (`"include": ["src"]`) sigue compilando y type-checando el archivo aunque esté sin uso.

## Guía específica y accionable
- Antes de editar `mock-data.ts` esperando ver el cambio reflejado en el dashboard, ejecutar `grep -rn "mock-data" frontend/src` para confirmar si sigue sin importarse.
- Si el objetivo es proveer datos de fallback/offline para el dashboard, se debe importar explícitamente `mockMovements` en `App.tsx` (o donde corresponda) y documentar esa decisión — no asumir que ya está conectado.
- Si se determina que el archivo ya no tiene propósito, su eliminación debe hacerse en un cambio separado y explícito, no como efecto secundario de otra tarea.

## Prueba de validación propuesta
Tarea concreta: agregar un comentario de una línea en la primera línea de `mock-data.ts` (por ejemplo `// TODO: verificar uso`), correr `npm run build` en `frontend/` y confirmar que el build no falla ni cambia el comportamiento del dashboard renderizado — demostrando que el archivo es independiente del flujo real de datos.
