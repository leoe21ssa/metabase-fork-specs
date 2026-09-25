# Roadmap y backlog

Único backlog del producto. Las specs enlazan aquí desde "Fuera de alcance". Cuando el
repo esté en GitHub puede reflejarse en Issues, pero este archivo manda.

## 1. Fase 2 - producto (tras el MVP)
| Id | Tema | Origen | Notas |
|---|---|---|---|
| R-1 | Cabecera de tarjeta con total del periodo y variación respecto al periodo anterior (estilo Ontraport "TOTAL +7,2 %") | Análisis 2026-09-23 | Futura spec 002; depende de [spec 001](../../specs/001-selector-de-metrica/spec.md). Estimación 4 a 8 días |
| R-2 | Filtros de dashboard mostrados como botones (una opción por botón) en lugar de desplegables | Análisis 2026-09-23 | Futura spec 003; independiente de 001. Estimación 3 a 5 días |
| R-3 | Métrica seleccionada recordada por usuario entre visitas, sin enlace | [Spec 001](../../specs/001-selector-de-metrica/spec.md), fuera de alcance | La dirección del dashboard ya lleva la selección (RF-20 de esa spec); falta recordarla sin enlace |
| R-4 | Selector en tarjetas con desglose (una serie por valor de otra columna) | [Spec 001](../../specs/001-selector-de-metrica/spec.md), fuera de alcance | Requiere decidir si el botón cambia la métrica o el valor del desglose |
| R-5 | Selector en tarjetas que combinan varias preguntas (series añadidas) | [Spec 001](../../specs/001-selector-de-metrica/spec.md), fuera de alcance | |
| R-6 | Suscripciones y alertas renderizadas con la métrica inicial en lugar de todas las series | [Spec 001](../../specs/001-selector-de-metrica/spec.md), fuera de alcance | Toca el render estático del backend |
| R-7 | Selector en la vista de pregunta, fuera de dashboards | [Spec 001](../../specs/001-selector-de-metrica/spec.md), fuera de alcance | |
| R-8 | Cambio de unidad calculado en servidor (importe frente a cantidad cuando no son columnas de la misma pregunta) | Análisis 2026-09-23 | Necesita ADR nuevo (backend). Estimación 10 a 20 días |
| R-9 | Cambio entre preguntas guardadas distintas dentro de una misma tarjeta | Análisis 2026-09-23 | Alternativa a R-8 sin backend. Estimación 5 a 10 días |

## 2. Backlog técnico
| Id | Tema | Origen | Notas |
|---|---|---|---|
| T-1 | CI mínima del fork en GitHub Actions (lint, formato, tipos, tests unitarios de las áreas tocadas) | [ADR-0002](../decisions/ADR-0002-stack.md) | Los flujos de Metabase no se activan en el fork |
| T-2 | Procedimiento de integración de cada versión de Metabase (`master` desde upstream, PR a `develop`, suite en verde) | [ADR-0001](../decisions/ADR-0001-repositorios.md) | Entre medio día y un día por versión |
| T-3 | Entorno WSL con los repos fuera de OneDrive | [ADR-0001](../decisions/ADR-0001-repositorios.md) | Prerrequisito de la [spec 001](../../specs/001-selector-de-metrica/tasks.md) |
| T-4 | Imagen Docker del fork para desplegar en el servidor del propietario | Despliegue | Metabase publica su propio Dockerfile; el fork lo reutiliza |
| T-5 | Flujo de traducciones del fork (entradas nuevas en el catálogo español) | Constitución, principio 7 | |

## 3. Ideas (sin compromiso)
- Visualizaciones personalizadas con el SDK de pago de Metabase, si algún día se contrata la edición comercial.
- Proponer el selector de métrica a upstream como pull request cuando esté estable.
