# Spec 001 - Selector de métrica en tarjetas de dashboard

> Estado: planificada · Depende de: ninguna · Marcadores abiertos: 0

## Contexto y objetivo
Una pregunta con varias métricas (por ejemplo, correos enviados, abiertos, con clic y rebotados por semana) se dibuja hoy con todas las series superpuestas. Para ver una sola, el espectador tiene que ocultar el resto desde la leyenda, opción que el editor no configura, que no destaca y que se pierde al recargar. Los informes de Ontraport resuelven esto con una fila de botones sobre el gráfico ([comportamiento de referencia](../../docs/reference/ontraport-cards.md)). Esta spec añade a la tarjeta de dashboard un selector de métrica configurable por el editor: un botón por métrica incluida, uno activo a la vez, que muestra solo esa serie sin volver a consultar los datos. El selector solo existe cuando hay dos o más puntos de vista, es decir, dos o más métricas incluidas; con menos, la tarjeta no muestra botones. Términos según el [glosario](../../docs/product/glossary.md).

## Actores
- Editor de dashboard: usuario con permiso de edición del dashboard; activa y configura el selector en la tarjeta.
- Espectador: usuario con acceso al dashboard, también en dashboards públicos o embebidos; pulsa los botones.
- Administrador de Metabase: despliega el fork; no interviene en el uso.

## Historias de usuario
- H1: Como editor de dashboard quiero activar un selector de métrica en una tarjeta con varias métricas para que quien la mire elija qué métrica ver.
- H2: Como editor de dashboard quiero decidir qué métricas aparecen como botón, en qué orden y cuál se muestra al abrir el dashboard, para que la tarjeta cuente primero lo importante.
- H3: Como espectador quiero pulsar un botón y ver solo esa métrica al instante, con los mismos filtros y el mismo periodo, para comparar sin reconfigurar nada.
- H4: Como espectador quiero que la métrica que elegí se mantenga mientras cambio los filtros del dashboard y que vaya en la dirección de la página, para no repetir la selección en cada cambio y poder compartir el enlace con esa métrica ya elegida.
- H5: Como editor de dashboard quiero que las tarjetas sin selector se vean y funcionen exactamente igual que antes, para que el cambio no afecte a los dashboards existentes.

## Requisitos funcionales (criterios de aceptación en EARS)

### Activación del selector (H1)
- RF-1: DONDE la tarjeta de dashboard muestra un gráfico cartesiano con dos o más métricas, sin desglose y sin series añadidas, EL SISTEMA ofrece en los ajustes de visualización de la tarjeta la opción "Selector de métrica", desactivada por defecto.
- RF-2: SI la tarjeta no cumple las condiciones de RF-1, ENTONCES EL SISTEMA no muestra la opción ni ninguno de sus ajustes.
- RF-3: CUANDO el editor activa la opción y guarda el dashboard, EL SISTEMA guarda la configuración del selector con la tarjeta (igual que el título o los colores) y la aplica a todos los espectadores.
- RF-4: EL SISTEMA guarda el selector por tarjeta de dashboard, de modo que la misma pregunta puede tener selector en una tarjeta y no en otra.

### Configuración (H2)
- RF-5: MIENTRAS el selector está activo, EL SISTEMA muestra en los ajustes la lista de métricas de la pregunta, cada una con una casilla para incluirla como botón y con posibilidad de reordenarlas arrastrando.
- RF-6: EL SISTEMA incluye por defecto todas las métricas de la pregunta, en el orden en que la tarjeta las dibuja.
- RF-7: MIENTRAS el selector está activo, EL SISTEMA ofrece un ajuste "Métrica inicial" cuyas opciones son las métricas incluidas, con la primera de la lista como valor por defecto.
- RF-8: EL SISTEMA usa como texto de cada botón el nombre visible de la métrica en la tarjeta, de modo que renombrar la serie cambia a la vez el botón y la leyenda.
- RF-9: SI el editor deja menos de dos métricas incluidas, ENTONCES EL SISTEMA muestra un aviso en los ajustes y dibuja la tarjeta sin selector, con todas sus series.

### Uso por el espectador (H3)
- RF-10: MIENTRAS el selector está activo y hay al menos dos métricas incluidas, EL SISTEMA muestra sobre el gráfico, bajo el título de la tarjeta, una fila de botones con las métricas incluidas en el orden configurado.
- RF-11: CUANDO se abre el dashboard, EL SISTEMA marca como seleccionado el botón de la métrica inicial (o el primero de la fila si la inicial no está incluida o no existe) y dibuja solo su serie.
- RF-12: CUANDO el espectador pulsa el botón de otra métrica, EL SISTEMA dibuja solo la serie de esa métrica, marca ese botón como seleccionado y desmarca el anterior, sin afectar a otras tarjetas.
- RF-13: EL SISTEMA cambia de métrica sin enviar ninguna consulta ni petición nueva al servidor, porque los datos ya están en la tarjeta.
- RF-14: CUANDO cambia la métrica seleccionada, EL SISTEMA conserva el eje horizontal, los filtros aplicados, el color y el formato numérico que esa serie tenía en la vista con todas las series, y ajusta la escala del eje vertical a la serie mostrada; la leyenda sigue la regla actual del producto, que no la muestra con una sola serie, y el botón marcado la sustituye.
- RF-15: CUANDO el espectador pulsa el botón ya seleccionado, EL SISTEMA no cambia nada.
- RF-16: EL SISTEMA muestra el selector con la misma configuración y comportamiento en el dashboard interno, en pantalla completa y en dashboards públicos o embebidos.
- RF-17: MIENTRAS el dashboard está en modo edición, EL SISTEMA muestra los botones en su posición y funcionando igual que fuera de la edición, para que el editor compruebe el resultado; pulsar un botón en edición no cambia la métrica inicial guardada (RF-7).

### Persistencia de la selección (H4)
- RF-18: CUANDO la tarjeta recarga sus datos (cambio de un filtro del dashboard, actualización automática o cambio de periodo), EL SISTEMA conserva la métrica seleccionada si sigue existiendo en el resultado nuevo.
- RF-19: SI la métrica seleccionada deja de existir en el resultado nuevo, ENTONCES EL SISTEMA selecciona la métrica inicial o, si tampoco existe, la primera métrica incluida que exista.
- RF-20: CUANDO el espectador cambia la métrica seleccionada, EL SISTEMA la refleja en la dirección de la página sin recargarla y sin crear una entrada nueva en el historial del navegador, de modo que copiar la dirección comparte la selección; si la seleccionada es la inicial, la dirección no la lleva.
- RF-26: CUANDO se abre o se recarga el dashboard con una dirección que indica una métrica para una tarjeta con selector, EL SISTEMA marca esa métrica como seleccionada si está incluida y existe en el resultado; si no, aplica RF-11.
- RF-27: SI la dirección indica una métrica para una tarjeta sin selector, ENTONCES EL SISTEMA la ignora sin error.

### Compatibilidad (H5)
- RF-21: SI la tarjeta no tiene el selector activo, ENTONCES EL SISTEMA la dibuja y la configura exactamente igual que antes de esta funcionalidad, leyenda incluida.
- RF-22: SI la pregunta de una tarjeta con selector cambia y ninguna métrica incluida existe en el resultado, ENTONCES EL SISTEMA dibuja la tarjeta sin selector, con todas sus series y sin error.
- RF-23: EL SISTEMA envía las suscripciones y alertas de una tarjeta con selector con todas sus series, como hasta ahora.
- RF-24: MIENTRAS el selector muestra una sola métrica, EL SISTEMA mantiene las acciones de la tarjeta (ampliar, descargar resultados, ir a la pregunta, filtrar al hacer clic en un punto) con su comportamiento actual.
- RF-25: SI la tarjeta muestra un estado de carga, vacío o de error, ENTONCES EL SISTEMA mantiene visibles los botones y la métrica seleccionada.

## Requisitos no funcionales aplicables
RNF-1, RNF-2, RNF-3, RNF-5, RNF-6, RNF-7, RNF-8, RNF-9, RNF-10, RNF-11 y RNF-12 del catálogo [`docs/product/requisitos-transversales.md`](../../docs/product/requisitos-transversales.md). Ninguno específico de esta spec.

## Casos límite
- Pregunta con una sola métrica → RF-2.
- Tarjeta con desglose (una serie por valor de otra columna) → RF-2.
- Tarjeta con series añadidas de otras preguntas → RF-2.
- El editor desmarca todas las métricas menos una → RF-9.
- La pregunta se edita y una métrica incluida desaparece → RF-19, RF-22.
- La métrica inicial se desmarcó después de elegirla → RF-11.
- Dos métricas con el mismo nombre visible → RF-8 (los botones repiten el texto y se distinguen por posición).
- Muchas métricas o nombres largos en una tarjeta estrecha o en móvil → RNF-7, RF-10.
- Resultado sin filas, error de consulta o carga lenta → RF-25.
- Dos tarjetas con selector en el mismo dashboard → RF-4, RF-12.
- Dashboard público, embebido o en pantalla completa → RF-16.
- Dirección con una métrica que ya no está incluida, o para una tarjeta sin selector → RF-26, RF-27.
- Dashboard embebido: la dirección es la del marco y el espectador no la ve; quien embebe puede incluir la selección en ella → RF-16, RF-26.
- Dashboard guardado con una versión anterior del producto → RF-21, RNF-3.

## Fuera de alcance
Registrado en el [roadmap](../../docs/product/roadmap.md):
- Cabecera con total del periodo y variación respecto al periodo anterior (R-1).
- Filtros de dashboard como botones (R-2).
- Métrica seleccionada recordada por usuario entre visitas, sin enlace (R-3).
- Selector en tarjetas con desglose (R-4) o con series añadidas (R-5).
- Suscripciones y alertas con la métrica inicial en lugar de todas las series (R-6).
- Selector en la vista de pregunta, fuera de dashboards (R-7).
- Cambio de unidad calculado en el servidor (R-8) y cambio entre preguntas guardadas distintas (R-9).

## Dependencias
- Ninguna spec previa.
- Comportamiento actual del producto base: [`docs/reference/metabase-fork.md`](../../docs/reference/metabase-fork.md).
- Referencia visual: [`docs/reference/ontraport-cards.md`](../../docs/reference/ontraport-cards.md).
- Recursos: máquina de desarrollo y repos según [`docs/sdd/recursos-externos.md`](../../docs/sdd/recursos-externos.md).

## Criterios de finalización
- Cada RF tiene al menos un test automatizado en verde que cita su id.
- Demo manual en local con la base de datos de ejemplo: una tarjeta de pedidos con suma del total, número de pedidos y media del total por mes, con selector activo, en el dashboard interno y en un enlace público.
- Una tarjeta sin selector se ve igual que antes del cambio (comparación de capturas).
- Suite del producto base de las áreas tocadas, lint, formato y tipos en verde (RNF-1, RNF-11).
- Textos en inglés con traducción al español (RNF-5).
- Marcadores a cero y spec en estado `implementada`.

## Dudas abiertas
- Ninguna. Las decisiones (selector desactivado por defecto, selección en la dirección de la página y no guardada con el dashboard, botones funcionales en modo edición, sin límite de botones) quedaron confirmadas por el propietario el 2026-09-25; cualquier cambio posterior pasa por la fase de cambio.

## Cambios
- 2026-09-24: redacción inicial (propuesta).
- 2026-09-25: revisión con el propietario, historias H1 y H2 confirmadas: solo gráficos cartesianos con dos o más métricas, sin desglose ni series añadidas (RF-1); selector desactivado por defecto (RF-1); todas las métricas incluidas y la primera como inicial (RF-6, RF-7); texto del botón igual al nombre visible (RF-8); sin límite de botones (regla 3 del modelo de dominio ajustada); con menos de dos métricas incluidas la tarjeta se dibuja con todas sus series y un aviso (RF-9). El contexto explicita que el selector solo existe con dos o más puntos de vista. Historias H3 a H5 confirmadas: botones bajo el título (RF-10); sin leyenda con una sola serie, regla actual del producto (RF-14); escala del eje vertical ajustada a la métrica mostrada (RF-14); suscripciones, alertas y descargas con todas las métricas (RF-23, RF-24). Cambios respecto a la redacción inicial: botones funcionales en modo edición (RF-17) y métrica seleccionada en la dirección de la página (RF-20 reescrito, RF-26 y RF-27 nuevos; R-3 del roadmap queda solo para recordarla por usuario).
- 2026-09-25: aprobada por el propietario tras la revisión guiada (27 RF, 0 marcadores).
- 2026-09-25: [plan](plan.md) y [tareas](tasks.md) aprobados por el propietario (P1); pasa a planificada. Antes de implementar faltan los prerrequisitos P2..P5 de las tareas.
