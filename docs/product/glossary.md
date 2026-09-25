# Glosario

Términos de dominio con su equivalente en inglés (identificadores de código). Las specs
usan **solo** estos términos. Si falta uno, se añade aquí antes de usarlo.

## Método (SDD)

| Término | Código (EN) | Definición |
|---|---|---|
| SDD | `spec-driven development` | Desarrollo dirigido por especificación: constitución → spec → clarificación → plan → tareas → implementación → validación → cambio. Guía en [`docs/sdd/README.md`](../sdd/README.md). |
| Constitución | `constitution` | Principios innegociables del producto, cada uno con su forma de verificarse. [`docs/constitution.md`](../constitution.md). |
| Spec | `spec` | Especificación de una funcionalidad: el QUÉ y el POR QUÉ, sin tecnología. `specs/NNN-nombre/spec.md`. |
| ADR | `architecture decision record` | Registro de una decisión de arquitectura o producto: contexto, decisión, consecuencias y alternativas descartadas. `docs/decisions/ADR-NNNN-tema.md`. |
| RF | `functional requirement` | Requisito funcional: una frase verificable en notación EARS, numerada `RF-n` dentro de cada spec. Las tareas y los tests citan su id. |
| RNF | `non-functional requirement` | Requisito no funcional transversal, numerado `RNF-n` en un catálogo único: [`docs/product/requisitos-transversales.md`](requisitos-transversales.md). |
| EARS | `EARS notation` | Patrones para escribir requisitos sin ambigüedad: CUANDO…, SI… ENTONCES…, MIENTRAS…, DONDE…, EL SISTEMA… |
| Historia de usuario | `user story` | "Como <rol> quiero <acción> para <beneficio>", numerada `H-n`; agrupa los RF. |
| Marcador de aclaración | `clarification marker` | `[NECESITA ACLARACIÓN: pregunta]`: hueco visible en una spec que alguien debe responder antes de aprobarla. |
| Plan | `plan` | El CÓMO de una spec: módulos, datos, contratos, decisiones justificadas y estrategia de tests. Solo con el stack decidido ([ADR-0002](../decisions/ADR-0002-stack.md)). |
| Tarea | `task` | Paso de menos de 30 minutos, numerado `T-n`, con RF que cubre y "Hecho cuando". Etiquetada `[A]` agente, `[H]` humano o `[M]` mixta. |
| Prerrequisito humano | `human prerequisite` | `P-n`: credencial, cuenta, dato o decisión que debe existir antes de empezar las tareas. |
| Skill | `skill` | Instrucciones reutilizables para un agente de IA que ejecutan una fase (`sdd-spec`, `sdd-plan`, `sdd-implement`, `sdd-run`, `sdd-validate`, `sdd-change`). Una sola copia en `.agents/skills/`. |
| Punto de inserción | `insertion point` | Archivo existente de Metabase que el fork modifica lo mínimo para enganchar código nuevo. Cada plan los lista (constitución, principio 2). |

## Metabase

| Término | Código (EN) | Definición |
|---|---|---|
| Producto base | `upstream` | El repositorio oficial `metabase/metabase` del que parte el fork. |
| Fork | `fork` | Copia del producto base con las mejoras propias; repo `metabase` del workspace. |
| Edición de código abierto | `OSS edition` | Parte de Metabase bajo licencia AGPL; todo lo que está fuera de `enterprise/`. Es la única que el fork modifica. |
| Edición comercial | `enterprise edition` | Código bajo licencia comercial en `enterprise/`; no se modifica ni se usa sin licencia. |
| Pregunta | `card` / `question` | Consulta guardada con su tipo de gráfico. Su resultado tiene columnas y filas. |
| Dashboard | `dashboard` | Página con tarjetas y filtros. |
| Tarjeta de dashboard | `dashcard` | Una pregunta colocada en un dashboard, con tamaño y ajustes de visualización propios que se guardan con el dashboard. |
| Ajustes de visualización | `visualization settings` | Configuración de cómo se dibuja una pregunta o tarjeta (título, colores, ejes, formato). Se guardan como un mapa de claves. |
| Gráfico cartesiano | `cartesian chart` | Gráfico de líneas, área, barras o combinado, con eje horizontal y eje vertical. |
| Dimensión | `dimension` | Columna que define el eje horizontal (por ejemplo, la semana). |
| Métrica | `metric` (de gráfico) | Columna numérica que el gráfico dibuja en el eje vertical. Una pregunta puede tener varias. Distinta de la métrica guardada de Metabase. |
| Métrica guardada | `metric card` | Entidad de Metabase que guarda una agregación reutilizable. No interviene en el selector. |
| Serie | `series` | Conjunto de puntos que el gráfico dibuja para una métrica (o para un valor de desglose). Tiene nombre y color. |
| Desglose | `breakout` | Segunda dimensión que divide una métrica en una serie por valor (por ejemplo, por propietario). |
| Series añadidas | `added series` | Preguntas adicionales combinadas en una misma tarjeta. |
| Leyenda | `legend` | Lista de series bajo o junto al gráfico; hoy permite ocultar series sin guardar el cambio. |
| Filtro de dashboard | `parameter` | Control del dashboard (fecha, categoría...) conectado a las preguntas; al cambiar, las tarjetas recargan datos. |
| Modo edición | `edit mode` | Estado del dashboard en el que el editor mueve tarjetas y cambia ajustes. |
| Dashboard público o embebido | `public / embedded dashboard` | Dashboard mostrado fuera de la aplicación (enlace público o iframe). |
| Suscripción | `subscription` / `alert` | Envío programado de un dashboard o pregunta por correo u otro canal, renderizado por el backend. |
| Render estático | `static viz` | Imagen de una tarjeta generada por el backend para suscripciones y alertas. |
| Base de datos de ejemplo | `Sample Database` | Base de datos incluida en Metabase (pedidos, productos, personas) usada en tests y demos. |

## Selector de métrica

| Término | Código (EN) | Definición |
|---|---|---|
| Selector de métrica | `metric selector` | Fila de botones sobre el gráfico de una tarjeta, uno por métrica incluida, que muestra solo la métrica elegida. Solo existe con dos o más métricas incluidas. |
| Botón de métrica | `metric button` | Cada opción del selector. Su texto es el nombre visible de la métrica. |
| Métrica incluida | `included metric` | Métrica de la pregunta que el editor ha marcado para aparecer como botón. |
| Métrica inicial | `default metric` | Métrica seleccionada al abrir el dashboard. |
| Métrica seleccionada | `selected metric` | Métrica cuya serie se muestra en este momento; estado de la página, va en la dirección del dashboard y no se guarda con él. |
| Nombre visible | `display name` | Texto con el que la tarjeta muestra una métrica (en leyenda, tooltip y botón); el editor puede cambiarlo. |
