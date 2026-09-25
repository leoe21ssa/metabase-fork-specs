# Recursos externos

Registro de todo lo que vive fuera de este repo y que una tarea puede necesitar.
Sirve para etiquetar tareas `[H]`/`[M]` en `tasks.md`: si un recurso dice
"agente: no", la tarea que lo necesita no es `[A]`.

| Recurso | Para qué | Propietario | ¿Agente puede usarlo? | Notas |
|---|---|---|---|---|
| GitHub - `leoe21ssa/metabase` (fork, repo de código) | Código, pull requests, CI | Esteban | Con `gh` autenticado, sí | Crear `develop` y `main`, activar Actions y proteger ramas es `[H]`. |
| GitHub - `leoe21ssa/sdd-template` (este repo) | Specs, ADR, pull requests | Esteban | Con `gh` autenticado, sí | Mezclar pull requests es `[H]`. |
| GitHub - `metabase/metabase` (upstream) | Leer código y traer versiones nuevas (`git fetch upstream`) | Metabase, Inc. | Sí, solo lectura | Nunca se hace push. |
| Máquina de desarrollo (Windows 11 con WSL) | Build, tests unitarios y de extremo a extremo | Esteban | Sí, una vez preparada | Instalar WSL, mise y clonar los repos fuera de OneDrive es `[H]` ([ADR-0001](../decisions/ADR-0001-repositorios.md)). |
| Context7 (clave personal en `CONTEXT7_API_KEY`) | Documentación de la versión instalada de cada librería | Esteban | Sí, con la clave en el entorno | Nunca en el repo. |
| Instancia de Metabase en producción del propietario | Uso real del producto | Esteban | No | La demo y la validación se hacen en local con la base de datos de ejemplo. |
| Ontraport (cuenta del propietario) | Referencia visual | Esteban | No | Solo capturas descritas en [`docs/reference/ontraport-cards.md`](../reference/ontraport-cards.md). |
| Servidor de despliegue del fork | Producción | Esteban | No | Fuera del alcance de la [spec 001](../../specs/001-selector-de-metrica/spec.md); backlog T-4. |
| Datos de clientes (correos, contactos, ventas) | - | Esteban, responsable del tratamiento | No; nunca en repos ni logs | Ley de protección de datos aplicable al propietario. |
| Plataforma de traducciones de Metabase | No se usa | Metabase, Inc. | No | Las traducciones del fork van en el catálogo `locales/es.po` del fork. |
