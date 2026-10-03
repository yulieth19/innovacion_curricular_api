# Plan técnico — V1 Innovación Curricular

## 1. Propósito

Este documento define cómo se organizará técnicamente la implementación de V1,
sin escribir código ni sustituir las decisiones funcionales de
`2_spec.md`. La especificación vigente es la fuente de verdad sobre lo que se
construirá; este plan describe su organización en los dos repositorios del
proyecto.

Las fases descritas son planificación, no tareas ejecutadas. La secuencia
detallada y sus compuertas se establecerán posteriormente en `8_tasks.md`.

## 2. Contexto y alcance de V1

V1 comprende API y Frontend para exactamente estos siete recursos:

- `area_conocimiento`
- `universidad`
- `aspecto_normativo`
- `practica_estrategia`
- `enfoque`
- `car_innovacion`
- `aliado`

El plan no incorpora CRUD para otros recursos de la base de datos, aunque sus
tablas existan. No planifica tablas con claves foráneas de V2, seguridad y
usuarios de V3, ni dashboard, consultas multitabla, PWA o publicación de V4.
La base de datos puede existir completa desde V1, pero el código de esta versión
solo podrá utilizar las tablas autorizadas para estos siete recursos.

El Spec Kit vive en `innovacion_curricular_api`. El trabajo de API y Frontend
se mantiene en los repositorios independientes `innovacion_curricular_api` y
`innovacion_curricular_front`.

## 3. Stack técnico

### API

- C# y ASP.NET Core sobre .NET 10.
- SQL Server como motor de base de datos.
- `Microsoft.Data.SqlClient` para la conexión y `Dapper` para ejecutar SQL.
- `Swashbuckle.AspNetCore` para la documentación Swagger de la API.
- `async`/`await` para las operaciones de acceso a datos.
- SQL escrito explícitamente, parametrizado y ubicado en los repositorios
	específicos.
- No se utiliza Entity Framework, ningún otro ORM ni generación automática de
	SQL.

### Frontend

- C# y Blazor Server sobre .NET 10.
- Comunicación con la API exclusivamente por HTTP.
- Sin Dapper, `Microsoft.Data.SqlClient`, conexión directa a SQL Server ni
	conocimiento de la cadena de conexión de la base de datos.
- Sin referencia al proyecto API ni reutilización de sus clases C#.

## 4. Arquitectura de la API

El flujo y las dependencias respetarán esta secuencia:

```text
HTTP
→ Controller
→ IServicio
→ Servicio
→ IRepositorio
→ RepositorioSqlServer
→ SQL Server
```

Responsabilidades previstas:

- **Controller:** recibe HTTP, valida las peticiones conforme a los contratos
	aprobados, invoca `IServicio` y traduce resultados y excepciones a respuestas
	HTTP. No contiene SQL.
- **IServicio:** define el contrato entre Controller y Servicio.
- **Servicio:** aplica las reglas funcionales de V1 y depende de
	`IRepositorio`. No conoce HTTP ni detalles de SQL Server.
- **IRepositorio:** define el contrato de persistencia utilizado por Servicio.
- **RepositorioSqlServer:** implementación específica de `IRepositorio` que
	utiliza Dapper y `Microsoft.Data.SqlClient`, con SQL explícito y
	parametrizado. No conoce HTTP.
- **Program.cs:** funciona como ensamblador y registra las implementaciones
	concretas mediante inyección de dependencias.

Las dependencias concretas se conectan en `Program.cs`; las capas consumidoras
dependen de interfaces. Cada recurso conserva sus propias interfaces y
componentes, sin una abstracción CRUD dinámica que reciba el nombre de una
tabla.

## 5. Organización prevista del repositorio API

La estructura se organizará por responsabilidades. Los nombres de carpetas se
mantendrán en español, de acuerdo con la constitución:

```text
innovacion_curricular_api/
├── Controllers/
├── Modelos/
├── Peticiones/
├── Servicios/
├── Repositorios/
├── Excepciones/
├── pruebas/
├── Program.cs
├── appsettings.json
└── proyecto .csproj
```

Para cada uno de los siete recursos se prevé su modelo de entidad, sus
peticiones específicas cuando correspondan, `IRepositorio` y su implementación
SQL Server, `IServicio` y su implementación, y un Controller específico. Las
peticiones se organizarán en `Peticiones/`; el resto de los elementos se
ubicarán según su responsabilidad. Los nombres concretos de tipos y archivos
se determinarán durante las tareas de implementación, respetando los nombres
en español y las convenciones obligatorias de las tecnologías.

Esta estructura es una organización prevista: no se crean carpetas, proyectos
ni archivos en este paso.

## 6. Estrategia de peticiones

Cada recurso tendrá peticiones diferenciadas por operación, sin enumerar
propiedades hasta que se apruebe `5_data_model.md`:

- **Crear:** representa una solicitud completa para crear el recurso conforme
	a los requisitos confirmados.
- **Reemplazo para `PUT`:** representa el recurso completo y aplica las reglas
	de reemplazo completo.
- **Actualizar para `PATCH`:** permite enviar los campos modificables que
	estén presentes y conserva los datos no incluidos.

Una petición `PATCH` debe contener al menos un campo modificable; si no contiene
ningún cambio, se rechaza. El código HTTP concreto y el formato de la respuesta
se establecerán en `6_contracts.md`.

La forma exacta de validar cada petición y de representar sus campos se basará
en los contratos aprobados y en el esquema oficial. Este plan no introduce
campos, validaciones por propiedad ni códigos HTTP.

## 7. Persistencia y borrado lógico

Cada repositorio será específico para su recurso y contendrá las consultas SQL
explícitas requeridas por los contratos aprobados. Las sentencias se
parametrizarán y el acceso a datos será asíncrono. El código se limitará a las
tablas habilitadas para V1.

La estrategia funcional de persistencia será:

- Las consultas de colecciones normales consideran únicamente filas con
	`activo = 1`.
- Las consultas individuales normales localizan únicamente registros activos.
- La operación funcional de borrado ejecuta una actualización que establece
	`activo = 0`; no ejecuta un borrado físico de la fila.
- Las claves y restricciones utilizadas por cada repositorio se obtendrán del
	esquema oficial y se documentarán en `5_data_model.md`.
- No se agregan reglas de unicidad no presentes en el esquema oficial.
- El SQL concreto se diseña durante la implementación, una vez aprobados
	`5_data_model.md` y `6_contracts.md`.

## 8. Estrategia de pruebas y límites de capas

Se planifican pruebas de Servicio usando repositorios falsos que implementen
`IRepositorio`. Estas pruebas permitirán:

- comprobar las reglas de Servicio sin ejecutar SQL Server;
- demostrar que Servicio depende del contrato de persistencia y no de una
	implementación concreta;
- ejecutar y verificar ese conjunto de pruebas con SQL Server apagado.

Los repositorios falsos pertenecen al ámbito de pruebas y sustituyen la
implementación de persistencia a través de la interfaz. Las pruebas no
implementarán ni simularán reglas fuera de la especificación vigente. Las
pruebas concretas y sus casos se desglosarán en `8_tasks.md` y se verificarán
conforme a `9_checklist.md`.

## 9. Arquitectura del Frontend

El Frontend se implementará como una aplicación Blazor Server independiente en
`innovacion_curricular_front`. Tendrá los modelos o DTO propios necesarios para
interpretar el JSON de la API; no referenciará el proyecto API ni reutilizará
clases C# de ese repositorio.

Se planifica un servicio HTTP específico para cada uno de los siete recursos.
Cada servicio consume la API mediante HTTP y no recibe conexiones o
credenciales de SQL Server. No se creará un servicio CRUD dinámico que reciba
el nombre de una tabla.

Habrá pantallas funcionales para las operaciones de cada recurso descritas en
`2_spec.md`. Las acciones se expresarán para las personas usuarias sin
obligarlas a conocer `PUT`, `PATCH` ni códigos HTTP. El Frontend contemplará:

- estado de consulta con datos recibidos de la API;
- estado legítimo sin registros, sin datos ficticios;
- estado de error de comunicación con la API;
- conservación adecuada de la información introducida si una operación falla,
	según el comportamiento que se concrete en los documentos correspondientes.

Ante indisponibilidad de la API, la interfaz no presentará datos inventados ni
como si la consulta hubiera sido exitosa. Los mensajes y estados específicos
quedarán sujetos a las decisiones documentadas para V1.

## 10. Docker e integración

Docker Compose será el mecanismo de ejecución conjunta de tres procesos o
servicios:

1. SQL Server.
2. API.
3. Frontend.

El Frontend depende funcionalmente de la API y no depende directamente de SQL
Server. La API es la única parte de la aplicación que accede a SQL Server. La
configuración permitirá iniciar el sistema en conjunto conforme a la
constitución.

Este plan no fija puertos, contraseñas, nombres de contenedores, nombres de
variables ni cadenas de conexión. Esos detalles se documentarán posteriormente
en `4_research.md`, `7_quickstart.md` y/o en los archivos de configuración que
correspondan, sin inventar valores en este documento.

## 11. Seguridad de configuración

Desde la preparación del proyecto se planifica:

- mantener `.env` fuera de Git;
- versionar `.env.example` sin credenciales reales ni secretos utilizables;
- suministrar secretos y cadenas de conexión mediante variables de entorno;
- no escribir contraseñas reales ni cadenas de conexión con credenciales reales
	en código o archivos versionados.

Los nombres y la estructura concreta de la configuración se determinarán en
los documentos de configuración y ejecución. Este plan no especifica valores
ni nombres de variables.

## 12. Fases de implementación previstas

Las siguientes fases son una planificación; no indican que su trabajo haya
sido realizado. El desglose de tareas, dependencias y compuertas se escribirá
posteriormente en `8_tasks.md`.

### Fase 0 — Preparación y verificación

- Verificar la disponibilidad del SDK .NET 10 y Docker.
- Revisar el script oficial de base de datos y confirmar las tablas y
	restricciones pertinentes para V1.
- Identificar la configuración necesaria sin inventar valores ni nombres que
	correspondan a documentos posteriores.
- Preparar exclusiones de Git para secretos y archivos locales, y preparar el
	esquema de variables de entorno según las decisiones aprobadas.

### Fase 1 — Esqueleto de API

- Crear el proyecto API y su configuración mínima.
- Preparar la capacidad de diagnóstico necesaria para verificar su inicio, sin
	introducir recursos funcionales fuera de V1.
- Establecer la organización base de modelos y excepciones comunes cuando
	corresponda a los contratos y al plan aprobados.

### Fase 2 — Modelos y peticiones de V1

- Preparar entidades para los siete recursos usando el esquema oficial.
- Preparar peticiones específicas de creación, reemplazo y actualización.
- Aplicar los campos y requisitos únicamente después de aprobar
	`5_data_model.md` y `6_contracts.md`.

### Fase 3 — Repositorios

- Definir interfaces específicas de repositorio para los siete recursos.
- Implementar sus repositorios SQL Server con Dapper y
	`Microsoft.Data.SqlClient`.
- Escribir SQL explícito y parametrizado para las operaciones aprobadas.
- Implementar las consultas de activos y el borrado lógico.

### Fase 4 — Servicios y pruebas de capas

- Definir las interfaces de servicio específicas.
- Implementar los servicios con las reglas funcionales de V1.
- Preparar repositorios falsos que implementen las interfaces de persistencia.
- Probar las reglas de Servicio sin requerir SQL Server.

### Fase 5 — Controllers y contratos HTTP

- Implementar Controllers específicos para los siete recursos.
- Registrar dependencias concretas en `Program.cs`.
- Configurar Swagger mediante `Swashbuckle.AspNetCore`.
- Validar peticiones y traducir resultados y excepciones conforme a
	`6_contracts.md`.

### Fase 6 — Frontend V1

- Crear la aplicación Blazor Server independiente.
- Definir sus propios modelos o DTO para el contrato JSON aprobado.
- Implementar servicios HTTP específicos por recurso.
- Construir las pantallas CRUD de los siete recursos.
- Contemplar estados con datos, sin registros y de error, sin inventar datos.

### Fase 7 — Docker e integración

- Preparar la ejecución conjunta de SQL Server, API y Frontend mediante Docker
	Compose.
- Configurar el flujo de dependencias: Frontend con API, API con SQL Server.
- Conectar la configuración a variables de entorno y mantener secretos fuera
	de Git.
- Verificar la integración de las operaciones definidas para V1.

### Fase 8 — Verificación y cierre

- Ejecutar las pruebas y verificaciones definidas para V1.
- Comprobar los criterios de aceptación funcionales y la regresión.
- Revisar el checklist de V1 y completar la documentación correspondiente.
- Cada integrante trabaja en su propia rama; se preparan commits pequeños y
	descriptivos para el Pull Request, sin trabajar directamente en `main`.
- Integrar y crear el tag `v1` únicamente después de aprobar las compuertas y
	cumplir los criterios de aceptación de API y Frontend.

## 13. Dependencias documentales

Al redactar este plan, las decisiones siguientes se dejaron deliberadamente
abiertas y se asignaron a los documentos enumerados. Esos documentos
posteriores ya las desarrollaron para V1; su estado posterior se resume en la
sección «Actualización de decisiones aplazadas».

- `4_research.md`: decisiones técnicas y alternativas que deban investigarse.
- `5_data_model.md`: esquema oficial, fuentes, columnas, tipos, claves y
	restricciones de los recursos permitidos en V1.
- `6_contracts.md`: rutas, cuerpos, formatos, respuestas, códigos HTTP y
	validaciones contractuales de V1.
- `7_quickstart.md`: preparación y pasos de ejecución del sistema.
- `8_tasks.md`: secuencia detallada de trabajo y sus compuertas.
- `9_checklist.md`: lista de verificación de la especificación y cierre.

La implementación seguirá esos documentos aprobados y no resolverá por
suposición las decisiones que les corresponden.

## 14. Riesgos y restricciones

- **Esquema y datos oficiales:** antes de concretar entidades, SQL o datos
	iniciales es necesario revisar el script y las fuentes oficiales. No se
	infieren columnas ni restricciones a partir de los nombres.
- **Unicidad:** no se crearán reglas funcionales de unicidad adicionales a las
	claves o restricciones que existan en el esquema oficial.
- **Dos repositorios:** API y Frontend se desarrollan por separado y se
	coordinan mediante el contrato JSON aprobado, sin referencias compartidas.
- **Disponibilidad de la API:** el Frontend debe distinguir datos recibidos,
	colección vacía y error de comunicación; nunca presentar datos ficticios
	como respuesta correcta.
- **Alcance acotado:** la existencia de otras tablas no habilita recursos fuera
	de los siete de V1 ni adelanta funcionalidades de V2, V3 o V4.
- **Configuración:** al redactarse este plan, los nombres de variables,
  puertos y parámetros de ejecución se remitieron a los documentos asignados.
  `7_quickstart.md` los definió posteriormente; no se incorporan credenciales
  reales a archivos versionados.
- **Cierre:** V1 requiere criterios de aceptación aprobados para API y
	Frontend; ni la implementación parcial ni la existencia de la base completa
	bastan para cerrar la versión.

## 15. Constitution Check

| Regla constitucional | Cómo la cumple este plan | Estado |
|---|---|---|
| Desarrollo por versiones y precedencia de la especificación | Este plan se limita a V1 y establece que la implementación sigue `2_spec.md` y los documentos aprobados de la versión. | CUMPLE |
| Implementar solo V1 | El alcance enumera únicamente los siete recursos V1 y excluye capacidades futuras. | CUMPLE |
| Cada versión incluye API y Frontend | Se planifican la API y la aplicación Blazor funcional para los siete recursos. | CUMPLE |
| Uso de .NET 10 | Se define ASP.NET Core con .NET 10 para API y Blazor Server con .NET 10 para Frontend. | CUMPLE |
| Dapper y SQL parametrizado | Se planifican Dapper, SQL explícito y parametrizado en repositorios específicos. | CUMPLE |
| Prohibición de Entity Framework y otros ORM | El stack los excluye expresamente junto con la generación automática de SQL. | CUMPLE |
| Controller → IServicio → Servicio → IRepositorio → RepositorioSqlServer | La arquitectura y las responsabilidades describen exactamente este flujo, seguido por SQL Server. | CUMPLE |
| API y Frontend independientes | El plan usa dos repositorios, sin referencia al proyecto API ni clases C# compartidas. | CUMPLE |
| Frontend consume únicamente HTTP | Los servicios HTTP del Frontend acceden a la API; no hay acceso directo a SQL Server. | CUMPLE |
| Borrado lógico y exclusión de inactivos | El borrado actualiza `activo = 0`; las consultas normales consideran registros activos. | CUMPLE |
| Secretos mediante variables de entorno | `.env` queda fuera de Git, `.env.example` se versiona sin secretos reales y la configuración usa variables de entorno. | CUMPLE |
| Proyecto en español | El documento y la organización prevista usan español, conservando los nombres técnicos obligatorios. | CUMPLE |
| Recursos específicos, sin CRUD dinámico por tabla | Se prevén componentes e interfaces por recurso; se prohíbe seleccionar recursos mediante el nombre de tabla. | CUMPLE |
| No anticipar V2, V3 ni V4 | Se excluyen explícitamente sus tablas, seguridad, dashboard y demás capacidades. | CUMPLE |
| Ejecución mediante Docker Compose | Se planifican los tres servicios: SQL Server, API y Frontend, sin inventar parámetros de entorno. | CUMPLE |
| Proceso Git constitucional | Cada integrante trabaja en su propia rama; se usan commits pequeños y descriptivos; los cambios llegan a `main` mediante Pull Request y el tag `v1` se crea solo después de superar criterios y compuertas. | CUMPLE |

## Actualización de decisiones aplazadas

Esta nota registra decisiones que el plan dejó abiertas al redactarse y que
posteriormente se detallaron en los documentos previstos. No modifica las
decisiones técnicas originales de este plan ni anticipa V2, V3 o V4:

- `5_data_model.md` definió el esquema, tipos, claves, restricciones y datos
	oficiales de los siete recursos V1.
- `6_contracts.md` definió las rutas, JSON, códigos HTTP y validaciones
	contractuales de V1.
- `7_quickstart.md` definió la configuración de ejecución de V1, los nombres
	de variables, la base `innovacion_curricular` y los puertos de desarrollo.
- `8_tasks.md` definió la secuencia detallada de implementación y sus
	compuertas.

## 16. Aclaraciones pendientes

No se detectaron aclaraciones necesarias para formular este plan. Los detalles
de esquema, contratos HTTP, nombres de variables y parámetros de ejecución
corresponden a los documentos posteriores indicados en la sección 13 y no se
fijan aquí.
