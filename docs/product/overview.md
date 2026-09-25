# metabase-fork - visión de producto

## Qué es
Metabase es la herramienta de informes que el propietario ya usa. Este fork le añade capacidades
visuales tomadas de los informes de Ontraport: tarjetas con un gráfico y, sobre él, una fila de
botones para cambiar la métrica mostrada (enviados, abiertos, clics, rebotados...). Hoy una tarjeta
de Metabase con varias métricas las dibuja todas a la vez y la única forma de ver una sola es
ocultar el resto desde la leyenda, algo que no se configura, no destaca y no se guarda.

La primera entrega es el **selector de métrica** configurable por tarjeta de dashboard. Después
vendrán la cabecera con total y variación respecto al periodo anterior y los filtros de dashboard
como botones. Todo se construye sobre lo que Metabase ya tiene (preguntas, dashboards, filtros,
ajustes de visualización) y sin tocar la edición comercial.

## Actores y roles
| Rol | Quién | Qué ve y hace |
|---|---|---|
| Editor de dashboard | Usuario de Metabase con permiso de edición del dashboard | Activa y configura el selector en una tarjeta |
| Espectador | Cualquier usuario con acceso al dashboard, incluidos dashboards públicos y embebidos | Pulsa los botones para cambiar de métrica |
| Administrador de Metabase | Quien despliega y actualiza el fork | Integra versiones de Metabase, aplica traducciones |
| Propietario del producto | Esteban (StreamSolve Automation) | Aprueba specs, ADR y pull requests |

## Origen de los datos
| Dato | Fuente | Cómo llega |
|---|---|---|
| Resultado de la pregunta | Base de datos conectada a Metabase (cualquier motor) | La tarjeta ejecuta su consulta como hoy; todas las métricas vienen como columnas del mismo resultado |
| Configuración del selector | Ajustes de visualización de la tarjeta | Se guarda con el dashboard, como el título o los colores |
| Datos de prueba y demo | Base de datos de ejemplo incluida en Metabase | Sin datos de clientes |

## Alcance del MVP
| Spec | Nombre | Para quién |
|---|---|---|
| [001](../../specs/001-selector-de-metrica/spec.md) | Selector de métrica en tarjetas de dashboard | Editor de dashboard y espectador |

## Fuera del MVP
Registrado en [`roadmap.md`](roadmap.md): total y variación en la cabecera (R-1), filtros como
botones (R-2), métrica seleccionada recordada por usuario entre visitas (R-3), tarjetas con desglose (R-4) o
con varias preguntas (R-5), suscripciones con la métrica inicial (R-6), selector en la vista de
pregunta (R-7), cambio de unidad calculado en servidor (R-8) y cambio entre preguntas guardadas (R-9).

## Principios de producto
- Lo que Metabase ya hace no se duplica; se completa.
- Una tarjeta sin selector se ve exactamente igual que antes.
- Cambiar de métrica nunca lanza una consulta nueva.
- Toda cifra mostrada viene de la pregunta tal cual; el selector no calcula nada.
