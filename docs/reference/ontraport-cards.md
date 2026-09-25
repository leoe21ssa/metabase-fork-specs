# Tarjetas de informe de Ontraport (referencia visual)

Fuente: capturas de pantalla de la aplicación Ontraport aportadas por el propietario (2026-09-23).
No se ha leído código de Ontraport; esto describe solo el comportamiento observado.

## Comportamiento observado
- Cada tarjeta muestra un título, una cabecera con el total del periodo y su variación respecto al
  periodo anterior, una fila de botones y un gráfico temporal debajo.
- Los botones cambian lo que dibuja el gráfico: la métrica (enviados, abiertos, clics, rebotados)
  o la unidad (importe frente a cantidad). Solo uno está activo a la vez.
- Al pulsar un botón, el gráfico cambia al instante; el periodo y los filtros no cambian.
- Los filtros globales (periodo, propietario) se aplican a todas las tarjetas.

## Qué se imita y en qué spec
| Elemento de Ontraport | Spec | Notas |
|---|---|---|
| Fila de botones que cambia la métrica del gráfico | [001](../../specs/001-selector-de-metrica/spec.md) | Cada botón es una métrica de la misma pregunta |
| Cabecera con total y variación | Futura 002 (roadmap R-1) | |
| Filtros como botones | Futura 003 (roadmap R-2) | |
| Cambio de unidad calculado en servidor | Roadmap R-8 | Necesita backend |

## Datos
Las capturas contienen datos reales de la cuenta del propietario: no se copian a este repo. Los
ejemplos de las specs usan métricas ficticias de correo (enviados, abiertos, clics, rebotados) o la
base de datos de ejemplo de Metabase.
