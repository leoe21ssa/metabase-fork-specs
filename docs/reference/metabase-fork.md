# Metabase: mapa del producto base y del fork

Fuente: `metabase/metabase` @ `2fef5f61f4` (2026-09-29); fork `leoe21ssa/metabase` idéntico en `master` y
en `develop` (PR #1 del fork, sin impacto en los puntos de inserción de la
[spec 001](../../specs/001-selector-de-metrica/spec.md)). Versión revisada antes: `fa7362a1e4` (2026-09-23).
Rutas relativas a la raíz del repo de código. Revisar este archivo al integrar cada versión nueva.

## Qué es y licencia
- Metabase: herramienta de informes y dashboards. Rama `master` = desarrollo de la versión 0.65; la
  documentación pública va por la 0.64.
- Licencia dual: todo lo que está fuera de `enterprise/` es AGPL (se puede modificar; si se ofrece
  el servicio a terceros hay que publicar u ofrecer el código). `enterprise/` está bajo licencia
  comercial: no se modifica ni se usa sin licencia. Fuente: `LICENSE.txt` y `enterprise/README.md`
  del repo de código.
- Tamaño en el commit leído (líneas): frontend TypeScript 911.578 (más 177.071 en la edición
  comercial), backend Clojure 343.597 (más 58.343 comercial), tests de extremo a extremo 260.658.

## Herramientas y comandos
- Versiones fijadas por `mise.toml` y `package.json`: JDK Temurin 25, Clojure CLI 1.12.3, Node 22,
  Bun 1.3 (npm y yarn bloqueados), TypeScript 6 (la comprobación de tipos usa el compilador nativo de
  TypeScript 7), React 18, Mantine 8.3, ECharts 6.1, ttag 1.7, Jest 30, Testing Library 16, Cypress 15.
- Windows: solo con WSL. Instalación paso a paso (Windows y Mac) en
  [entorno-desarrollo.md](../sdd/entorno-desarrollo.md); no se usa `./bin/dev-install` (interactivo,
  instala mise y una segunda copia de las herramientas). Copia de una instancia real en la local:
  [migracion-instancia-local.md](../sdd/migracion-instancia-local.md).
- Comandos (desde la raíz del repo de código, en WSL):

```bash
bun install                                   # dependencias
bun run build-hot                             # frontend en modo desarrollo (recarga en caliente)
clojure -M:run                                # backend en localhost:3000
TZ=UTC bun run test-unit <ruta-del-spec>      # tests unitarios (Jest); compila ClojureScript antes (necesita java)
TZ=UTC bun run test-unit-keep-cljs <ruta>     # igual, sin recompilar ClojureScript
bun run lint-eslint-pure                      # ESLint
bun run lint-format-pure                      # formato (oxfmt); `bun run format` corrige
bun run type-check-pure                       # tipos
CYPRESS_GUI=false bun run test-cypress --spec <archivo.cy.spec.ts>   # extremo a extremo, con backend en marcha
```
- Los tests unitarios suponen la hora UTC, la de los servidores de GitHub: con otra zona horaria
  fallan algunos de fechas (2 de `SmartScalar/compute.unit.spec.ts` con la hora de Bogotá). Medido el
  2026-09-29 en WSL: las cuatro carpetas del plan suman 191 suites y 2148 tests (81 s); tipos, 17 s.
- Tests de backend: `bin/test-agent :only '[namespace]'`; no se necesitan en la [spec 001](../../specs/001-selector-de-metrica/spec.md).
- Guías del repo de código: `docs/developers-guide/devenv.md` (entorno), `docs/developers-guide/frontend.md`
  (estilo y tests unitarios), `docs/developers-guide/e2e-tests.md` (Cypress).

## Lo que Metabase ya hace y condiciona las specs
- Una pregunta con varias métricas dibuja todas las series en el mismo gráfico; la leyenda permite
  ocultar series, pero solo mientras dura la página y sin configuración del editor.
- Los ajustes de visualización de una tarjeta de dashboard son un mapa de claves guardado con el
  dashboard; el backend lo trata como opaco (sin esquema). Las claves desconocidas se ignoran.
- Las tarjetas admiten filtros en línea (parámetros dentro de la tarjeta) y series añadidas de
  otras preguntas.
- Las suscripciones y alertas se renderizan en el backend con un bundle estático separado.
- Los dashboards públicos y embebidos usan los mismos componentes de tarjeta que el dashboard interno.

## Puntos de extensión relevantes para el selector de métrica
| Ruta | Qué es |
|---|---|
| `frontend/src/metabase/viz-core/lib/settings/graph.ts` | Definiciones de ajustes de los gráficos cartesianos: `graph.dimensions`, `graph.metrics` (eje vertical, en orden), `graph.series_order`. Patrón de cada ajuste: `widget`, `getHidden`, `isValid`, `getDefault`, `readDependencies`, `dashboard`. |
| `frontend/src/metabase/viz-core/lib/settings/visualization.ts` | Ajustes comunes que solo aparecen en tarjetas de dashboard (`card.title`, `card.hide_empty`) mediante `dashboard: true`. |
| `frontend/src/metabase/viz-core/shared/settings/series.ts` | `series_settings`: por serie, `title` (nombre visible) y `color`. |
| `frontend/src/metabase/viz-core/lib/settings/widgets.ts` | Registro de widgets de ajustes (toggle, select, listas ordenables). |
| `frontend/src/metabase/visualizations/components/settings/` | Componentes de los widgets (`ChartSettingToggle`, `ChartSettingSelect`, `ChartSettingOrderedSimple`, `ChartSettingSeriesOrder`, `ChartSettingOrderedItems`). |
| `frontend/src/metabase/visualizations/visualizations/CartesianChart/` | Gráficos cartesianos; `CartesianChart.tsx` guarda en estado local las series ocultas desde la leyenda. |
| `frontend/src/metabase/visualizations/components/Visualization/Visualization.tsx` | Componente genérico que dibuja una tarjeta: título, acciones, gráfico, estados de carga y error. |
| `frontend/src/metabase/dashboard/components/DashCard/DashCardVisualization.tsx` | Compone la tarjeta de dashboard: combina ajustes de la tarjeta con los de la pregunta, filtros en línea, y renderiza `Visualization` con `canToggleSeriesVisibility={!isEditing}`. |
| `frontend/src/metabase/dashboard/components/DashCard/Dashcard.unit.spec.tsx` | Tests unitarios existentes de la tarjeta de dashboard. |
| `frontend/src/metabase/dashboard/hooks/use-dashboard-url-query.ts` | Sincroniza filtros y pestaña activa con la dirección del dashboard (interno, público y embebido); reescribe la cadena de consulta y solo conserva los nombres de `QUERY_PARAMS_ALLOW_LIST`. Punto de inserción del parámetro `metric_selector`. |
| `frontend/src/metabase/ui/components/inputs/SegmentedControl/` | Control segmentado (Mantine) exportado desde `metabase/ui`. |
| `frontend/src/metabase-types/api/card.ts` y `visualization-settings.ts` | Tipos de los ajustes de visualización. |
| `frontend/test/__support__/ui.tsx` | `renderWithProviders` y utilidades para tests de componentes. |
| `e2e/test/scenarios/dashboard/` y `e2e/support/helpers/e2e-dashboard-helpers.ts` | Tests de Cypress de dashboards y sus ayudas (`visitDashboard`, `editDashboard`, `saveDashboard`, `showDashboardCardActions`). |
| `locales/es.po` y `bin/i18n/` | Catálogo de traducciones al español y scripts de traducción. |
| `frontend/src/metabase/static-viz/` y `src/metabase/channel/render/card.clj` | Render estático para suscripciones (no se toca en la [spec 001](../../specs/001-selector-de-metrica/spec.md)). |
| `.github/workflows/` | Flujos de CI de Metabase (`frontend.yml`, `e2e-tests.yml`); pesados, pensados para el repo oficial. |
| `enterprise/` | Edición comercial. No se toca. |

## Deudas y riesgos conocidos
- Build y suite pesados: la primera instalación tarda minutos y ocupa varios GB.
- Cada versión de Metabase puede mover archivos (por ejemplo, los ajustes de gráficos se movieron a
  `viz-core`); los puntos de inserción del fork deben revisarse en cada integración.
- Integrar una versión nueva de Metabase: `git fetch upstream`; `master` avanza sin commits propios y
  se sube; rama `chore/upstream-<fecha>` con PR a `develop`, suite en verde y mezcla con **merge
  commit** (squash o rebase copian los commits con otra identidad y la siguiente integración daría
  conflictos en cientos de archivos). El token de `gh` necesita el permiso `workflow`, porque casi
  todas las versiones tocan `.github/workflows/`. Primera integración: 2026-09-29, 69 commits, PR #1.
