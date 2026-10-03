# Contratos HTTP — V1 Innovación Curricular

## 1. Propósito y convenciones globales

Este documento fija las rutas, cuerpos JSON, respuestas y códigos HTTP de V1
para los siete recursos autorizados. API y Frontend deben implementar estos
contratos; los nombres de ruta identifican recursos concretos y no representan
un endpoint genérico que seleccione tablas dinámicamente.

Los nombres de propiedades JSON usan `camelCase`. Los nombres de las rutas
conservan los identificadores concretos de los recursos. Las lecturas
individuales devuelven el objeto público directamente; los listados usan el
sobre de colección de la sección 4.

El campo `activo` es interno: no forma parte de las respuestas públicas ni de
los cuerpos públicos normales. Al crear, el registro inicia con `activo = 1`.
`PUT` y `PATCH` no pueden modificarlo. Si se envía `activo` en un cuerpo de
escritura, se ignora y no cambia el estado del registro.

Ninguna clave V1 es `IDENTITY`. POST debe contener la clave primaria; en PUT,
PATCH y DELETE la clave se toma de la ruta. La clave no se modifica por el
cuerpo de PUT o PATCH.

## 2. Rutas

Cada recurso tiene su propia ruta base y sus propias operaciones:

| Recurso | GET colección / POST | Clave de ruta | GET / PUT / PATCH / DELETE por clave |
|---|---|---|---|
| `area_conocimiento` | `/api/area_conocimiento` | `{id}` (texto, máximo 6 caracteres) | `/api/area_conocimiento/{id}` |
| `universidad` | `/api/universidad` | `{id}` (entero) | `/api/universidad/{id}` |
| `aspecto_normativo` | `/api/aspecto_normativo` | `{id}` (entero) | `/api/aspecto_normativo/{id}` |
| `practica_estrategia` | `/api/practica_estrategia` | `{id}` (entero) | `/api/practica_estrategia/{id}` |
| `enfoque` | `/api/enfoque` | `{id}` (entero) | `/api/enfoque/{id}` |
| `car_innovacion` | `/api/car_innovacion` | `{id}` (entero) | `/api/car_innovacion/{id}` |
| `aliado` | `/api/aliado` | `{nit}` (entero) | `/api/aliado/{nit}` |

Para cada ruta por clave se exponen únicamente GET, PUT, PATCH y DELETE. No
existe `/api/{tabla}` como endpoint que elija el recurso en tiempo de ejecución.

## 3. Códigos HTTP globales

V1 utiliza estos códigos para los resultados y errores documentados:

| Código | Significado contractual |
|---|---|
| `200` | Lectura con datos, creación correcta, reemplazo correcto, actualización correcta o eliminación lógica correcta. |
| `204` | Listado correcto sin filas activas; respuesta sin cuerpo. |
| `400` | Regla de petición u operación inválida que no es validación estructural del DTO: `limite` no positivo y PATCH sin campos modificables. Un valor de `limite` que no pueda interpretarse como entero también es una consulta inválida. |
| `404` | La clave solicitada no corresponde a un registro activo, o se intenta modificar/eliminar uno inexistente o inactivo. |
| `422` | Cuerpo estructuralmente inválido, campo obligatorio ausente en POST o PUT, tipo JSON incompatible, o texto que supera la longitud contractual derivada del modelo. |
| `500` | La base de datos rechaza la operación, incluida una clave primaria duplicada, o se produce un error interno o de infraestructura no traducido a otro caso contractual. |

No se definen otros códigos HTTP para los casos contractuales de V1. Los
resultados `204` no incluyen cuerpo; los demás resultados y errores indicados
llevan los cuerpos descritos en este documento.

## 4. Formato de listados

Cada GET de colección acepta el parámetro opcional `limite`:

- Debe ser un entero positivo.
- Si se omite, su valor es `1000`.
- Si es menor o igual a cero, la respuesta es `400`.
- Si no puede interpretarse como entero, la respuesta es `400`.
- No se establece un máximo adicional en este contrato.

Solo se incluyen filas activas. `activo` no aparece en los objetos de `datos`.
Si hay filas activas que devolver, la respuesta es `200` y tiene esta forma:

```json
{
  "tabla": "universidad",
  "limite": 1000,
  "total": 1,
  "datos": [
    {
      "id": 1,
      "nombre": "...",
      "tipo": "...",
      "ciudad": "..."
    }
  ]
}
```

`tabla` contiene exactamente el nombre concreto del recurso de la ruta;
`limite` contiene el valor aplicado; `total` es el número de objetos incluidos
en `datos`. Si no hay filas activas, la respuesta es `204` sin cuerpo.

Ejemplos de rutas de colección:

```text
GET /api/area_conocimiento?limite=1000
GET /api/universidad?limite=1000
GET /api/aspecto_normativo?limite=1000
GET /api/practica_estrategia?limite=1000
GET /api/enfoque?limite=1000
GET /api/car_innovacion?limite=1000
GET /api/aliado?limite=1000
```

## 5. Formato de errores

Para los errores `400`, `404` y `500`, el cuerpo general es:

```json
{
  "estado": 404,
  "mensaje": "Universidad no encontrada.",
  "detalle": "No existe un registro activo con la clave solicitada."
}
```

`estado` coincide con el código HTTP. `mensaje` está en español y es
específico del recurso cuando corresponda. `detalle` explica el motivo sin
exponer stack traces, secretos, cadenas de conexión ni credenciales.

Para `422`, se usa este formato y se incluye `errores`:

```json
{
  "estado": 422,
  "mensaje": "Datos inválidos.",
  "detalle": "Uno o más campos no cumplen el contrato.",
  "errores": [
    "El campo nombreContacto es obligatorio."
  ]
}
```

`errores` se utiliza para las validaciones `422` y no se incluye en los otros
formatos de error. Sus elementos describen en español los campos ausentes, los
tipos JSON incompatibles o las longitudes excedidas.

Para errores de persistencia `500`, `detalle` comunica la información necesaria
para el diagnóstico, por ejemplo el rechazo de una clave primaria duplicada,
sin revelar stack traces, secretos, cadenas de conexión ni credenciales.

Mensajes de error `404` por recurso:

| Recurso | `mensaje` |
|---|---|
| `area_conocimiento` | `Área de conocimiento no encontrada.` |
| `universidad` | `Universidad no encontrada.` |
| `aspecto_normativo` | `Aspecto normativo no encontrado.` |
| `practica_estrategia` | `Práctica y estrategia no encontradas.` |
| `enfoque` | `Enfoque no encontrado.` |
| `car_innovacion` | `Característica de innovación no encontrada.` |
| `aliado` | `Aliado no encontrado.` |

La misma respuesta `404` se utiliza si el registro no existe o está inactivo; no
se revela una diferencia entre esos dos estados.

## 6. Diagnóstico y Swagger

`GET /` no recibe parámetros ni cuerpo, no consulta SQL Server y responde `200`
con un objeto que incluye como mínimo:

```json
{
  "mensaje": "API Innovación Curricular disponible.",
  "version": "v1",
  "contratos": "/swagger"
}
```

Swagger se publica en `GET /swagger` como documentación y herramienta de
verificación durante el desarrollo. No reemplaza el Frontend, las pruebas, los
criterios de aceptación ni las verificaciones de regresión.

## 7. `area_conocimiento`

Clave primaria: `id`, texto de máximo 6 caracteres. Las lecturas devuelven
únicamente estos campos públicos, sin `activo`:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | texto | 6 | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `granArea` | texto | 60 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `area` | texto | 60 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `disciplina` | texto | 150 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe el objeto completo:

```json
{
  "id": "1A01",
  "granArea": "Ciencias Naturales",
  "area": "...",
  "disciplina": "..."
}
```

PUT recibe obligatoriamente todos los campos modificables y no incluye `id`:

```json
{
  "granArea": "Ciencias Naturales",
  "area": "...",
  "disciplina": "..."
}
```

PATCH recibe cualquier subconjunto no vacío de `granArea`, `area` y
`disciplina`, por ejemplo:

```json
{
  "disciplina": "..."
}
```

GET por clave responde `200` con un objeto de la forma del POST, sin `activo`.
Una clave inexistente o inactiva responde `404`.

## 8. `universidad`

Clave primaria: `id`, entero. Las lecturas devuelven únicamente los campos
públicos siguientes:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `nombre` | texto | 60 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `tipo` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `ciudad` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "id": 1,
  "nombre": "...",
  "tipo": "...",
  "ciudad": "..."
}
```

PUT recibe obligatoriamente `nombre`, `tipo` y `ciudad`, sin `id` en el cuerpo.
PATCH recibe cualquier subconjunto no vacío de esos tres campos. GET por clave
devuelve el objeto público con `id`, `nombre`, `tipo` y `ciudad`, sin `activo`.

## 9. `aspecto_normativo`

Clave primaria: `id`, entero. Sus campos públicos son:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `tipo` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `descripcion` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `fuente` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "id": 1,
  "tipo": "...",
  "descripcion": "...",
  "fuente": "..."
}
```

PUT recibe obligatoriamente `tipo`, `descripcion` y `fuente`, sin `id` en el
cuerpo. PATCH recibe cualquier subconjunto no vacío de esos tres campos. GET
por clave devuelve `id`, `tipo`, `descripcion` y `fuente`, sin `activo`.

## 10. `practica_estrategia`

Clave primaria: `id`, entero. Sus campos públicos son:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `tipo` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `nombre` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `descripcion` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "id": 1,
  "tipo": "...",
  "nombre": "...",
  "descripcion": "..."
}
```

PUT recibe obligatoriamente `tipo`, `nombre` y `descripcion`, sin `id` en el
cuerpo. PATCH recibe cualquier subconjunto no vacío de esos tres campos. GET
por clave devuelve `id`, `tipo`, `nombre` y `descripcion`, sin `activo`.

## 11. `enfoque`

Clave primaria: `id`, entero. Sus campos públicos son:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `nombre` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `descripcion` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "id": 1,
  "nombre": "...",
  "descripcion": "..."
}
```

PUT recibe obligatoriamente `nombre` y `descripcion`, sin `id` en el cuerpo.
PATCH recibe cualquier subconjunto no vacío de esos dos campos. GET por clave
devuelve `id`, `nombre` y `descripcion`, sin `activo`.

## 12. `car_innovacion`

Clave primaria: `id`, entero. Sus campos públicos son:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `id` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `nombre` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `descripcion` | texto | No especificada (`VARCHAR(MAX)`) | Obligatorio | Obligatorio | Opcional, si se modifica |
| `tipo` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "id": 1,
  "nombre": "...",
  "descripcion": "...",
  "tipo": "..."
}
```

PUT recibe obligatoriamente `nombre`, `descripcion` y `tipo`, sin `id` en el
cuerpo. PATCH recibe cualquier subconjunto no vacío de esos tres campos. GET
por clave devuelve `id`, `nombre`, `descripcion` y `tipo`, sin `activo`. No se
establece un máximo contractual numérico para `descripcion`.

## 13. `aliado`

Clave primaria: `nit`, entero. Sus campos públicos son:

| Campo JSON | Tipo | Longitud máxima | POST | PUT | PATCH |
|---|---|---:|:---:|:---:|:---:|
| `nit` | entero | — | Obligatorio | Ruta, no va en el cuerpo | Ruta, no va en el cuerpo |
| `razonSocial` | texto | 60 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `nombreContacto` | texto | 60 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `correo` | texto | 70 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `telefono` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |
| `ciudad` | texto | 45 | Obligatorio | Obligatorio | Opcional, si se modifica |

POST recibe:

```json
{
  "nit": 900123456,
  "razonSocial": "...",
  "nombreContacto": "...",
  "correo": "...",
  "telefono": "...",
  "ciudad": "..."
}
```

PUT recibe obligatoriamente `razonSocial`, `nombreContacto`, `correo`,
`telefono` y `ciudad`, sin `nit` en el cuerpo. PATCH recibe cualquier
subconjunto no vacío de esos cinco campos. No se valida el formato del correo.
GET por clave devuelve `nit`, `razonSocial`, `nombreContacto`, `correo`,
`telefono` y `ciudad`, sin `activo`.

## 14. Reglas comunes de POST, PUT y PATCH

### POST — Crear

POST requiere todos los campos públicos de la tabla, incluida la clave primaria
y excluido `activo`. Ninguna clave se genera automáticamente. Un cuerpo
estructuralmente inválido, la ausencia de un campo obligatorio, un tipo JSON
incompatible o un texto que excede la longitud máxima aplicable responde `422`.

Si falta un campo obligatorio o se envía `null` para un campo `NOT NULL`, se
incluye un mensaje de validación en `errores`. No se establecen longitudes
mínimas ni reglas adicionales de negocio para los campos. Una clave primaria
duplicada responde `500`, conforme a la convención V1.

Una creación correcta responde `200` con:

```json
{
  "estado": 200,
  "mensaje": "Universidad creada exitosamente."
}
```

### PUT — Reemplazo completo

PUT reemplaza todos los campos modificables del recurso. Cada campo modificable
es obligatorio; la clave de la ruta no se incluye como campo del cuerpo ni
cambia. `activo` no es modificable. Si falta cualquier campo modificable, el
resultado es `422`; un valor con tipo JSON incompatible o texto sobre la
longitud máxima también responde `422`. Si la clave no identifica un registro
activo, responde `404`.

El reemplazo correcto responde `200` con:

```json
{
  "estado": 200,
  "mensaje": "Universidad reemplazada.",
  "filasAfectadas": 1
}
```

### PATCH — Actualización parcial

PATCH permite suministrar cualquier subconjunto no vacío de los campos
modificables. Solo cambia los campos incluidos; los campos ausentes conservan
su valor. La clave se toma de la ruta y no se modifica. `activo` no es
modificable.

Un cuerpo vacío (`{}`) o uno que no contenga campos modificables responde `400`.
Un campo suministrado con tipo JSON incompatible, `null` para una columna
`NOT NULL` o texto sobre la longitud máxima aplicable responde `422`. Si la
clave no identifica un registro activo, responde `404`.

La actualización correcta responde `200` con:

```json
{
  "estado": 200,
  "mensaje": "Universidad actualizada.",
  "filasAfectadas": 1
}
```

## 15. PUT frente a PATCH

La diferencia es contractual y se aplica de forma equivalente a los siete
recursos. Por ejemplo, para `aliado`, este cuerpo omite `correo`:

```json
{
  "razonSocial": "...",
  "nombreContacto": "...",
  "telefono": "...",
  "ciudad": "..."
}
```

En `PUT /api/aliado/{nit}`, la ausencia de `correo`, que es obligatorio para el
reemplazo completo, produce `422`. El mismo cuerpo enviado a
`PATCH /api/aliado/{nit}` produce `200` si los campos enviados son válidos,
porque PATCH solo actualiza los campos presentes.

`PATCH /api/aliado/{nit}` con `{}` produce `400` porque no contiene ningún
campo modificable. Estos resultados no determinan otros códigos HTTP.

## 16. Borrado lógico

DELETE es exclusivamente lógico. Para cualquier recurso, la operación actualiza
el registro activo a `activo = 0`; no se ejecuta un DELETE físico funcional.

Si se elimina correctamente, responde `200` con `filasAfectadas: 1`. Si la clave
no existe o el registro ya está inactivo, responde `404`; por ello una segunda
eliminación del mismo registro también responde `404`. Después de eliminarlo,
la fila permanece en la base con `activo = 0`, pero deja de aparecer en GET de
colección y GET por clave.

Mensajes de éxito de escritura por recurso:

| Recurso | POST: creación | PUT: reemplazo | PATCH: actualización | DELETE: borrado lógico |
|---|---|---|---|---|
| `area_conocimiento` | `Área de conocimiento creada exitosamente.` | `Área de conocimiento reemplazada.` | `Área de conocimiento actualizada.` | `Área de conocimiento eliminada.` |
| `universidad` | `Universidad creada exitosamente.` | `Universidad reemplazada.` | `Universidad actualizada.` | `Universidad eliminada.` |
| `aspecto_normativo` | `Aspecto normativo creado exitosamente.` | `Aspecto normativo reemplazado.` | `Aspecto normativo actualizado.` | `Aspecto normativo eliminado.` |
| `practica_estrategia` | `Práctica y estrategia creadas exitosamente.` | `Práctica y estrategia reemplazadas.` | `Práctica y estrategia actualizadas.` | `Práctica y estrategia eliminadas.` |
| `enfoque` | `Enfoque creado exitosamente.` | `Enfoque reemplazado.` | `Enfoque actualizado.` | `Enfoque eliminado.` |
| `car_innovacion` | `Característica de innovación creada exitosamente.` | `Característica de innovación reemplazada.` | `Característica de innovación actualizada.` | `Característica de innovación eliminada.` |
| `aliado` | `Aliado creado exitosamente.` | `Aliado reemplazado.` | `Aliado actualizado.` | `Aliado eliminado.` |

Para una operación de escritura correcta, los cuerpos de PUT, PATCH y DELETE
usan el mensaje correspondiente de la tabla con esta forma:

```json
{
  "estado": 200,
  "mensaje": "Universidad eliminada.",
  "filasAfectadas": 1
}
```

POST usa el mensaje de creación de la tabla en el cuerpo `{ "estado": 200,
"mensaje": "..." }`.

## 17. Matriz mínima de casos contractuales

Cada fila de esta matriz aplica independientemente a cada uno de los siete
recursos. Donde corresponde, `clave` es `id`, salvo para `aliado`, cuya clave
es `nit`.

| # | Solicitud/caso | Resultado |
|---:|---|---|
| 1 | GET de colección con filas activas | `200`, sobre de listado; solo filas activas y sin `activo`. |
| 2 | GET de colección sin filas activas | `204`, sin cuerpo. |
| 3 | GET de colección con `limite <= 0` o no entero | `400`, sobre de error. |
| 4 | GET por clave de un registro activo | `200`, objeto público directo, sin `activo`. |
| 5 | GET por clave inexistente | `404`, sobre de error. |
| 6 | GET por clave inactiva | `404`, igual que una clave inexistente. |
| 7 | POST con todos los campos públicos válidos, incluida la clave | `200`, `{ "estado": 200, "mensaje": "… creado exitosamente." }`; el registro inicia activo. |
| 8 | POST incompleto, de tipo incompatible o con texto sobre el máximo | `422`, sobre de validación con `errores`. |
| 9 | POST con clave primaria duplicada | `500`, sobre de error con detalle de diagnóstico sin secretos. |
| 10 | PUT completo sobre registro activo | `200`, mensaje de reemplazo y `filasAfectadas: 1`. |
| 11 | PUT sin un campo modificable obligatorio | `422`, sobre de validación con `errores`. |
| 12 | PUT sobre registro inexistente o inactivo | `404`, sobre de error. |
| 13 | PATCH con uno o más campos modificables válidos | `200`, mensaje de actualización y `filasAfectadas: 1`; los ausentes no cambian. |
| 14 | PATCH vacío o sin campos modificables | `400`, sobre de error. |
| 15 | PATCH con valor de tipo incompatible, `null` en campo NOT NULL o texto sobre el máximo | `422`, sobre de validación con `errores`. |
| 16 | PATCH sobre registro inexistente o inactivo | `404`, sobre de error. |
| 17 | DELETE sobre registro activo | `200`, mensaje de borrado lógico y `filasAfectadas: 1`; la fila queda con `activo = 0`. |
| 18 | Segundo DELETE o DELETE sobre clave inexistente | `404`; la fila eliminada lógicamente no se borra físicamente. |

La persistencia debe comprobar que DELETE conserva la fila y deja `activo = 0`;
las consultas HTTP normales posteriores deben excluirla y responder `404` al
consultarla por clave.

## 18. Trazabilidad de recursos

| Recurso | Ruta base | Clave | POST | PUT | PATCH | DELETE lógico |
|---|---|---|---|---|---|---|
| `area_conocimiento` | `/api/area_conocimiento` | `id` texto | `id`, `granArea`, `area`, `disciplina` | `granArea`, `area`, `disciplina` | Subconjunto no vacío de los campos PUT | Sí |
| `universidad` | `/api/universidad` | `id` entero | `id`, `nombre`, `tipo`, `ciudad` | `nombre`, `tipo`, `ciudad` | Subconjunto no vacío de los campos PUT | Sí |
| `aspecto_normativo` | `/api/aspecto_normativo` | `id` entero | `id`, `tipo`, `descripcion`, `fuente` | `tipo`, `descripcion`, `fuente` | Subconjunto no vacío de los campos PUT | Sí |
| `practica_estrategia` | `/api/practica_estrategia` | `id` entero | `id`, `tipo`, `nombre`, `descripcion` | `tipo`, `nombre`, `descripcion` | Subconjunto no vacío de los campos PUT | Sí |
| `enfoque` | `/api/enfoque` | `id` entero | `id`, `nombre`, `descripcion` | `nombre`, `descripcion` | Subconjunto no vacío de los campos PUT | Sí |
| `car_innovacion` | `/api/car_innovacion` | `id` entero | `id`, `nombre`, `descripcion`, `tipo` | `nombre`, `descripcion`, `tipo` | Subconjunto no vacío de los campos PUT | Sí |
| `aliado` | `/api/aliado` | `nit` entero | `nit`, `razonSocial`, `nombreContacto`, `correo`, `telefono`, `ciudad` | `razonSocial`, `nombreContacto`, `correo`, `telefono`, `ciudad` | Subconjunto no vacío de los campos PUT | Sí |

En todos los recursos, `activo` queda excluido de POST, PUT, PATCH y de los
objetos públicos de lectura.

## 19. Consumo desde el Frontend

El Frontend consume exactamente estas rutas y representaciones mediante HTTP.
No conoce SQL Server, no utiliza `activo` para realizar un borrado físico y no
comparte clases C# con la API. Debe tratar `204` como colección sin registros,
interpretar los códigos y cuerpos definidos aquí y presentar mensajes
comprensibles para la persona usuaria.

El Frontend no inventa códigos HTTP ni altera el significado de las respuestas.
Los nombres técnicos de métodos, códigos y base de datos no necesitan mostrarse
en los textos visibles de la interfaz.

## 20. Fuera de alcance

Este contrato no define autenticación, JWT, usuarios, roles, relaciones de V2,
reactivación, filtros adicionales, búsqueda, paginación distinta de `limite`,
ordenamiento configurable, validación de formato de correo, generación
automática de identificadores, endpoints genéricos ni funcionalidades de V3 o
V4.

## 21. Aclaraciones pendientes

No se detectaron aclaraciones pendientes para los contratos HTTP de V1.
