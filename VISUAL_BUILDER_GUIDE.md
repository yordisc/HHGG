# Visual Builder Guide - Certification Management

## Overview

Esta guia describe la experiencia visual real para administrar certificaciones en HHGG. La interfaz no es un editor generico: es una pantalla de edicion de certificacion con acciones concretas, validacion en vivo del banco de preguntas, historial de versiones y un asistente de creacion por pasos.

## Dónde se usa

- Edicion de certificacion: `/admin/certifications/{certification}/edit`
- Historial de versiones: `/admin/certifications/{certification}/versions`
- Asistente de creacion: `/admin/certifications/wizard/{step?}`
- Borradores del asistente: `/admin/certifications-wizard/drafts`
- Reinicio del asistente: `/admin/certifications/wizard-reset`

La pantalla de administracion usa middleware de administrador y sesion activa de usuario admin.

## Pantalla de edicion

La vista principal esta en [resources/views/admin/certifications/edit.blade.php](resources/views/admin/certifications/edit.blade.php).

### Acciones disponibles

- Guardar cambios de la certificacion.
- Volver al listado.
- Probar funcionamiento de la certificacion.
- Editar la plantilla del certificado.
- Ver historial de versiones.
- Clonar la certificacion.
- Generar preguntas de prueba si el banco es insuficiente.
- Borrar preguntas de prueba.

### Campos principales

- Slug.
- Nombre.
- Descripcion.
- Preguntas requeridas.
- Porcentaje de aprobacion.
- Dias de cooldown.
- Modo de resultado.
- Vista PDF.
- Orden en el home.
- Imagen destacada.
- Settings JSON.

### Configuracion de salida

El campo `result_mode` permite elegir entre:

- `binary_threshold`: resultado por umbral de aprobacion.
- `custom`: resultado con reglas o llaves personalizadas.
- `generic`: resultado simple.

El campo `pdf_view` indica que vista Blade genera el certificado PDF, por ejemplo `pdf.certificate`.

## Panel de preguntas

El panel lateral esta en [resources/views/admin/certifications/\_questions-panel.blade.php](resources/views/admin/certifications/_questions-panel.blade.php).

### Que muestra

- Cantidad de preguntas activas.
- Cantidad de preguntas requeridas.
- Estado de validacion.
- Lista de preguntas activas.

### Como funciona

- Se carga por AJAX desde `/admin/api/certifications/{id}/available-questions`.
- Se actualiza cuando cambia el valor de `questions_required`.
- Si hay menos preguntas activas que las requeridas, la interfaz muestra un aviso de insuficiencia.

### Que debe saber el usuario

- Solo cuentan las preguntas activas.
- Si el banco no alcanza, no conviene publicar la certificacion todavia.
- El boton de recarga permite volver a consultar el estado del banco.

## Historial de versiones

El bloque de versiones esta en [resources/views/admin/certifications/\_versions.blade.php](resources/views/admin/certifications/_versions.blade.php).

### Que ofrece

- Lista de versiones guardadas.
- Fecha y motivo del cambio.
- Visualizacion del detalle de cambios por campo.
- Opcion de restaurar una version anterior.

### Cuando usarlo

- Cuando una certificacion cambia de forma importante.
- Cuando necesitas volver a un estado anterior.
- Cuando quieres revisar que cambio entre revisiones.

## Asistente de creacion

El flujo de creacion redirige desde `create()` al asistente por pasos. El asistente usa borradores guardados en sesion y puede reanudar el flujo si el usuario sale antes de terminar.

### Flujo general

1. Datos basicos.
2. Configuracion de evaluacion.
3. Configuracion de presentacion.
4. Imagen y opciones avanzadas.
5. Revision final y creacion.

### Funciones del asistente

- Guardado automatico del borrador.
- Reanudacion del draft por sesion.
- Reinicio manual del borrador.
- Lista de borradores activos.

### Resultado final

Al completar el paso final, el sistema crea la certificacion y redirige a la vista de edicion.

## Diagnostico rapido

La vista de edicion usa un resumen automatico de validacion para mostrar si la certificacion esta lista o si requiere atencion. Ese resumen ayuda a detectar:

- Falta de preguntas activas.
- Configuracion inconsistente.
- Valores fuera de rango.

## Que no hace esta interfaz

- No es un constructor visual de paginas libre.
- No expone atajos de teclado especiales.
- No depende de una API publica para editar la certificacion.
- No reemplaza el panel de preguntas ni el historial; los complementa.

## Resumen practico

Si quieres entender el editor, piensa en tres capas:

- La edicion central de la certificacion.
- El panel lateral que valida el banco de preguntas.
- El historial que permite revisar y restaurar versiones.

Esa es la experiencia visual real que HHGG expone hoy.
