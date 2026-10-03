# Especificación funcional — V1 Innovación Curricular

## 1. Propósito de V1

V1 entrega una API y un Frontend funcionales para administrar los siete recursos
sin claves foráneas definidos en este documento. Esta especificación establece
qué comportamiento funcional debe ofrecer la versión; la constitución del
proyecto y el mapa de versiones siguen siendo obligatorios.

Los contratos HTTP exactos, incluidos rutas, cuerpos JSON, formatos de
respuesta, códigos HTTP, validaciones particulares y límites de consulta, se
documentarán posteriormente en `6_contracts.md`. Esta especificación no los
sustituye ni los anticipa.

## 2. Alcance

V1 comprende el CRUD completo mediante API y Frontend para exactamente siete
recursos: `area_conocimiento`, `universidad`, `aspecto_normativo`,
`practica_estrategia`, `enfoque`, `car_innovacion` y `aliado`.

Para cada recurso, las personas usuarias podrán listar registros activos,
consultar un registro activo, crear, reemplazar completamente, actualizar
parcialmente y eliminar lógicamente registros. Los resultados de consultas
normales excluirán registros inactivos.

La base de datos puede contener otras tablas, pero la API y el Frontend de V1
no implementarán CRUD ni funcionalidades para recursos fuera de esta lista.

## 3. Recursos incluidos

Los únicos recursos incluidos son:

1. `area_conocimiento`
2. `universidad`
3. `aspecto_normativo`
4. `practica_estrategia`
5. `enfoque`
6. `car_innovacion`
7. `aliado`

## 4. Reglas funcionales comunes

- Cada recurso permite consultar una colección de sus registros activos. La
  colección puede estar vacía y esa ausencia legítima de registros debe poder
  representarse sin fabricar datos.
- Cada recurso permite consultar individualmente un registro activo. Un registro
  inexistente o inactivo no está disponible en las operaciones normales de
  consulta.
- Cada recurso permite crear un registro que cumpla los requisitos aplicables,
  reemplazar completamente un registro existente mediante `PUT`, actualizarlo
  parcialmente mediante `PATCH` y eliminarlo lógicamente.
- El reemplazo completo debe cumplir los requisitos del registro completo. Un
  reemplazo incompleto no sustituye el registro existente.
- Una actualización parcial cambia únicamente los datos incluidos en la
  solicitud y conserva los demás datos existentes.
- Una actualización parcial debe incluir al menos un campo modificable. Una
  solicitud que no contenga ningún cambio se rechaza; el código HTTP exacto se
  definirá en `6_contracts.md`.
- V1 no inventa reglas adicionales de unicidad. Un registro solo se considera
  duplicado cuando entra en conflicto con una clave primaria, una restricción
  `UNIQUE` u otra restricción de unicidad existente en el esquema oficial. No
  se impide repetir valores para campos o combinaciones de campos para los que
  el esquema oficial no establezca unicidad. Las claves y restricciones de cada
  recurso se documentarán en `5_data_model.md`; el comportamiento HTTP ante
  una violación se definirá en `6_contracts.md`.
- El borrado funcional marca el registro como inactivo (`activo = 0`); nunca
  elimina físicamente la fila. Después del borrado, el registro deja de
  aparecer en listados y consultas normales.
- Un registro inexistente o inactivo no puede modificarse ni eliminarse como si
  estuviera disponible.
- Los requisitos por campo y la representación de cada operación se concretarán
  en los documentos de datos y contratos de V1, a partir de las fuentes oficiales
  del proyecto. No se deducen columnas, tipos, límites ni reglas particulares en
  esta especificación.
- El comportamiento de cada operación ante errores se documentará en los
  contratos. No se fijan aquí códigos HTTP ni formatos de error.

## 5. Comportamiento esperado por recurso

La matriz de operaciones y escenarios de la sección 8 aplica por separado a cada
uno de los siguientes recursos. No se establecen relaciones funcionales entre
ellos en V1.

### `area_conocimiento`

Debe permitir todas las operaciones comunes de V1. Representa un catálogo
jerárquico relacionado con gran área, área y disciplina. El material de
referencia informa 218 registros y contempla identificadores alfanuméricos,
como `1A01`; no deben suponerse identificadores enteros. La disciplina debe
poder representar los valores reales del catálogo. La estructura exacta de
columnas, tipos y capacidad se documentará a partir del esquema y los datos
oficiales.

### `universidad`

Debe permitir todas las operaciones comunes de V1 para el catálogo de
universidades/sedes definido por el proyecto. El material de referencia incluye
seis registros, cuya fuente oficial debe respetarse. No se especifican aquí sus
campos ni su forma de presentación.

### `aspecto_normativo`

Debe permitir todas las operaciones comunes de V1. No se agregan registros
iniciales ficticios: solo se cargarán datos si existe una fuente oficial del
proyecto.

### `practica_estrategia`

Debe permitir todas las operaciones comunes de V1. No se agregan registros
iniciales ficticios: solo se cargarán datos si existe una fuente oficial del
proyecto.

### `enfoque`

Debe permitir todas las operaciones comunes de V1. No se agregan registros
iniciales ficticios: solo se cargarán datos si existe una fuente oficial del
proyecto.

### `car_innovacion`

Debe permitir todas las operaciones comunes de V1. No se agregan registros
iniciales ficticios: solo se cargarán datos si existe una fuente oficial del
proyecto.

### `aliado`

Debe permitir todas las operaciones comunes de V1 y puede comenzar sin
registros. No se inventan aliados ni se agregan datos para llenar la tabla o la
interfaz.

## 6. Frontend de V1

V1 incluye una interfaz funcional para cada uno de los siete recursos. Desde la
interfaz, una persona debe poder realizar las operaciones CRUD previstas sin
usar Swagger ni escribir solicitudes HTTP manualmente.

La interfaz debe presentar los resultados y las acciones en lenguaje
comprensible para las personas usuarias. No debe exponer innecesariamente
métodos HTTP, códigos de estado, SQL Server ni nombres internos de
implementación.

El Frontend obtiene y modifica los datos exclusivamente mediante HTTP a través
de la API. No se conecta a SQL Server, no recibe una cadena de conexión y no
comparte proyectos ni clases C# con la API. API y Frontend intercambian
únicamente el contrato JSON de V1.

Si la API no está disponible, el Frontend no presenta datos inventados ni los
representa como si se hubieran obtenido correctamente. La forma concreta de
informar y gestionar ese error se definirá en los documentos posteriores de
V1.

La interfaz debe poder representar una colección legítimamente vacía y los
catálogos que no tengan datos iniciales, sin insertar registros ficticios.

## 7. Datos iniciales y catálogos

- Se respetan los datos oficiales suministrados para los catálogos cuando
  existan y su fuente esté disponible en el proyecto.
- Para `area_conocimiento`, se tienen como referencia los 218 registros del
  Excel del módulo, incluidos identificadores que pueden ser alfanuméricos y
  los valores reales de disciplina.
- Para `universidad`, se tienen como referencia los seis registros de
  universidades/sedes del material del módulo.
- No se inventan datos iniciales para `aspecto_normativo`,
  `practica_estrategia`, `enfoque` ni `car_innovacion` si el material del
  proyecto no proporciona una fuente oficial.
- `aliado` puede comenzar sin registros; no se crean aliados ficticios.
- La ausencia legítima de datos debe ser manejada correctamente por API y
  Frontend.
- La identificación exacta de las fuentes, columnas y tipos se documentará en
  `5_data_model.md`. Este apartado no autoriza a inferir columnas ni tipos a
  partir de los nombres de los recursos o de este resumen.

## 8. Escenarios funcionales

Los escenarios siguientes se ejecutan **independientemente para cada uno de
los siete recursos**. Los resultados observables en HTTP y sus códigos exactos
se definirán en `6_contracts.md`.

1. **Listar registros activos:** al consultar la colección, se muestran los
   registros activos del recurso y no se incluyen los inactivos.
2. **Listar colección vacía:** cuando no hay registros activos, API y Frontend
   representan la colección vacía como un estado válido; no crean datos para
   simular contenido.
3. **Consultar un registro activo:** al solicitar un registro existente y
   activo, se obtiene su información.
4. **Consultar un registro inexistente o inactivo:** el registro no se presenta
   como disponible en la consulta normal.
5. **Crear un registro válido:** se incorpora el registro conforme a los
   requisitos aplicables y queda disponible para las consultas normales.
6. **Crear un registro inválido:** si no satisface los requisitos obligatorios
   aplicables, no se incorpora como un registro válido.
7. **Reemplazar completamente un registro existente:** la operación reemplaza
   el contenido conforme a los requisitos del registro completo.
8. **Intentar un reemplazo incompleto:** no se acepta como reemplazo completo y
   no deja el registro en un estado parcialmente reemplazado.
9. **Actualizar parcialmente un registro existente:** se aplican los cambios
   solicitados y se conservan los datos no incluidos en la actualización.
10. **Intentar una actualización parcial sin cambios:** se rechaza una
    solicitud que no incluya ningún campo modificable. El código HTTP concreto
    se define en `6_contracts.md`.
11. **Eliminar lógicamente un registro existente:** el registro se marca
    inactivo, sin borrado físico.
12. **Verificar después del borrado:** el registro eliminado lógicamente deja
    de aparecer en listados y consultas normales.
13. **Operar sobre un registro inexistente o inactivo:** la API y la interfaz no
    lo tratan como un registro disponible para consulta, modificación o
    eliminación. La respuesta concreta se define en los contratos.
14. **Operar desde el Frontend:** una persona puede consultar y realizar las
    operaciones CRUD del recurso desde la interfaz, sin Swagger ni solicitudes
    manuales, y ve datos provenientes de la API.

## 9. Fuera de alcance

No forman parte de V1:

- CRUD de tablas con claves foráneas ni relaciones funcionales reservadas para
  V2;
- autenticación JWT, inicio de sesión, sesiones, roles, administración de
  usuarios o permisos, reservados para V3;
- dashboard, consultas multitabla de V4, páginas corporativas, PWA y publicación
  final;
- requisitos de identidad visual que no hayan sido adoptados expresamente para
  este proyecto;
- cualquier otro recurso o funcionalidad que no esté incluido en el alcance de
  V1, aunque su tabla exista en la base de datos.

## 10. Criterios de aceptación

Los siguientes criterios permanecen **pendientes de verificación**; ninguno se
marca como cumplido en esta especificación:

- [ ] Los siete recursos tienen CRUD completo en la API.
- [ ] Los siete recursos pueden utilizarse desde el Frontend.
- [ ] El Frontend obtiene y modifica datos exclusivamente a través de la API.
- [ ] El borrado de los siete recursos es lógico y marca `activo = 0`.
- [ ] Los registros inactivos no aparecen en consultas normales.
- [ ] No se implementan recursos de V2, V3 o V4 en V1.
- [ ] Se respetan los datos oficiales disponibles.
- [ ] No se inventan datos para catálogos sin fuente.
- [ ] Pasan los criterios de aceptación específicos que se documenten para V1.
- [ ] Pasan las verificaciones de regresión definidas para V1.
- [ ] Se cumplen los demás requisitos de esta especificación y los contratos
      aprobados para V1.

## 11. Aclaraciones pendientes

No se detectaron aclaraciones funcionales pendientes con la información
disponible para esta etapa.

## 12. Condición de cierre de V1

V1 no está completa hasta que todos sus criterios de aceptación y los criterios
específicos definidos para la versión se hayan verificado para API y Frontend,
las verificaciones de regresión hayan pasado y las aclaraciones funcionales
necesarias se hayan resuelto en los documentos correspondientes.

Después de superar las compuertas de la versión, V1 se integra a `main` mediante
Pull Request y se etiqueta `v1`. La integración y la etiqueta ocurren después
de cumplir los criterios; hasta entonces V1 no se considera cerrada.
