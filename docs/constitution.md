# Constitución - metabase-fork

Estado: aceptada · Fecha: 2026-09-24 · Aceptada por el propietario: 2026-09-25

Principios innegociables. Toda spec, plan, tarea y línea de código debe cumplirlos.
Cada principio termina con cómo se verifica, para que `sdd-validate` pueda comprobarlo.
Cambiar un principio requiere un ADR aceptado. Están adaptados a un producto que es un
**fork de un proyecto vivo** (Metabase): la mayor amenaza no es el diseño desde cero sino
romper lo que ya funciona y perder la capacidad de absorber versiones nuevas.

1. **La spec manda.** Ningún comportamiento se implementa si no está en la spec activa y
   aprobada. Si falta una decisión o un marcador `[NECESITA ACLARACIÓN]` bloquea un RF, el
   trabajo se detiene y se pregunta.
   *Se verifica:* cada tarea de `tasks.md` cita al menos un RF; cada RF tiene un test con su id.

2. **Cambios aditivos y acotados.** Lo nuevo vive en archivos nuevos o en ajustes de
   visualización nuevos. Los archivos de Metabase que se modifican son solo puntos de
   inserción, están listados en el `plan.md` de la spec y cada uno cambia lo mínimo. Nada
   se modifica dentro de `enterprise/`, ni en el backend ni en el esquema de la base de
   datos de aplicación salvo que una spec lo exija con un ADR aceptado.
   *Se verifica:* `git diff upstream/master --stat` en el repo de código solo lista archivos previstos en el plan; `enterprise/` no aparece en el diff.

3. **Sin regresión del producto base.** Un dashboard guardado antes del cambio se ve y se
   comporta igual después. La suite de Metabase de las áreas tocadas sigue en verde.
   *Se verifica:* tests unitarios y de extremo a extremo de dashboards y visualizaciones en verde en CI; un test comprueba que una tarjeta sin la funcionalidad activa renderiza igual que antes.

4. **Lógica de visualización pura.** Las reglas de qué serie se muestra, con qué nombre y
   cuál es la inicial viven en funciones sin interfaz, testeables con datos en memoria.
   Los componentes de interfaz solo las invocan.
   *Se verifica:* el módulo de lógica no importa nada de interfaz ni del navegador; sus tests corren sin DOM.

5. **Convenciones de Metabase, no las nuestras.** Se usan el sistema de ajustes de
   visualización, la biblioteca de componentes interna, el mecanismo de traducción y las
   herramientas de lint, formato y tipos del repo. Ninguna dependencia nueva.
   *Se verifica:* lint, formato y comprobación de tipos del repo en verde; el archivo de dependencias no cambia.

6. **Tests como puerta.** Cada RF aparece en el nombre o la descripción de al menos un
   test automático. Una suite en rojo bloquea la integración. No se avanza a la siguiente
   tarea con tests en rojo.
   *Se verifica:* CI falla si algún test falla; `sdd-validate` no reporta RF sin test.

7. **Idiomas e internacionalización.** Specs, documentación y mensajes al usuario en
   español; identificadores, nombres de archivo de código y commits en inglés. Los textos
   de interfaz se escriben en inglés a través de la capa de traducción de Metabase y
   reciben su traducción al español en el catálogo del repo.
   *Se verifica:* grep de cadenas de interfaz literales en los componentes nuevos devuelve vacío; cada texto nuevo tiene entrada en el catálogo español.

8. **Al día con upstream.** El fork sigue la rama principal de Metabase. Cada versión
   mensual y cada parche de seguridad se integran con un merge (rama `chore/upstream-<versión>`)
   y la suite debe quedar en verde antes de desplegar; los parches intermedios sin
   corrección de seguridad pueden esperar a la siguiente versión mensual. Los commits del
   fork citan spec y RF para que cada integración sea legible.
   *Se verifica:* `git log upstream/master..develop` lista solo commits del fork con spec y RF en el mensaje; CI en verde en `develop` tras cada integración de upstream; ninguna versión mensual de Metabase lleva más de un mes sin integrar.

9. **Licencia respetada.** El fork modifica solo código AGPL. Es de uso interno de
   StreamSolve y no se ofrece a terceros por red, por lo que no aplica la obligación de
   ofrecer el código modificado (AGPL, sección 13); además el repo del fork es público, así
   que el código queda disponible en cualquier caso. Si el uso cambia, se abre un ADR. El
   código de `enterprise/` no se usa sin licencia comercial.
   *Se verifica:* no hay diff bajo `enterprise/`; el uso interno y la visibilidad del repo constan en el [ADR-0001](decisions/ADR-0001-repositorios.md).

10. **Datos y secretos.** Tests, capturas y ejemplos usan la base de datos de ejemplo de
    Metabase o datos ficticios; nunca datos de clientes. Ningún secreto en los repos.
    *Se verifica:* grep de nombres, correos o identificadores reales en docs, specs y código devuelve vacío; `.env` está ignorado.
