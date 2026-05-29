<div align="center">

# HHGG — Plataforma Satírica de Certificaciones

**Diseñada para hacer reír. Construida para impresionar.**

[![Laravel](https://img.shields.io/badge/Laravel-11-FF2D20?style=flat-square&logo=laravel&logoColor=white)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-8.4-777BB4?style=flat-square&logo=php&logoColor=white)](https://php.net)
[![Livewire](https://img.shields.io/badge/Livewire-4-4E56A6?style=flat-square&logo=livewire&logoColor=white)](https://livewire.laravel.com)
[![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?style=flat-square&logo=mysql&logoColor=white)](https://mysql.com)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Deploy](https://img.shields.io/badge/Deploy-Render-46E3B7?style=flat-square&logo=render&logoColor=white)](https://hhgg-5c2f.onrender.com/)

[**Ver demo en vivo →**](https://hhgg-5c2f.onrender.com/)

</div>

---

![HHGG Home](./documentation/images/00%20-%20HOME.png)

---

## ¿Qué es HHGG?

HHGG es una plataforma de certificaciones completamente funcional con un giro satírico. Simula el ciclo de vida completo de un sistema de acreditación profesional — desde el registro del candidato hasta la emisión y verificación pública del certificado — construida sobre un stack PHP moderno con prácticas de ingeniería reales.

> La sátira es el producto. La ingeniería es el portafolio.

---

## ¿Por qué es interesante técnicamente?

| Área | Qué hay aquí |
|---|---|
| 🧠 **Motor de examen** | Preguntas aleatorizadas, scoring ponderado, modo sudden-death, cooldowns y control de intentos |
| 🏅 **Certificados verificables** | Serial único, hash SHA-256 y endpoint de verificación pública por certificado |
| 🌍 **Multiidioma real** | 7 locales (`en`, `es`, `pt`, `zh`, `hi`, `ar`, `fr`) con traducciones por pregunta y fallback automático |
| 📦 **Pipeline import/export** | Empaquetado completo en ZIP (`manifest.json` + `questions.csv` + assets), validado y despachado a cola |
| 🛠️ **Panel admin** | CRUD de certificaciones, constructor de preguntas, editor de plantillas HTML/CSS, snapshots, rollback y audit log |
| 🔁 **Sistema de versiones** | Snapshots inmutables en cada cambio, diff entre versiones, rollback con un clic |

---

## Stack

```
Backend   Laravel 11 · PHP 8.4
Frontend  Livewire 4 · Blade · Tailwind CSS · Vite · Alpine.js
Base de datos  MySQL 8 / PostgreSQL (configurable por entorno)
PDF       barryvdh/laravel-dompdf
Colas     Laravel Queue (sync / database / redis)
Cache     Redis (Upstash en producción)
Deploy    Docker · Nginx · PHP-FPM · Render
```

---

## Capturas

<div align="center">

| Panel admin | Constructor de preguntas |
|---|---|
| ![Admin](./documentation/images/09%20-%20admin.png) | ![Builder](./documentation/images/19-admin-questions-builder.png) |

| Wizard de certificación | Certificado emitido |
|---|---|
| ![Wizard](./documentation/images/11%20-%20admin-certifications-wizard-1.png) | ![Cert](./documentation/images/07%20-%20certificate.png) |

</div>

> Galería completa en [`documentation/images/`](./documentation/images/)

---

## Flujo de la plataforma

```
Catálogo → Registro → Examen → Resultado → Certificado → Verificación pública
```

**Rutas principales**

| Ruta | Función |
|---|---|
| `/` | Catálogo público con filtros y búsqueda |
| `/exam/{certType}/register` | Registro e identificación del candidato |
| `/exam/{certType}` | Motor de quiz en tiempo real |
| `/result/{serial}` | Resultado público por serial |
| `/cert/{serial}` | Vista pública del certificado |
| `/cert/{serial}/pdf` | Descarga del PDF generado |
| `/admin/*` | Panel de administración completo |

---

## Arranque local

```bash
cp .env.example .env
composer install
npm install
php artisan key:generate
php artisan migrate --seed
npm run dev
php artisan serve --host=0.0.0.0 --port=8000
```

Para Docker: ver `docker/` y `docker/start-container.sh`.

---

## Tests

```bash
composer test:unit    # Suite rápida, sin servicios externos
composer test:feature # Suite completa, requiere base de datos
```

---

## Documentación

| Documento | Qué cubre |
|---|---|
| [Guía de despliegue](./documentation/DEPLOY_RENDER_NEON_AIVEN.md) | Render + Neon/Aiven, variables de entorno, workflows CI/CD |
| [Guía del builder visual](./documentation/VISUAL_BUILDER_GUIDE.md) | Editor admin, panel de preguntas, UI de versiones |
| [Sistema de versionado](./documentation/VERSIONING_SYSTEM.md) | Snapshots, rollback, comparación de versiones |
| [Banco de preguntas](./documentation/QUESTION_BANK_CERTIFICATION_GUIDE.md) | Estructura del banco, contrato CSV, scoring, i18n |
| [Guía de ZIPs de cursos](./documentation/COURSE_ZIP_GUIDE.md) | Formato del paquete, contrato de importación/exportación |
| [Esquema questions.csv](./model/questions_csv_schema.md) | Especificación completa de columnas con layout multilenguaje |
| [Prompt de certificación](./model/CLAUDE_CERTIFICATION_PROMPT.md) | Prompt de IA para generar certificaciones completas |
| [Troubleshooting](./documentation/TROUBLESHOOTING.md) | Problemas conocidos, soluciones, checklist de debugging |

---

## Arquitectura del proyecto

```
app/
├── Enums/              QuestionType · SuddenDeathMode
├── Http/Controllers/
│   ├── Admin/          Certificaciones · Preguntas · Plantillas · Import/Export
│   └── Api/            Endpoints internos y webhooks
├── Models/             Certification · Question · Certificate · CertificationVersion
├── Observers/          CertificationObserver (versionado automático)
└── Support/            Servicios: scoring · versioning · ZIP import/export · CSV validation
database/
├── migrations/         Esquema completo + seeds de plantillas
└── seeders/
resources/views/
├── admin/              Panel admin con Blade + Livewire
└── pdf/                Plantillas de certificado para DOMPDF
```

---

<div align="center">

**HHGG no es una certificación real ni sustituye evaluaciones médicas, psicológicas o legales.**
Este repositorio existe para demostrar el alcance técnico, el producto y el mantenimiento de una aplicación Laravel moderna.

</div>
