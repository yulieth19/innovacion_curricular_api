# Investigación técnica — V1 Innovación Curricular

## 1. Propósito

Este documento registra las decisiones técnicas que condicionan la implementación
de V1 y explica sus consecuencias. No redefine el alcance funcional de
`2_spec.md` ni sustituye la organización de trabajo de `3_plan.md`.

Las decisiones que la constitución hace obligatorias se presentan como
restricciones del proyecto, no como opciones abiertas. Los detalles de esquema,
contratos, ejecución y secuencia de tareas se reservan para los documentos que
les corresponden.

## 2. Restricciones técnicas de partida

V1 se desarrolla con API y Frontend en repositorios independientes y se limita
a los siete recursos autorizados por la especificación. El Spec Kit reside en
`innovacion_curricular_api`; el contrato JSON de la API es la frontera de
intercambio con `innovacion_curricular_front`.

La constitución determina el stack .NET, la arquitectura por interfaces, el uso
de SQL explícito y parametrizado, el borrado lógico, la gestión de secretos y
la prohibición de anticipar versiones futuras. Esas reglas no son decisiones
pendientes de esta investigación. Sus consecuencias se detallan en las
secciones siguientes.

## 3. API: ASP.NET Core y .NET 10

La API será un proyecto Web API escrito en C# con ASP.NET Core sobre .NET 10.
Esta es una restricción constitucional y, por tanto, no se evalúa aquí otro
framework o versión como alternativa. Mantener el stack establecido concentra
la implementación en el alcance aprobado y conserva la compatibilidad con las
decisiones comunes del proyecto.

La API utilizará Controllers específicos para los recursos. No se sustituirán
por Minimal API: la arquitectura constitucional establece que cada recurso
corresponde a un Controller y que el flujo HTTP debe pasar por esa capa.

El recorrido permitido se conserva de extremo a extremo:

```text
HTTP
→ Controller
→ IServicio
→ Servicio
→ IRepositorio
→ RepositorioSqlServer
→ SQL Server
```

El Controller valida la petición de acuerdo con los contratos aprobados y
traduce resultados o errores al límite HTTP. No ejecuta consultas ni contiene
SQL.

## 4. Acceso a datos y SQL Server

SQL Server es el motor establecido por el proyecto. La base de datos se
considera un artefacto provisto por el proyecto: V1 no diseña una base nueva ni
crea tablas arbitrarias. El código de V1 utiliza únicamente las siete tablas
autorizadas por la especificación y el modelo de datos aprobado.

El acceso se realizará mediante:

- `Microsoft.Data.SqlClient` para conectarse con SQL Server;
- Dapper para ejecutar sentencias y mapear sus resultados;
- SQL escrito explícitamente en los repositorios específicos;
- parámetros para todos los valores variables;
- `async`/`await` en las operaciones de acceso a datos.

La combinación conserva la consulta visible y revisable: Dapper ejecuta el SQL
que escribe el proyecto, no lo genera a partir de un modelo. Parametrizar todo
valor variable evita incorporar valores recibidos del usuario a la sentencia
mediante concatenación y permite que los parámetros se envíen como datos, no
como parte del texto SQL.

Entity Framework y cualquier otro ORM están prohibidos por la constitución.
Tampoco se utilizará SQL generado automáticamente. El SQL concreto depende de
las columnas, tipos, claves y restricciones del esquema oficial; se podrá
redactar después de documentarlos en `5_data_model.md` y de aprobar los
contratos de `6_contracts.md`.

No se fijan en este documento nombres de base de datos, servidor, usuario,
contraseña, puertos ni cadenas de conexión.

## 5. Arquitectura por capas e inyección de dependencias

La separación de responsabilidades es obligatoria. `IServicio` permite que
Controller dependa del contrato de negocio, y `IRepositorio` permite que
Servicio dependa de un contrato de persistencia sin conocer el motor. El
RepositorioSqlServer implementa ese contrato y concentra SQL y acceso al
motor.

El contenedor de inyección de dependencias integrado en ASP.NET Core conectará
las implementaciones concretas con sus interfaces. `Program.cs` será el punto
de composición. Las capas consumidoras recibirán las abstracciones; no se
creará un Service Locator ni se instanciarán repositorios manualmente dentro de
Controllers o Servicios.

Esta composición mantiene las dependencias dirigidas hacia los contratos y
permite probar Servicio sustituyendo el repositorio SQL por una implementación
falsa de la misma interfaz. También evita que detalles HTTP o de SQL Server se
propaguen a capas que no son responsables de ellos.

## 6. Frontend Blazor Server

El Frontend se implementará en C# con Blazor Server sobre .NET 10, de acuerdo
con la constitución. Será una aplicación independiente en el repositorio
`innovacion_curricular_front`.

El Frontend tendrá sus propios modelos o DTO para interpretar el JSON. No
referenciará el proyecto API ni compartirá clases C# con él. Esta separación
mantiene los repositorios desacoplados: los tipos internos de una aplicación
pueden cambiar sin convertirse en una dependencia directa de la otra, mientras
ambas se coordinan mediante el contrato HTTP publicado y documentado.

El Frontend no instalará Dapper ni `Microsoft.Data.SqlClient`, no recibirá la
cadena de conexión y no se conectará a SQL Server. La persistencia pertenece a
la API.

## 7. Comunicación HTTP y servicios del Frontend

El Frontend consumirá la API exclusivamente mediante `HttpClient`/HTTP. Se
mantendrá un servicio HTTP específico para cada uno de estos recursos:

- `area_conocimiento`
- `universidad`
- `aspecto_normativo`
- `practica_estrategia`
- `enfoque`
- `car_innovacion`
- `aliado`

Los servicios por recurso hacen explícito qué contrato consume cada parte de la
interfaz y evitan seleccionar dinámicamente una tabla mediante un parámetro.
No se implementará un servicio CRUD genérico que reciba el nombre de tabla.

La frontera entre API y Frontend será exclusivamente el JSON intercambiado por
HTTP. Las rutas, estructuras exactas del JSON y respuestas aún no se establecen
en este documento; deben aprobarse y registrarse en `6_contracts.md` antes de
implementar consumidores y proveedores.

## 8. Swagger

La API utilizará `Swashbuckle.AspNetCore` para exponer Swagger durante el
desarrollo. Su propósito es facilitar la inspección manual y la comprobación
de los contratos de API mientras se implementan.

Swagger es una herramienta de desarrollo y verificación. No reemplaza el
Frontend, las pruebas automatizadas que se definan, los criterios de aceptación
o las verificaciones de regresión de V1. La funcionalidad de la versión debe
poder utilizarse desde el Frontend, conforme a `2_spec.md`.

## 9. Estrategia de pruebas de capas

Los Servicios se probarán usando repositorios falsos que implementen las
mismas interfaces `IRepositorio` que utilizan los Servicios productivos. De
este modo, las reglas funcionales y la dependencia de la abstracción pueden
comprobarse sin abrir conexiones ni ejecutar SQL Server.

Esta estrategia sirve para verificar la separación de capas y permite ejecutar
las pruebas correspondientes con SQL Server apagado. Los repositorios falsos
son sustitutos de prueba, no una segunda implementación de persistencia para
la aplicación ejecutada.

No se selecciona ni agrega aquí un framework de pruebas adicional. La elección
y configuración de herramientas de prueba se concretarán en el documento de
investigación o tareas que corresponda, solo si resulta necesario y no está ya
establecida. Este documento no contiene código de prueba.

## 10. Configuración y secretos

La configuración sensible, incluidas credenciales y cadenas de conexión, se
suministrará mediante variables de entorno. `.env` permanecerá fuera de Git y
`.env.example` podrá versionarse únicamente como plantilla, sin credenciales
reales ni secretos utilizables.

Se distinguen dos clases de información:

- **Nombres y estructura de configuración:** pueden definirse posteriormente
  en los documentos de ejecución y configuración. No se inventan aquí nombres
  concretos de variables.
- **Valores secretos reales:** contraseñas, credenciales y cadenas de conexión
  que las contengan nunca se escriben en código ni se versionan en archivos.

No se copian credenciales ni excepciones didácticas del repositorio de
referencia. Tampoco se propone guardar secretos en `appsettings.json`.

## 11. Docker Compose

Docker Compose será el mecanismo de ejecución conjunta y reproducible de los
tres servicios:

1. SQL Server.
2. API.
3. Frontend.

La dependencia funcional es `Frontend → API → SQL Server`: el Frontend consume
la API y no depende directamente de SQL Server; la API es quien realiza el
acceso a datos. El objetivo es que el sistema pueda iniciarse de forma
reproducible con Docker Compose, según la constitución y las instrucciones de
`7_quickstart.md`.

No se fijan aquí puertos, nombres de contenedor, contraseñas ni nombres de
variables. Los datos concretos se definirán formalmente en los documentos
correspondientes antes de configurar la ejecución.

## 12. Borrado lógico

La regla constitucional de borrado lógico determina la operación de
persistencia: el borrado funcional de un recurso se implementará como una
actualización que establece `activo = 0`, no como eliminación física.

Las consultas normales de colecciones y de registros individuales considerarán
únicamente filas activas (`activo = 1`). Esto mantiene la información eliminada
lógicamente fuera de las operaciones normales sin destruir la fila.

No se redactan sentencias concretas aquí. Su forma depende del esquema oficial
y se documentará después de aprobar `5_data_model.md`.

## 13. PUT y PATCH

La diferencia de comportamiento está establecida por la constitución y
`2_spec.md`:

- **PUT:** reemplaza completamente los campos modificables del recurso y debe
  representar la operación completa.
- **PATCH:** actualiza parcialmente el recurso; solo modifica los campos
  suministrados, no cambia automáticamente campos ausentes y debe incluir al
  menos un campo modificable. Una solicitud sin cambios se rechaza.

Las formas exactas de los cuerpos, las reglas de validación contractual y los
códigos HTTP se definirán en `6_contracts.md`. Este documento no añade campos
ni códigos.

## 14. Manejo de errores

La comunicación de errores respeta los límites de las capas:

- **Repositorio:** comunica problemas de persistencia hacia las capas
  superiores; no genera respuestas HTTP.
- **Servicio:** comunica problemas funcionales mediante el mecanismo previsto
  por la arquitectura y no conoce HTTP.
- **Controller:** traduce resultados y errores al contrato HTTP aprobado.
- **Frontend:** interpreta la respuesta HTTP y presenta información
  comprensible para la persona usuaria.

Los códigos HTTP, los cuerpos de error y su tratamiento concreto se fijarán en
`6_contracts.md`. Las decisiones de experiencia de usuario que correspondan se
precisarán en los documentos de V1 sin atribuir responsabilidades HTTP a
Servicio o Repositorio.

## 15. Dependencia del esquema oficial

No se diseñarán modelos, DTO, SQL o validaciones basándose únicamente en el
nombre de una tabla o en suposiciones. Antes de implementar esos elementos se
revisará el script oficial de la base de datos y las fuentes de datos del
proyecto.

`5_data_model.md` registrará, para cada una de las siete tablas autorizadas:

- columnas y tipos;
- nulabilidad;
- claves primarias y restricciones;
- campo de estado activo;
- datos oficiales disponibles;
- correcciones documentadas del material, cuando correspondan.

Este documento no inventa esos detalles ni autoriza cambios al esquema. Las
restricciones de unicidad que implemente V1 serán únicamente las que existan
en el esquema oficial.

## 16. Alternativas descartadas

| Alternativa | Motivo del descarte |
|---|---|
| Entity Framework | Está prohibido por la constitución, que exige SQL explícito ejecutado con Dapper y `Microsoft.Data.SqlClient`. |
| Otro ORM o SQL generado automáticamente | Incumpliría la restricción de que las consultas sean visibles, explícitas y parametrizadas. |
| SQL directo en Controllers | Mezclaría HTTP con persistencia y saltaría Servicio, `IRepositorio` y `RepositorioSqlServer`. |
| Acceso directo a SQL Server desde el Frontend | Viola la arquitectura de dos aplicaciones y expondría persistencia y configuración de base de datos al cliente. |
| Compartir proyecto o modelos C# entre API y Frontend | Acoplaría dos repositorios independientes; el límite contractual acordado es el JSON por HTTP. |
| CRUD genérico seleccionado por nombre de tabla | Contradice la obligación de recursos específicos y oculta contratos y responsabilidades particulares. |
| Borrado físico | Contradice la regla de borrado lógico mediante `activo = 0` y la conservación de registros inactivos. |
| Credenciales en `appsettings.json` o en código | Expondría secretos en archivos versionados; la configuración sensible debe llegar mediante variables de entorno. |
| Implementar capacidades de V2, V3 o V4 durante V1 | Contradice SDD, la precedencia de la especificación vigente y el alcance delimitado de V1. |
| Minimal API en lugar de Controllers | No respeta la arquitectura constitucional de Controllers específicos por recurso. |

## 17. Decisiones aplazadas

Las decisiones de la tabla siguiente están deliberadamente asignadas a otros
documentos; no constituyen vacíos que deban resolverse aquí.

| Decisión aplazada | Documento donde se resolverá |
|---|---|
| Estructura exacta de las siete tablas: columnas, tipos, nulabilidad, claves, restricciones y datos oficiales | `5_data_model.md` |
| Rutas de API | `6_contracts.md` |
| JSON de entrada y salida | `6_contracts.md` |
| Códigos HTTP | `6_contracts.md` |
| Validaciones contractuales | `6_contracts.md` |
| Pasos exactos de ejecución | `7_quickstart.md` |
| Secuencia detallada de implementación | `8_tasks.md` |
| Verificación humana de la especificación | `9_checklist.md` |
| Nombres concretos de variables de entorno y parámetros de ejecución | Documentos de configuración y ejecución, incluidos `7_quickstart.md` cuando corresponda |

## 18. Aclaraciones pendientes

No se detectaron aclaraciones técnicas pendientes para esta etapa. Los detalles
aplazados en la sección anterior tienen documentos asignados y no requieren
resolución en este research.

## 19. Conclusión técnica

Las decisiones técnicas obligatorias para V1 quedan delimitadas por el stack
.NET 10, Controllers y capas con interfaces, SQL Server con Dapper y consultas
explícitas parametrizadas, y un Frontend Blazor Server independiente que se
comunica por HTTP. Las pruebas de Servicio podrán sustituir la persistencia con
repositorios falsos, y Docker Compose integrará los tres servicios sin
adelantar valores de configuración no aprobados.

El siguiente paso técnico depende de documentar el esquema oficial en
`5_data_model.md` y los contratos en `6_contracts.md`. Solo después de aprobar
esas decisiones se concretarán los modelos, las consultas y los clientes HTTP.
