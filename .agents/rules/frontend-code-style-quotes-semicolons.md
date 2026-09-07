# Regla: Seguir el estilo de comillas/punto y coma del archivo hermano, no de un archivo aislado

## Propósito
Evitar que un agente mezcle aún más los dos estilos de código coexistentes en el frontend, ya que no hay un formateador automático que los arbitre.

## Alcance (Scope)
Todos los archivos `.ts`/`.tsx` bajo `frontend/src/`.

## Justificación basada en el código base
No existe `.prettierrc` en el repositorio, y [frontend/eslint.config.js](../../frontend/eslint.config.js) solo carga `js.configs.recommended`, `tseslint.configs.recommended`, `reactHooks` y `reactRefresh` — ninguna regla `quotes` o `semi` está activa. Esto permite que convivan dos estilos verificados:
- Comillas simples, sin `;`: `financial-types.ts`, `kpi-row.tsx`, `dashboard-header.tsx`, `kpi-card.tsx`, `income-outcome-chart.tsx`, `profit-percent-chart.tsx`, `ui/card.tsx`, `main.tsx`.
- Comillas dobles, con `;`: `App.tsx`, `financial-utils.ts`, `vite.config.ts`.

## Guía específica y accionable
- Antes de escribir o editar un archivo `.ts`/`.tsx`, revisar el estilo del archivo hermano más reciente en la misma carpeta (`components/dashboard/`, `components/ui/`, o `lib/`) y replicarlo.
- El patrón dominante en `components/` y en la mayoría de `lib/` es comillas simples sin `;`; `App.tsx` y `financial-utils.ts` son la excepción documentada.
- No introducir un tercer estilo (p. ej. backticks por defecto) sin justificación.

## Prueba de validación propuesta
Tarea concreta: crear un componente nuevo trivial en `frontend/src/components/dashboard/` (por ejemplo, un badge de estado) siguiendo comillas simples sin `;`, y correr `npm run lint` en `frontend/` para confirmar que ESLint no reporta error de estilo (ya que hoy no hay regla que lo valide) — esto documenta que la única defensa contra la inconsistencia es la disciplina del agente, no una herramienta automática.
