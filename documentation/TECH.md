# Arquitectura, stack y operaciones (TECH)

Este documento amplía información técnica y operativa del proyecto: stack, desarrollo local, despliegue, pruebas, CI, bases de datos, observabilidad y buenas prácticas.

## Stack principal

- Framework: Laravel 11
- Frontend: Livewire 4 + Blade, Tailwind CSS, Vite
- Lenguaje: PHP 8.4
- DB: MySQL o PostgreSQL (según entorno)
- PDF: barryvdh/laravel-dompdf
- Jobs/Cola: configurables (por defecto `sync` en testing)

## Arquitectura y componentes clave

- Modelos: `Certification`, `Question`, `QuestionTranslation`, `Certificate`, `CertificateTemplate`, `CertificationVersion`, `ImportLog`.
- Controladores públicos: home, búsqueda, quiz, certificados, verificación.
- Panel admin: gestión de certificaciones, preguntas, plantillas, import/export, usuarios y dashboard.
- Servicios: scoring, elegibilidad, import/export ZIP, validación CSV, versionado y reglas automáticas.
- Jobs/Comandos: import queue, purga/retención de datos, exportaciones, scheduler commands.

## Desarrollo local

- Requisitos: PHP 8.4, Composer, Node.js (npm/yarn), MySQL/Postgres o Docker.
- Pasos rápidos:

```bash
cp .env.example .env
composer install --no-interaction --prefer-dist
npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
```

- Si usas Docker, ver `docker/` y `docker/start-container.sh` para empezar servicios.

## Ejecutar tests localmente

- Unitarios:

```bash
composer test:unit
```

- Feature (requieren DB/servicios):

```bash
composer test:feature
```

- Ejecutar un fichero o test específico:

```bash
php artisan test --filter="NombreDeTest"
vendor/bin/phpunit tests/Feature/SomeTest.php
```

Nota: ver `phpunit.xml` para variables de entorno recomendadas (DB de testing, cache en array, etc.).

## Integración continua (CI)

- Recomendación de jobs mínimos para GitHub Actions / GitLab CI:
    - `lint` (pint/php-cs-fixer, markdownlint)
    - `test:unit` (rápido, sin servicios externos)
    - `test:feature` (en runner con DB; usar service containers para MySQL)
    - `build_assets` (compilar CSS/JS con Vite para releases)
    - `security_scan` (opcional: Composer audit, SAST)

- Ejemplo: separar `unit` y `feature` en jobs distintos para paralelizar y acelerar feedback.

## Base de datos y migraciones

- Las migraciones se almacenan en `database/migrations`. Use `php artisan migrate --graceful` en despliegues.
- Para pruebas usar DB de testing indicada en `phpunit.xml` (`certificados_test`).
- Seeds para datos de ejemplo en `database/seeders`.

## Import / Export (paquetes ZIP)

- El sistema soporta importación/exportación de certificaciones en paquetes ZIP con `manifest.json`, `questions.csv`, `template.html` y carpeta `assets/`.
- Hay validación previa para evitar corrupciones y mecanismos de rollback para evitar persistir partiales.

## Observabilidad y registros

- Logging: Laravel log (`storage/logs/laravel.log`) y canal configurable vía `config/logging.php`.
- Errores críticos: usar Sentry/LogRocket o similar en producción.
- Monitorización: exportar métricas a Prometheus o usar servicios APM para endpoints críticos (import, export, generación PDF).

## Backups y recuperación

- Hacer backups regulares de la base de datos y de `public/Certificates` (assets exportados).
- Mantener un proceso de restauración documentado y probado en staging.

## Seguridad y buenas prácticas

- Variables sensibles en el entorno `.env`; no subir nunca a VCS.
- Validar y sanitizar archivos ZIP y CSV antes de procesarlos.
- Limitar tamaños de uploads y escanear/limpiar nombres de archivos.
- Usar HTTPS y cabeceras de seguridad (CSP, HSTS, X-Frame-Options).

## Rendimiento

- Cache: usar Redis para cache y sesiones en producción si está disponible.
- Optimizar consultas: revisar N+1 en lista de certificados/preguntas.
- Assets: servir imágenes estáticas desde CDN/objeto storage en producción.

## Operaciones y despliegue

- Imagen Docker / Compose: proveer servicio `php-fpm`, `nginx`, `db` y `runner` para queues.
- Migraciones en despliegue: ejecutar `php artisan migrate --force` en release step.
- Colas: en entornos con tráfico, configurar `queue:work` con supervisor/systemd o usar servicios gestionados.

## Contribuir y estilo de código

- Formato y linting: `composer run-script pint` / `npx eslint` para JS si aplica.
- Tests: PRs deben incluir pruebas unitarias para lógica y tests feature para cambios en endpoints.
- Documentación: actualizar `docs/` y `docs/public-docs/TECH.md` para cambios de alto nivel.

## Comandos útiles

```bash
composer install
composer test:unit
composer test:feature
php artisan migrate --seed
php artisan queue:work
npm run build
```

Si quieres, puedo:

- Añadir un workflow de GitHub Actions que implemente la propuesta de CI.
- Documentar cada test en `tests/Unit` como hicimos con `tests/Feature`.
