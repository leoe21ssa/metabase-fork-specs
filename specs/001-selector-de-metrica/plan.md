# Plan técnico - Spec 001

> Requiere: [ADR-0002](../../docs/decisions/ADR-0002-stack.md) (stack heredado de Metabase, `aceptada` el 2026-09-25). Cubre: RF-1..RF-27 de [`spec.md`](spec.md).

Base de código: `../metabase` en el commit `2fef5f61f4` de `upstream/master` (2026-09-29; el plan se escribió sobre `fa7362a1e4` y la actualización no toca sus puntos de inserción), descrito en [`docs/reference/metabase-fork.md`](../../docs/reference/metabase-fork.md). Todas las rutas de este plan son relativas a la raíz de `../metabase`. El cambio es solo de frontend: ni backend, ni `static-viz/`, ni `enterprise/`.

## Estructura de módulos

| Módulo | Ruta | Responsabilidad | RF |
|---|---|---|---|
| Lógica pura del selector (nuevo) | `frontend/src/metabase/visualizations/lib/metric-selector.ts` | Decidir si el selector aplica, qué botones tiene, cuál está seleccionado y qué serie se dibuja. Funciones puras sobre `RawSeries` y ajustes; sin React ni Redux. | RF-2, RF-6, RF-8, RF-9, RF-11..RF-14, RF-18, RF-19, RF-22 |
| Definiciones de ajustes (nuevo) | `frontend/src/metabase/viz-core/lib/settings/metric-selector.ts` | Las tres claves de ajuste con widget, valor por defecto, dependencias y condición de ocultación. | RF-1..RF-7, RF-9 |
| Widget de métricas incluidas (nuevo) | `frontend/src/metabase/visualizations/components/settings/ChartSettingMetricSelectorMetrics.tsx` | Envoltorio de `ChartSettingOrderedSimple` (casillas y arrastre) que añade el aviso de RF-9. | RF-5, RF-9 |
| Componente de botones (nuevo) | `frontend/src/metabase/visualizations/components/MetricSelector/` (`MetricSelector.tsx`, `MetricSelector.module.css`, `MetricSelector.unit.spec.tsx`, `index.ts`) | Fila de botones sobre `SegmentedControl` de `metabase/ui`; recibe métricas, clave seleccionada y `onChange`; sin estado de edición, porque los botones funcionan también en modo edición. | RF-10, RF-12, RF-15 |
| Hook de estado (nuevo) | `frontend/src/metabase/dashboard/components/DashCard/use-metric-selector.ts` | Guarda la métrica seleccionada de la tarjeta, la lee de la dirección al montar, la escribe en la dirección al pulsar y la reconcilia cuando cambian datos o ajustes. | RF-11, RF-12, RF-18..RF-20, RF-26, RF-27 |
| Parámetro de dirección (nuevo) | `frontend/src/metabase/dashboard/components/DashCard/metric-selector-url.ts` | Leer y escribir el parámetro `metric_selector` de la dirección (pares tarjeta:clave). Funciones puras sobre cadenas; sin React ni navegador. | RF-20, RF-26, RF-27 |
| Integración en la tarjeta (inserción) | `frontend/src/metabase/dashboard/components/DashCard/DashCardVisualization.tsx` | Usa el hook, pasa la serie filtrada a `Visualization` y los botones por `toolbar`. | RF-10, RF-16, RF-17, RF-21, RF-25 |
| Hueco para la barra (inserción) | `frontend/src/metabase/visualizations/components/Visualization/Visualization.tsx` | Prop opcional `toolbar`, pintada tras la cabecera en todos los estados del contenido. | RF-10, RF-25 |
| Conservación del parámetro (inserción) | `frontend/src/metabase/dashboard/hooks/use-dashboard-url-query.ts` | Añadir `metric_selector` a la lista de nombres que la sincronización de filtros y pestaña conserva en la dirección (una línea). | RF-20, RF-26 |
| Registro de ajustes (inserción) | `frontend/src/metabase/visualizations/visualizations/CartesianChart/definition.ts` y `frontend/src/metabase/visualizations/register.ts` | Sumar las definiciones a los gráficos cartesianos y el widget nuevo al registro de widgets. | RF-1 |
| Tipos (inserción) | `frontend/src/metabase-types/api/card.ts` | Las tres claves como opcionales en `VisualizationSettings`. | RF-3 |
| Traducciones (inserción) | `locales/metabase.po`, `locales/es.po` | Cadenas nuevas extraídas y traducidas al español. | RNF-5 |
| Pruebas de extremo a extremo (nuevo) | `e2e/test/scenarios/dashboard/metric-selector.cy.spec.ts` | Recorridos completos con la base de datos de ejemplo. | RF-1..RF-22, RF-24..RF-27 |

Fuera del cambio: `static-viz/`, el backend Clojure, `enterprise/`, la vista de pregunta (los ajustes quedan ocultos fuera del dashboard) y la leyenda del gráfico.

Dependencias entre módulos: lógica pura ← definiciones de ajustes (comparten la regla de cuándo aplica el selector) ← hook ← integración en la tarjeta. El hook usa además el módulo del parámetro de dirección y los hooks de navegación de `metabase/router`. El componente de botones solo depende de `metabase/ui`. La lógica pura no importa nada de React ni de `dashboard/` (constitución, principio 4).

## Modelo de datos
Sin entidades nuevas ni migraciones. Tres claves nuevas en los ajustes de visualización de la tarjeta de dashboard (`dashcard.visualization_settings`, JSON opaco para el backend, que no cambia):

```json
{
  "graph.metric_selector.enabled": true,
  "graph.metric_selector.metrics": [
    { "key": "sum", "enabled": true },
    { "key": "count", "enabled": true },
    { "key": "avg", "enabled": false }
  ],
  "graph.metric_selector.default": "count"
}
```

- `key` es el `name` de la columna de la métrica en el resultado: el mismo identificador que usa `graph.metrics` y la misma clave con la que `series_settings` guarda el nombre visible y el color de cada serie en una tarjeta de una sola pregunta (`_seriesKey = col.name` en `CartesianChart/definition-legacy.ts`).
- Tipos: las tres claves se añaden como opcionales a `VisualizationSettings` en `frontend/src/metabase-types/api/card.ts`, junto a `graph.metrics`.
- Compatibilidad (RNF-3): un Metabase sin el fork ignora las claves desconocidas; el fork ignora las claves cuyas métricas ya no existen (RF-22).
- Métrica seleccionada: estado de React en la tarjeta de dashboard, reflejado en el parámetro `metric_selector` de la dirección de la página: pares `<id de tarjeta>:<clave>` separados por comas, con la clave codificada con `encodeURIComponent` (ejemplo: `?metric_selector=12:count,15:sum`). El par se omite cuando la seleccionada es la inicial. No se guarda con el dashboard (RF-20, RF-26).

## Contratos
Sin cambios de API, de eventos ni de esquema. Contratos internos del frontend:

| Contrato | Firma | RF |
|---|---|---|
| Estado del selector | `getMetricSelectorState(rawSeries, settings): MetricSelectorState \| null`, con `MetricSelectorState = { metrics: { key: string; label: string }[]; defaultKey: string }`; `null` cuando el selector no aplica | RF-2, RF-6, RF-8, RF-9, RF-22 |
| Serie a dibujar | `selectMetric(rawSeries, key): RawSeries`; devuelve la misma referencia si `key` no existe | RF-12, RF-13, RF-14 |
| Clave vigente | `resolveSelectedKey(state, currentKey): string` | RF-11, RF-18, RF-19 |
| Componente | `<MetricSelector metrics selectedKey onChange />` | RF-10, RF-12, RF-15 |
| Parámetro de dirección | `parseMetricSelectorParam(search: string): Record<number, string>` y `writeMetricSelectorParam(search: string, dashcardId: number, key: string \| null): string` (`null` elimina el par; sin pares se elimina el parámetro; el resto de la cadena se conserva) | RF-20, RF-26, RF-27 |
| Hook | `useMetricSelector(dashcardId, series, settings): { state, selectedKey, select }` | RF-11, RF-12, RF-17..RF-20, RF-26, RF-27 |
| Hueco en la tarjeta | `Visualization` acepta `toolbar?: ReactNode` y lo pinta entre la cabecera (`VisualizationHeader`) y el contenido | RF-10, RF-25 |
| Definiciones de ajustes | `GRAPH_METRIC_SELECTOR_SETTINGS` con las tres claves; `getHidden` devuelve `true` fuera de dashboards (`extra.isDashboard`), con desglose (`graph.dimensions` con más de una columna), con series añadidas (`rawSeries.length > 1`) o con menos de dos métricas | RF-1, RF-2, RF-5, RF-7 |

## Algoritmos y cálculos
No hay cálculos numéricos: el selector no agrega ni transforma valores (principio de producto "toda cifra viene de la pregunta"), por lo que no aplica ninguna fórmula de referencia.

```
getMetricSelectorState(rawSeries, settings):
  si rawSeries.length != 1 o no settings["graph.metric_selector.enabled"]: devolver null
  si settings["graph.dimensions"] tiene más de una columna (desglose): devolver null
  cols = rawSeries[0].data.cols
  metricsInResult = settings["graph.metrics"] filtradas a las que existen en cols
  si metricsInResult.length < 2: devolver null
  configured = settings["graph.metric_selector.metrics"] ?? metricsInResult.map(key => { key, enabled: true })
  included = configured.filter(m => m.enabled y m.key está en metricsInResult)
  si included.length < 2: devolver null
  label(key) = settings["series_settings"]?[key]?.title ?? col(key).display_name
  defaultKey = settings["graph.metric_selector.default"] si está en included; si no, included[0].key
  devolver { metrics: included.map(m => { key: m.key, label: label(m.key) }), defaultKey }

selectMetric(rawSeries, key):
  si key no está en graph.metrics del primer card: devolver rawSeries
  devolver rawSeries con rawSeries[0].card.visualization_settings["graph.metrics"] = [key]
  (updateIn de icepick: filas, columnas y el resto de ajustes se comparten sin copiar)

resolveSelectedKey(state, currentKey):
  devolver currentKey si state.metrics lo contiene; si no, state.defaultKey

parseMetricSelectorParam(search):
  valor = new URLSearchParams(search).get("metric_selector") ?? ""
  para cada par "id:clave" separado por comas: si id es un entero y la clave no está vacía, resultado[id] = decodeURIComponent(clave)
  devolver resultado (vacío si no hay parámetro o está mal formado)

writeMetricSelectorParam(search, dashcardId, key):
  pares = parseMetricSelectorParam(search); si key es null borrar pares[dashcardId], si no pares[dashcardId] = key
  params = new URLSearchParams(search); si no quedan pares borrar "metric_selector", si no ponerlo con "id:" + encodeURIComponent(clave) unidos por comas
  devolver "?" + params (o "" si no queda ningún parámetro)

useMetricSelector(dashcardId, series, settings):
  state = memo(getMetricSelectorState(series, settings))
  location = useMaybeLocation(); navigate = useNavigate()
  inicial = resolveSelectedKey(state, parseMetricSelectorParam(location?.search ?? "")[dashcardId])
  selectedKey = useState(inicial), reconciliado con resolveSelectedKey cuando cambia state
  select(key):
    si key == selectedKey: no hacer nada
    setSelectedKey(key)
    si location existe, dashcardId > 0 y no isEmbedPreview():
      navigate({ ...location, search: writeMetricSelectorParam(location.search, dashcardId, key == state.defaultKey ? null : key) }, { replace: true })
```

- `settings` son los ajustes calculados (`getComputedSettingsForSeries(series)`, memoizados por `series`), para leer `graph.metrics`, `graph.dimensions` y `series_settings` con sus valores por defecto.
- Al dibujar una sola métrica, el gráfico cartesiano oculta la leyenda por sí mismo, conserva `series_settings[key]` (color y nombre) y recalcula el eje vertical (RF-14).
- `selectMetric` no copia filas ni columnas; cambiar de métrica es un re-render con los mismos datos, sin petición (RF-13, RNF-9, RNF-10).
- `useMaybeLocation` devuelve `null` fuera del router (SDK de embebido): entonces la selección vive solo en memoria. En la vista previa de embebido (`isEmbedPreview`) no se escribe la dirección, igual que hace la sincronización de filtros de Metabase.
- RF-27 se cumple sin código extra: cada tarjeta lee solo su propio par; los pares de tarjetas sin selector o de métricas no incluidas se ignoran y quedan en la dirección sin efecto.

## Decisiones técnicas
1. **Filtrar por ajuste, no por datos.** `selectMetric` cambia solo `graph.metrics` de la serie que se pasa al gráfico; las columnas, filas e ids de columna quedan intactos, así que el drill-through, la descarga y los tooltips siguen funcionando (RF-14, RF-24). Alternativas descartadas: recortar `data.cols`/`data.rows` (rompe los objetos de clic y los índices de columna) y reutilizar el estado `hiddenSeries` de `CartesianChart` (vive dentro del gráfico, no es configurable y la leyenda seguiría mostrando todo). Constitución, principios 4 y 5.
2. **Estado en la tarjeta de dashboard, reflejado en la dirección.** El hook vive en `DashCardVisualization`, que no se desmonta cuando la tarjeta recarga datos; por eso la selección sobrevive a los filtros (RF-18). Al pulsar, el hook escribe el parámetro `metric_selector` con `useNavigate` y `replace: true` (sin entrada nueva en el historial) y al montar lo lee con `useMaybeLocation` (RF-20, RF-26); ambos vienen de `metabase/router`, el mismo mecanismo con el que `useDashboardUrlQuery` lleva la pestaña y los filtros a la dirección en el dashboard interno y en los públicos o embebidos. Esa sincronización reescribe la dirección y solo conserva los nombres de `QUERY_PARAMS_ALLOW_LIST`, así que `metric_selector` se añade a esa lista (una línea). Alternativas descartadas: guardar la selección en Redux y sincronizarla desde `useDashboardUrlQuery` (más puntos de inserción en acciones, reductor y selectores del dashboard); un parámetro por tarjeta (`metric_12=count`), que obligaría a cambiar la lista de nombres por un filtro de prefijo; recordar la selección por usuario (roadmap R-3). Principio 2.
3. **Tres ajustes nuevos con `dashboard: true`** en un archivo propio de definiciones, registrados en `COMBO_CHARTS_SETTINGS_DEFINITIONS` (línea, área, barras y combinado). Alternativa descartada: extender `graph.series_order` (semántica distinta: oculta series de forma permanente y se muestra en otros contextos). Principio 5.
4. **`SegmentedControl` de `metabase/ui` para los botones**: es un grupo de radios accesible por teclado y con estado expuesto a lectores de pantalla, ya tematizado (RNF-6). Alternativa descartada: `Button.Group` con `aria-pressed` manual. Ponytail: sin componente nuevo de base.
5. **Prop `toolbar` en `Visualization`** (inserción de pocas líneas tras la cabecera) para que el orden sea título, botones, gráfico, como en Ontraport. Alternativa descartada: envolver `Visualization` desde la tarjeta (los botones quedarían encima del título) o duplicar la cabecera. Principio 2: punto de inserción único y listado.
6. **Sin cambios en el render estático** (RF-23): las suscripciones leen los ajustes guardados, donde `graph.metrics` no se toca; el fork no altera nada en `static-viz/` ni en el backend. Principio 2.
7. **Widget de lista reutilizado**: el ajuste de métricas incluidas usa `ChartSettingOrderedSimple` (casillas y arrastre ya existentes) envuelto en un widget mínimo que añade el aviso de RF-9. Alternativa descartada: un editor propio.
8. **Botones funcionales en modo edición** (RF-17): el hook no depende de `isEditing` y la selección nunca toca `dashcard.visualization_settings`, así que guardar el dashboard no cambia la métrica inicial. La leyenda sigue bloqueada en edición como hoy (`canToggleSeriesVisibility={!isEditing}`); con una sola serie no se muestra, así que no hay conflicto.
9. **Sin dependencias nuevas**: `icepick`, `ttag`, `@testing-library/react` y Cypress ya están en el repo. Ponytail.
10. **Puntos de inserción en archivos de Metabase** (lista cerrada; principio 2): `frontend/src/metabase-types/api/card.ts`, `frontend/src/metabase/visualizations/visualizations/CartesianChart/definition.ts`, el registro de widgets de ajustes, `frontend/src/metabase/visualizations/components/Visualization/Visualization.tsx`, `frontend/src/metabase/dashboard/components/DashCard/DashCardVisualization.tsx`, `frontend/src/metabase/dashboard/components/DashCard/Dashcard.unit.spec.tsx`, `frontend/src/metabase/dashboard/hooks/use-dashboard-url-query.ts` con su `use-dashboard-url-query.unit.spec.tsx`, y `locales/es.po`. Todo lo demás son archivos nuevos.
11. **Parámetro `metric_selector` con valor `id:clave`**: un nombre fijo (no hace falta filtrar por prefijo), legible, y la clave codificada para que valgan nombres de columna con comas o dos puntos. Limitación conocida: un filtro de dashboard cuyo nombre genere el slug `metric_selector` pisaría el parámetro; se documenta en [`docs/reference/metabase-fork.md`](../../docs/reference/metabase-fork.md) en T30.

## Herramientas obligatorias del agente
Regla 8 de [`AGENTS.md`](../../AGENTS.md) y `AGENTS.md` del repo de código (se crea en T2). En cada tarea:

- **Context7** antes de escribir código con cada librería, siempre para la versión instalada en `../metabase` (confirmarla en `package.json` antes de consultar): React 18.2 (`useState`, `useMemo`, `useEffect`), `@mantine/core` 8.3.18 (`SegmentedControl`: `data`, `value`, `onChange`, `disabled`, `size`, `fullWidth`), `ttag` 1.7.21 (`t`, `jt`), Jest 30, `@testing-library/react` 16 (`renderHook`, consultas por rol), Cypress 15.14 (`cy.intercept`, `cy.findByRole`), `icepick` 2.4 (`updateIn`), `react-router` 7.18 solo a través de `metabase/router` (`useMaybeLocation`, `useNavigate` con `replace`).
- Para el código propio de Metabase no hay Context7: se leen `docs/developers-guide/frontend.md`, `docs/developers-guide/e2e-tests.md` y las skills del fork en `.claude/skills/` (`typescript-write`, `typescript-review`, `e2e-test-create`, `e2e-test-review`), que fijan convenciones de componentes, tests y helpers `H.*`.
- **Ponytail** en cada tarea: el mínimo código que cumple el RF, ninguna dependencia nueva (decisión 9) y ningún archivo de Metabase tocado fuera de la lista de la decisión 10.
- **impeccable** en el componente de botones y en el widget de ajustes (T15): tamaño, espaciado respecto al título, contraste, foco visible y comportamiento en tarjetas estrechas y en móvil, con capturas en el pull request.
- Cada commit de tarea cita la consulta de Context7 usada (TX-3).

## Estrategia de tests
No hay fechas ni cálculos: no aplican fechas inyectadas ni golden tests. Cuatro niveles, todos con la base de datos de ejemplo o con series ficticias:

1. **Unitarios de lógica pura** (Jest, `metric-selector.unit.spec.ts` junto al módulo): una prueba por regla de `getMetricSelectorState`, `selectMetric` y `resolveSelectedKey`, con series construidas con `createMockCard`, `createMockDataset` y `createMockColumn` de `metabase-types/api/mocks`; más `metric-selector-url.unit.spec.ts` para el parámetro de dirección (lectura, escritura, borrado del par y del parámetro, claves con caracteres especiales, valores mal formados). Cubren RF-2, RF-6, RF-8, RF-9, RF-11, RF-13, RF-14, RF-19, RF-20, RF-22, RF-26, RF-27.
2. **Unitarios de ajustes** (Jest, `metric-selector.unit.spec.ts` junto a las definiciones): `getSettingsWidgetsForSeries` (`viz-core/lib/widgets.ts`) con `extra.isDashboard` verdadero y falso, con y sin desglose, con una y con tres métricas; valores por defecto y `readDependencies`. Cubren RF-1, RF-2, RF-5, RF-6, RF-7.
3. **Unitarios de componente e integración de la tarjeta** (Jest y Testing Library: `MetricSelector.unit.spec.tsx`, `use-metric-selector.unit.spec.ts`, `Dashcard.unit.spec.tsx` y `use-dashboard-url-query.unit.spec.tsx` ampliados): botones como radios, clic, clic repetido, teclado; el hook con `renderHookWithProviders` (`withRouter`, `initialRoute`): arranque desde una dirección con parámetro, pulsación que la reescribe sin entrada de historial, par ignorado sin selector; en la tarjeta: métrica inicial, cambio de serie sin nueva consulta (el mock de la consulta no se vuelve a llamar), botones funcionales en edición sin tocar los ajustes guardados, nueva carga de datos con la selección conservada, métrica desaparecida, tarjeta sin selector idéntica a la actual, botones presentes en carga, vacío y error; en la sincronización de la dirección: un cambio de filtro conserva `metric_selector`. Cubren RF-10, RF-11, RF-12, RF-15, RF-17, RF-18, RF-19, RF-20, RF-21, RF-25, RF-26, RF-27.
4. **Extremo a extremo** (Cypress, `metric-selector.cy.spec.ts`): pregunta sobre pedidos (suma del total, número de pedidos y media del total por mes) creada por API con `H.createQuestionAndDashboard`; ajustes visibles solo cuando aplica; activar, reordenar, desmarcar, elegir inicial, guardar y recargar; renombrar una serie se refleja en el botón; el clic dibuja una serie y `cy.intercept` confirma que no hay petición; un cambio de filtro conserva la selección; aviso con menos de dos incluidas; edición de la pregunta; dashboard público, embebido y modo edición con botones funcionales y la inicial guardada intacta; tarjeta sin selector; la dirección lleva la métrica pulsada, se mantiene al recargar y un par para una tarjeta sin selector no da error. Cubren RF-1..RF-22, RF-24..RF-27.

RF-23 (suscripciones) se comprueba a mano con `H.setupSMTP` y `H.sendEmailAndVisitIt` en local (T28) y con una captura en el pull request; no entra en el CI del fork porque necesita el backend completo. RF-16 y RF-24 se completan con la comprobación manual de T21 (menú de acciones, descarga, pantalla completa, dirección compartida).

Cada nombre de test incluye el id del RF que cubre (TX-2). Los unitarios corren en el CI del fork (T3). Los de Cypress corren en local antes del pull request:

```
CYPRESS_GUI=false bun run test-cypress --spec e2e/test/scenarios/dashboard/metric-selector.cy.spec.ts
```

## Cobertura de requisitos
| RF | Módulo | Test previsto |
|---|---|---|
| RF-1 | Definiciones de ajustes, registro | Ajustes: "RF-1 ofrece el selector solo en gráficos cartesianos con dos o más métricas"; e2e: visibilidad de los ajustes |
| RF-2 | Definiciones de ajustes, lógica pura | Ajustes: "RF-2 oculta los ajustes con desglose o series añadidas"; lógica: `getMetricSelectorState` devuelve `null` |
| RF-3 | Tipos, integración | e2e: activar, guardar, recargar el dashboard y ver los botones |
| RF-4 | Integración | e2e: dos tarjetas de la misma pregunta, solo una con selector |
| RF-5 | Widget de métricas incluidas | e2e: reordenar y desmarcar; ajustes: el widget `metricSelectorMetrics` está registrado |
| RF-6 | Definiciones de ajustes, lógica pura | Lógica: "RF-6 incluye todas las métricas en el orden del gráfico por defecto" |
| RF-7 | Definiciones de ajustes | Ajustes: "RF-7 la métrica inicial solo ofrece las incluidas y vale la primera por defecto" |
| RF-8 | Lógica pura | Lógica: "RF-8 usa el nombre visible de la serie"; e2e: renombrar una serie |
| RF-9 | Widget, lógica pura | Lógica: "RF-9 devuelve null con menos de dos incluidas"; e2e: aviso en los ajustes |
| RF-10 | Componente, hueco `toolbar`, integración | Tarjeta: "RF-10 pinta los botones bajo el título" |
| RF-11 | Hook, lógica pura | Tarjeta: "RF-11 abre con la métrica inicial y una sola serie" |
| RF-12 | Componente, hook | Tarjeta: "RF-12 al pulsar dibuja solo esa métrica"; e2e: la otra tarjeta no cambia |
| RF-13 | Lógica pura, integración | Tarjeta: "RF-13 no vuelve a consultar"; e2e: `cy.intercept` sin peticiones |
| RF-14 | Lógica pura | Lógica: "RF-14 conserva dimensiones, columnas y filas"; e2e: color y formato de la serie |
| RF-15 | Componente | Componente: "RF-15 no llama a onChange al pulsar el seleccionado" |
| RF-16 | Integración | e2e: dashboard público y embebido; T21: pantalla completa |
| RF-17 | Hook, integración | Tarjeta: "RF-17 los botones funcionan en edición y no cambian la inicial guardada"; e2e: modo edición y guardar |
| RF-18 | Hook | Tarjeta: "RF-18 conserva la selección al recargar datos"; e2e: cambio de filtro |
| RF-19 | Hook, lógica pura | Lógica y tarjeta: "RF-19 vuelve a la inicial si la seleccionada desaparece" |
| RF-20 | Parámetro de dirección, hook, inserción en la sincronización | Parámetro: "RF-20 escribe y limpia el par"; hook: "RF-20 pulsar reescribe la dirección sin historial"; sincronización: "RF-20 conserva metric_selector"; e2e: la dirección cambia al pulsar |
| RF-21 | Integración | Tarjeta: "RF-21 una tarjeta sin selector se dibuja igual"; e2e: tarjeta sin selector |
| RF-22 | Lógica pura | Lógica: "RF-22 sin métricas existentes devuelve null"; e2e: editar la pregunta |
| RF-23 | Ninguno (sin cambio de código) | Comprobación manual T28 con captura de la suscripción |
| RF-24 | Lógica pura (decisión 1) | e2e: menú de acciones de la tarjeta; T21: descarga y drill-through |
| RF-25 | Hueco `toolbar`, integración | Tarjeta: "RF-25 mantiene los botones en carga, vacío y error" |
| RF-26 | Parámetro de dirección, hook | Parámetro: "RF-26 lee los pares"; hook: "RF-26 arranca con la métrica de la dirección"; e2e: recargar y abrir con la dirección |
| RF-27 | Parámetro de dirección, hook | Hook: "RF-27 ignora el par de una tarjeta sin selector"; e2e: dirección con par para una tarjeta sin selector |
