# Architecture & Stack — HHGG

Technical reference for the stack, local setup, testing, CI, deployment, and operational practices.

---

## Stack

| Layer | Technology |
|---|---|
| Framework | Laravel 11 |
| Frontend | Livewire 4 · Blade · Tailwind CSS · Vite |
| Language | PHP 8.4 |
| Database | MySQL or PostgreSQL (env-switchable) |
| PDF generation | barryvdh/laravel-dompdf |
| Queues | Laravel Queue — `sync` for dev, `database`/`redis` for production |
| Cache / Sessions | Redis (Upstash in production) |
| Container | Docker · Nginx · PHP-FPM |
| Deploy target | Render (web service) |

---

## Architecture overview

**Models**
`Certification` · `Question` · `QuestionTranslation` · `Certificate` · `CertificateTemplate` · `CertificationVersion` · `ImportLog`

**Controllers (public)**
Home · Search · Quiz · Certificates · Verification

**Controllers (admin)**
Certifications · Questions · Templates · Import/Export · Users · Dashboard

**Services**
| Service | Responsibility |
|---|---|
| `WeightedScoringService` | Scoring with per-question weights |
| `SuddenDeathRuleService` | Sudden-death pass/fail rule enforcement |
| `CertificationVersioningService` | Snapshot creation, rollback, diffing |
| `CertificationZipImporterService` | ZIP import pipeline with queue dispatch |
| `CertificationZipExporterService` | ZIP export with asset bundling |
| `CsvValidator` / `QuestionsCsvValidator` | CSV validation with multilingual column support |
| `ZipManifestValidator` | Path-traversal-safe manifest validation |

**Jobs / Commands**
Import queue processor · data purge/retention · exports · scheduler commands

---

## Local development

**Requirements**: PHP 8.4, Composer, Node.js, MySQL or PostgreSQL (or Docker)

```bash
cp .env.example .env
composer install --no-interaction --prefer-dist
npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
```

For Docker: see `docker/` and `docker/start-container.sh`.

---

## Testing

```bash
# Unit tests (fast, no services)
composer test:unit

# Feature tests (require DB)
composer test:feature

# Single test or file
php artisan test --filter="NombreDeTest"
vendor/bin/phpunit tests/Feature/SomeTest.php
```

See `phpunit.xml` for recommended env vars (test DB, array cache, etc.).

**Key test suites**
- `AdminCertificationVersioningTest` — snapshot creation, rollback, version increment
- `AdminCertificationImportTest` / `AdminCertificationExportTest` — full ZIP round-trip
- `AdminQuestionsCsvContractTest` — CSV import/export contract validation

---

## CI recommendations

Suggested GitHub Actions / GitLab CI jobs:

| Job | What it runs |
|---|---|
| `lint` | `pint` / `php-cs-fixer` · `markdownlint` |
| `test:unit` | Fast, no external services |
| `test:feature` | MySQL service container required |
| `build_assets` | `npm run build` for release artifacts |
| `security_scan` | `composer audit` (optional) |

Tip: split `unit` and `feature` into parallel jobs for faster CI feedback.

---

## Database

- Migrations: `database/migrations/` — use `php artisan migrate --graceful` on deploy
- Test DB: configured in `phpunit.xml` (`certificados_test`)
- Seeds: `database/seeders/` for demo data

---

## Import/Export (ZIP packages)

Format: `manifest.json` + `questions.csv` + optional `template.html` + `assets/`

- Pre-validation before any write operation
- Rollback on partial failure (no half-imported state)
- Heavy imports dispatched to queue with full `ImportLog` audit trail
- Round-trip tested: export → import → asset restoration

Reference: [`COURSE_ZIP_GUIDE.md`](./COURSE_ZIP_GUIDE.md) · [`questions_csv_schema.md`](../model/questions_csv_schema.md)

---

## Deployment

Production target: Render (web service) + Neon (PostgreSQL) or Aiven (MySQL) + Upstash Redis.

```bash
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
npm run build
```

- Migrations run as a separate GitHub Actions workflow (not inside the container startup)
- Scheduler: external cron calling `php artisan schedule:run` every minute (Render free plan has no built-in cron)
- Queue: `sync` on free plan; switch to `database`/`redis` + `queue:work` when scaling

Full guide: [`DEPLOY_RENDER_NEON_AIVEN.md`](./DEPLOY_RENDER_NEON_AIVEN.md)

---

## Observability

| Concern | Approach |
|---|---|
| Application logs | `storage/logs/laravel.log` · configurable via `config/logging.php` |
| Error tracking | Sentry / LogRocket recommended for production |
| Performance | APM for critical endpoints (import, export, PDF generation) |
| Backups | Regular DB snapshots + `public/Certificates` asset backup |

---

## Security practices

- Sensitive config in `.env` — never committed to VCS
- ZIP and CSV uploads validated and sanitized before processing
- Upload size limits enforced
- HTTPS + security headers (CSP, HSTS, X-Frame-Options) in Nginx config
- Path-traversal protection in ZIP importer (`ZipManifestValidator`)

---

## Performance notes

- Redis for cache and sessions in production (avoid file/array drivers under load)
- Review N+1 queries in certification/question list views
- Static assets: move to CDN/object storage in production to reduce container I/O

---

## Contributing

```bash
# Code style
composer run-script pint

# Before submitting a PR
- Add unit tests for new logic
- Add feature tests for endpoint changes
- Update docs/ for architectural changes
```
