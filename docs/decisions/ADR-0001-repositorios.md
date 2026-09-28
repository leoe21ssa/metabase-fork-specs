# ADR-0001 - Organización de repositorios; las specs como repo hermano del fork

Estado: aceptada · Fecha: 2026-09-24 · Aceptada por el propietario: 2026-09-25 · Enmienda 1 (repo de specs propio): 2026-09-28

## Contexto
- El código es un fork de Metabase (`metabase/metabase`, alrededor de 1,5 millones de líneas entre
  Clojure y TypeScript) clonado en `15_Metabase_fork/metabase` con remotos `origin`
  (`leoe21ssa/metabase`) y `upstream` (`metabase/metabase`). Su rama por defecto es `master` y hoy
  es idéntica a upstream en el commit `fa7362a1e4` (2026-09-23). El repo es público en GitHub
  y el fork es de uso interno de StreamSolve: no se ofrece a terceros (constitución, principio 9).
- Este repo nació como clon de `leoe21ssa/sdd-template` (fork de `julian-ssa/sdd-template`, marcado
  como plantilla en GitHub) y hasta el 2026-09-28 vivió en su rama `metabase/add-ontraport-views`.
  Desde la enmienda 1 es el repo `leoe21ssa/metabase-fork-specs`, con la historia completa de la
  plantilla. También es público.
- Metabase se desarrolla en Linux o macOS; en Windows el proyecto exige WSL. La carpeta actual
  del workspace está dentro de OneDrive, que sincronizaría `node_modules` y artefactos de build.

## Decisión
- Un repositorio git por pieza. La carpeta local de trabajo (**workspace**) es una carpeta
  normal, sin git, con los repos clonados uno al lado del otro:

  ```
  15_Metabase_fork/
  ├── metabase-fork-specs/ specs, ADRs, docs, skills (este repo)
  └── metabase/            repo de código: fork de metabase/metabase
  ```
- `metabase-fork-specs` es la fuente de verdad del producto (constitución, specs, ADRs, docs, skills).
  No contiene código y **no es dependencia de construcción ni de despliegue** del fork.
- El fork lleva un `AGENTS.md` propio (creado desde [`templates/code-repo/AGENTS.md`](../../templates/code-repo/AGENTS.md))
  que dice "las specs están en `../metabase-fork-specs`; si la carpeta no existe, clónala ahí", y enlaza
  las skills con `.agents/skills/sdd-* -> ../../../metabase-fork-specs/.agents/skills/sdd-*` (enlaces
  simbólicos relativos, uno por skill). Metabase ya trae sus propios archivos de contexto para
  agentes; el `AGENTS.md` del fork se añade sin borrarlos ni editarlos.
- Trazabilidad: `plan.md` y `tasks.md` viven en este repo; cada tarea marcada anota el commit del
  fork que la implementó, y los commits del fork citan spec y RF (`spec 001 T6 (RF-12): ...`).
- Upstream no se clona aparte: se lee desde el propio fork con `git fetch upstream`. Su contrato
  (rutas, comandos, puntos de inserción) está en [`docs/reference/metabase-fork.md`](../reference/metabase-fork.md)
  con el commit leído.
- El backlog vive en [`docs/product/roadmap.md`](../product/roadmap.md).

## Ramas y entornos
- Repo de código (fork):
  - `master`: espejo de `upstream/master`. Solo recibe `git pull upstream master`. Nunca commits propios.
  - `develop`: rama por defecto del fork e integración (entorno de pruebas). Se crea desde `master`.
    Recibe por pull request las ramas `spec-NNN/...` y `chore/...`.
  - `main`: producción. Solo recibe un pull request de liberación desde `develop`.
  - Versión nueva de Metabase: `master` se actualiza desde upstream y entra en `develop` por un
    pull request `chore/upstream-<versión>` con la suite en verde antes de mezclar.
  - CI en cada pull request y en cada push a `develop` y `main`.
- Repo de specs (`leoe21ssa/metabase-fork-specs`): `main`; todo cambio por rama y pull request. Su
  `main` nace del contenido de la rama `metabase/add-ontraport-views` (enmienda 1); desde entonces
  cada cambio entra por pull request.
- Los dos repos son públicos, así que la protección de ramas de GitHub es gratuita y se activa:
  `develop` y `main` en el fork y `main` en el repo de specs solo aceptan cambios por pull request
  (sin push directo, ni siquiera del propietario). La convención escrita en `AGENTS.md` se mantiene
  como respaldo.

## Entorno de trabajo
- El desarrollo se hace en WSL (Ubuntu) con los dos repos clonados como hermanos dentro del
  sistema de archivos de Linux (por ejemplo `~/work/15_Metabase_fork/`), no en `/mnt/d/...` ni
  en una carpeta sincronizada por OneDrive: el build de Metabase (webpack, Clojure, `node_modules`
  de varios GB) no es viable sobre carpetas sincronizadas o montadas.
- La máquina de trabajo es Windows 11; el editor es VS Code con la extensión WSL, que abre la
  carpeta de Linux directamente, por lo que no hace falta una copia en Windows para desarrollar.
- La copia actual en OneDrive queda para consulta desde Windows, con la sincronización de OneDrive
  desactivada para esa carpeta. Las dos copias se sincronizan solo por git (push y pull); nunca
  copiando carpetas. Trabajar en WSL sobre `/mnt/d/...` funciona pero es varias veces más lento
  (instalación de dependencias y build) y el vigilante de archivos de webpack no es fiable; se
  reserva como opción de emergencia.
- Paso a paso para preparar una máquina nueva (Windows con WSL o Mac), con verificación y
  problemas conocidos: [docs/sdd/entorno-desarrollo.md](../sdd/entorno-desarrollo.md).

## Consecuencias
- Cada repo se construye solo; el fork no necesita credenciales para este repo.
- Cambiar una spec es un commit aquí, sin tocar el fork; la referencia es siempre `main` de este repo.
- Un agente que trabaje en el fork necesita este repo clonado como hermano; el `AGENTS.md` del fork lo dice.
- Las integraciones de upstream pueden chocar con los puntos de inserción del fork; la constitución
  (principio 2) los mantiene mínimos y listados en cada plan.
- La carpeta local se llama `metabase-fork-specs/` (enmienda 1); el `AGENTS.md` del fork y los
  enlaces de las skills usan ese nombre.

## Enmienda 1 (2026-09-28): repo de specs propio
- Motivo: `leoe21ssa/sdd-template` es la plantilla del propietario para varios proyectos (fork de
  `julian-ssa/sdd-template`, marcado como plantilla en GitHub, y su README pide un repo `<nombre>-specs`
  por proyecto). Mezclar las specs de Metabase en su `main` habría contaminado todos los proyectos
  futuros creados desde ella.
- Decisión: se crea `leoe21ssa/metabase-fork-specs` (público, vacío) y se empuja en él la rama
  `metabase/add-ontraport-views` como `main`, con protección de rama solo por pull request; después
  la rama se borra de `leoe21ssa/sdd-template`, que queda como plantilla limpia. Las carpetas locales
  pasan a llamarse `metabase-fork-specs` (`~/work/metabase-fork-specs` en WSL y la copia de consulta
  en Windows). El remoto `upstream` del repo de specs sigue apuntando a `julian-ssa/sdd-template`,
  solo para traer mejoras de la plantilla.
- Descartado: mezclar en `main` de la plantilla (lo previsto en la versión original de este ADR).

## Alternativas descartadas
- **Specs dentro del fork (monorepo)**: cada integración de upstream arrastraría las specs, el diff
  con upstream dejaría de ser solo código y la documentación interna viajaría con el código AGPL.
- **Copiar upstream en el workspace**: queda obsoleto al primer cambio; el fork ya lo tiene como remoto.
- **Specs como submódulo git en el fork**: fija la versión por commit, obliga a `clone --recurse-submodules`
  y a un commit por cada cambio de spec, y no aporta nada al build.
- **Trabajar desde Windows nativo sobre la carpeta de OneDrive**: Metabase no soporta Windows sin WSL y
  OneDrive sincronizaría artefactos de build.
