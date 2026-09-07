# Regla: Nombrar archivos nuevos en kebab-case

## Propósito
Preservar la convención de nomenclatura de archivos ya establecida en el 100% del código frontend existente.

## Alcance (Scope)
`frontend/src/components/**` y `frontend/src/lib/**`.

## Justificación basada en el código base
Todos los archivos en `frontend/src/components/dashboard/` (`kpi-card.tsx`, `kpi-row.tsx`, `income-outcome-chart.tsx`, `profit-percent-chart.tsx`, `dashboard-header.tsx`), `frontend/src/components/ui/` (`card.tsx`, `skeleton.tsx`) y `frontend/src/lib/` (`financial-utils.ts`, `financial-types.ts`, `mock-data.ts`, `utils.ts`) usan kebab-case, sin una sola excepción detectada.

## Guía específica y accionable
- Todo componente nuevo debe nombrarse `nombre-del-componente.tsx` (no `PascalCase.tsx` ni `camelCase.tsx`).
- Todo módulo utilitario nuevo en `lib/` debe seguir el mismo patrón (`nombre-utilitario.ts`).
- El nombre del componente exportado dentro del archivo sí puede ser PascalCase (p. ej. `export function KPICard` en `kpi-card.tsx`) — la regla aplica solo al nombre de archivo.

## Prueba de validación propuesta
Tarea concreta: agregar un componente nuevo (p. ej. un footer del dashboard) como `frontend/src/components/dashboard/dashboard-footer.tsx` y confirmar visualmente que su nombre es consistente con los 5 componentes existentes en la misma carpeta.
