# Alcance funcional

HHGG combina una fachada formal con contenido satirico para simular una plataforma de certificaciones. El flujo principal permite explorar certificaciones, resolver un examen, revisar el resultado y descargar un certificado con validacion publica.

## Flujo publico

1. Inicio con catalogo y busqueda de certificaciones.
2. Registro del aspirante por tipo de certificacion.
3. Inicio del examen con reglas de elegibilidad y limite de intentos.
4. Resolucion del quiz con preguntas reordenadas y resultado segun umbral.
5. Emision del certificado con serial unico.
6. Consulta publica por serial y descarga PDF.
7. Verificacion adicional por documento o consulta interna.

## Flujo administrativo

- Gestion de certificaciones activas e inactivas.
- Gestion de preguntas, importacion CSV y exportacion de plantillas.
- Edicion visual de certificados y plantillas.
- Versionado, rollback y comparacion de versiones.
- Revision de registros de importacion y auditoria.
- Gestion de usuarios administradores.

## Funcionalidades visibles

- Busqueda publica y home con listado de certificaciones.
- Evaluacion con preguntas de opcion multiple y verdadero/falso.
- Certificados con serial unico y vista publica verificable.
- Descarga de PDF.
- Selector de idioma.
- Panel admin protegido por acceso administrativo.
- Limpieza y purga automatica de certificados caducados.

## Rutas principales

- `/`
- `/search`
- `/exam/{certType}/register`
- `/exam/start`
- `/exam/{certType}`
- `/result/{serial}`
- `/cert/{serial}`
- `/cert/{serial}/pdf`
- `/admin/login`
- `/admin/certifications`
- `/admin/questions`
- `/api/webhooks/scheduler`

## Idiomas soportados

- `en`
- `es`
- `pt`
- `zh`
- `hi`
- `ar`
- `fr`
