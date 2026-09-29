# AGENTS.md - metabase-fork-specs

Contexto canónico para cualquier agente de código o LLM (Claude Code, Codex, Cursor,
Gemini CLI, opencode, Jules, xAI, chat plano…). [`CLAUDE.md`](CLAUDE.md) solo contiene `@AGENTS.md`;
el resto de herramientas leen este archivo directamente. Si tu herramienta no lo carga
sola, pega su contenido al inicio de la sesión.

## Proyecto

**metabase-fork** - fork de Metabase (edición de código abierto, licencia AGPL) al que se
añaden mejoras de visualización inspiradas en los informes de Ontraport. La primera es el
selector de métrica en tarjetas de dashboard: botones sobre el gráfico para cambiar la
métrica mostrada con un clic.
Visión, actores y alcance: [`docs/product/overview.md`](docs/product/overview.md). Modelo de dominio: [`docs/product/domain-model.md`](docs/product/domain-model.md).

Este repositorio es la **fuente de verdad del producto**: constitución, specs, decisiones
y documentación. No contiene código de aplicación. El código vive en el repo hermano
`../metabase` (fork de `metabase/metabase`).

## Estado del proyecto (2026-09-29)

- Fase: **[spec 001](specs/001-selector-de-metrica/spec.md) `planificada` (2026-09-25)**: [plan](specs/001-selector-de-metrica/plan.md) y [tareas](specs/001-selector-de-metrica/tasks.md)
  aprobados por el propietario (P1). Prerrequisitos P1..P5 hechos el 2026-09-28 (WSL, ramas del fork, clave de Context7, instancia local); la implementación (T1..T34) empieza cuando el propietario lo indique.
- Fork al día con `metabase/metabase` `2fef5f61f4` (2026-09-29, PR #1 del fork, sin impacto en la [spec 001](specs/001-selector-de-metrica/spec.md)). Este repo es
  `leoe21ssa/metabase-fork-specs` desde la enmienda 1 del [ADR-0001](docs/decisions/ADR-0001-repositorios.md) (2026-09-28).
- Constitución: `aceptada` (2026-09-25) en [`docs/constitution.md`](docs/constitution.md).
- Stack: heredado del producto base y descrito en [`docs/decisions/ADR-0002-stack.md`](docs/decisions/ADR-0002-stack.md)
  (`aceptada`, 2026-09-25). El [plan de la spec 001](specs/001-selector-de-metrica/plan.md) queda habilitado.
- Prohibido crear `apps/`, `packages/`, `src/` o cualquier código aquí.
- Repos, ramas y entorno de trabajo: [`docs/decisions/ADR-0001-repositorios.md`](docs/decisions/ADR-0001-repositorios.md);
  máquina nueva paso a paso en [`docs/sdd/entorno-desarrollo.md`](docs/sdd/entorno-desarrollo.md).

## Mapa del workspace y de los repos

```
../                       15_Metabase_fork/ (carpeta local, no es repo git)
├── metabase-fork-specs/  este repo (specs, ADRs, docs, skills)
└── metabase/             repo de código: fork de metabase/metabase (master = espejo de upstream)
```

Repos externos (NO se clonan en el workspace; se leen bajo demanda y su contrato está
documentado con el commit leído):

| Repo | Qué es | Contrato |
|---|---|---|
| https://github.com/metabase/metabase | Producto base (`upstream` del fork) | [`docs/reference/metabase-fork.md`](docs/reference/metabase-fork.md) |
| Ontraport (aplicación SaaS, solo capturas) | Referencia visual de las tarjetas que se imitan | [`docs/reference/ontraport-cards.md`](docs/reference/ontraport-cards.md) |

## Mapa de este repo

| Ruta | Contenido |
|---|---|
| [`docs/constitution.md`](docs/constitution.md) | Principios innegociables. Léelo antes de cualquier tarea. |
| `docs/sdd/` | El método: [`README.md`](docs/sdd/README.md) (cómo trabajar), [`prompts.md`](docs/sdd/prompts.md) (prompts por fase, para cualquier LLM), [`recursos-externos.md`](docs/sdd/recursos-externos.md) (accesos y quién los tiene). |
| `docs/product/` | Visión, glosario, modelo de dominio, requisitos transversales (RNF), roadmap/backlog. |
| `docs/reference/` | Lo que ya existe: mapa del fork de Metabase y comportamiento de referencia de Ontraport. |
| `docs/decisions/` | ADRs. Una decisión por archivo. |
| `specs/` | Una carpeta por funcionalidad: `NNN-nombre/spec.md` (+ `plan.md`, `tasks.md` cuando toque). Índice y estado en [`specs/README.md`](specs/README.md). |
| [`tools/links.py`](tools/links.py) | Único script del repo: comprueba (`--check`) o corrige (`--fix`) los enlaces entre archivos markdown. No es código de aplicación. |
| `.agents/skills/` | Skills SDD canónicas (formato Agent Skills: `SKILL.md` con `name`/`description`). `.claude/skills` y `.opencode/skill` son enlaces simbólicos a ellas. |
| `templates/code-repo/` | Base para el `AGENTS.md` y la configuración del repo de código. |

## Convenciones de specs

- Carpeta `specs/NNN-nombre-en-kebab-case/`, numeración de tres dígitos, la siguiente libre.
- `spec.md` sigue [`specs/_templates/spec.md`](specs/_templates/spec.md) sin saltar secciones.
- Requisitos funcionales numerados `RF-n` (únicos dentro de la spec) en notación **EARS**:
  `CUANDO … , EL SISTEMA …` · `SI … , ENTONCES EL SISTEMA …` · `MIENTRAS … , EL SISTEMA …` ·
  `DONDE … , EL SISTEMA …` · `EL SISTEMA …`.
- Un requisito, una frase, verificable. Sin adjetivos no medibles.
- Lo que no se sabe se marca `[NECESITA ACLARACIÓN: pregunta concreta]` y se lista en "Dudas abiertas".
- Requisitos no funcionales: se citan por id `RNF-n` desde [`docs/product/requisitos-transversales.md`](docs/product/requisitos-transversales.md).
- Términos de dominio: los del glosario [`docs/product/glossary.md`](docs/product/glossary.md). No inventes sinónimos.
- La spec describe el QUÉ y el POR QUÉ. Nada de stack, arquitectura, tablas, endpoints ni nombres de archivo.
- Idiomas: specs, docs y mensajes al usuario en **español**; identificadores, esquema y commits de código en **inglés**.
- `tasks.md` etiqueta cada tarea `[A]` (agente), `[H]` (humano) o `[M]` (mixta) y abre con "Prerrequisitos humanos".
- **Toda referencia a otro archivo del repo es un enlace markdown relativo** que se pueda abrir con un clic: [`docs/constitution.md`](docs/constitution.md), `[ADR-0002](docs/decisions/ADR-0002-stack.md)`, `[spec 001](specs/001-selector-de-metrica/spec.md)`. Nunca una ruta suelta ni solo el nombre "[ADR-0002](docs/decisions/ADR-0002-stack.md)" o el número de una spec sin enlace. Vale para specs, docs, ADRs, skills y plantillas. `python3 tools/links.py --check` lo verifica y `--fix` lo corrige.

## Flujo SDD

Fases y skill que las ejecuta (detalle en [`docs/sdd/README.md`](docs/sdd/README.md); prompt equivalente en [`docs/sdd/prompts.md`](docs/sdd/prompts.md)):

| Fase | Skill | Produce |
|---|---|---|
| Spec y clarificación | `sdd-spec` | `specs/NNN-*/spec.md` |
| Plan y tareas | `sdd-plan` | `plan.md` + `tasks.md` (requiere [ADR-0002](docs/decisions/ADR-0002-stack.md) decidido) |
| Implementación de una tarea | `sdd-implement` | código + tests, en `../metabase` |
| Validación de una spec | `sdd-validate` | veredicto RF por RF |
| Cambio de requisito | `sdd-change` | spec actualizada primero, luego plan y tareas |
| Lote de tareas sin supervisión | `sdd-run` | rama `spec-NNN/Ta-Tb` + pull request con informe; solo con autorización del propietario ([`docs/sdd/ejecucion-desatendida.md`](docs/sdd/ejecucion-desatendida.md)) |

## Recursos externos

Registro completo con propietario y si un agente puede usarlo: [`docs/sdd/recursos-externos.md`](docs/sdd/recursos-externos.md).
Si una tarea necesita una credencial, cuenta o acceso que no tienes, **no la simules**:
márcala `[H]` o `[M]` y párate.

## Reglas de autoría y estilo (para toda persona y agente, en cualquier herramienta)

- Los commits, pull requests y archivos van únicamente a nombre de la persona que los hace. **Nunca** se añaden trailers `Co-Authored-By`, `Claude-Session`, líneas "Generated with Claude Code" ni ninguna otra atribución a una herramienta o modelo. El CI rechaza los commits que las lleven.
- **Nunca el guion largo** (em dash, U+2014) en ningún texto. Se usa "-". El CI lo comprueba en todo texto propio; queda fuera el texto de terceros vendido tal cual (skills `ponytail` e `impeccable`, bloque que genera Next.js).
- Estas reglas viven en el repositorio (este archivo, `.claude/settings.json` y el CI) para que apliquen a cualquiera que lo clone, sin depender de configuración local.

## Reglas

1. Lee [`docs/constitution.md`](docs/constitution.md) y la spec activa antes de escribir nada.
2. El stack es el de Metabase ([ADR-0002](docs/decisions/ADR-0002-stack.md)): ninguna librería nueva, ningún cambio de arquitectura y nada dentro de `enterprise/` del fork.
3. No modifiques el material de referencia del workspace. Es solo lectura.
4. Nunca copies a este repo datos personales reales. Los ejemplos usan valores ficticios y la base de datos de ejemplo de Metabase.
5. Si un marcador `[NECESITA ACLARACIÓN]` bloquea un requisito, pregunta; no rellenes el hueco con una suposición silenciosa.
6. Toda decisión que cambie el modelo de dominio, el alcance de las tarjetas afectadas o toque el backend del fork requiere un ADR nuevo, no una edición silenciosa.
7. Actualiza [`specs/README.md`](specs/README.md) (estado y número de marcadores) cada vez que toques una spec.
8. **Herramientas obligatorias en todo repo de código, plan y tarea**: Context7 antes de escribir código con cualquier librería (documentación de la versión instalada, nunca de memoria), Ponytail en cada tarea (el mínimo código que funciona; ninguna dependencia sin justificar) e impeccable en toda interfaz. Cada `plan.md` lleva la sección "Herramientas obligatorias del agente" y cada `tasks.md` la tarea transversal TX-3.
9. **Ramas y pull requests, siempre**: nadie escribe directamente en `main` (este repo) ni en `develop`, `main` o `master` (repo de código). Todo cambio va en una rama (`spec-NNN/...`, `chore/...`, `docs/...`) y termina en un pull request que revisa y mezcla el propietario. En el repo de código los PR van contra `develop` (entorno de pruebas) y `develop` pasa a `main` (producción) por un PR de liberación; `master` solo recibe `upstream/master`. **Antes de cualquier cambio**: `git fetch` y `git pull` de la rama base (`main` aquí, `develop` en el repo de código) y crear la rama nueva desde ahí; si hay cambios sin commit o la rama base está por detrás, párate. Detalle en [ADR-0001](docs/decisions/ADR-0001-repositorios.md).

## Al terminar cualquier tarea

- Ejecuta las comprobaciones de [`docs/sdd/README.md`](docs/sdd/README.md) → "Checklist de calidad" que apliquen.
- Resume en tu respuesta qué archivos cambiaste y qué marcadores quedan abiertos.
