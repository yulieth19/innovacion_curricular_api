# Checklist de requisitos — V1 Innovación Curricular

## Propósito y reglas de revisión humana

Este checklist es la compuerta documental final antes de comenzar a programar
V1. Revisa la especificación y planificación de los documentos fuente, no la
implementación.

- Las casillas las revisa y marca una persona responsable.
- Copilot u otra IA puede ayudar a localizar inconsistencias, pero no puede
autoaprobar ni marcar este checklist.
- `[x]` significará que una persona verificó que el criterio se cumple en los
documentos actuales; no significa que se planea implementarlo más adelante.
- Todas las casillas de este documento se entregan sin marcar. No se ha
realizado una aprobación humana.
- Si una casilla documental A-K queda en rojo o requiere una aclaración, no se
comienza código y se devuelve el documento correspondiente a revisión.
- La revisión humana se realiza antes de iniciar la Fase 1 de `8_tasks.md`.
- La sección L contiene verificaciones posteriores de implementación/cierre;
no se usa para impedir el comienzo del código.

## A. Claridad

Revisar los documentos especificados y marcar solo tras comprobar su contenido
actual:

- [ ] No hay expresiones ambiguas o subjetivas sin un criterio verificable o una asignación explícita a un documento posterior.
- [ ] No queda ningún marcador `[NECESITA ACLARACIÓN: ...]` pendiente en el Spec Kit de V1 ni en la Constitución.
- [ ] `2_spec.md` define qué debe cumplir V1 y no mezcla detalles innecesarios de implementación.
- [ ] Cada requisito y escenario funcional tiene un propósito identificable y se puede relacionar con el alcance de V1.
- [ ] PUT está definido como reemplazo completo y PATCH como actualización parcial, sin contradicción entre documentos.
- [ ] El borrado lógico mediante `activo = 0` y la exclusión de inactivos están definidos claramente.
- [ ] Una colección sin registros activos se trata como estado normal, no como error funcional.
- [ ] Los siete recursos aparecen nombrados consistentemente: `area_conocimiento`, `universidad`, `aspecto_normativo`, `practica_estrategia`, `enfoque`, `car_innovacion` y `aliado`.

## B. Medibilidad

Comprobar que `6_contracts.md` y `7_quickstart.md` contienen resultados
observables y verificaciones documentales concretas:

- [ ] `6_contracts.md` define códigos HTTP concretos para los casos contractuales de V1.
- [ ] `200` está definido para lecturas con datos y operaciones de escritura exitosas según contrato.
- [ ] `204` está definido para listados sin filas activas y especifica que no lleva cuerpo.
- [ ] `400` está definido para `limite <= 0` y PATCH sin campos modificables.
- [ ] `404` está definido para clave inexistente o inactiva y para operaciones de escritura sobre registros no disponibles.
- [ ] `422` está definido para cuerpo inválido, campos obligatorios ausentes, tipos incompatibles y longitudes excedidas según contrato.
- [ ] `500` está definido para rechazo de base de datos, incluida PK duplicada, y otros errores internos o de infraestructura no traducidos a otro caso.
- [ ] `limite` tiene valor por defecto `1000` y debe ser positivo.
- [ ] PUT incompleto produce `422`.
- [ ] PATCH parcial válido produce `200`.
- [ ] PATCH `{}` produce `400`.
- [ ] DELETE de un registro activo produce `200`.
- [ ] GET posterior al DELETE del registro queda definido como `404`.
- [ ] Un segundo DELETE queda definido como `404`.
- [ ] El borrado lógico puede verificarse en persistencia comprobando que la fila permanece con `activo = 0`.
- [ ] `7_quickstart.md` establece cómo comprobar los contratos para cada uno de los siete recursos.
- [ ] El diagnóstico `GET /` tiene resultado verificable con `version: "v1"` y `contratos: "/swagger"`, y se indica que no consulta SQL Server.

## C. Completitud funcional y de recursos

- [ ] V1 cubre exactamente los siete recursos permitidos y ningún otro CRUD.
- [ ] Para cada recurso están contemplados GET de colección, GET por clave, POST, PUT, PATCH y DELETE.
- [ ] Los siete recursos llegan en el plan a API y Frontend.
- [ ] `8_tasks.md` traza cada recurso desde modelo/peticiones hasta API, Frontend y smoke test.
- [ ] `activo` no forma parte del contrato JSON público normal.
- [ ] POST incluye la clave primaria porque el modelo no establece generación `IDENTITY`.
- [ ] PUT no modifica la clave primaria, que se identifica en la ruta.
- [ ] PATCH no modifica la clave primaria, que se identifica en la ruta.
- [ ] DELETE está definido como borrado lógico y no como eliminación física.
- [ ] Los listados normales excluyen registros inactivos.
- [ ] Las consultas individuales normales excluyen registros inactivos.
- [ ] El Frontend consume los recursos únicamente a través de la API.
- [ ] El Frontend no accede directamente a SQL Server ni recibe su cadena de conexión.
- [ ] API y Frontend no comparten proyectos, referencias ni clases C#.
- [ ] Se exige que el Frontend permanezca disponible durante una caída temporal de la API y muestre un aviso comprensible.
- [ ] Se exige que el Frontend no invente ni presente datos como obtenidos correctamente cuando la API falla.

## D. Modelo de datos

Comprobaciones documentales de `5_data_model.md`:

- [ ] `5_data_model.md` documenta exactamente las siete tablas V1 y no detalla modelos de recursos de V2, V3 o V4.
- [ ] La clave primaria de cada una de las siete tablas está identificada.
- [ ] Ninguna clave primaria V1 se documenta como `IDENTITY`.
- [ ] `area_conocimiento.id` está definido como `VARCHAR(6) NOT NULL`.
- [ ] `area_conocimiento.disciplina` está definida como `VARCHAR(150) NOT NULL`.
- [ ] `car_innovacion.descripcion` conserva `VARCHAR(MAX) NOT NULL` y no se le inventa un límite contractual numérico.
- [ ] `activo BIT NOT NULL DEFAULT 1` está documentado para las siete tablas V1.
- [ ] La corrección de dato `Cienias Naturales` → `Ciencias Naturales` está documentada como la corrección confirmada.
- [ ] El modelo documenta 218 registros oficiales de `area_conocimiento`.
- [ ] El modelo documenta 6 registros oficiales de `universidad`.
- [ ] No se inventan datos iniciales para `aspecto_normativo`, `practica_estrategia`, `enfoque` y `car_innovacion`.
- [ ] `aliado` puede iniciar vacío y no se inventan aliados.
- [ ] Los tipos SQL y el mapeo conceptual SQL → C# documentados son coherentes.

La ausencia actual de un script SQL ejecutable no es una falla documental: el
script y su verificación física se completarán durante las fases de
implementación. Esos controles se enumeran por separado en la sección L.

## E. Contratos HTTP

- [ ] `6_contracts.md` define una ruta específica para cada recurso y sus operaciones.
- [ ] No define ni requiere un endpoint dinámico `/api/{tabla}`.
- [ ] Los nombres de propiedades JSON usan `camelCase`.
- [ ] Están documentados los campos y cuerpos POST de los siete recursos.
- [ ] Están documentados los campos y cuerpos PUT de los siete recursos.
- [ ] Están documentados los campos y cuerpos PATCH de los siete recursos.
- [ ] Están documentadas las respuestas de lecturas de colección y por clave.
- [ ] Están documentados los sobres de error y los casos en que se usan.
- [ ] El sobre de listado contiene `tabla`, `limite`, `total` y `datos`.
- [ ] `activo` no se expone en respuestas públicas y no es modificable mediante POST, PUT o PATCH.
- [ ] PUT y PATCH tienen reglas distintas, inequívocas y compatibles con `2_spec.md`.
- [ ] PATCH vacío está documentado con su resultado contractual.
- [ ] La PK duplicada está documentada con su resultado contractual.
- [ ] Los registros inexistentes e inactivos tienen un tratamiento normal coherente.
- [ ] No se inventa una validación de formato de correo para V1.
- [ ] No se fijan autenticación ni JWT como parte de V1.

## F. Arquitectura

Revisar la coherencia entre la Constitución, `3_plan.md`, `4_research.md` y
`8_tasks.md`:

```text
HTTP
→ Controller
→ IServicio
→ Servicio
→ IRepositorio
→ RepositorioSqlServer
→ SQL Server
```

- [ ] El Controller recibe HTTP y no contiene SQL.
- [ ] Servicio no conoce HTTP.
- [ ] Servicio no escribe ni conoce SQL Server.
- [ ] Repositorio no conoce HTTP.
- [ ] `Program.cs` es el ensamblador donde se registran las implementaciones concretas con sus interfaces.
- [ ] El acceso a datos utiliza Dapper.
- [ ] El acceso a SQL Server utiliza `Microsoft.Data.SqlClient`.
- [ ] El SQL será explícito y todos los valores variables serán parámetros.
- [ ] El acceso a datos utiliza `async`/`await`.
- [ ] Se excluye Entity Framework.
- [ ] Se excluyen otros ORM y SQL generado automáticamente.
- [ ] Cada recurso tendrá servicio y repositorio específicos.
- [ ] Se excluye CRUD dinámico seleccionado por nombre de tabla.
- [ ] Se planifican pruebas de Servicio con repositorios falsos que implementen las interfaces de persistencia.
- [ ] Las pruebas de capas están planificadas para ejecutarse sin SQL Server.

## G. Frontend

Comprobar que la documentación exige lo siguiente al Frontend V1:

- [ ] Blazor Server sobre .NET 10.
- [ ] Repositorio independiente `innovacion_curricular_front`.
- [ ] Comunicación exclusivamente HTTP con la API.
- [ ] Dirección base de API `UrlApi` obtenida desde configuración.
- [ ] Un servicio HTTP específico por recurso.
- [ ] Siete pantallas, una para cada recurso V1.
- [ ] Listado de registros obtenidos de la API.
- [ ] Estado vacío comprensible cuando no hay registros.
- [ ] Creación de registros mediante la API.
- [ ] Reemplazo/edición completa mediante la API.
- [ ] Actualización parcial mediante la API.
- [ ] Confirmación antes de retirar/eliminar lógicamente.
- [ ] Mensajes de error en español.
- [ ] Conservación de los valores escritos cuando una operación falla, conforme a lo que documenten los contratos y tareas.
- [ ] Los textos de interfaz no exponen innecesariamente PUT, PATCH, 422 o SQL Server.
- [ ] El Frontend sigue disponible y muestra un aviso si la API cae.
- [ ] El Frontend no muestra datos ficticios como si vinieran de la API.
- [ ] El Frontend no utiliza Dapper, `Microsoft.Data.SqlClient` ni conexión directa a SQL Server.
- [ ] El Frontend no referencia el proyecto API ni comparte clases C#.

## H. Configuración, secretos y Docker

Comprobar la documentación y planificación, no la existencia de artefactos de
implementación:

- [ ] `.env` se documenta como archivo local, ignorado por Git y no versionable.
- [ ] `.env.example` se documenta como plantilla versionada sin secretos reales.
- [ ] `appsettings.json` no contendrá una contraseña real ni credenciales reales.
- [ ] Ningún documento del proyecto incluye una contraseña real.
- [ ] Se documenta `MSSQL_SA_PASSWORD` para la contraseña local de SQL Server.
- [ ] Se documenta `ConnectionStrings__SqlServer` para la cadena de conexión de la API.
- [ ] Se documenta `UrlApi` para que el Frontend localice la API.
- [ ] La base documentada es `innovacion_curricular`.
- [ ] Se documenta SQL Server 2022.
- [ ] Se documenta el puerto de desarrollo API `8072`.
- [ ] Se documenta el puerto de desarrollo Frontend `8073`.
- [ ] Se documenta SQL publicado en `11471 → 1433`.
- [ ] `docker-compose.yml` está planificado en la raíz del repositorio API.
- [ ] Se documenta `../innovacion_curricular_front` como contexto del Frontend.
- [ ] Se conserva la independencia de los dos repositorios.
- [ ] El Frontend no depende directamente de SQL Server.
- [ ] Se documenta como comando objetivo `docker compose up -d --build` desde la raíz del repositorio API.
- [ ] Se aclara que Docker Compose, Dockerfiles, scripts y plantillas se crearán durante implementación y que su ausencia actual no es una falla documental.

## I. Coherencia y trazabilidad

- [ ] Todo recurso de `2_spec.md` aparece en `5_data_model.md`.
- [ ] Todo recurso de `5_data_model.md` aparece en `6_contracts.md`.
- [ ] Todo contrato tiene tareas correspondientes en `8_tasks.md`.
- [ ] `7_quickstart.md` describe cómo verificar los contratos definidos.
- [ ] `8_tasks.md` incluye explícitamente los siete recursos.
- [ ] Cada recurso tiene trazabilidad a Modelo, Peticiones, Repositorio, Servicio, Controller, Servicio Frontend, Pantalla y Smoke.
- [ ] `3_plan.md` es coherente con la secuencia de tareas de `8_tasks.md`.
- [ ] `4_research.md` es coherente con el plan y la Constitución, incluidos los detalles que luego quedaron fijados en documentos posteriores.
- [ ] No quedan decisiones contradictorias o referencias desactualizadas entre documentos.
- [ ] Los nombres de configuración de `7_quickstart.md` se mantienen dentro de la regla general del Artículo 7 y no se convierten en reglas permanentes para futuras versiones.
- [ ] La aclaración previa de variables de entorno está cerrada en la Constitución sin fijar esos nombres como regla permanente.
- [ ] La condición de Fase 0 en `8_tasks.md` corresponde al estado actual de la Constitución y de los marcadores pendientes.

## J. Alcance y no anticipación

Comprobar que V1 no incluye trabajo reservado para versiones futuras ni
características que no estén aprobadas:

- [ ] Relaciones o CRUD de tablas con claves foráneas de V2.
- [ ] Autenticación.
- [ ] JWT.
- [ ] Sesiones.
- [ ] Roles.
- [ ] Administración de usuarios.
- [ ] Consultas multitabla de V4.
- [ ] Dashboard de V4.
- [ ] PWA de V4.
- [ ] Publicación de V4.
- [ ] Reactivación de registros no especificada.
- [ ] Búsqueda, ordenamiento o paginación adicional no especificados.
- [ ] Validación de formato de correo no especificada.
- [ ] Generación automática de identificadores.
- [ ] CRUD genérico dinámico por nombre de tabla.
- [ ] La mención de V2, V3 o V4 como fuera de alcance se entiende como exclusión y no como autorización para anticipar su implementación.

## K. Git y cierre

Comprobar documentalmente el proceso, sin afirmar que PR, integración o tag ya
existan:

- [ ] Nunca se trabaja directamente en `main`.
- [ ] Está prevista una rama personal para cada integrante en ambos repositorios.
- [ ] Los cambios llegan a `main` mediante Pull Request.
- [ ] Los commits serán pequeños y descriptivos en español.
- [ ] El tag `v1` solo se crea después de superar aceptación y compuertas.
- [ ] No se declara V1 cerrada solo porque compile la API o el Frontend.
- [ ] El cierre exige API, Frontend, pruebas, smoke test y verificación manual.
- [ ] No se crea el tag antes de integrar y verificar los cambios en `main`.
- [ ] Las tareas de cierre describen pasos futuros y no afirman que ya haya PR, aprobación, integración o tag.

## Comprobaciones diferidas a implementación

Las siguientes casillas requieren código, scripts, infraestructura o revisión de
Git real. No forman parte de la decisión documental A-K para autorizar el
inicio de implementación. Se completarán progresivamente durante las fases de
`8_tasks.md`; no se marcan como implementadas por estar descritas aquí. Son
necesarias para las compuertas y el cierre de V1.

- [ ] El script SQL real coincide con `5_data_model.md`.
- [ ] La base ejecutable contiene realmente 218 registros oficiales de `area_conocimiento`.
- [ ] La base ejecutable contiene realmente 6 registros oficiales de `universidad`.
- [ ] `activo` existe físicamente en las siete tablas V1.
- [ ] Las longitudes, nulabilidad, PK y demás estructura física coinciden con el modelo aprobado.
- [ ] API compila.
- [ ] `GET /` responde según el contrato.
- [ ] Swagger responde según el contrato.
- [ ] Las pruebas de capas pasan con SQL Server apagado.
- [ ] Frontend compila.
- [ ] Las siete pantallas funcionan.
- [ ] Docker Compose levanta SQL Server, inicializador si aplica, API y Frontend.
- [ ] El smoke test completo de los siete recursos pasa.
- [ ] El Frontend permanece disponible cuando la API está caída, sin mostrar datos ficticios.
- [ ] No existen secretos versionados al cierre de V1.
- [ ] El Pull Request está aprobado e integrado conforme al proceso del proyecto.
- [ ] El tag `v1` está creado sobre `main` tras superar todas las compuertas.

Estas casillas diferidas no autorizan a tratar como implementado ningún
artefacto que todavía no exista y no bloquean el comienzo del código una vez
aprobada la compuerta documental A-K.

## Resultado de la compuerta documental

| Campo | Resultado |
|---|---|
| Revisada por | Pendiente |
| Fecha | Pendiente |
| Casillas documentales A-K en rojo | Pendiente de revisión humana |
| Aclaraciones pendientes | Pendiente de revisión humana |
| Veredicto documental | ⬜ Puede comenzar implementación / ⬜ Debe volver a especificación |

La sección L contiene comprobaciones de implementación y cierre; no forma parte
del conteo de casillas documentales A-K para autorizar el inicio de
implementación.

## Aclaraciones detectadas durante la construcción

No se detectan aclaraciones pendientes para iniciar la revisión humana de la
compuerta documental. La inconsistencia identificada durante la construcción
del checklist fue resuelta mediante la actualización documental de
`3_plan.md`, `4_research.md` y `8_tasks.md`.
