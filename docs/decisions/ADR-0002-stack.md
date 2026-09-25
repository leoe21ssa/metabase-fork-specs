# ADR-0002 - Stack de implementación: heredado de Metabase, cambios solo en el frontend

Estado: aceptada · Fecha: 2026-09-24 · Aceptada por el propietario: 2026-09-25

> El stack lo fija el producto base; este ADR no elige tecnologías: delimita en qué capa se
> implementan las mejoras y con qué herramientas se trabaja. Aceptado el ADR, el plan de la
> [spec 001](../../specs/001-selector-de-metrica/plan.md) queda habilitado.

## Contexto
- Equipo: el propietario (Windows 11) más agentes de IA. Sin equipo de backend Clojure.
- Producto base: Metabase en el commit `fa7362a1e4` (rama de desarrollo de la versión 0.65),
  descrito en [`docs/reference/metabase-fork.md`](../reference/metabase-fork.md).
- Stack fijo por Metabase: frontend TypeScript 6 + React 18 + Mantine 8 (expuesto como `metabase/ui`)
  + ECharts 6 + ttag (traducciones); tests con Jest 30 + Testing Library y Cypress 15; backend Clojure
  1.12 sobre JDK Temurin 25; herramientas fijadas con mise; Bun 1.3 como gestor de paquetes (npm y
  yarn bloqueados).
- El objetivo (selector de métrica) se resuelve con datos que la tarjeta ya recibe: todas las métricas
  llegan como columnas del mismo resultado.

## Costuras que hay que decidir
- Dónde vive la funcionalidad: en los ajustes de visualización de la tarjeta (un JSON opaco que el
  backend guarda sin cambios), en una función pura de selección de serie y en un componente de
  interfaz. El backend, la API y el esquema de la base de datos de aplicación no cambian.
- Render estático (suscripciones por correo, generado por el backend con un bundle aparte): fuera
  del MVP; las tarjetas se envían como hasta ahora.

## Opciones
| Opción | Descripción | Pros | Contras |
|---|---|---|---|
| A | Solo frontend: ajuste de visualización nuevo en la tarjeta y filtrado de la serie en el cliente | Sin backend, sin migraciones, sin consulta nueva; reversible; integración de upstream sencilla | Limitado a lo que la consulta ya devuelve; el render estático no lo ve |
| B | Backend: consulta o agregación por métrica y API nueva | Permite unidades calculadas en servidor (importe frente a cantidad de otra tabla) | Toca Clojure, API y tests de backend; más conflictos con upstream; 10 a 20 días |
| C | Visualización personalizada con el SDK de Metabase (funcionalidad de pago) | Sin fork | Licencia comercial; el SDK no cubre botones sobre los gráficos nativos |

## Decisión
Opción A. En concreto:
- Frontend únicamente, con las convenciones del repo: ajustes de visualización, componentes de
  `metabase/ui`, ttag, oxfmt, ESLint y comprobación de tipos.
- Sin dependencias nuevas.
- Tests: Jest para lógica y componentes, Cypress para el flujo completo. Sin tests de backend porque
  no hay cambios de backend.
- Traducciones al español en el catálogo del fork (`locales/es.po`).
- CI del fork: un flujo propio y mínimo en GitHub Actions (lint, formato, tipos y los tests unitarios
  de las áreas tocadas) separado de los flujos de Metabase, que no se activan en el fork.
- Herramientas obligatorias del agente: Context7 para la versión instalada de cada librería, Ponytail
  en cada tarea, impeccable en cada interfaz (regla 8 de [`AGENTS.md`](../../AGENTS.md)).

## Consecuencias
- El build es pesado: la primera instalación y el primer `build-hot` tardan minutos y ocupan varios GB;
  los tests unitarios necesitan compilar antes la parte ClojureScript; los tests de Cypress necesitan
  el backend en marcha.
- Desarrollo en WSL, con los repos dentro del sistema de archivos de Linux ([ADR-0001](ADR-0001-repositorios.md)).
- Cada versión mensual de Metabase y cada parche de seguridad obligan a integrar upstream y a pasar la suite antes de desplegar (constitución, principio 8).
- Lo que exija cálculo en servidor (unidades distintas, totales que la consulta no devuelve) queda en
  el [roadmap](../product/roadmap.md) y necesitará un ADR nuevo.

## Alternativas descartadas
- **B (backend)**: coste y riesgo de conflicto con upstream sin necesidad para el MVP; se reconsidera
  solo para R-8 del roadmap.
- **C (SDK de visualizaciones personalizadas)**: de pago y no cubre el caso.
- **Usar la leyenda existente como "selector"**: no es configurable, no es prominente y no se guarda;
  no cumple la spec.
