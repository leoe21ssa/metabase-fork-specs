# Migrar una instancia de Metabase existente a la instancia local

Cómo traer todo el contenido de un Metabase en producción (usuarios, colecciones, preguntas,
modelos, métricas, dashboards, permisos, ajustes y conexiones a datos) a la instancia local del
fork, para desarrollar y probar el [selector de métrica](../../specs/001-selector-de-metrica/spec.md)
sobre contenido real. Requiere el entorno de [entorno-desarrollo.md](entorno-desarrollo.md).

Caso cubierto: la instancia origen está en un servidor propio (VPS) al que se entra por SSH, y
quien migra es administrador de ese Metabase y del servidor. Los nombres de servidor, usuarios y
rutas de este documento son marcadores (`<servidor>`, `<contenedor>`); nunca se escriben los
reales aquí ni en un chat.

Ejecutado por primera vez el 2026-09-28 (origen 0.62 con base Postgres, copia de unos 10 MB): los
tiempos, salidas y avisos de este documento vienen de esa ejecución.

## Qué se migra y qué no

- **Se migra**: la base de datos de la aplicación de Metabase entera. Es un solo archivo (H2) o
  una base Postgres/MySQL. Contiene todo lo listado arriba, incluidas las contraseñas de los
  usuarios (cifradas: cada uno entra con la suya) y los datos de conexión a las fuentes.
- **No se migra**: los datos de negocio. Las preguntas siguen apuntando a las bases de datos
  originales; la instancia local tiene que poder conectarse a ellas (sección 5).
- **Se pierde**: lo que dependa de la edición comercial (si el origen es Pro o Enterprise), porque
  el fork es la edición libre. Caché de resultados y sesiones abiertas: se regeneran.
- **Sentido único**: es una copia. Nada de lo que se haga en local vuelve al servidor. Para
  actualizar, se repite la copia con otra fecha.

Versiones: el origen corre una versión publicada (0.5x o 0.6x); el fork es más nuevo que
cualquiera publicada. Al arrancar, Metabase actualiza el esquema de la copia hacia delante y no
hay vuelta atrás. Por eso el servidor no se toca y en local se guarda una copia intacta antes del
primer arranque.

## 1. Reconocimiento en el servidor (sin secretos)

Entra por SSH y averigua cómo corre Metabase. Los comandos de esta sección solo leen; ninguno
imprime contraseñas ni claves, así que su salida se puede pegar en un chat.

```bash
ssh <usuario>@<servidor>
docker ps --format '{{.Names}}\t{{.Image}}\t{{.Status}}'
```

Si aparece un contenedor de Metabase, apunta su nombre (`<contenedor>`) y su imagen (la etiqueta
es la versión, por ejemplo `metabase/metabase:v0.56.9`). Si la imagen no lleva etiqueta, apunta
su identificador exacto y la versión que sirve:

```bash
docker inspect <contenedor> --format '{{.Image}}'
curl -s http://localhost:3000/api/session/properties | grep -o '"tag":"[^"]*"'
```

Luego:

```bash
# tipo de base de la aplicación y versión (solo esas variables, con valor)
docker inspect <contenedor> --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -E '^MB_(DB_TYPE|DB_FILE|VERSION|SITE_URL)='
# si existe MB_DB_CONNECTION_URI, solo su esquema (la URI entera lleva la contraseña)
docker inspect <contenedor> --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -oE '^MB_DB_CONNECTION_URI=[a-z]+:'
# nombres de todas las variables MB_ definidas, sin valores
docker inspect <contenedor> --format '{{range .Config.Env}}{{println .}}{{end}}' | grep -oE '^MB_[A-Z_]+' | sort
# carpetas montadas (dónde vive el archivo H2 o los datos)
docker inspect <contenedor> --format '{{range .Mounts}}{{.Type}} {{.Source}} -> {{.Destination}}{{println}}{{end}}'
```

Interpretación:

| Lo que ves | Significa |
|---|---|
| Ni `MB_DB_TYPE=` ni `MB_DB_CONNECTION_URI`, o `MB_DB_TYPE=h2` | Base H2: un archivo. Rama A de la sección 2. |
| `MB_DB_TYPE=postgres` o `mysql` | Base externa. Rama B de la sección 2. |
| `MB_DB_CONNECTION_URI=postgresql:` o `mysql:` | Base externa definida por URI (sin `MB_DB_TYPE`). Rama B de la sección 2. |
| `MB_ENCRYPTION_SECRET_KEY` en la lista de nombres | Conexiones cifradas: la instancia local necesita esa misma clave (sección 4). Apunta solo que existe. |
| `MB_EMAIL_SMTP_*` en la lista de nombres | El correo se configura por variables, no en la base: la copia local no lo hereda (sección 5). |
| Un montaje `bind /ruta/en/el/servidor -> /metabase.db` | El archivo H2 está en esa ruta del servidor. |

Si `docker ps` no muestra nada, Metabase corre como servicio con el JAR:

```bash
systemctl list-units --type=service | grep -i metabase
systemctl cat <servicio> | grep -oE 'MB_[A-Z_]+' | sort -u
systemctl cat <servicio> | grep -E 'WorkingDirectory|ExecStart'
```

`ExecStart` dice dónde está `metabase.jar`; sin `MB_DB_TYPE`, el archivo H2 `metabase.db.mv.db`
está en `WorkingDirectory`.

Completa desde el navegador: menú de ajustes, "Acerca de Metabase": versión y edición (si dice
Pro o Enterprise, parte del contenido no funcionará en el fork).

## 2. Obtener la copia en el servidor

Avisa a los usuarios: el paso A para Metabase entre uno y tres minutos; el B no lo para.

### Rama A: base H2 (archivo)

Copiar el archivo con Metabase en marcha da una copia corrupta: hay que pararlo.

Con Docker (el archivo está dentro del contenedor o en el montaje visto en la sección 1):

```bash
docker stop <contenedor>
docker cp <contenedor>:/metabase.db/metabase.db.mv.db ~/metabase-copia-AAAA-MM-DD.mv.db
docker start <contenedor>
ls -lh ~/metabase-copia-AAAA-MM-DD.mv.db
```

Si `docker inspect` mostró un montaje `bind`, la ruta origen es `<ruta-del-montaje>/metabase.db.mv.db`
y vale `cp` en lugar de `docker cp` (con el contenedor parado igualmente).

Con el JAR como servicio:

```bash
sudo systemctl stop <servicio>
cp <WorkingDirectory>/metabase.db.mv.db ~/metabase-copia-AAAA-MM-DD.mv.db
sudo systemctl start <servicio>
```

Comprueba que el servicio volvió: abre Metabase en el navegador.

### Rama B: base Postgres o MySQL

Metabase incluye el comando `dump-to-h2`, que vuelca la base de la aplicación a un archivo H2
usando la misma versión y las mismas variables de conexión que el servidor. Se ejecuta en un
contenedor aparte, con la misma imagen, compartiendo la red del contenedor real para llegar a la
base:

```bash
mkdir -p ~/mb-mig && chmod 700 ~/mb-mig
docker inspect <contenedor> --format '{{range .Config.Env}}{{println .}}{{end}}' | grep '^MB_DB_' > ~/mb-mig/mb-tmp.env && chmod 600 ~/mb-mig/mb-tmp.env
docker run --name mb-volcado --env-file ~/mb-mig/mb-tmp.env --network container:<contenedor> <id-imagen> "dump-to-h2 /tmp/metabase-copia-AAAA-MM-DD" 2>&1 | tail -20
docker cp mb-volcado:/tmp/metabase-copia-AAAA-MM-DD.mv.db ~/mb-mig/
docker rm mb-volcado && rm ~/mb-mig/mb-tmp.env
ls -la ~/mb-mig
```

Detalles que importan:

- `<id-imagen>` es el identificador exacto de la imagen del contenedor en marcha (sección 1), no
  una etiqueta más nueva: `dump-to-h2` aplica las migraciones de su versión a la base origen antes
  de copiar, y una imagen más nueva cambiaría producción.
- Solo se copian las variables `MB_DB_` (la conexión), no las de correo. El archivo contiene la
  contraseña de la base: carpeta y archivo solo legibles por ti, y se borra en la misma sesión.
- El comando y la ruta van juntos en un único argumento entre comillas. El script de arranque de
  la imagen solo pasa a Java el primer argumento; separados, Metabase responde `The 'dump-to-h2'
  command requires the following arguments: [h2-filename & opts], but received: []`.
- Contenedor con nombre y `docker cp` en lugar de montar una carpeta: el archivo queda de tu
  usuario, no de root. Java corre dentro como usuario sin privilegios y escribe en `/tmp`.
- Metabase de producción no se para; hazlo en una hora de poco uso (arranca una segunda máquina
  Java). Con una base de unos 10 MB tarda menos de un minuto y termina con `Dump complete`.

Con el JAR: `java -jar metabase.jar dump-to-h2 ~/metabase-copia-AAAA-MM-DD` desde el
`WorkingDirectory` y con las mismas variables `MB_DB_*` exportadas.

## 3. Traer la copia a la máquina local

En Ubuntu (WSL) o en el Mac, fuera del repo y fuera de OneDrive:

```bash
mkdir -p ~/work/data && chmod 700 ~/work/data
scp <usuario>@<servidor>:mb-mig/metabase-copia-AAAA-MM-DD.mv.db ~/work/data/
cd ~/work/data && chmod 600 metabase-copia-AAAA-MM-DD.mv.db && cp -p metabase-copia-AAAA-MM-DD.mv.db metabase-copia-AAAA-MM-DD.original.mv.db
md5sum ~/work/data/metabase-copia-AAAA-MM-DD.mv.db
```

Con la Rama A el archivo está en `~` del servidor, sin la carpeta `mb-mig`. La tercera línea guarda
la copia intacta: la primera se transformará al arrancar. Carpeta y archivos quedan solo legibles
por ti porque, si el servidor no tenía clave de cifrado (sección 4), la copia lleva en claro las
contraseñas de las conexiones a datos. Si prefieres un programa gráfico (WinSCP, FileZilla), el
destino en Windows es `\wsl$\Ubuntu\home\<usuario-linux>\work\data`.

Calcula en el servidor la misma huella, `md5sum ~/mb-mig/metabase-copia-AAAA-MM-DD.mv.db`, y si
coincide borra allí la carpeta: `rm -rf ~/mb-mig && ls ~/mb-mig` (el `ls` debe fallar con
`No such file or directory`: es la confirmación).

## 4. Clave de cifrado (solo si existía en el servidor)

Si la sección 1 mostró `MB_ENCRYPTION_SECRET_KEY`, la instancia local necesita el mismo valor o
no podrá leer las contraseñas de las conexiones a datos ni algunos ajustes. Guárdala en un
archivo fuera del repo, legible solo por ti, y nunca la pegues en un chat:

```bash
touch ~/work/scripts/mb-secrets.sh && chmod 600 ~/work/scripts/mb-secrets.sh
```

Edita `~/work/scripts/mb-secrets.sh` con `nano` y escribe una sola línea:
`export MB_ENCRYPTION_SECRET_KEY="<valor copiado del servidor>"`. El valor se obtiene en el
servidor con `docker inspect` sin filtrar (o `systemctl cat`) y se copia a mano.

Si la clave no está disponible, Metabase arranca igual: las conexiones a datos aparecen y hay que
volver a escribir su contraseña en Administración, Bases de datos. Si el servidor nunca tuvo
clave, no hay nada que hacer: las contraseñas de las conexiones van en claro dentro de la copia y
funcionan tal cual (de ahí los permisos de la sección 3).

## 5. Arrancar la instancia local con la copia

Para el backend actual y arráncalo apuntando a la copia (ruta sin `.mv.db`). El primer arranque
aplica todas las migraciones del fork y tarda varios minutos; el log muestra líneas de
`liquibase` y termina con `Metabase Initialization COMPLETE`.

Con dos terminales:

```bash
cd ~/work/metabase
source ~/work/scripts/mb-secrets.sh   # solo si existe (sección 4)
MB_DISABLE_SCHEDULER=true MB_SITE_URL=http://localhost:3000 MB_ANON_TRACKING_ENABLED=false MB_DB_FILE=$HOME/work/data/metabase-copia-AAAA-MM-DD clojure -M:run
```

Con el servicio de systemd (Windows, sección 5d de la guía de entorno):

```bash
systemctl --user stop mb-backend
systemd-run --user --collect --unit=mb-backend -p WorkingDirectory="$HOME/work/metabase" -p SuccessExitStatus=143 --setenv=MB_DISABLE_SCHEDULER=true --setenv=MB_SITE_URL=http://localhost:3000 --setenv=MB_ANON_TRACKING_ENABLED=false --setenv=MB_DB_FILE="$HOME/work/data/metabase-copia-AAAA-MM-DD" bash -c 'source ~/work/scripts/mb-env.sh; [ -f ~/work/scripts/mb-secrets.sh ] && source ~/work/scripts/mb-secrets.sh; exec clojure -M:run > ~/work/logs/backend.log 2>&1'
tail -f ~/work/logs/backend.log
```

Si `systemd-run` responde `Unit mb-backend.service already exists`, ejecuta
`systemctl --user reset-failed mb-backend` y repítelo (guía de entorno, sección 7).

Las tres variables nuevas:

- `MB_DISABLE_SCHEDULER=true` apaga en el primer arranque las suscripciones, alertas y
  sincronizaciones: si la copia trae la configuración de correo o Slack del servidor, sin esto la
  instancia local mandaría los mismos envíos que producción a destinatarios reales.
- `MB_SITE_URL=http://localhost:3000` sustituye la URL del sitio guardada en la copia (la de
  producción). Por variable y no en el formulario: el panel de administración local es idéntico
  al de producción y es fácil cambiar el de producción por error; además sobrevive a copias
  nuevas. El campo aparece como fijado por `MB_SITE_URL`.
- `MB_ANON_TRACKING_ENABLED=false` evita que la instancia de pruebas envíe estadísticas de uso.

El primer arranque de una copia de unos 10 MB (origen 0.62) tardó algo más de un minuto.

Entra en http://localhost:3000 con tu usuario y contraseña de siempre (los de producción), en una
pestaña cuya barra de direcciones diga `localhost:3000`. Antes de quitar `MB_DISABLE_SCHEDULER`:

1. Administración, Ajustes, General: la URL del sitio debe aparecer fijada por `MB_SITE_URL`.
2. Administración, Ajustes, Correo electrónico: borra la configuración SMTP si existe (si el
   servidor la tenía por variables `MB_EMAIL_*`, aquí no hay nada).
3. Administración, Ajustes, Notificaciones (Slack): desconecta la app si existe.
4. Administración, Bases de datos: revisa cada conexión (siguiente sección).

Sin sesión, los puntos 2 y 3 se comprueban por API: `curl -s localhost:3000/api/session/properties`
contiene `"email-configured?":false` y `"slack-token-valid?":null` cuando no hay nada.

Después reinicia el backend sin `MB_DISABLE_SCHEDULER` (mismo comando sin esa variable, con las
otras dos): así funcionan la sincronización de esquemas y el índice de búsqueda. En el log, las
líneas `reset from ERROR state to: WAITING` son avisos normales de tareas programadas que venían
marcadas en producción, no errores.

## 6. Conexiones a las fuentes de datos

Las conexiones guardan el host tal y como lo veía el servidor. Si es `localhost`, un nombre de
red de Docker o una IP privada, desde tu máquina no existe. Si es una IP o un dominio público
(una base gestionada), puede que no haga falta nada: comprueba primero.

```bash
timeout 5 bash -c 'exec 3<>/dev/tcp/<host>/<puerto>' && echo alcanzable
```

Si responde `alcanzable`, abre un dashboard en local: si carga datos, la conexión ya funciona
(que el puerto responda no garantiza que la base acepte tu IP; solo la consulta lo confirma). Si
no, para cada base, en Administración, Bases de datos, elige una opción:

- **Túnel SSH** (recomendado, viene de serie): activa "Usar un túnel SSH" en la conexión, con el
  `<servidor>` como host del túnel y tu usuario SSH; el host de la base sigue siendo el nombre
  interno (`localhost` o el del contenedor, visto desde el servidor). Necesita una clave SSH o
  contraseña, que se guarda cifrada en la copia local.
- **Abrir el puerto** en el firewall del proveedor solo para tu IP pública, y poner en la conexión
  el host público.
- **Dejarla sin conexión**: las preguntas se ven pero no se ejecutan. Vale para probar el editor de
  visualización, no para ver datos.

La base de datos de ejemplo (Sample Database) ya está en la copia si el servidor la tenía.

## 7. Comprobar que todo llegó

En el navegador de la instancia local, compara con el servidor: colecciones de primer nivel y su
contenido, número de usuarios en Administración, Personas, y las bases de datos en Administración.
Abre un dashboard representativo y una pregunta de cada base: si carga datos, la conexión
funciona.

Por API, en las dos instancias (la contraseña la pide `read`, no va en la línea):

```bash
read -rsp 'Contraseña: ' P; echo
T=$(curl -s -X POST <url>/api/session -H 'Content-Type: application/json' -d "{\"username\":\"<correo>\",\"password\":\"$P\"}" | python3 -c 'import sys,json; print(json.load(sys.stdin)["id"])'); unset P
for r in dashboard card user database collection; do printf '%s=' "$r"; curl -s -H "X-Metabase-Session: $T" "<url>/api/$r" | python3 -c 'import sys,json; d=json.load(sys.stdin); print(len(d if isinstance(d,list) else d.get("data",d)))'; done
```

Los recuentos deben coincidir (los de la instancia local pueden ser mayores si ya creaste algo).

Recuento rápido sin API: al arrancar, el log local imprime `Index initialized in ... (["card" N]
["table" N] ["collection" N] ["dashboard" N] ...)`; si la versión del servidor lo tiene, `docker
logs <contenedor>` muestra la misma línea de su último arranque.

## 8. Convivencia con la instancia de ejemplo

- Sin `MB_DB_FILE`, el backend usa `metabase.db` en la raíz del repo (la del asistente de primer
  arranque, con la base de ejemplo). Con `MB_DB_FILE`, la copia real. Se alterna parando y
  arrancando el backend; el frontend no cambia.
- Los tests de extremo a extremo y las tareas de la spec usan la de ejemplo; la copia real sirve
  para probar a mano.
- Para refrescar la copia: repetir las secciones 2, 3 y 5 con una fecha nueva. La anterior se
  puede borrar; la `.original` de la última se conserva.
- `~/work/data` no está en git ni en OneDrive; si la máquina se pierde, se repite la migración
  desde el servidor.
