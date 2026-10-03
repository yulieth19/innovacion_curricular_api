# Mapa de versiones — Innovación Curricular

## Propósito

Este documento organiza el alcance general del proyecto de aula Innovación
Curricular en cuatro versiones incrementales. Sirve para distinguir qué
corresponde a cada versión y evitar que una implementación adelante trabajo
reservado para una versión futura.

El proyecto está compuesto por dos repositorios independientes:

- `innovacion_curricular_api`
- `innovacion_curricular_front`

El Spec Kit se mantiene únicamente en el repositorio de la API. Cada versión,
sin embargo, contempla el trabajo de API y Frontend que establezca su
especificación. La constitución del proyecto, en
`docs/spec_kit/1_constitution.md`, es obligatoria y prevalece sobre este mapa.

## Resumen

| Versión | Objetivo general | Alcance de recursos o capacidades | Estado inicial |
|---|---|---|---|
| V1 | CRUD de tablas sin claves foráneas | API y Frontend para siete recursos definidos en este mapa | Especificación en preparación |
| V2 | Incorporar tablas con claves foráneas y sus relaciones | El alcance exacto se definirá en el Spec Kit de V2 | Pendiente |
| V3 | Incorporar seguridad y administración de usuarios | JWT, sesiones, roles y administración de usuarios, según su futura especificación | Pendiente |
| V4 | Integración y presentación final | Consultas multitabla, dashboard, páginas corporativas, responsive/PWA, identidad visual y publicación, según su futura especificación | Pendiente |

Ninguna versión se considera completada por aparecer en este mapa. Su estado
solo cambia mediante el proceso de cierre definido más adelante.

## V1 — CRUD de tablas sin claves foráneas

**Objetivo:** implementar API y Frontend para el CRUD completo de exactamente
estos siete recursos:

- `area_conocimiento`
- `universidad`
- `aspecto_normativo`
- `practica_estrategia`
- `enfoque`
- `car_innovacion`
- `aliado`

El alcance contractual del CRUD se especificará en
`docs/spec_kit/versiones/v1_innovacion_curricular/6_contracts.md`. El borrado
será lógico mediante `activo = 0`, y las consultas normales excluirán los
registros inactivos, de acuerdo con la constitución.

Este mapa no define campos, endpoints, códigos HTTP, validaciones, datos semilla
ni detalles de interfaz. Esos aspectos corresponden a los documentos
específicos de V1.

**Estado inicial:** especificación en preparación.

## V2 — Tablas con claves foráneas

**Objetivo:** incorporar las tablas del módulo que tienen claves foráneas y las
relaciones correspondientes.

Este mapa no enumera tablas, diseña relaciones ni define sus operaciones. El
alcance de API y Frontend, así como las tablas autorizadas para uso, se
establecerá exclusivamente en el futuro Spec Kit de V2. Hasta que esa
especificación lo determine, no se implementan tablas ni funcionalidades de
V2.

**Estado inicial:** pendiente.

## V3 — Seguridad y usuarios

**Objetivo:** incorporar autenticación JWT, sesiones, roles y administración de
usuarios.

La especificación futura de V3 definirá el alcance correspondiente para API y
Frontend. Este mapa no diseña endpoints, claims, permisos, tablas ni detalles
de implementación de seguridad. Ninguna de estas capacidades se anticipa en
V1 o V2.

**Estado inicial:** pendiente.

## V4 — Integración y presentación final

**Objetivo:** abordar, según la futura especificación, consultas multitabla,
dashboard, páginas corporativas, responsive/PWA, identidad visual y
publicación.

El alcance específico de API y Frontend se definirá en el futuro Spec Kit de
V4. Este mapa no diseña ni autoriza anticipadamente la implementación de esas
funcionalidades. La identidad visual y cualquier decisión de publicación
dependerán de lo que establezca esa especificación.

**Estado inicial:** pendiente.

## Dependencias entre versiones

Las versiones siguen una evolución incremental: V2 se construye sobre la base
entregada por V1; V3, sobre la base de las versiones anteriores; y V4, sobre la
base acumulada hasta V3. Cada versión conserva su propio alcance y su propia
especificación; la dependencia de una versión anterior no habilita adelantar
funcionalidades de una posterior.

La existencia completa de la base de datos desde V1 no cambia esta secuencia ni
amplía lo que el código puede utilizar. En cada versión, el código solo puede
utilizar las tablas habilitadas por la especificación vigente.

## Regla de no anticipación

Durante una versión solo se implementa el alcance aprobado en su especificación
vigente. Las capacidades descritas para versiones posteriores son una guía de
organización, no requisitos de la versión actual. No se agregan por adelantado
tablas, relaciones, campos, validaciones, endpoints, pantallas ni otras
funcionalidades futuras.

Si una necesidad parece exceder el alcance de la versión activa, no se resuelve
por suposición ni se incorpora como adelanto: debe quedar para la especificación
que corresponda, de acuerdo con la constitución.

## Regla de cierre de versión

Una versión solo se cierra después de cumplir todos sus criterios de aceptación
para la API y el Frontend que incluya su especificación. Cumplidos esos
criterios, los cambios se integran a `main` mediante Pull Request y se crea la
etiqueta Git correspondiente (`v1`, `v2`, `v3` o `v4`). Hasta completar el
proceso, la versión no se marca como cerrada ni como completada.

## Estados iniciales

- **V1:** especificación en preparación.
- **V2:** pendiente.
- **V3:** pendiente.
- **V4:** pendiente.

No hay versiones completadas al momento de crear este mapa.