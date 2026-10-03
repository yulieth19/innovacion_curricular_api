# Modelo de datos — V1 Innovación Curricular

## 1. Propósito

Este documento fija el modelo de datos que V1 utilizará para sus siete recursos.
Es la referencia para diseñar posteriormente modelos, peticiones, repositorios y
contratos, sin generar aquí código ni SQL.

Se registran el esquema aplicable a V1, las claves y restricciones conocidas,
los datos oficiales disponibles y las correcciones confirmadas respecto del
DDL original. No se deducen columnas, relaciones ni reglas adicionales.

## 2. Fuentes y alcance

El modelo se basa en el DDL oficial del módulo, los datos de referencia
suministrados para el proyecto y las correcciones confirmadas en este documento.
El alcance se limita a:

- `area_conocimiento`
- `universidad`
- `aspecto_normativo`
- `practica_estrategia`
- `enfoque`
- `car_innovacion`
- `aliado`

Estas son las siete tablas sin claves foráneas autorizadas para V1. La base de
datos puede contener otras tablas, pero no se documentan ni habilitan aquí para
uso funcional.

## 3. Correcciones confirmadas respecto del DDL original

| Elemento | DDL original o material | Corrección aplicable a V1 | Motivo |
|---|---|---|---|
| `area_conocimiento.id` | `INT NOT NULL` | `VARCHAR(6) NOT NULL` y clave primaria | Los identificadores oficiales incluyen valores alfanuméricos, por ejemplo `1A01`. |
| `area_conocimiento.disciplina` | `VARCHAR(60) NOT NULL` | `VARCHAR(150) NOT NULL` | Los valores oficiales superan 60 caracteres; el máximo identificado en el material es 124. |
| Campo `activo` | No incluido en el DDL original de las tablas del módulo | En cada tabla V1: `BIT NOT NULL DEFAULT 1` | V1 requiere borrado lógico. |
| Dato de `area_conocimiento` | `Cienias Naturales` | `Ciencias Naturales` | Corrección de calidad del dato documentada en el material. No se aplican otras correcciones no confirmadas. |

## 4. Reglas generales del modelo V1

- Cada definición de recurso especifica columnas, tipos, nulabilidad y clave
  primaria según el modelo confirmado.
- Ninguna de las siete claves tiene propiedad `IDENTITY`. Los identificadores
  forman parte de los datos proporcionados o recibidos según el contrato futuro.
- Las únicas restricciones de unicidad documentadas para estas tablas son sus
  respectivas claves primarias. No se agrega `UNIQUE` a otras columnas o
  combinaciones.
- Las siete tablas V1 no tienen claves foráneas ni relaciones entre sí dentro
  del alcance de esta versión.
- Todas las tablas incluyen `activo BIT NOT NULL DEFAULT 1` para soportar el
  borrado lógico.
- No se infieren restricciones adicionales a partir del significado de los
  nombres de columnas o de los recursos.

## 5. `area_conocimiento`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `VARCHAR(6)` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `gran_area` | `VARCHAR(60)` | NOT NULL | Ninguna adicional documentada |
| `area` | `VARCHAR(60)` | NOT NULL | Ninguna adicional documentada |
| `disciplina` | `VARCHAR(150)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

Datos oficiales disponibles: 218 registros. El identificador es alfanumérico y
la corrección confirmada `Cienias Naturales` a `Ciencias Naturales` se aplica a
los datos de referencia. No se reproducen aquí los registros del catálogo.

## 6. `universidad`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `nombre` | `VARCHAR(60)` | NOT NULL | Ninguna adicional documentada |
| `tipo` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `ciudad` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

Datos oficiales disponibles: 6 registros.

## 7. `aspecto_normativo`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `tipo` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `descripcion` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `fuente` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

No se documentan datos iniciales ficticios.

## 8. `practica_estrategia`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `tipo` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `nombre` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `descripcion` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

No se documentan datos iniciales ficticios.

## 9. `enfoque`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `nombre` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `descripcion` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

No se documentan datos iniciales ficticios.

## 10. `car_innovacion`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `id` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `nombre` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `descripcion` | `VARCHAR(MAX)` | NOT NULL | Ninguna adicional documentada |
| `tipo` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

No se documentan datos iniciales ficticios.

## 11. `aliado`

| Columna | Tipo SQL | Nulabilidad | Restricción / valor predeterminado |
|---|---|---|---|
| `nit` | `INT` | NOT NULL | PRIMARY KEY; no IDENTITY |
| `razon_social` | `VARCHAR(60)` | NOT NULL | Ninguna adicional documentada |
| `nombre_contacto` | `VARCHAR(60)` | NOT NULL | Ninguna adicional documentada |
| `correo` | `VARCHAR(70)` | NOT NULL | Ninguna adicional documentada |
| `telefono` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `ciudad` | `VARCHAR(45)` | NOT NULL | Ninguna adicional documentada |
| `activo` | `BIT` | NOT NULL | DEFAULT 1 |

Puede comenzar sin registros. No se inventan aliados.

## 12. Claves y restricciones

Las claves primarias V1 son:

| Recurso | Clave primaria |
|---|---|
| `area_conocimiento` | `id` |
| `universidad` | `id` |
| `aspecto_normativo` | `id` |
| `practica_estrategia` | `id` |
| `enfoque` | `id` |
| `car_innovacion` | `id` |
| `aliado` | `nit` |

Las claves primarias son las únicas restricciones de unicidad documentadas para
estas siete tablas. No se agrega unicidad para `nombre`, `correo`, `telefono`,
`razon_social`, `ciudad`, `disciplina` ni para ningún otro campo o combinación
de campos.

No se documentan claves foráneas ni relaciones entre las tablas V1. Las tablas
de versiones posteriores pueden referenciarlas, pero esas relaciones quedan
fuera del modelo y del CRUD de V1.

Ninguna clave está marcada como `IDENTITY`; no se presupone generación
automática de identificadores.

## 13. Mapeo conceptual SQL → C#

| Tipo SQL | Representación conceptual C# |
|---|---|
| `VARCHAR(...) NOT NULL` | `string` no nulo |
| `VARCHAR(MAX) NOT NULL` | `string` no nulo |
| `INT NOT NULL` | `int` |
| `BIT NOT NULL` | `bool` |

En particular, `area_conocimiento.id` (`VARCHAR(6)`) se representa como
`string`, y `aliado.nit` (`INT`) como `int`. Este mapeo documenta el modelo para
orientar el diseño posterior; no genera ni define aquí clases C#.

## 14. Borrado lógico

El campo `activo` representa el estado técnico del registro:

- `activo = 1`: registro activo disponible para operaciones normales.
- `activo = 0`: registro eliminado lógicamente.

Las consultas normales deben considerar únicamente registros con `activo = 1`.
El DELETE funcional actualizará el campo a `0`; no se eliminará físicamente la
fila.

La presencia de `activo` no autoriza implementar reactivación de registros en
V1. El campo soporta únicamente la regla de borrado lógico dentro del alcance
aprobado.

## 15. Datos iniciales

| Recurso | Datos iniciales documentados |
|---|---|
| `area_conocimiento` | 218 registros oficiales; se documenta su procedencia y cantidad, no se copian las filas en este documento. Incluye la única corrección confirmada: `Cienias Naturales` a `Ciencias Naturales`. |
| `universidad` | 6 registros oficiales. |
| `aspecto_normativo` | No inventar datos iniciales. |
| `practica_estrategia` | No inventar datos iniciales. |
| `enfoque` | No inventar datos iniciales. |
| `car_innovacion` | No inventar datos iniciales. |
| `aliado` | Puede comenzar vacío; no inventar aliados. |

No se agregan registros ficticios para que una tabla o pantalla parezca tener
contenido.

## 16. Trazabilidad

| Recurso | Clave primaria | Tipo de clave | Activo | Datos iniciales |
|---|---|---|---|---|
| `area_conocimiento` | `id` | `VARCHAR(6)` | `BIT DEFAULT 1` | 218 oficiales |
| `universidad` | `id` | `INT` | `BIT DEFAULT 1` | 6 oficiales |
| `aspecto_normativo` | `id` | `INT` | `BIT DEFAULT 1` | Sin datos oficiales indicados |
| `practica_estrategia` | `id` | `INT` | `BIT DEFAULT 1` | Sin datos oficiales indicados |
| `enfoque` | `id` | `INT` | `BIT DEFAULT 1` | Sin datos oficiales indicados |
| `car_innovacion` | `id` | `INT` | `BIT DEFAULT 1` | Sin datos oficiales indicados |
| `aliado` | `nit` | `INT` | `BIT DEFAULT 1` | Puede iniciar vacío |

## 17. Fuera de alcance

No se documentan modelos detallados ni operaciones para `facultad`, `programa`,
`acreditacion`, `registro_calificado`, `activ_academica`, `pasantia`, `premio`,
tablas puente, usuarios, roles ni otras tablas de V2, V3 o V4. Su posible
existencia en la base de datos no las habilita para uso en V1.

## 18. Aclaraciones pendientes

No se detectaron aclaraciones pendientes del modelo de datos para V1.
