# Entorno de desarrollo paso a paso (Windows y Mac)

Guía para dejar una máquina lista para trabajar en el fork de Metabase con el método SDD:
herramientas, cuentas, clones y una instancia local de Metabase funcionando. Está escrita para
alguien que no usa la terminal a diario: cada paso trae el comando, lo que debe imprimir y cómo
comprobarlo. Verificada en Windows 11 con WSL el 2026-09-28 (prerrequisitos P2..P5 de la
[spec 001](../../specs/001-selector-de-metrica/tasks.md)); los pasos marcados **Mac** usan los
instaladores oficiales pero no se han verificado en un Mac.

Las versiones son las que fija el repo de código (`.nvmrc`, `packageManager` en `package.json`,
`.github/scripts/cache-keys.sh`); ver [metabase-fork.md](../reference/metabase-fork.md). No se usa
`./bin/dev-install` del repo: es interactivo, instala `mise` y una segunda copia de todas las
herramientas.

## 0. Qué tendrás al final y cuánto tarda

| Bloque | Tiempo aproximado |
|---|---|
| Windows: WSL con Ubuntu | 15 min y un reinicio |
| Herramientas (Java, Clojure, Node, Bun, gh, uv, Claude Code) | 20 min |
| Cuentas (GitHub, Context7, Claude) | 10 min |
| Clones de los dos repos | 10 min (el de Metabase pesa 1,4 GB) |
| Instancia local (primer arranque) | 10 min |

Reglas para toda la guía:

- Pega **una línea por vez** y espera a que vuelva el símbolo `$` antes de la siguiente.
- Revisa que la línea pegada empiece exactamente como aquí: un carácter de más al principio
  (por ejemplo `~cp`) hace que falle.
- `sudo` va en minúsculas y pide tu contraseña de Ubuntu; no se ve mientras la escribes.
- Nunca pegues una clave o contraseña en un chat ni en un archivo del repo.

## 1. Sistema operativo

### 1a. Solo Windows: WSL 2 con Ubuntu 24.04

1. Abre PowerShell como administrador y ejecuta:

   ```powershell
   wsl --install -d Ubuntu
   ```

   Reinicia cuando lo pida. Al abrir "Ubuntu" desde el menú Inicio te pedirá un nombre de usuario
   (minúsculas, sin espacios) y una contraseña. Apúntala: es la que pide `sudo`.

2. Comprueba en PowerShell que la distribución corre en la versión 2:

   ```powershell
   wsl -l -v
   ```

   Debe listar `Ubuntu` con `VERSION 2`.

3. Reserva recursos para la máquina virtual. Crea el archivo `C:\Users\<tu-usuario>\.wslconfig`
   con este contenido, ajustando a la mitad de los núcleos y de la memoria de tu PC (el build
   de Metabase necesita al menos 4 núcleos y 12 GB):

   ```ini
   [wsl2]
   processors=8
   memory=16GB
   swap=4GB
   localhostForwarding=true
   ```

   Aplica con `wsl --shutdown` en PowerShell y vuelve a abrir Ubuntu.

4. Instala VS Code en Windows con las extensiones "WSL" (`ms-vscode-remote.remote-wsl`) y
   "Claude Code" (`anthropic.claude-code`). Las carpetas de Linux se abren desde Ubuntu con
   `code ~/work/metabase`; la primera vez instala un servidor de VS Code dentro de WSL.

5. Instala Git para Windows si no lo tienes: aporta el gestor de credenciales que WSL reutiliza
   para no volver a iniciar sesión en GitHub (paso 3a).

En el resto de la guía, "la terminal" es la ventana de Ubuntu.

### 1b. Solo Mac

1. Herramientas de compilación de Apple (abre un diálogo; acepta):

   ```bash
   xcode-select --install
   ```

2. Homebrew, siguiendo la línea única de https://brew.sh. Al terminar, ejecuta las dos líneas
   que el instalador imprime para añadir `brew` a tu `~/.zprofile`.

En el resto de la guía, "la terminal" es la aplicación Terminal. Donde la guía diga
`~/.profile` (Ubuntu) usa `~/.zprofile` (Mac); donde diga `apt`, usa el comando **Mac** indicado.

## 2. Herramientas

Todo se instala en tu carpeta personal salvo lo que va con `apt` o `brew`. Cuando un instalador
diga que cierres la terminal, ciérrala y ábrela de nuevo antes de seguir: si no, el comando
recién instalado "no se encuentra".

### 2a. Paquetes base

Ubuntu:

```bash
sudo apt update && sudo apt install -y curl git unzip zip rlwrap
```

**Mac** (curl y git ya vienen con las herramientas de Apple):

```bash
brew install rlwrap
```

### 2b. Java 25 (Temurin) con SDKMAN

```bash
curl -s "https://get.sdkman.io" | bash
```

Cierra y abre la terminal. Después:

```bash
sdk install java 25.0.4-tem
java -version
```

La última línea debe empezar por `openjdk version "25.0.4"`. Igual en Mac.

### 2c. Clojure CLI 1.12.0.1488

Ubuntu (el instalador necesita `sudo`; deja el programa en `/usr/local/bin`):

```bash
curl -L -O https://download.clojure.org/install/linux-install-1.12.0.1488.sh
chmod +x linux-install-1.12.0.1488.sh
sudo ./linux-install-1.12.0.1488.sh
rm linux-install-1.12.0.1488.sh
clojure --version
```

**Mac**:

```bash
brew install clojure/tools/clojure@1.12.0.1488
```

Si Homebrew dice que esa fórmula no existe, instala `clojure/tools/clojure` (la última versión;
cualquier 1.12 sirve). Comprobación en ambos: `clojure --version` imprime
`Clojure CLI version 1.12.0.1488` (o la instalada).

### 2d. Node 22 con nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.8/install.sh | bash
```

Cierra y abre la terminal. Después:

```bash
nvm install 22.23.2
node --version
```

Debe imprimir `v22.23.2` (la versión de `.nvmrc` del repo de código). Igual en Mac.

### 2e. Bun 1.3.14

```bash
curl -fsSL https://bun.sh/install | bash -s "bun-v1.3.14"
```

Cierra y abre la terminal; `bun --version` imprime `1.3.14`. Igual en Mac. El repo de código
bloquea npm y yarn: usa siempre `bun`.

### 2f. gh (línea de comandos de GitHub)

Ubuntu:

```bash
sudo apt install -y gh
```

**Mac**:

```bash
brew install gh
```

`gh --version` imprime `gh version 2.x`. Se usa para iniciar sesión en GitHub sin contraseña y para
administrar el repo (ramas, protección, flujos).

### 2g. uv 0.10.2

```bash
curl -LsSf https://astral.sh/uv/0.10.2/install.sh | sh
```

Cierra y abre la terminal; `uv --version` imprime `uv 0.10.2`. Metabase lo usa la primera vez que
analiza una consulta SQL (instala una librería de Python por su cuenta); sin `uv` esa parte falla
con un error en el log del backend. Igual en Mac.

### 2h. Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Cierra y abre la terminal; `claude --version` imprime un número de versión. Igual en Mac. La
sesión se inicia en el paso 3c.

## 3. Cuentas

### 3a. GitHub

```bash
gh auth login
```

Respuestas: `GitHub.com`; `HTTPS`; a "Authenticate Git with your GitHub credentials?" responde
**No** en Windows (git usará el gestor de credenciales de Windows, siguiente bloque) y **Yes** en
Mac; `Login with a web browser`. Copia el código de ocho letras, abre en tu navegador la dirección
que imprime (Ubuntu no puede abrirlo por ti) y pega el código. Comprueba con `gh auth status`:
debe decir `Logged in to github.com account <tu-usuario>`.

Solo Windows: que git dentro de Ubuntu reutilice el inicio de sesión de Git para Windows:

```bash
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

### 3b. Clave de Context7

Context7 da documentación actualizada a los agentes; la [ejecución desatendida](ejecucion-desatendida.md)
la registra a nivel de usuario. Crea una clave en https://context7.com (panel de la cuenta,
apartado API keys). La clave va en el perfil de inicio de sesión, no en `~/.bashrc`: los
procesos no interactivos (agentes, servicios) leen `~/.profile` (Ubuntu) o `~/.zprofile` (Mac)
y se saltan `~/.bashrc`.

1. En el Bloc de notas (o TextEdit) prepara esta línea con tu clave en lugar de `TU_CLAVE`:

   ```bash
   printf 'export CONTEXT7_API_KEY="%s"\n' 'TU_CLAVE' >> ~/.profile
   ```

2. Pégala en la terminal **de una vez** y pulsa Enter. Después borra el historial:

   ```bash
   history -c; clear
   ```

3. Cierra y abre la terminal y comprueba, sin mostrar la clave:

   ```bash
   bash -lc 'test -n "$CONTEXT7_API_KEY" && echo definida'
   curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer $CONTEXT7_API_KEY" 'https://context7.com/api/v1/search?query=react'
   ```

   Debe imprimir `definida` y `200`. Una clave mal copiada devuelve `401`; una clave vacía
   también devuelve `200`, por eso importa la palabra `definida`.

Si te equivocas, borra las líneas antiguas antes de repetir el paso 1:
`sed -i '/^export CONTEXT7_API_KEY=/d' ~/.profile` (en Mac, `sed -i '' ...` y `~/.zprofile`).

### 3c. Claude Code

```bash
claude auth login
```

Copia la dirección que imprime, ábrela en el navegador con tu cuenta de Claude y pega el código
de vuelta. `claude auth status` confirma la sesión. Hazlo desde Ubuntu, no desde PowerShell: la
sesión se guarda en la máquina donde ejecutas el comando.

## 4. Clones

Los dos repos van como hermanos dentro de `~/work`, en el disco de Linux (Windows) o de tu
usuario (Mac), nunca en una carpeta sincronizada por OneDrive, iCloud o Dropbox
([ADR-0001](../decisions/ADR-0001-repositorios.md)).

```bash
mkdir -p ~/work && cd ~/work
git clone https://github.com/leoe21ssa/metabase.git
git clone https://github.com/leoe21ssa/sdd-template.git
```

El primero pesa 1,4 GB y tarda varios minutos. Después, en cada repo, la identidad con la que
firmarás commits (usa el correo `noreply` que muestra tu perfil de GitHub, ajustes, Emails) y el
remoto `upstream` para traer cambios del proyecto original:

```bash
cd ~/work/metabase
git config user.name "<tu-usuario-github>"
git config user.email "<id>+<tu-usuario-github>@users.noreply.github.com"
git remote add upstream https://github.com/metabase/metabase.git
git checkout develop

cd ~/work/sdd-template
git config user.name "<tu-usuario-github>"
git config user.email "<id>+<tu-usuario-github>@users.noreply.github.com"
git remote add upstream https://github.com/julian-ssa/sdd-template.git
```

Comprobación: `git remote -v` lista `origin` (tu fork) y `upstream`; `git branch --show-current`
imprime `develop` en el fork (las ramas de trabajo salen de ahí, ver [AGENTS.md](../../AGENTS.md)).

## 5. Instancia local de Metabase

### 5a. Dependencias del frontend (una vez)

```bash
cd ~/work/metabase
bun install
```

Tarda unos minutos y termina con `NNNN packages installed` (5.868 en el commit de referencia).
Crea `node_modules` (1,7 GB) y no cambia nada que git vea (`git status` limpio).

### 5b. Arrancar backend y frontend

Hacen falta dos terminales (en Ubuntu, abre dos ventanas; o usa `tmux`).

Terminal 1, backend (la primera vez descarga unas 1.000 librerías de Java, 5 min):

```bash
cd ~/work/metabase && clojure -M:run
```

Está listo cuando el log dice `Metabase Initialization COMPLETE`. Escucha en http://localhost:3000.

Terminal 2, frontend en modo desarrollo (compila ClojureScript y luego sirve el JavaScript con
recarga en caliente; 3 min la primera vez):

```bash
cd ~/work/metabase && bun run build-hot
```

Está listo cuando imprime `Rspack compiled successfully`. Deja las dos terminales abiertas:
cerrarlas apaga los servicios. Con `Ctrl+C` los paras.

Alternativa en una sola terminal: `bun run dev` (repite `bun install`, arranca backend, frontend
y procesos auxiliares). Es la del repo de código; la de dos terminales se verificó en la
máquina de referencia.

### 5c. Asistente de primer arranque (en el navegador)

Abre http://localhost:3000. La primera vez aparece el asistente:

1. Idioma.
2. Cuenta de administrador: nombre, correo y contraseña. Es una cuenta local de esta instancia;
   no tiene que ver con GitHub ni con Claude.
3. Datos: elige "añadir mis datos más tarde". La base de datos de ejemplo (Sample Database)
   viene incluida.
4. Envío de datos de uso: como prefieras.

Termina en la página de inicio con los ejemplos de la base de datos de ejemplo. Comprobación desde
la terminal:

```bash
curl -s localhost:3000/api/session/properties | grep -o '"has-user-setup":[a-z]*'
```

Debe imprimir `"has-user-setup":true`. Mientras diga `false`, el asistente no se ha completado.

La base de datos de la aplicación (usuarios, preguntas, dashboards) es el archivo
`metabase.db.mv.db` en la raíz del repo de código, ignorado por git. Para arrancar con otra (por
ejemplo una copia de producción) ver [migracion-instancia-local.md](migracion-instancia-local.md).

### 5d. Solo Windows: dejar los servicios en segundo plano

Los procesos lanzados desde una llamada `wsl.exe` (por ejemplo por un agente desde Windows) mueren
cuando esa llamada termina. Los servicios de usuario de systemd sobreviven. Requiere systemd
activado en WSL (`/etc/wsl.conf` con `[boot]` y `systemd=true`; en Ubuntu 24.04 viene así).

Una vez, crea `~/work/scripts/mb-env.sh` que carga las herramientas del paso 2 en shells no
interactivos:

```bash
mkdir -p ~/work/scripts ~/work/logs
cat > ~/work/scripts/mb-env.sh <<'FIN'
# Carga las herramientas instaladas en shells no interactivos (systemd, wsl.exe)
source "$HOME/.sdkman/bin/sdkman-init.sh"
export NVM_DIR="$HOME/.nvm"; . "$NVM_DIR/nvm.sh"
export PATH="$HOME/.bun/bin:$HOME/.local/bin:$PATH"
FIN
```

Arrancar (el backend primero; el frontend cuando el backend haya terminado de descargar):

```bash
systemd-run --user --unit=mb-backend -p WorkingDirectory="$HOME/work/metabase" bash -c 'source ~/work/scripts/mb-env.sh && exec clojure -M:run > ~/work/logs/backend.log 2>&1'
systemd-run --user --unit=mb-frontend -p WorkingDirectory="$HOME/work/metabase" bash -c 'source ~/work/scripts/mb-env.sh && exec bun run build-hot > ~/work/logs/frontend.log 2>&1'
```

Ver estado, logs y parar:

```bash
systemctl --user status mb-backend mb-frontend --no-pager
tail -n 5 ~/work/logs/backend.log ~/work/logs/frontend.log
systemctl --user stop mb-backend mb-frontend
```

Sobreviven a cerrar la terminal, no a `wsl --shutdown` ni a reiniciar Windows (se vuelven a
lanzar con los mismos dos comandos).

## 6. Verificación completa

Un solo bloque; pégalo entero en una terminal recién abierta:

```bash
java -version 2>&1 | head -1; clojure --version; node --version; bun --version; gh --version | head -1; uv --version; claude --version
bash -lc 'test -n "$CONTEXT7_API_KEY" && echo context7=definida'
gh auth status 2>&1 | grep -E 'Logged in|not logged'
cd ~/work/metabase && git remote -v | grep -c upstream && cd ~/work/sdd-template && git remote -v | grep -c upstream
curl -s localhost:3000/api/health; echo; curl -s -o /dev/null -w 'frontend=%{http_code}\n' http://127.0.0.1:8080/app/dist/app-main.hot.bundle.js
```

Esperado (máquina de referencia, 2026-09-28):

| Línea | Valor |
|---|---|
| java | `openjdk version "25.0.4"` |
| clojure | `Clojure CLI version 1.12.0.1488` |
| node | `v22.23.2` |
| bun | `1.3.14` |
| gh | `gh version 2.45.0` (Ubuntu) o superior |
| uv | `uv 0.10.2` |
| claude | `2.1.282 (Claude Code)` o superior |
| context7 | `context7=definida` |
| gh auth | `Logged in to github.com account ...` |
| upstream | `2` y `2` |
| health | `{"status":"ok"}` (solo con el backend arrancado) |
| frontend | `frontend=200` (solo con el frontend arrancado) |

## 7. Problemas conocidos

- **`Command 'xxx' not found` justo después de instalarlo.** Cierra y abre la terminal: el
  instalador cambió `~/.bashrc` o `~/.zshrc` y la terminal abierta no lo ha releído.
- **Pegaste dos líneas de golpe y la segunda "desapareció" o se usó como respuesta.** Cuando un
  comando pide un dato (contraseña, clave), pega solo ese comando y espera el aviso.
- **`Command '~cp' not found` u otro comando con un carácter raro delante.** Se coló un carácter al
  copiar; vuelve a pegar la línea limpia. Si era parte de un bloque, revisa que el resto sí se
  ejecutó.
- **`bun run build-hot` muere con `JavaScript heap out of memory` o la máquina se congela.** Sube
  `memory` en `.wslconfig` (Windows) o cierra otras aplicaciones; el build pide unos 8 GB.
- **El backend arranca pero `localhost:3000` no responde en el navegador.** Espera a
  `Metabase Initialization COMPLETE`; comprueba con `ss -ltn | grep 3000` (Ubuntu) o
  `lsof -i :3000` (Mac) que el puerto está a la escucha.
- **`Address already in use` en 3000 u 8080.** Queda un arranque anterior: en Windows
  `systemctl --user stop mb-backend mb-frontend`; en general, busca el proceso con `ss -ltnp`
  (Ubuntu) o `lsof -i :3000` (Mac) y páralo.
- **El editor SQL falla y el log del backend menciona `uv`, `pip` o `sqlglot`.** Falta el paso 2g.
- **Lanzaste el backend desde Windows con `wsl.exe` y desapareció.** Es lo esperado; usa el
  paso 5d.
- **`gh auth login` o `claude auth login` no abren el navegador.** Ubuntu en WSL no tiene navegador:
  copia la dirección que imprimen y ábrela en Windows.
- **`git push` desde Ubuntu pide usuario y contraseña.** En Windows falta el bloque de
  `credential.helper` del paso 3a; en Mac responde Yes a "Authenticate Git" o ejecuta
  `gh auth setup-git`.
- **Avisos `LF will be replaced by CRLF`.** Solo en copias en Windows; inofensivos. Los commits
  se hacen desde Ubuntu o con `git -c core.safecrlf=false`.
- **Docker Desktop en Windows.** Comparte la máquina virtual de WSL: la memoria de `.wslconfig` es
  para los dos. Si Docker Desktop no arranca tras cambiar `.wslconfig`, revisa la sintaxis del
  archivo.
