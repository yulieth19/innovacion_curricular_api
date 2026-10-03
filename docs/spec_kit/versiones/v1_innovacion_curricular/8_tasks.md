# Tareas de implementación — V1 Innovación Curricular

## 1. Propósito

Este documento convierte la especificación V1 en una secuencia verificable de
trabajo para los dos repositorios independientes. Define fases, dependencias,
artefactos previstos y compuertas; no indica que ninguna tarea se haya
realizado.

**Estado inicial de todas las fases: pendiente.** Los archivos y áreas
mencionados son previsiones para su implementación posterior. Este documento no
crea código, proyectos, SQL, configuración ni artefactos de ejecución.

## 2. Reglas de ejecución

- Trabajar en una rama personal y nunca directamente en `main`.
- Cada integrante trabaja en su propia rama en ambos repositorios. Para esta
  secuencia se verificará la rama indicada `rama-yulieth` en cada uno.
- Los cambios llegan a `main` únicamente mediante Pull Request.
- Los commits son pequeños y descriptivos, escritos en español. Los ejemplos
  de checkpoints de este documento son propuestas, no comandos ni acciones
  ejecutadas.
- No avanzar a la fase siguiente si falla la verificación o no se satisface
  la compuerta de la fase actual.
- Implementar exclusivamente los siete recursos de V1 definidos en
  `2_spec.md` y `5_data_model.md`. No agregar tablas, campos, restricciones,
  datos, códigos HTTP o reglas que no estén aprobados.
- Mantener API y Frontend en los repositorios independientes. El Frontend
  consume la API exclusivamente por HTTP, no accede a SQL Server y no comparte
  clases C# con la API.
- Cada recurso conserva modelos, peticiones, interfaces, repositorio, servicio
  y Controller específicos cuando correspondan. No crear CRUD dinámico por
  nombre de tabla.
- No implementar capacidades de V2, V3 o V4.
- No versionar `.env`, contraseñas reales ni cadenas de conexión con
  credenciales reales.

## 3. Fase 0 — Compuerta documental

**Objetivo:** aprobar la fuente de verdad de V1 antes de escribir código.

**Tareas previstas:**

- Confirmar que existen y están revisados: `1_constitution.md`,
  `0_mapa_versiones.md`, `2_spec.md`, `3_plan.md`, `4_research.md`,
  `5_data_model.md`, `6_contracts.md`, `7_quickstart.md`, `8_tasks.md` y
  `9_checklist.md`.
- Revisar si queda algún marcador `[NECESITA ACLARACIÓN: ...]` en los
  documentos de la compuerta.
- Revisar manualmente `9_checklist.md` y resolver los puntos rojos conforme al
  proceso aprobado; no modificar la constitución para hacer pasar la lista.

**Archivos o áreas previstas:** los documentos de
`docs/spec_kit/` y `docs/spec_kit/versiones/v1_innovacion_curricular/`; en esta
fase no se crean archivos de aplicación.

**Verificación obligatoria:** documentos completos y coherentes, sin
aclaraciones pendientes que bloqueen implementación, y checklist revisado y
aprobado manualmente.

**Criterio de salida:** `9_checklist.md` aprobado manualmente y sin puntos
rojos. **`8_tasks.md` por sí solo no autoriza comenzar a programar.**

**Estado:** pendiente. La aclaración sobre los nombres de variables de entorno
ya fue resuelta para V1 en `7_quickstart.md`, conforme al Artículo 7; la
Constitución ya no contiene el marcador correspondiente. Fase 0 continúa
pendiente únicamente porque todavía debe realizarse y aprobarse la revisión
humana de `9_checklist.md`. Si durante esa revisión aparece otra aclaración
real, debe resolverse antes de comenzar código. Esta fase no se considera
completada hasta que la persona revisora registre su aprobación.

## 4. Fase 1 — Infraestructura base y repositorios

**Objetivo:** confirmar los dos repositorios, sus límites y la base de
configuración segura para el trabajo posterior.

**Tareas previstas:**

- Verificar `rama-yulieth` en `innovacion_curricular_api` y
  `innovacion_curricular_front`, sin trabajar directamente en `main`.
- Revisar o preparar `.gitignore` en cada repositorio según su contenido.
- Asegurar que `.env` esté ignorado en ambos repositorios.
- Preparar `.env.example` como plantilla sin secretos reales ni valores
  utilizables como credenciales.
- Definir `.gitattributes` si se necesita normalizar a LF los scripts `.sh`.
- Preparar las áreas iniciales necesarias para la futura ejecución con Docker
  Compose, sin crear todavía los archivos de ejecución como parte de este
  documento.
- Mantener ambos repositorios independientes, sin submódulos ni copia del
  Frontend dentro de API.
- Ubicar posteriormente `docker-compose.yml` en la raíz de API y configurar el
  contexto de construcción del Frontend como `../innovacion_curricular_front`.

**Archivos o áreas previstas:** `.gitignore`, `.gitattributes` cuando se
requiera, `.env.example` en los repositorios correspondientes; estructura de
trabajo del repositorio API y contexto hermano del Frontend. No se crea aquí
ninguno de ellos.

**Verificación obligatoria:** Git no muestra secretos; los repositorios siguen
separados; la rama personal está verificada en ambos; la estructura prevista
coincide con `7_quickstart.md`.

**Criterio de salida:** ambos repositorios quedan preparados sin secretos y con
su independencia y contexto hermano comprobados.

**Estado:** pendiente.

## 5. Fase 2 — Base de datos

**Objetivo:** preparar y verificar la base de datos proporcionada por el
proyecto antes de implementar persistencia.

**Tareas previstas:**

- Incorporar el script SQL oficial/adaptado que se apruebe para la base
  `innovacion_curricular`.
- Incluir el esquema completo proporcionado por el proyecto; la API V1 seguirá
  limitada a las siete tablas autorizadas.
- Conservar las correcciones registradas en `5_data_model.md`:
  `area_conocimiento.id VARCHAR(6)`,
  `area_conocimiento.disciplina VARCHAR(150)`, el campo `activo BIT NOT NULL
  DEFAULT 1` en las siete tablas y la corrección de dato
  `Cienias Naturales` a `Ciencias Naturales`.
- Cargar los 218 registros oficiales de `area_conocimiento` y los 6 registros
  oficiales de `universidad`.
- No inventar datos iniciales para `aspecto_normativo`,
  `practica_estrategia`, `enfoque`, `car_innovacion` ni `aliado`.
- Preparar una inicialización idempotente de SQL Server si la estrategia de
  ejecución aprobada lo requiere.
- No implementar CRUD en esta fase.

**Archivos o áreas previstas:** script oficial/adaptado de base de datos y
artefacto de inicialización, si se adopta. La ubicación y los nombres concretos
se detallarán en `7_quickstart.md`/implementación.

**Verificación obligatoria:** la base `innovacion_curricular` existe; el
esquema completo previsto existe; se verifican 218 filas oficiales en
`area_conocimiento`, 6 en `universidad`, `activo` en las siete tablas V1 y los
tipos y longitudes coinciden con `5_data_model.md`.

**Criterio de salida:** esquema y datos oficiales verificados contra
`5_data_model.md`, sin datos inventados y sin CRUD implementado en esta fase.

**Estado:** pendiente.

## 6. Fase 3 — Esqueleto de API

**Objetivo:** disponer de una base Web API .NET 10 que pueda compilarse,
arrancar, exponer diagnóstico y Swagger y leer configuración sin secretos
versionados.

**Tareas previstas:**

- Crear el proyecto ASP.NET Core Web API en C# sobre .NET 10. El nombre podrá
  ser `ApiInnovacion` o uno coherente con las convenciones aprobadas.
- Preparar la estructura `Controllers/`, `Modelos/`, `Peticiones/`,
  `Servicios/`, `Repositorios/`, `Excepciones/` y `pruebas/`, además de
  `Program.cs`, `appsettings.json` y el `.csproj`.
- Incorporar como dependencias previstas `Microsoft.Data.SqlClient`, Dapper y
  `Swashbuckle.AspNetCore`.
- Configurar Controllers, Swagger y el contenedor de inyección de dependencias.
- Preparar la configuración mediante variables de entorno; `appsettings.json`
  no contendrá una contraseña real ni una cadena con credenciales reales.
- Implementar el diagnóstico `GET /` con `version: "v1"` y
  `contratos: "/swagger"`, sin conexión a SQL Server.
- Configurar el puerto de desarrollo API `8072` conforme a
  `7_quickstart.md`.
- Preparar la respuesta de validación estructural `422` conforme a
  `6_contracts.md`; las operaciones de recursos se incorporan en fases
  posteriores.

**Archivos o áreas previstas:** proyecto `.csproj`, `Program.cs`,
`appsettings.json`, `Controllers/`, configuración de Swagger y configuración de
la API. No se crean en esta fase durante la redacción de tareas.

**Verificación obligatoria:** el proyecto compila; `GET /` responde `200` con
`version` igual a `v1` y `contratos` igual a `/swagger`; Swagger abre; el
endpoint de diagnóstico responde sin SQL Server.

**Criterio de salida:** esqueleto compilable y diagnóstico/Swagger verificados
sin depender de la base.

**Estado:** pendiente.

## 7. Fase 4 — Modelos, peticiones e interfaces

**Objetivo:** declarar las formas de datos y fronteras por recurso a partir de
`5_data_model.md` y `6_contracts.md`.

**Tareas previstas para cada uno de los siete recursos:**

- Crear el modelo correspondiente con tipos y nulabilidad del modelo aprobado.
- Crear peticiones específicas de Crear, Reemplazo y Actualizar.
- Definir una interfaz `IRepositorio` específica del recurso.
- Definir una interfaz `IServicio` específica del recurso.
- Mantener las clases de API separadas de los DTO/modelos que posteriormente
  tendrá el Frontend.

**Reglas de los cuerpos:**

- Crear contiene todos los campos públicos, incluida la clave primaria; no
  expone `activo`.
- Reemplazo no incluye la clave como campo modificable y exige todos los demás
  campos públicos modificables.
- Actualizar no incluye clave modificable, permite individualmente los campos
  modificables que lleguen y conserva los ausentes.
- La regla PATCH de al menos un campo debe implementarse de forma que produzca
  el resultado establecido en `6_contracts.md`; no debe introducir una
  respuesta distinta.
- Respetar tipos y longitudes de `5_data_model.md` y `6_contracts.md`.

**Recursos:** `area_conocimiento`, `universidad`, `aspecto_normativo`,
`practica_estrategia`, `enfoque`, `car_innovacion` y `aliado`.

**Archivos o áreas previstas:** `Modelos/`, `Peticiones/`,
`Repositorios/` para interfaces y `Servicios/` para interfaces, con tipos
específicos por recurso.

**Verificación obligatoria:** build correcto; cuerpos y tipos cotejados con el
contrato; PK ausente de los cuerpos PUT/PATCH; `activo` no manipulable desde
el contrato; ningún recurso de V2 o posterior aparece.

**Criterio de salida:** los siete conjuntos de modelos, peticiones e interfaces
coinciden con el esquema y contratos aprobados.

**Estado:** pendiente.

## 8. Fase 5 — Repositorios SQL Server

**Objetivo:** implementar persistencia SQL Server específica por recurso,
respetando la interfaz y el modelo aprobado.

**Tareas previstas para cada recurso:**

- Implementar el repositorio SQL Server específico con operaciones equivalentes
  a `ObtenerTodos(limite)`, `ObtenerPorClave`, `Crear`, `Reemplazar`,
  `Actualizar` y `EliminarLogico`.
- Usar Dapper y `Microsoft.Data.SqlClient` con operaciones `async`/`await`.
- Escribir SQL explícitamente y parametrizar todos los valores.
- Filtrar `activo = 1` en listados y consultas individuales normales.
- Implementar DELETE funcional como actualización de `activo` a `0`; no usar
  DELETE físico.
- No modificar la clave primaria ni permitir que el cliente controle `activo`.
- No agregar restricciones de unicidad que no estén en el esquema oficial.
- Para PATCH, seleccionar de una lista expresa de columnas permitidas según el
  recurso. Si se arma dinámicamente la lista de asignaciones, los nombres de
  columna solo procederán de esa lista fija; todos los valores continuarán
  siendo parámetros. Nunca concatenar valores recibidos en SQL.

**Archivos o áreas previstas:** implementaciones de repositorio específicas
por recurso en `Repositorios/`.

**Verificación obligatoria:** build; revisión del SQL de los siete recursos;
confirmación de parámetros en los valores; filtros de activos; y ausencia de
DELETE físico funcional.

**Criterio de salida:** los siete repositorios respetan contratos, esquema,
parámetros, async y borrado lógico.

**Estado:** pendiente.

## 9. Fase 6 — Servicios y pruebas de capas

**Objetivo:** implementar reglas funcionales detrás de interfaces y demostrar
que pueden probarse sin SQL Server.

**Tareas previstas:**

- Crear un Servicio específico para cada recurso, implementando su `IServicio`
  y dependiendo únicamente de su `IRepositorio`.
- Mantener Servicio libre de HTTP y SQL; Repositorio libre de HTTP; y Controller
  libre de SQL.
- Implementar `NoEncontradoExcepcion` conforme al diseño de errores aprobado y
  sin trasladar códigos HTTP a Servicio.
- Crear repositorios falsos que implementen las mismas interfaces usadas por
  los Servicios productivos.
- Preparar pruebas de Servicio que cubran como mínimo recurso existente,
  inexistente/inactivo, reemplazo, actualización parcial, PATCH sin cambios
  según la capa donde se haya ubicado esa regla y eliminación lógica.
- No elegir ni agregar un framework de pruebas que no haya sido aprobado como
  necesario.

**Archivos o áreas previstas:** clases de Servicio y excepciones en
`Servicios/` y `Excepciones/`; repositorios falsos y pruebas en `pruebas/`.

**Verificación obligatoria:** pruebas pasan con SQL Server apagado; build
correcto; Servicios conocen interfaces y ninguna dependencia concreta aparece
fuera del ensamblador previsto en `Program.cs`.

**Criterio de salida:** reglas de Servicio verificadas sin infraestructura SQL
y límites de capa comprobados.

**Estado:** pendiente.

## 10. Fase 7 — Controllers y contratos HTTP

**Objetivo:** exponer los siete recursos mediante Controllers específicos y
cumplir exactamente `6_contracts.md`.

**Tareas previstas:**

- Crear un Controller por cada recurso con GET de colección, GET por clave,
  POST, PUT, PATCH y DELETE.
- Usar las siete rutas de `6_contracts.md`; no crear `/api/{tabla}` ni
  selección dinámica de recursos.
- Validar cuerpos de acuerdo con las peticiones aprobadas y traducir resultados
  o excepciones al contrato HTTP.
- Registrar en `Program.cs` las implementaciones concretas correspondientes:
  `IRepositorio` a RepositorioSqlServer e `IServicio` a Servicio para cada
  recurso.
- Mantener Swagger disponible para verificación y cumplir nombres JSON
  `camelCase`.
- Implementar solo los códigos definidos: `200`, `204`, `400`, `404`, `422` y
  `500`.

**Comprobaciones contractuales obligatorias:** `limite` predeterminado `1000`;
`limite <= 0` produce `400`; listado sin filas produce `204`; registro
inactivo produce `404`; POST incompleto produce `422`; PK duplicada produce
`500`; PUT incompleto produce `422`; PATCH parcial válido produce `200`; PATCH
`{}` produce `400`; segundo DELETE produce `404`; `activo` no aparece en JSON
público.

**Archivos o áreas previstas:** `Controllers/`, composición en
`Program.cs`, mapeo de errores y configuración Swagger.

**Verificación obligatoria:** build, Swagger, pruebas HTTP iniciales de cada
ruta y comparación de respuestas con `6_contracts.md`.

**Criterio de salida:** los seis métodos de cada uno de los siete recursos
cumplen las rutas, cuerpos, mensajes y códigos documentados.

**Estado:** pendiente.

## 11. Fase 8 — Frontend Blazor Server

**Objetivo:** implementar en `innovacion_curricular_front` una aplicación
Blazor Server .NET 10 independiente que permita utilizar los siete recursos.

**Tareas previstas:**

- Crear el proyecto Frontend .NET 10 y sus modelos/DTO propios para interpretar
  el JSON; no referenciar el proyecto API ni compartir clases C#.
- Leer `UrlApi` desde configuración.
- Usar `HttpClient` para consumir únicamente la API.
- Crear un servicio HTTP específico para cada recurso.
- Crear una pantalla funcional para `area_conocimiento`, `universidad`,
  `aspecto_normativo`, `practica_estrategia`, `enfoque`, `car_innovacion` y
  `aliado`.
- Ofrecer acciones de listado, creación, reemplazo completo, actualización
  parcial y retirada/eliminación lógica conforme al contrato.
- Presentar confirmación antes de retirar.
- Presentar errores en español, conservar los valores del formulario al fallar
  una operación y evitar términos visibles como PUT, PATCH, 422 o SQL Server.
- Mantener la aplicación levantada si la API deja de responder; mostrar un
  aviso comprensible y no inventar ni presentar como recientes datos que no
  fueron obtenidos correctamente.
- No instalar Dapper ni `Microsoft.Data.SqlClient`, incluir cadenas de conexión
  ni conectar a SQL Server.

**Archivos o áreas previstas en el repositorio Frontend:** proyecto Blazor,
DTO/modelos propios, servicios HTTP por recurso, pantallas de los siete
recursos, componentes compartidos que no conviertan la API en un CRUD genérico,
configuración y estilos propios.

**Verificación obligatoria:** build del Frontend; siete pantallas accesibles;
consumo HTTP únicamente; estados con datos, vacío y error; operaciones y
mensajes; prueba de caída de API; ausencia de dependencia SQL.

**Criterio de salida:** siete flujos de pantalla utilizables y verificados de
acuerdo con `2_spec.md`, `6_contracts.md` y `7_quickstart.md`.

**Estado:** pendiente.

## 12. Fase 9 — Docker e integración

**Objetivo:** integrar la base, API y Frontend con la configuración local segura
y el contexto de repositorios hermanos.

**Tareas previstas:**

- Crear `Dockerfile` de API en el repositorio API y `Dockerfile` de Frontend en
  el repositorio Frontend.
- Crear `docker-compose.yml` en la raíz de `innovacion_curricular_api`, con
  contexto de Frontend `../innovacion_curricular_front`.
- Incorporar SQL Server 2022 y, si se adoptó, el inicializador auxiliar del
  script; agregar API y Frontend como servicios.
- Configurar SQL Server 2022, base `innovacion_curricular`, API en `8072`,
  Frontend en `8073` y publicación SQL `11471 → 1433`.
- Conectar `MSSQL_SA_PASSWORD`, `ConnectionStrings__SqlServer` y `UrlApi` sin
  versionar valores secretos.
- Usar `sqlserver:1433` desde API y el nombre del servicio API en `UrlApi` desde
  Frontend; definir el nombre final del servicio en este punto.
- Configurar dependencias y comprobaciones de salud para que SQL Server esté
  saludable y la inicialización necesaria haya terminado antes de depender de
  la API.
- Mantener `.env` local e ignorado y `.env.example` sin secretos reales.

**Archivos o áreas previstas:** Dockerfiles en ambos repositorios,
`docker-compose.yml` en la raíz API, inicializador SQL si corresponde,
`.env.example` y reglas de exclusión Git.

**Verificación obligatoria:** desde `innovacion_curricular_api`, ejecutar el
procedimiento objetivo `docker compose up -d --build`; revisar `docker compose
ps`; verificar estados saludables/activos, diagnóstico API, Swagger, Frontend y
lecturas del Frontend a través de la API; confirmar ausencia de credenciales
reales en Git.

**Criterio de salida:** ejecución integrada reproducible y los tres servicios
funcionales, más el inicializador auxiliar si se adoptó.

**Estado:** pendiente.

## 13. Fase 10 — Smoke test completo

**Objetivo:** comprobar por HTTP y persistencia los contratos de los siete
recursos y las pruebas de capas.

**Tareas previstas:**

- Usar `7_quickstart.md` y `6_contracts.md` como guías, sin reemplazar sus
  contratos.
- Ejecutar la matriz siguiente independientemente para cada recurso, usando
  claves libres y registros de prueba controlados.
- No hacer PUT o DELETE sobre las 218 filas oficiales de
  `area_conocimiento` ni las 6 de `universidad`.
- Probar el caso de lista vacía con el método aislado previsto en
  `7_quickstart.md`; no borrar catálogos oficiales para simular vacío.
- Comprobar que la fila de prueba sobrevive al borrado con `activo = 0`.
- Ejecutar las pruebas de capas con SQL Server apagado.

| Caso | Resultado esperado según contrato |
|---|---|
| GET colección con datos | `200` |
| GET colección sin filas activas | `204` |
| `limite` inválido | `400` |
| GET existente | `200` |
| GET inexistente o inactivo | `404` |
| POST válido | `200` |
| POST incompleto | `422` |
| POST con PK duplicada | `500` |
| PUT completo | `200` |
| PUT incompleto | `422` |
| PATCH parcial | `200` |
| PATCH vacío | `400` |
| DELETE de registro activo | `200` |
| Segundo DELETE | `404` |
| Verificación de persistencia posterior a DELETE | La fila permanece con `activo = 0` y no aparece en lecturas normales. |

**Recursos a cubrir:** las siete filas de la matriz de trazabilidad de la
sección 17; la lista de casos se aplica completa a cada recurso.

**Verificación obligatoria:** todos los casos alcanzables pasan con los códigos
y cuerpos previstos; las pruebas de capas pasan sin SQL Server; no se han
alterado datos oficiales.

**Criterio de salida:** matriz de smoke test completa para los siete recursos y
resultados registrados para la revisión final.

**Estado:** pendiente.

## 14. Fase 11 — Verificación final del Frontend

**Objetivo:** validar de forma manual los flujos de usuario y la independencia
del Frontend respecto de API y SQL Server.

**Tareas previstas para cada pantalla:**

- Revisar listado con datos y estado vacío.
- Crear registro controlado, reemplazarlo completamente y actualizarlo
  parcialmente.
- Retirarlo con confirmación y comprobar su desaparición de consultas normales.
- Verificar mensajes en español y conservación de los valores de formulario
  cuando una operación falla.
- Confirmar que la interfaz no presenta datos ficticios ni expone jerga HTTP,
  códigos o SQL Server.

**Prueba de independencia:** detener solo la API; mantener Frontend disponible;
abrir las páginas, confirmar que responden y muestran aviso de indisponibilidad
sin inventar datos; volver a iniciar API y comprobar recuperación. Los comandos
con nombres de servicio se completan después de que Compose defina esos
nombres.

**Áreas previstas:** siete pantallas y servicios HTTP propios en
`innovacion_curricular_front`; escenario integrado Compose.

**Verificación obligatoria:** los siete flujos manuales se aprueban y la prueba
de indisponibilidad/restablecimiento de API pasa sin acceso directo a SQL.

**Criterio de salida:** todas las verificaciones manuales quedan aprobadas para
los siete recursos.

**Estado:** pendiente.

## 15. Fase 12 — Cierre de V1

**Objetivo:** reunir evidencias, revisar criterios y preparar el cierre
constitucional, sin marcar V1 terminada antes de superar todas las compuertas.

**Tareas previstas, solo tras aprobar las fases anteriores:**

- Revisar `git status` de ambos repositorios y confirmar ausencia de secretos.
- Ejecutar build, pruebas, smoke test y verificaciones de Frontend finales.
- Revisar y aprobar manualmente `9_checklist.md`.
- Actualizar documentación únicamente si la implementación confirmó detalles
  que debían completarse y el cambio está autorizado.
- Preparar commits finales pequeños y descriptivos en la rama personal; hacer
  push de esa rama y abrir Pull Request hacia `main` cuando corresponda.
- Revisar e integrar el Pull Request de acuerdo con el rol del proyecto.
- Comprobar el estado de `main` y crear/publicar el tag `v1` sobre `main` solo
  después de cumplir todos los criterios de aceptación y compuertas.

**Áreas previstas:** ambos repositorios, documentación aprobada, checklist,
Pull Request y etiqueta de versión.

**Verificación obligatoria:** todos los criterios de `2_spec.md`, pruebas y
regresión V1 pasan; el checklist está aprobado; el Pull Request se integró a
`main`; el tag `v1` apunta a `main`.

**Criterio de salida:** solo entonces puede declararse V1 cerrada. No se ejecuta
ninguna acción de cierre durante la redacción de este plan.

**Estado:** pendiente.

## 16. Dependencias entre fases

No se permite saltar fases por conveniencia. La evidencia de una fase debe
existir antes de iniciar la siguiente.

| Fase | Depende de | Evidencia para avanzar |
|---|---|---|
| 0 — Compuerta documental | Ninguna | Documentos revisados, `9_checklist.md` aprobado manualmente y sin puntos rojos ni aclaraciones bloqueantes. |
| 1 — Infraestructura base | Fase 0 | Ramas personales verificadas en ambos repositorios, repositorios independientes, secretos ignorados y estructura preparada. |
| 2 — Base de datos | Fase 1 | Script oficial/adaptado aplicado; esquema, correcciones y datos oficiales verificados. |
| 3 — Esqueleto API | Fase 1 y Fase 2 | Proyecto compila; diagnóstico y Swagger responden; diagnóstico no depende de SQL. |
| 4 — Modelos, peticiones e interfaces | Fases 2 y 3, más aprobaciones de `5_data_model.md` y `6_contracts.md` | Siete conjuntos definidos y cotejados con modelo y contratos. |
| 5 — Repositorios | Fases 2 y 4 | Siete repositorios revisados; SQL explícito/parametrizado; filtros activos y borrado lógico verificados. |
| 6 — Servicios y pruebas | Fase 4 y contratos aprobados | Servicios dependen de interfaces; pruebas con repositorios falsos pasan con SQL apagado. |
| 7 — Controllers y contratos HTTP | Fases 3, 4, 5 y 6 | Siete Controllers y rutas contrastados con `6_contracts.md`; pruebas HTTP iniciales pasan. |
| 8 — Frontend | Fase 1 y API funcional de Fase 7 | Proyecto independiente, siete pantallas y servicios HTTP consumen el contrato aprobado. |
| 9 — Docker e integración | Fases 2, 7 y 8 | Compose levanta servicios y la inicialización; API, Swagger y Frontend están disponibles y comunicados. |
| 10 — Smoke test completo | Fase 9 | Matriz de casos completa para los siete recursos; pruebas de capa y persistencia verificadas. |
| 11 — Verificación final Frontend | Fases 9 y 10 | Siete pantallas y prueba de API caída/restablecida aprobadas manualmente. |
| 12 — Cierre de V1 | Fases 0 a 11 | Checklist aprobado, criterios satisfechos, regresión y Pull Request integrados; tag `v1` creado después en `main`. |

Fase 0 bloquea cualquier implementación. El modelo y los contratos preceden a
los Controllers; la API funcional precede a la integración completa del
Frontend; Docker/integración preceden al smoke test final; el smoke test y la
verificación manual preceden al cierre y al tag.

## 17. Trazabilidad por recurso

Todas las filas y componentes están **planificados, no implementados**. En cada
fase se aplican a estos siete recursos las verificaciones de modelo, peticiones,
persistencia, API, Frontend y smoke test.

| Recurso | Modelo | Peticiones | Repositorio | Servicio | Controller | Servicio Frontend | Pantalla | Smoke |
|---|---|---|---|---|---|---|---|---|
| `area_conocimiento` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `universidad` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `aspecto_normativo` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `practica_estrategia` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `enfoque` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `car_innovacion` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |
| `aliado` | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Por crear | Pendiente |

## 18. Checkpoints Git propuestos

Los siguientes mensajes son ejemplos para futuros commits pequeños, no comandos
a ejecutar durante la planificación:

- `Preparar infraestructura base de V1`
- `Agregar esquema y datos iniciales de V1`
- `Crear esqueleto de API`
- `Definir modelos y peticiones de V1`
- `Implementar repositorios de V1`
- `Implementar servicios y pruebas de capas`
- `Implementar contratos HTTP de V1`
- `Crear frontend de recursos V1`
- `Integrar entorno Docker de V1`
- `Completar pruebas de aceptación de V1`

Los commits reales podrán dividirse todavía más por recurso y por cambio para
mantener cada unidad pequeña y descriptiva. Se realizarán en ramas personales;
no se trabajará directamente en `main`.

## 19. Aclaraciones pendientes

No se detectan aclaraciones pendientes sobre nombres de variables de entorno:
la Constitución registra su resolución para V1 y `7_quickstart.md` documenta
los nombres concretos acordados.
