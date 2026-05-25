# Sistema de versionado de certificaciones

## Resumen

El sistema de versionado guarda snapshots de una certificación cada vez que se actualiza y también cuando se intenta eliminar. La implementación real está en:

- [app/Observers/CertificationObserver.php](app/Observers/CertificationObserver.php)
- [app/Support/CertificationVersioningService.php](app/Support/CertificationVersioningService.php)
- [app/Models/CertificationVersion.php](app/Models/CertificationVersion.php)
- [app/Http/Controllers/Admin/CertificationAdminController.php](app/Http/Controllers/Admin/CertificationAdminController.php)

No existe una entidad de eventos separada para este flujo en el código actual. El versionado se activa desde el observer del modelo.

## Qué guarda una versión

Cada registro en `certification_versions` guarda:

- `certification_id`: certificación asociada
- `version_number`: número incremental por certificación
- `snapshot`: estado completo de la certificación en el momento del guardado
- `questions_snapshot`: snapshot de las preguntas y sus traducciones
- `change_reason`: motivo textual del cambio
- `changes`: diferencias detectadas respecto a la versión anterior
- `created_at` y `updated_at`
- `deleted_at` por soft delete

El esquema real de la tabla está en [database/migrations/2026_04_07_000700_create_certification_versions_table.php](database/migrations/2026_04_07_000700_create_certification_versions_table.php).

## Cómo se crea una versión

La creación automática ocurre en `CertificationObserver::updated()`.

Flujo actual:

1. Se actualiza la certificación.
2. El observer comprueba `getChanges()`.
3. Si hay cambios, llama a `CertificationVersioningService::createVersion()`.
4. Se genera un `version_number` nuevo sumando 1 al último existente.
5. Se guardan `snapshot`, `questions_snapshot`, `change_reason` y `changes`.

Si el update no deja cambios reales, no se crea una versión.

## Eliminación

Cuando se intenta eliminar una certificación, el observer también crea una versión con el motivo `Certificación eliminada` antes de que el modelo desaparezca.

Además, la eliminación del modelo `Certification` borra preguntas, traducciones, plantillas, certificados, estadísticas, logs y versiones asociadas mediante el hook `booted()` del modelo.

## Restauración de una versión

La restauración está disponible desde el admin:

- Vista de historial: la ruta existe como `/admin/certifications/{certification}/versions` y está definida en [routes/web.php](routes/web.php).
- Acción de rollback: `POST /admin/certifications/{certification}/versions/{version}/rollback`

El controlador real es `CertificationAdminController::rollbackVersion()`.

El flujo es:

1. Se busca la versión dentro de la certificación.
2. `CertificationVersioningService::rollbackToVersion()` aplica el `snapshot` guardado sobre la certificación.
3. Se crea una nueva versión registrando el rollback.

Es decir, un rollback no reescribe historial: genera una nueva versión adicional.

## Historial en la interfaz

La vista de historial está en:

- [resources/views/admin/certifications/versions.blade.php](resources/views/admin/certifications/versions.blade.php)
- [resources/views/admin/certifications/\_versions.blade.php](resources/views/admin/certifications/_versions.blade.php)

La UI muestra:

- número de versión
- motivo del cambio
- fecha de creación
- estado actual de la versión
- acción de restauración para versiones anteriores

## Comparación de versiones

Existe un endpoint de comparación en la API admin:

- `GET /admin/api/certifications/{id}/versions/{versionId}/compare?to={toVersionId}`

Implementación:

- [app/Http/Controllers/Api/CertificationApiController.php](app/Http/Controllers/Api/CertificationApiController.php)

La comparación revisa estos campos:

- `name`
- `description`
- `questions_required`
- `pass_score_percentage`
- `cooldown_days`
- `result_mode`
- `active`
- `pdf_view`
- `settings`

Si no se pasa `to`, la comparación se hace contra el estado actual de la certificación.

## Estructura del modelo

El modelo de versión es [app/Models/CertificationVersion.php](app/Models/CertificationVersion.php).

Campos fillable:

- `certification_id`
- `version_number`
- `snapshot`
- `questions_snapshot`
- `change_reason`
- `changes`

Relación principal:

- `CertificationVersion::certification()`

Helpers útiles:

- `getChangeSummary()` devuelve `changes`
- `getChangedFields()` devuelve las claves modificadas

## Límites y observaciones

- No existe `created_by_user_id` en la tabla real, aunque versiones antiguas del doc lo mencionaban.
- No existe `data` ni `metadata` como columnas separadas; el contrato real usa `snapshot`, `questions_snapshot`, `change_reason` y `changes`.
- No hay un endpoint REST de listado de versiones separado; el listado se consume desde la vista admin.
- Las versiones usan soft deletes, así que no se eliminan físicamente por defecto.
- Los tests actuales cubren creación automática, incremento de número, historial, rollback y comparación.

## Pruebas relacionadas

- [tests/Feature/AdminCertificationVersioningTest.php](tests/Feature/AdminCertificationVersioningTest.php)
- [tests/Feature/AdminCertificationApiTest.php](tests/Feature/AdminCertificationApiTest.php)
- [tests/Feature/AdminCertificationEditTest.php](tests/Feature/AdminCertificationEditTest.php)

## Resumen operativo

En la práctica, el sistema ofrece:

- historial automático de cambios en certificaciones
- número de versión incremental por certificación
- rollback con creación de nueva versión
- comparación entre versiones o contra el estado actual
- snapshot de preguntas y traducciones para auditoría
