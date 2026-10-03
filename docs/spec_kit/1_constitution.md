# Constitución del proyecto de aula — Innovación Curricular

> **Documento permanente.** Estas reglas rigen todas las versiones del proyecto.
> La especificación de cada versión define su alcance y sus contratos; esta
> constitución establece las reglas que ninguna especificación puede contradecir.
> El Spec Kit se mantiene en `innovacion_curricular_api`; el proyecto tiene dos
> repositorios independientes: `innovacion_curricular_api` y
> `innovacion_curricular_front`.

---

## Artículo 1 — SDD por versiones: la especificación manda

- El proyecto se desarrolla mediante **Spec-Driven Development (SDD)**,
	versión por versión. Antes de implementar una versión debe existir su
	especificación correspondiente en el Spec Kit.
- La especificación vigente es la fuente de verdad sobre el código. Si el código
	hace algo que no está especificado, no se incorpora; si la especificación
	exige algo que el código no cumple, la versión no está terminada.
- No se anticipan tablas en uso, campos, validaciones, contratos ni
	funcionalidades de versiones futuras. El código de una versión solo utiliza
	lo permitido por la especificación de esa versión.
- Una versión se cierra únicamente cuando cumple todos sus criterios de
	aceptación y se crea su etiqueta Git `vN` correspondiente. Hasta entonces no
	se considera terminada.

## Artículo 2 — Cada versión entrega API y Frontend

Cada versión incluye la parte de API y la parte de Frontend definidas en su
especificación, ambas funcionando en conjunto. Una versión no se cierra si uno
de esos dos componentes no cumple sus criterios de aceptación.

## Artículo 3 — Stack del Backend y SQL explícito

- El Backend se desarrolla en **C# con ASP.NET Core sobre .NET 10**.
- El acceso a SQL Server utiliza **Dapper** y **Microsoft.Data.SqlClient**.
- Está prohibido usar Entity Framework o cualquier otro ORM.
- Toda sentencia SQL se escribe explícitamente y se parametriza. Está prohibido
	concatenar valores recibidos o variables dentro del SQL.
- El acceso a datos utiliza `async`/`await`.

## Artículo 4 — Arquitectura obligatoria por interfaces

La única dirección permitida para el flujo de una petición es:

```text
HTTP
→ Controller
→ IServicio
→ Servicio
→ IRepositorio
→ RepositorioSqlServer
→ SQL Server
```

- El Controller atiende HTTP y no contiene SQL.
- El Servicio implementa las reglas definidas por la especificación y no conoce
	HTTP ni SQL Server.
- El Repositorio ejecuta el acceso a datos y no conoce HTTP.
- Las capas se comunican mediante interfaces. Las dependencias concretas se
	registran únicamente en `Program.cs`.
- Ninguna capa puede saltarse las capas intermedias ni asumir responsabilidades
	que pertenecen a otra.

## Artículo 5 — Recursos específicos y contratos HTTP

- Cada recurso tiene sus propios modelos, peticiones, interfaces, servicio,
	repositorio y controlador cuando corresponda.
- Los endpoints son específicos por recurso. Está prohibido crear controladores
	o repositorios genéricos que reciban el nombre de una tabla para decidir qué
	recurso consultar o modificar.
- `PUT` representa reemplazo completo y `PATCH` representa actualización
	parcial.
- Las rutas, cuerpos, respuestas y códigos HTTP exactos de cada versión se
	documentan en el `6_contracts.md` de su especificación y se implementan de
	acuerdo con ese documento. Esta constitución no fija esos contratos.

## Artículo 6 — Base de datos y borrado lógico

- La base de datos completa existe desde V1. Su existencia no autoriza al
	código a utilizar todas sus tablas: la aplicación solo puede utilizar las
	tablas permitidas por la especificación de la versión actual.
- El borrado siempre es lógico: se marca `activo = 0`; no se elimina
	físicamente el registro como operación funcional del sistema.
- Los listados y búsquedas normales excluyen los registros inactivos.
- No se inventan tablas, campos, relaciones, restricciones, validaciones ni
	otros requisitos que no estén definidos en la especificación vigente.

## Artículo 7 — Secretos y configuración

- Las credenciales y cadenas de conexión se suministran mediante variables de
	entorno; no se escriben en el código ni en archivos versionados.
- El archivo `.env` nunca se versiona.
- Debe existir un `.env.example` con los nombres y la forma de la configuración
	requerida, pero sin credenciales reales ni secretos utilizables.
- Los nombres concretos de las variables, sus valores de desarrollo y la
	configuración de ejecución se determinan en los documentos de planificación
	y puesta en marcha, sin incluir secretos en ellos.

## Artículo 8 — Dos repositorios, comunicación por HTTP

- `innovacion_curricular_api` y `innovacion_curricular_front` son repositorios
	independientes. El Spec Kit vive en el repositorio de la API.
- El Frontend se implementa con **Blazor Server sobre .NET 10** y consume la
	API exclusivamente mediante HTTP. Nunca se conecta directamente a SQL
	Server.
- API y Frontend no comparten proyectos, referencias ni clases C#. Su único
	contrato compartido es el JSON intercambiado por HTTP, documentado para cada
	versión en `6_contracts.md`.

## Artículo 9 — El idioma del proyecto es español

Los nombres, rutas, mensajes, comentarios y documentación del proyecto se
escriben en español. Esta regla se aplica a ambos repositorios y a los
documentos del Spec Kit, respetando únicamente los nombres obligatorios de
tecnologías y convenciones del lenguaje.

## Artículo 10 — Inicio mediante Docker Compose

La aplicación debe poder iniciarse mediante Docker Compose, de acuerdo con lo
que definan posteriormente el plan y el quickstart de la versión. Esta
constitución no establece comandos, puertos, imágenes, credenciales ni cadenas
de conexión.

## Artículo 11 — Trabajo con Git y cierre de versiones

- Nunca se trabaja directamente en `main`.
- Cada integrante trabaja en su propia rama. Los cambios llegan a `main` por
	medio de Pull Request.
- Los commits son pequeños y descriptivos.
- Al cerrar una versión, se crea su etiqueta correspondiente (`v1`, `v2`, etc.)
	solo después de cumplir sus criterios de aceptación.

## Artículo 12 — Mapa general de versiones

Este mapa indica el alcance general y no reemplaza ni adelanta las
especificaciones de cada versión:

- **V1:** CRUD de API y Frontend para las tablas sin clave foránea:
	`area_conocimiento`, `universidad`, `aspecto_normativo`,
	`practica_estrategia`, `enfoque`, `car_innovacion` y `aliado`.
- **V2:** tablas con claves foráneas y sus relaciones, según la especificación
	correspondiente.
- **V3:** autenticación JWT, sesiones, roles y administración de usuarios,
	según la especificación futura.
- **V4:** consultas multitabla, dashboard, páginas corporativas,
	responsive/PWA, identidad visual y publicación, según la especificación
	futura.

La implementación detallada de V2, V3 y V4 queda pendiente de sus respectivas
especificaciones. No se derivan de este mapa requisitos adicionales.

## Artículo 13 — Enmiendas a la constitución

Esta constitución solo cambia mediante una decisión explícita y documentada del
proyecto. Una implementación o una especificación no puede introducir una
excepción tácita: ante un conflicto, se detiene el cambio y se resuelve
formalmente antes de continuar.

---

*Constitución del proyecto de aula. Las especificaciones de versión determinan
los contratos y detalles propios de cada entrega, dentro de estas reglas
permanentes.*

> **Aclaración resuelta para V1:** los nombres concretos de configuración se
> definieron posteriormente en `7_quickstart.md`, conforme al Artículo 7.
