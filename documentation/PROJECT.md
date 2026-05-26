# Alcance funcional

HHGG combina una fachada formal con contenido satírico para simular una plataforma de certificaciones. El flujo principal permite explorar certificaciones, resolver exámenes, revisar resultados y descargar certificados con verificación pública.

## Flujo público (resumen)

1. Inicio: catálogo, filtros y búsqueda de certificaciones.
2. Registro/identificación del aspirante según tipo de certificación.
3. Inicio del examen: comprobación de elegibilidad y límites de intentos.
4. Resolución del quiz: preguntas aleatorizadas, opciones mezcladas y reglas de scoring.
5. Emisión del certificado: generación de serial único y firma/hasheo para verificación.
6. Consulta pública por serial y descarga de PDF.
7. Verificación adicional por documento o API interna.

## Flujo administrativo (resumen)

- Gestión completa de certificaciones: crear, editar, activar/desactivar y reordenar.
- Gestión de preguntas: CRUD, importación/exportación CSV y herramientas de generación de preguntas de prueba.
- Plantillas de certificado: edición HTML/CSS, previews y plantillas por certificación.
- Versionado: snapshots, comparación y rollback de configuraciones.
- Auditoría: logs de importación y trazabilidad de cambios.
- Gestión de usuarios y roles administrativos.

## Funcionalidades destacadas

- Búsqueda y listado público de certificaciones.
- Motor de quiz: MCQ, True/False, sudden-death y scoring ponderado.
- Certificados verificables públicamente (serial + token de verificación).
- Exportación e importación de paquetes ZIP (`manifest.json`, `questions.csv`, `assets/`).
- Soporte multilenguaje y traducciones por pregunta.
- Panel admin con vistas y API privadas protegidas.

## Rutas principales

- `/` — home / catálogo
- `/search` — busqueda pública
- `/exam/{certType}/register` — registro para examen
- `/exam/start` — iniciar examen
- `/exam/{certType}` — vista del examen
- `/result/{serial}` — resultado público
- `/cert/{serial}` — vista pública certificado
- `/cert/{serial}/pdf` — descarga PDF
- `/admin/*` — panel admin (login, certificaciones, preguntas, templates)
- `/api/*` — endpoints internos y webhooks

## Idiomas soportados

- en, es, pt, zh, hi, ar, fr

## Resumen técnico

- Framework: Laravel 11 con Livewire 4 y Blade.
- PHP 8.4, Vite para frontend, Tailwind CSS.
- Base de datos: MySQL o PostgreSQL (según entorno).
- Dependencias clave: barryvdh/laravel-dompdf, livewire, paquetes de testing (phpunit).

## Estructura del repositorio (relevante)

- `app/` — código de aplicación (Models, Http, Services, Jobs, Actions).
- `tests/` — pruebas `Feature` y `Unit` (ver `docs/public-docs/TEST.md`).
- `docs/` — documentación, fixtures y guías.
- `public/Certificates/` — assets públicos de certificaciones y plantillas exportadas.
- `database/` — migraciones, seeders y factories.

## Desarrollo local (rápido)

1. Copia el entorno y configura variables:

```bash
cp .env.example .env
```

2. Instala dependencias y prepara la base de datos:

```bash
composer install
npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
```

3. Si usas Docker: consultar `docker/` y `docker/start-container.sh`.

## Variables de entorno relevantes

- `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` — DB de la app.
- `APP_ENV`, `APP_DEBUG`, `APP_URL` — entorno de ejecución.
- `ADMIN_PRIMARY_*` — provisión del admin primario.
- `QUEUE_CONNECTION` — configurar `sync` para testing o `database/redis` en producción.
- `MAIL_MAILER` — `array` para testing; configurar SMTP para producción.

## Ejecutar tests

- Unitarios (rápidos):

```bash
composer test:unit
```

- Feature (requieren DB y potencialmente otros servicios):

```bash
composer test:feature
```

- Ejecutar un test concreto:

```bash
php artisan test --filter="NombreDeTest"
vendor/bin/phpunit tests/Unit/CertificationScoringServiceTest.php
```

## Import / Export de paquetes ZIP

- Formato esperado: `manifest.json`, `questions.csv`, `template.html`, `assets/`.
- Validaciones: `ZipManifestValidator` y comprobaciones anti path-traversal.
- Uso típico: admin -> exportar paquete -> import en otra instancia o restauración.

## Despliegue y operaciones

- Recomendación mínima: Docker + Nginx + PHP-FPM + MySQL/Postgres.
- Paso de despliegue (ejemplo):

```bash
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
npm run build
```

- Colas: en producción usar `php artisan queue:work` o un worker gestionado.
- Scheduler: configurar cron para `php artisan schedule:run` cada minuto.

## Observabilidad y backups

- Logs en `storage/logs/laravel.log`; considerar Sentry/OPentelemetry para errores.
- Backups regulares de BD y `public/Certificates`.

## Seguridad

- Nunca versionar `.env` ni claves secretas.
- Validar y sanitizar uploads (ZIP, imágenes, CSV).
- Limitar tamaño de subida y aplicar controles de rates en endpoints públicos (quiz, búsqueda).

## Contribución

- Estilo de código: ejecutar `composer run-script pint` antes de subir PRs.
- Tests: añadir tests unitarios y feature para cambios de comportamiento.
- Documentación: actualizar `docs/` y `docs/public-docs/TECH.md` cuando se cambie arquitectura.

## Puntos abiertos / roadmap

- Integrar pruebas E2E visuales (Playwright) para regresiones de UI.
- Añadir CI que separe `unit` y `feature` jobs y capture screenshots en fallos.
- Mejora opcional: mover assets estáticos a CDN/objeto storage para reducir I/O.

---

Si quieres que expanda alguna sección (despliegue Docker-compose, workflow CI, o documentación por test individual), dímelo y lo incluyo.
