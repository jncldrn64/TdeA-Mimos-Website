# CLAUDE.md: estándar del proyecto

Reglas que valen en cada sesión. Se leen antes de tocar nada. Este archivo fija el
estándar de documentación y formato; el mapa técnico vive en `docs/ARCHITECTURE.md`.

## Documentación

La documentación vive en `docs/`. Los archivos canónicos son tres: `ARCHITECTURE.md`,
`ROADMAP.md` y `DECISIONS.md`. En la raíz quedan dos más: `CHANGELOG.md` y
`README-TESTS-WINDOWS.md` (guía de entorno Windows para correr los tests). No se crea
ningún archivo de documentación nuevo sin preguntar primero.

## CHANGELOG

`CHANGELOG.md` está en la raíz, en formato [Keep a Changelog](https://keepachangelog.com).
Es un solo archivo que crece por secciones, nunca uno por ronda. Lo más nuevo va arriba, en
orden descendente. Cada sección abre con `## vX.Y — YYYY-MM-DD` y adentro lleva
`### Added`, `### Changed`, `### Fixed` o `### Removed`. Todo lo anterior a v1.0 vive en el
historial de git (101 commits hasta 2025-11-13) y no se reconstruye de memoria.

## DECISIONS

`docs/DECISIONS.md` es append-only, estilo ADR. No se borra una entrada vieja aunque quede
obsoleta; se agrega una nueva que la reemplaza y la referencia. Cada entrada abre con
`## YYYY-MM-DD — <título>`. Antes de este registro el porqué de cada decisión vivía en
chats y en un documento de contexto que quedó desactualizado; acá la historia no se
reescribe.

## Fechas

Siempre ISO 8601 (`YYYY-MM-DD`). Nunca formato local.

## Commits

El mensaje es `<tipo>: <resumen imperativo corto>`, con `tipo` en `{add, chg, fix, rmv,
doc}`:

- `add`: nueva capacidad.
- `chg`: cambio de comportamiento.
- `fix`: corrección.
- `rmv`: se sacó algo.
- `doc`: solo documentación.

El cuerpo del commit no vuelve a narrar el cambio: máximo 1 o 2 líneas y una referencia a
la sección del CHANGELOG. El qué vive en el CHANGELOG, el porqué en DECISIONS, no en el
mensaje del commit. Los 101 commits previos usan otros prefijos (`Nuevo:`, `Fix:`,
`Feature:`); esa historia no se normaliza.

## Prosa

Docs y comentarios en español, aplicando dos skills:
`no-ai-slop-writing-rules:no-ai-slop` y `no-ai-slop-writing-rules:rossmann-voice`. Las dos
salen del plugin `no-ai-slop-writing-rules` (realrossmanngroup). No están vendoreadas en el
repo porque el upstream no trae licencia (ver "Vendoreo de dependencias de terceros"). Esas
referencias no resuelven hasta instalar el plugin: se instala por sesión con
`/plugin marketplace add realrossmanngroup/no_ai_slop_writing_rules` y después
`/plugin install no-ai-slop-writing-rules`
(https://github.com/realrossmanngroup/no_ai_slop_writing_rules). Sin relleno, sin frases de
IA. Cada afirmación cierra sobre un dato concreto: un número, una línea de código, una
fecha, un endpoint.

## Guion largo

Guion largo (`—`): prohibido en toda la prosa (regla 1 de no-ai-slop). Se permite
únicamente como token de formato en los encabezados de fecha de CHANGELOG
(`## vX.Y — YYYY-MM-DD`) y DECISIONS (`## YYYY-MM-DD — <título>`). La historia no se
normaliza: lo ya escrito queda como está.

## Honestidad de estado

Nada se declara "funciona" o "probado" sin una corrida real. Los tests de este proyecto
necesitan SQL Server corriendo; si el entorno de la sesión no lo tiene, se dice con esas
palabras en vez de citar resultados viejos como vigentes.

## Flujo de trabajo

Se trabaja vía Pull Request. Si `push`, `branch` o `PR` devuelve `403`, se para y se avisa
que falta permiso de escritura. No se arma una subida manual de archivos sueltos.

## Vendoreo de dependencias de terceros

Cuando se copia una skill, plantilla o cualquier código de terceros a este repo, se copia
también su LICENSE y su atribución, en la misma carpeta. Este repo es público y está bajo
GPL-3.0: no se redistribuye nada sin su nota de licencia. Si la fuente no la trae, se para
y se avisa antes de commitear.

## Scope de escritura

Este repo (TdeA-Mimos-Website) es el único destino de escritura. Cualquier otro
repositorio clonado en la sesión es solo lectura y contexto: se copia DESDE él, nunca se
escribe EN él. No se traen a este repo convenciones de otro (idioma, DECISIONS vs Known
gaps, formato). Ante duda de en qué repo estás escribiendo, se para y se pregunta.

## Versión mostrada

La versión del artefacto (el `<version>` de `pom.xml`) es fuente única con el CHANGELOG:
siempre es la última versión del CHANGELOG. Se bumpea en el mismo PR que trae el cambio de
código que la amerita, nunca en un PR doc-only. Hoy `pom.xml` todavía dice
`0.0.1-SNAPSHOT`; ese desfase está anotado como gap en `docs/ARCHITECTURE.md` §8 y se
cierra en el próximo PR de código.

## Convenciones de código

Las obligatorias, verificadas contra el código el 2026-07-05. El detalle con ejemplos está
en `docs/ARCHITECTURE.md` §7.

- Todo en español: clases, métodos, variables.
- Flujo hexagonal estricto: Controlador → Caso de Uso → Servicio → Adaptador → Puerto →
  Entidad. Un controlador nunca inyecta un repositorio ni un servicio directo.
- Máximo 2 niveles de indentación. Early return y métodos privados antes que un tercer nivel.
- Máximo 5 líneas de comentario por clase.
- Excepciones específicas con `throws`; el manejo vive en `ManejadorGlobalExcepciones`
  (`@ControllerAdvice`), no en try-catch por controlador.
- `@Transactional` en todo método que escribe a la base.
- Logging con SLF4J, no `System.out`.
