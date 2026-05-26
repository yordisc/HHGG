# Project Scope — HHGG

HHGG combines a formal certification facade with satirical content to simulate a complete credentialing platform. The goal: demonstrate a real production Laravel application end-to-end, from exam engine to verified PDF certificate.

---

## Public flow

| Step | Route | What happens |
|---|---|---|
| 1 · Catalog | `/` | Browse and search certifications with filters |
| 2 · Register | `/exam/{certType}/register` | Candidate identification and eligibility check |
| 3 · Start exam | `/exam/start` | Cooldown and attempt-limit enforcement |
| 4 · Quiz | `/exam/{certType}` | Randomized questions, shuffled options, timed rules |
| 5 · Result | `/result/{serial}` | Scoring result with public serial |
| 6 · Certificate | `/cert/{serial}` | Rendered certificate with verification data |
| 7 · PDF | `/cert/{serial}/pdf` | DOMPDF-generated certificate for download |
| 8 · Verify | public endpoint | Serial + token lookup for third-party verification |

---

## Admin flow

| Area | Capabilities |
|---|---|
| Certifications | Full CRUD · activate/deactivate · reorder · clone |
| Questions | Create/edit/delete · import CSV with preview · export CSV · bulk tools |
| Question builder | Visual builder for non-technical content teams |
| Templates | HTML/CSS editor · live preview · per-certification assignment |
| Versioning | Automatic snapshots on every save · diff view · one-click rollback |
| Import/Export | Full course ZIP packaging (`manifest.json` + `questions.csv` + assets) |
| Audit | Import logs · change history · trazabilidad |
| Users | Admin user management and roles |

---

## Engine highlights

**Quiz engine**
- Question types: `mcq_2`, `mcq_3`, `mcq_4`, `true_false`
- Weighted scoring via `WeightedScoringService`
- Sudden-death mode: `fail_if_wrong` / `pass_if_correct` — configurable at certification and question level
- Randomized question selection and option shuffling per attempt

**Certificate issuance**
- Unique serial per certificate
- SHA-256 hash + token for public verification
- DOMPDF-rendered PDF with swappable HTML/CSS templates
- Public verification endpoint (`/cert/{serial}`)

**Multilingual support**
- 7 locales: `en`, `es`, `pt`, `zh`, `hi`, `ar`, `fr`
- Per-question translation with fallback to base language
- CSV import/export with `prompt_<lang>` / `option_X_<lang>` column layout

**Import/Export pipeline**
- ZIP format: `manifest.json` + `questions.csv` + optional `template.html` + `assets/`
- Validated by `ZipManifestValidator` with path-traversal protection
- Heavy imports dispatched to queue with full audit log
- Round-trip tested: export → import → asset restoration

---

## API surface

| Type | Prefix |
|---|---|
| Public web | `/`, `/search`, `/exam/*`, `/result/*`, `/cert/*` |
| Admin panel | `/admin/*` |
| Internal API | `/api/*` (webhooks, scheduler, internal queries) |

---

## Technical summary

- **Framework**: Laravel 11 · Livewire 4 · Blade
- **Frontend**: Tailwind CSS · Vite · Alpine.js patterns
- **PHP**: 8.4
- **Database**: MySQL or PostgreSQL (env-switchable)
- **PDF**: `barryvdh/laravel-dompdf`
- **Key services**: `WeightedScoringService`, `SuddenDeathRuleService`, `CertificationVersioningService`, `CertificationZipImporterService`, `CertificationZipExporterService`, `CsvValidator`

---

## Local setup (quick)

```bash
cp .env.example .env
composer install
npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
```

For Docker: see `docker/` and `docker/start-container.sh`.

---

## Running tests

```bash
# Fast unit suite (no services needed)
composer test:unit

# Feature suite (requires DB)
composer test:feature

# Single test
php artisan test --filter="NombreDeTest"
```

---

## Deployment

Production stack: Docker + Nginx + PHP-FPM + MySQL/PostgreSQL + Redis (Upstash).

Full guide → [DEPLOY_RENDER_NEON_AIVEN.md](./DEPLOY_RENDER_NEON_AIVEN.md)

```bash
# Release steps
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
npm run build
```

---

## Visual reference

Screenshots for every major screen are in [`documentation/images/`](./images/).
Covers: home, exam flow, result, certificate, admin panel, wizard, question builder, templates, import, audit.
