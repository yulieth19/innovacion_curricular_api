# Quickstart — Arranque y verificación de V1

## 1. Propósito y estado

Este documento describe cómo deberá configurarse, iniciarse y verificarse V1 una
vez implementada. Los comandos y resultados son el procedimiento objetivo, no
afirmaciones de que la implementación ya exista o funcione.

En este momento, los archivos de ejecución y aplicaciones mencionados abajo aún
deben crearse durante las tareas de V1. En particular, se crearán el archivo de
Docker Compose, los proyectos API y Frontend, el inicializador de base de datos
si se requiere, y las plantillas locales de configuración. Las tareas y el orden
preciso de implementación se documentarán en `8_tasks.md`.

## 2. Prerrequisitos

Para el arranque integrado se requerirá Docker con Docker Compose disponible.
Para ejecutar o compilar API, Frontend o pruebas fuera de contenedores se
requerirá el SDK .NET 10. No se requiere instalar SQL Server localmente para el
arranque mediante Compose.

## 3. Estructura de repositorios

Los dos repositorios Git son directorios hermanos dentro de una carpeta común:

```text
innovacion_curricular/
├── innovacion_curricular_api/
└── innovacion_curricular_front/
```

`innovacion_curricular_api` contiene el Spec Kit y será también la raíz del
archivo `docker-compose.yml`. `innovacion_curricular_front` es un repositorio
independiente; no será un submódulo ni se copiará dentro del repositorio API.

El contexto de construcción del Frontend se referenciará desde Compose mediante
el directorio hermano `../innovacion_curricular_front`. Por ello, ambos
repositorios deben estar clonados con esos nombres y como hermanos para que el
arranque conjunto encuentre el contexto de construcción.

## 4. Configuración y secretos

La configuración local sensible se suministrará mediante variables de entorno.
`.env` se crea localmente, se mantiene fuera de Git y no debe contener valores
que se suban al repositorio. `.env.example` será una plantilla versionada, con
placeholders seguros y sin secretos utilizables. Ninguna contraseña real se
escribe en `appsettings.json`, en código o en otro archivo versionado.

Los nombres acordados son:

| Variable | Uso |
|---|---|
| `MSSQL_SA_PASSWORD` | Contraseña local del usuario `sa` de SQL Server. El valor real se configura localmente y no se versiona. |
| `ConnectionStrings__SqlServer` | Cadena de conexión que ASP.NET Core interpreta como `ConnectionStrings:SqlServer`. |
| `UrlApi` | Dirección base que usa el Frontend para comunicarse con la API mediante HTTP. |

La plantilla futura `.env.example` podrá mostrar únicamente valores de forma,
por ejemplo:

```dotenv
MSSQL_SA_PASSWORD=<definir-localmente>
ConnectionStrings__SqlServer=<cadena-de-conexion-local-sin-credenciales-reales>
UrlApi=<direccion-http-del-servicio-api>
```

Estos placeholders no son valores de ejecución. Al crear `.env`, cada integrante
los sustituye localmente por la configuración necesaria y no comparte ni
versiona el archivo resultante.

Dentro de Docker, la API se conectará al host SQL `sqlserver`, puerto interno
`1433`, base `innovacion_curricular` y usuario `sa`; la contraseña se obtiene de
la configuración. El Frontend usará `UrlApi` con el nombre del servicio API
asignado en Compose, no `localhost`. El nombre definitivo de ese servicio se
concretará en `8_tasks.md` y en la implementación. Para procesos ejecutados
fuera de Docker se podrá utilizar `localhost` con los puertos publicados.

## 5. Docker Compose e inicialización

El archivo `docker-compose.yml` se ubicará en la raíz de
`innovacion_curricular_api`. Esa ubicación mantiene los dos repositorios
independientes y permite que Compose use el contexto hermano
`../innovacion_curricular_front`. El archivo se creará durante las tareas de
implementación; no existe por efecto de este documento.

La ejecución integrada incluirá:

1. SQL Server 2022, con la imagen prevista `mcr.microsoft.com/mssql/server:2022-latest`.
2. Un inicializador SQL auxiliar, si hace falta para cargar el script oficial; prepara la base y termina, no es una aplicación funcional.
3. La API.
4. El Frontend.

El flujo de dependencia funcional es:

```text
Frontend → HTTP → API → SQL Server
```

El Frontend no se conecta directamente a SQL Server. Compose deberá esperar a
que SQL Server esté saludable y a que la inicialización necesaria termine antes
de considerar disponible la API. El inicializador, si se incorpora, será un
proceso auxiliar de infraestructura.

La base se llamará `innovacion_curricular`, no `innovacion_local`. La
inicialización partirá del script oficial/adaptado del proyecto, conservará las
correcciones aprobadas en `5_data_model.md` y contendrá el esquema completo
proporcionado por el proyecto. Esto no amplía las tablas que el código puede
utilizar: V1 implementa CRUD únicamente para los siete recursos autorizados.
El script no se escribe en este quickstart y deberá crearse durante la
implementación.

## 6. Puertos y direcciones de desarrollo

Los puertos de desarrollo acordados son:

| Servicio | Dirección desde el equipo local |
|---|---|
| API | `http://localhost:8072` |
| Swagger | `http://localhost:8072/swagger` |
| Frontend | `http://localhost:8073` |
| SQL Server | `localhost,11471` (puerto publicado `11471` hacia `1433` interno) |

Dentro de la red de Compose, la API usa `sqlserver:1433` para SQL Server. El
Frontend se comunica con la API a través del nombre de servicio Docker que se
establezca para ella. Los puertos indicados son para desarrollo local y no
implican ni publican credenciales.

## 7. Arranque, consulta de estado y detención

Una vez que las tareas de V1 hayan creado el archivo Compose, `.env.example`,
los proyectos, el script aplicable y la configuración necesaria, el objetivo
será ejecutar desde `innovacion_curricular_api/`:

```powershell
docker compose up -d --build
```

El comando deberá construir y levantar SQL Server, inicializar la base cuando
corresponda, y arrancar API y Frontend. No se debe ejecutar todavía como si
esos artefactos ya estuvieran disponibles.

Para consultar el estado de los servicios:

```powershell
docker compose ps
```

Para consultar los registros de ejecución:

```powershell
docker compose logs
```

Para detener el sistema conservando el volumen y los datos locales:

```powershell
docker compose down
```

**Advertencia:** `docker compose down -v` elimina el volumen local de SQL Server
y todos sus datos. Solo debe utilizarse cuando se desee reinicializar por
completo la base de desarrollo. Después de eliminar el volumen, se reconstruye
y levanta el entorno con:

```powershell
docker compose down -v
docker compose up -d --build
```

La inicialización vuelve a aplicar el esquema y los datos oficiales según los
archivos creados durante la implementación.

## 8. Diagnóstico de API

Después del arranque, la comprobación de diagnóstico será:

```powershell
curl.exe http://localhost:8072/
```

Se espera `200` y un objeto JSON que incluya `version` con valor `v1` y
`contratos` con valor `/swagger`, además de un mensaje de la API. Este endpoint
no consulta SQL Server; por tanto, su respuesta verifica que la API inició, no
que la base de datos esté disponible.

La documentación Swagger deberá estar disponible durante el desarrollo en:

```text
http://localhost:8072/swagger
```

Swagger permite inspeccionar y comprobar manualmente la API. No reemplaza el
Frontend ni las pruebas y criterios de aceptación de V1.

## 9. Verificación de la base de datos

Después de la inicialización, verificar con una herramienta de administración
SQL que:

- existe la base `innovacion_curricular`;
- existen las tablas del esquema completo proporcionado por el proyecto;
- las siete tablas V1 tienen la estructura aprobada en `5_data_model.md`;
- `area_conocimiento` contiene 218 registros oficiales;
- `universidad` contiene 6 registros oficiales;
- `activo` existe en cada una de las siete tablas V1;
- `area_conocimiento.id` admite identificadores alfanuméricos;
- `area_conocimiento.disciplina` admite la longitud aprobada en el modelo.

No se supone ni se documenta aquí una cantidad de datos iniciales para las
otras cinco tablas. La corrección de dato confirmada del catálogo de
`area_conocimiento` es `Cienias Naturales` a `Ciencias Naturales`; no se aplican
otras correcciones inventadas.

Para una conexión SQL externa se usará el destino de desarrollo
`localhost,11471`, base `innovacion_curricular` y usuario `sa`; la contraseña se
obtiene del `.env` local, nunca de un archivo versionado. El comando definitivo
de `sqlcmd` no se fija aquí porque depende de los nombres y mecanismos que se
implementen para el servicio de SQL Server y su inicializador.

## 10. Smoke test de la API

El smoke test deberá seguir los contratos de `6_contracts.md`. Se repetirá para
cada fila de la matriz y para los siete recursos. Las rutas, nombres de campos,
claves y tipos se toman de ese contrato; no se usarán nombres genéricos de tabla
para seleccionar recursos.

### A. Verificaciones comunes por recurso

Para cada recurso, comprobar:

1. GET de colección con filas activas: `200`, con el sobre de listado y sin el
   campo público `activo`.
2. GET de colección sin filas activas: `204`, sin cuerpo.
3. GET de colección con `limite <= 0`: `400`.
4. GET por clave activa existente: `200` con el objeto público del recurso.
5. GET por clave inexistente: `404`.
6. GET por clave inactiva: `404`.
7. POST válido con todos los campos públicos, incluida la clave: `200`.
8. POST incompleto: `422`.
9. POST con clave primaria duplicada: `500`.
10. PUT completo sobre el registro controlado: `200`.
11. PUT incompleto: `422`.
12. PATCH parcial sobre el registro controlado: `200`.
13. PATCH vacío (`{}`): `400`.
14. DELETE del registro de prueba activo: `200`; segundo DELETE: `404`.
15. Comprobar en SQL Server que el registro de prueba permanece con
    `activo = 0` y que ya no aparece en consultas normales.

### B. Prueba PUT frente a PATCH

Con un registro de prueba existente, enviar a PUT un cuerpo al que le falte un
campo modificable obligatorio: se espera `422`. Enviar el mismo subconjunto
válido a PATCH: se espera `200`, ya que los campos omitidos se conservan.
Enviar `{}` a PATCH: se espera `400`. Esta verificación se repetirá conforme a
la matriz para los recursos V1.

### C. Registros y entorno de prueba controlados

Las escrituras se ejecutarán únicamente con registros de prueba controlados y
claves primarias comprobadas como libres. La prueba de PK duplicada reutiliza
la clave del registro controlado dentro de la prueba; no debe provocar
colisiones con datos oficiales.

No se harán PUT ni DELETE sobre los 218 registros oficiales de
`area_conocimiento` ni sobre los 6 de `universidad`. Para probar `204` en una
colección que contiene datos oficiales, se usará una instancia de prueba
aislada con el esquema V1 y sin esos datos de catálogo. No se vaciarán ni
alterarán los catálogos oficiales para obtener una respuesta vacía. Las otras
colecciones también se verificarán según su estado real, sin asumir conteos no
documentados.

Las rutas por recurso que deben recorrer las verificaciones son:

| Recurso | Ruta base | Clave de prueba |
|---|---|---|
| `area_conocimiento` | `/api/area_conocimiento` | `id` de texto libre verificado |
| `universidad` | `/api/universidad` | `id` entero libre verificado |
| `aspecto_normativo` | `/api/aspecto_normativo` | `id` entero libre verificado |
| `practica_estrategia` | `/api/practica_estrategia` | `id` entero libre verificado |
| `enfoque` | `/api/enfoque` | `id` entero libre verificado |
| `car_innovacion` | `/api/car_innovacion` | `id` entero libre verificado |
| `aliado` | `/api/aliado` | `nit` entero libre verificado |

Las solicitudes deberán seguir los cuerpos y longitudes de `6_contracts.md`.
No se inventan registros oficiales para hacer que las pantallas o listados
parezcan llenos.

## 11. Borrado lógico: verificación completa

El caso de borrado no termina al recibir `200`. Para cada recurso, con un
registro de prueba controlado:

1. DELETE del registro activo: esperar `200`.
2. GET posterior por clave: esperar `404`.
3. Segundo DELETE: esperar `404`.
4. Revisar en SQL Server que la fila continúa almacenada.
5. Confirmar que el valor de `activo` es `0`.
6. Confirmar que la fila no aparece en el listado normal.

No se ejecuta un borrado físico como parte del smoke test.

## 12. Pruebas de capas

Deberán existir pruebas de los Servicios usando repositorios falsos que
implementan las mismas interfaces `IRepositorio` utilizadas por la aplicación.
Estas pruebas verifican las reglas funcionales y la separación entre Servicio y
persistencia, y deberán poder ejecutarse con SQL Server apagado.

No se fija todavía un comando definitivo de pruebas: el proyecto de pruebas y
su comando aún deben crearse y documentarse durante la implementación. Antes
del cierre de V1, el comando definitivo deberá quedar registrado y sus pruebas
deberán pasar.

## 13. Verificación manual del Frontend

V1 no termina con la API. Cuando exista la aplicación Blazor Server en
`innovacion_curricular_front`, revisar manualmente las pantallas de los siete
recursos y comprobar para cada una:

- la pantalla del recurso está accesible;
- se muestra un listado con datos cuando la API devuelve filas;
- el estado sin registros es comprensible y no contiene datos ficticios;
- se puede crear un registro de prueba controlado;
- se puede reemplazar el registro completo;
- se puede actualizar parcialmente;
- se solicita confirmación antes de retirar/eliminar;
- el registro desaparece de las consultas normales tras el borrado lógico;
- los errores se presentan en español;
- si una operación falla, la información introducida se conserva de acuerdo
  con el comportamiento definido para V1;
- los textos visibles no exponen jerga innecesaria como `PUT`, `PATCH`, `422` o
  SQL Server.

Las verificaciones de escritura deben usar datos controlados y claves libres;
no se modificarán ni retirarán filas de los catálogos oficiales.

## 14. Independencia del Frontend ante caída de API

Como verificación arquitectónica, una vez implementados los servicios y nombres
de Compose:

1. detener temporalmente solo el servicio API;
2. mantener SQL Server y Frontend levantados;
3. abrir el Frontend en `http://localhost:8073`;
4. confirmar que la aplicación sigue respondiendo y muestra un aviso
   comprensible de indisponibilidad;
5. confirmar que no muestra datos inventados ni presenta datos previos como si
   provinieran de una consulta exitosa;
6. volver a levantar la API y confirmar que la comunicación puede restablecerse.

Los comandos específicos de parada y arranque por servicio se completarán
cuando `8_tasks.md` y la implementación establezcan los nombres de servicio.
El Frontend usa el nombre del servicio API en Docker, no `localhost`, para las
solicitudes entre contenedores.

## 15. Regresión

V1 no tiene una versión funcional anterior que deba conservarse como regresión.
Antes de cerrar V1, sí deben pasar todos los smoke tests, pruebas de capas,
verificaciones manuales del Frontend y criterios de aceptación definidos para
esta versión.

Desde V2 en adelante, las verificaciones aprobadas de V1 deberán conservarse y
repetirse antes de cerrar cada versión posterior, además de las pruebas propias
de esa versión.

## 16. Condición de cierre de V1

V1 solo podrá considerarse terminada cuando:

- los siete CRUD de la API funcionen de acuerdo con `6_contracts.md`;
- las siete pantallas del Frontend funcionen y consuman únicamente la API;
- el borrado lógico y el filtrado de inactivos pasen sus verificaciones;
- estén cargados y verificados los catálogos oficiales disponibles;
- las pruebas de capas pasen sin depender de SQL Server;
- el smoke test de los siete recursos pase;
- la verificación manual del Frontend pase;
- no existan secretos versionados;
- el checklist de V1 esté aprobado;
- los cambios se integren a `main` mediante Pull Request;
- se cree la etiqueta `v1` sobre `main` después de superar las compuertas.

Este documento no ejecuta ni aprueba ninguna de esas acciones. El cierre queda
sujeto a los criterios y al proceso Git establecidos por la constitución y la
especificación de V1.

## 17. Problemas frecuentes

| Síntoma | Comprobación o respuesta prevista |
|---|---|
| SQL Server todavía no está saludable | Revisar `docker compose ps` y los logs de Compose; esperar a que el servicio de base de datos esté saludable y a que la inicialización termine antes de evaluar la API. |
| Se cambió la contraseña local, pero se conserva el volumen anterior | La instancia puede conservar credenciales y datos del volumen existente. Confirmar la configuración local; si se decide reiniciar toda la base de desarrollo, recordar que `docker compose down -v` elimina los datos locales. |
| La API no puede resolver el host SQL | Dentro de Docker, verificar que la conexión use `sqlserver:1433`, no `localhost`; revisar el estado del servicio y la inicialización. |
| El Frontend intenta usar `localhost` dentro de Docker | Configurar `UrlApi` con el nombre del servicio API en Compose y el puerto interno previsto; el nombre final se fijará en tareas/implementación. |
| Los inactivos aparecen en una consulta | Verificar que la consulta normal aplique el filtro `activo = 1` conforme a `5_data_model.md` y `6_contracts.md`. |
| Un listado vacío responde `200` en vez de `204` | Revisar la implementación del contrato de lista vacía en `6_contracts.md`; la respuesta vacía requerida es `204` sin cuerpo. |
| Un script `.sh` no se ejecuta en Windows por finales de línea | Comprobar que el archivo use finales de línea LF; el script se creará en las tareas si la inicialización lo requiere. |
| Falta uno de los repositorios hermanos | Confirmar que `innovacion_curricular_api` e `innovacion_curricular_front` estén presentes en el mismo directorio padre. |
| Docker Compose no encuentra el contexto del Frontend | Confirmar que el directorio hermano se llame exactamente `innovacion_curricular_front` y que el contexto relativo `../innovacion_curricular_front` exista. |

## 18. Aclaraciones pendientes

No se detectaron aclaraciones pendientes para el arranque y verificación de V1.
