# Requisitos no funcionales transversales (RNF)

Catálogo único. Las specs citan estos ids; no los redactan de nuevo. Cada RNF es
verificable; el umbral se fija aquí. Está adaptado a un fork de Metabase: la sesión, las
contraseñas, los permisos, las copias de seguridad y la operación los resuelve el producto base y
no se vuelven a especificar.

## Compatibilidad con el producto base
- **RNF-1** Sin regresión: la suite de Metabase (unitaria y de extremo a extremo) de las áreas tocadas por una spec queda en verde; un dashboard guardado antes del cambio se ve igual después.
- **RNF-2** Cambios aditivos: cada spec lista en su plan los archivos de Metabase que modifica; ninguno está bajo `enterprise/`; no cambian el backend, la API ni el esquema de la base de datos de aplicación salvo ADR aceptado.
- **RNF-3** Configuración compatible: toda configuración nueva se guarda en los ajustes de visualización existentes; una versión de Metabase sin el fork ignora esas claves sin error.
- **RNF-4** Integración de upstream: tras integrar una versión nueva de Metabase, la suite del fork queda en verde antes de desplegar.

## Interfaz
- **RNF-5** Internacionalización: todo texto visible pasa por la capa de traducción de Metabase, se escribe en inglés y tiene su traducción al español en el catálogo del fork.
- **RNF-6** Accesibilidad: los controles nuevos se manejan con teclado, exponen su rol y estado a los lectores de pantalla y cumplen contraste AA con el tema de Metabase.
- **RNF-7** Adaptación al espacio: un control nuevo cabe en una tarjeta de la anchura mínima de la rejilla de Metabase y en la vista móvil; si no cabe en una línea, pasa a varias o se desplaza dentro de la tarjeta, nunca desborda.
- **RNF-8** Navegadores: los mismos que soporta Metabase (últimas dos versiones de Chrome, Edge, Safari y Firefox).

## Rendimiento
- **RNF-9** Sin consultas adicionales: un cambio de estado en la interfaz (por ejemplo, elegir una métrica) no envía ninguna petición al servidor; se verifica interceptando las peticiones en el test de extremo a extremo.
- **RNF-10** Respuesta inmediata: el cambio visible tras una pulsación se completa en menos de 300 ms en una tarjeta con seis métricas y 1.000 puntos, medido en local.

## Calidad y datos
- **RNF-11** Calidad de código: lint, formato y comprobación de tipos del repo de Metabase en verde para todo cambio.
- **RNF-12** Datos: tests, capturas y ejemplos usan la base de datos de ejemplo de Metabase o valores ficticios; ningún dato de clientes en repos, tests ni logs.
- **RNF-13** Licencia: el código del fork se publica o se ofrece a quien use el servicio, conforme a AGPL; ningún uso de código de `enterprise/`.
