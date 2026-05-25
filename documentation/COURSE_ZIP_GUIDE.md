# Guía de ZIPs de Cursos

Este documento explica cómo construir un paquete `.zip` de cursos/certificaciones para importación y exportación en el sistema.

## Estado de compatibilidad

El sistema de importación/exportación ya soporta ZIP de forma nativa mediante `ZipArchive` y requiere `ext-zip` en PHP. No hay un problema estructural con los ZIP en el código principal.

El flujo real ya está implementado y validado; este documento solo resume el contrato operativo del paquete.

Punto a tener en cuenta:

- El importador espera exactamente `manifest.json` y `questions.csv` en la raíz del ZIP.
- Los archivos de fixtures de ejemplo usan nombres `manifest_example.json` y `questions_example.csv`, así que hay que renombrarlos al empaquetar un ZIP real.
- El importador acepta `slug` o `certification_slug`, y `name` o `title`.
- El exportador actual genera los assets solo bajo `assets/`.
- Las plantillas exportadas se reescriben para apuntar a `assets/...`.

## Estructura esperada

```text
course-package.zip
├── manifest.json
├── questions.csv
├── template.html        # opcional
├── template.css         # opcional
└── assets/              # opcional
    ├── image/
    └── signature/
```

## Contenido mínimo

### `manifest.json`

Debe incluir, como mínimo:

- `slug`
- `name`
- `questions_csv`

Ejemplo:

```json
{
    "slug": "good-girl",
    "name": "Good Girl Certificate",
    "questions_csv": "questions.csv",
    "settings": {
        "title_passed": "Good Girl Certificate",
        "title_failed": "Bitch Certificate"
    }
}
```

### `questions.csv`

Debe estar en UTF-8 y respetar el contrato multilenguaje del validador actual.

Ejemplo de encabezado:

```csv
cert_type,question_id,prompt_es,prompt_en,option_1_es,option_2_es,option_3_es,option_4_es,option_1_en,option_2_en,option_3_en,option_4_en,options_json_es,options_json_en,correct_option,content_hash,type,answer_payload_json,options_json,correct_options_json
```

Si el CSV incluye `options_json_{lang}` o `correct_options_json`, el importador lo acepta.

## Cómo generar un ZIP de prueba

Desde la raíz del repositorio:

```bash
mkdir -p /tmp/cert_example
cp docs/fixtures/manifest_example.json /tmp/cert_example/manifest.json
cp docs/fixtures/questions_example.csv /tmp/cert_example/questions.csv
cd /tmp/cert_example
zip -r /tmp/good-girl-package.zip .
```

Recuerda que los nombres finales deben ser `manifest.json` y `questions.csv`.

## Checklist antes de importar

- El ZIP contiene `manifest.json` y `questions.csv` en la raíz.
- `slug` es único en la base de datos.
- `questions.csv` no está vacío.
- Las rutas de assets en la plantilla apuntan a archivos realmente incluidos en el ZIP y preferiblemente usan `assets/...`.
- El entorno tiene `ext-zip` habilitado.

## Verificación rápida

Para comprobar que el ZIP se ve bien:

```bash
unzip -l /tmp/good-girl-package.zip
```

Para validar el manifest contra el schema:

```bash
python3 - <<'PY'
import json
from jsonschema import Draft7Validator

schema = json.load(open('docs/fixtures/manifest.schema.json'))
inst = json.load(open('docs/fixtures/manifest_example.json'))

validator = Draft7Validator(schema)
errors = sorted(validator.iter_errors(inst), key=lambda e: e.path)

if errors:
    print('INVALID')
    for e in errors:
        path = '.'.join([str(p) for p in e.path]) or '(root)'
        print(f'Path: {path} -> {e.message}')
else:
    print('VALID')
PY
```

## Problemas conocidos

- Si usas validación JSON Schema sin `FormatChecker`, el campo `created_at` puede no verificarse como `date-time` aunque el schema lo declare.
- Si el ZIP incluye nombres distintos a `manifest.json` y `questions.csv`, el importador lo rechazará.
