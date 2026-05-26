# Esquema de `questions.csv`

Este documento describe el CSV que exporta e importa el sistema de certificaciones dentro del ZIP. El flujo real está implementado en `app/Support/CertificationZipExporterService.php`, `app/Support/CertificationZipImporterService.php` y validado por `app/Support/QuestionsCsvValidator.php`.

El contrato es multilenguaje primero: el exportador genera columnas por idioma y el importador acepta tanto ese formato como algunas variantes de compatibilidad.

---

## Tipos de pregunta aceptados

El importador y el validador aceptan estos valores en la columna `type`:

| Valor | Descripción |
|---|---|
| `mcq_2` | Opción múltiple con 2 opciones |
| `mcq_3` | Opción múltiple con 3 opciones |
| `mcq_4` | Opción múltiple con 4 opciones |
| `true_false` | Verdadero / Falso |
| `single_choice` | Alias de MCQ con respuesta única (compatible con importador) |
| `multiple_choice` | Opción múltiple con varias respuestas correctas |
| `textarea` | Respuesta abierta de texto largo |
| `boolean` | Alias de true_false (compatible con importador) |
| `number` | Respuesta numérica |

> **Nota de consistencia:** El enum `QuestionType` del backend solo acepta `mcq_2`, `mcq_3`, `mcq_4` y `true_false` en el formulario de creación manual. Los tipos adicionales (`single_choice`, `multiple_choice`, `textarea`, `boolean`, `number`) son aceptados por el importador CSV/ZIP para compatibilidad con paquetes externos, pero pueden no estar disponibles en el builder visual.

---

## Encabezado exportado

El exportador genera estas columnas, en este orden:

1. `cert_type`
2. `question_id`
3. `prompt_<lang>` por cada idioma detectado, ej: `prompt_es`, `prompt_en`
4. `option_1_<lang>`, `option_2_<lang>`, `option_3_<lang>`, `option_4_<lang>` por cada idioma
5. `options_json_<lang>` por cada idioma detectado
6. `correct_option`
7. `content_hash`
8. `type`
9. `answer_payload_json`
10. `options_json`
11. `correct_options_json`

---

## Descripción de campos

| Campo | Tipo | Descripción |
|---|---|---|
| `cert_type` | string | Slug de la certificación a la que pertenece la fila |
| `question_id` | string o número | Identificador de la pregunta dentro del paquete; usado como clave para emparejar filas en el importador |
| `prompt_<lang>` | string | Enunciado de la pregunta para el idioma `lang`, ej: `prompt_es`, `prompt_en` |
| `option_X_<lang>` | string | Opción X traducida al idioma `lang`. Opcional si se usa `options_json_<lang>` |
| `options_json_<lang>` | JSON array | Array de strings con las opciones en el idioma `lang`. Si existe, el importador lo usa en preferencia a `option_X_<lang>` para esa traducción |
| `correct_option` | número 1-based | Índice de la opción correcta cuando hay una sola respuesta correcta. Ej: `1` apunta a `option_1` |
| `correct_options_json` | JSON array | Array de índices 1-based cuando hay varias respuestas correctas. Ej: `[1,3]` |
| `content_hash` | string | Hash de contenido; ayuda a flujos de deduplicación e importación inteligente. Puede dejarse vacío |
| `type` | string | Tipo de pregunta. Ver tabla de tipos aceptados arriba |
| `answer_payload_json` | JSON object | Payload opcional para tipos que necesiten estructura adicional. Ver ejemplos abajo |
| `options_json` | JSON array | Opciones por defecto (sin idioma) cuando no se usa una versión `options_json_<lang>` |
| `image_path` | string | Ruta relativa de la imagen dentro del ZIP, ej: `images/q1.png`. No puede contener `..` ni empezar con `/` |

---

## JSON dentro de CSV: escapado de comillas

Los campos JSON dentro de un CSV requieren que las comillas internas se dupliquen. Ejemplo:

```
# Correcto: el valor del campo está entre comillas CSV y las comillas internas del JSON se escapan duplicándolas
"[""Framework PHP"",""CMS""]"

# Alternativa equivalente con comillas simples en el JSON (solo si el parser lo acepta)
'["Framework PHP","CMS"]'
```

En la práctica, la mayoría de editores de hojas de cálculo (Excel, Google Sheets) hacen este escapado automáticamente al exportar como CSV. Si construyes el CSV manualmente o con código, asegúrate de que tu librería CSV maneje el quoting correctamente.

---

## `answer_payload_json` por tipo

Este campo es un objeto JSON serializado como string. Ejemplos por tipo:

| Tipo | Valor de `answer_payload_json` |
|---|---|
| `true_false` | `{"value": true}` |
| `multiple_choice` | `{"correct": [1, 3]}` — aunque `correct_options_json` es preferible |
| `textarea` / `short_answer` | `{"keywords": ["concepto A", "concepto B"], "min_words": 30}` |
| `number` | `{"answer": 42, "tolerance": 0.5}` |
| `mcq_2` / `mcq_3` / `mcq_4` | Normalmente vacío `{}` — usar `correct_option` en su lugar |

Si el campo no aplica para el tipo, dejarlo vacío `{}` o como cadena vacía.

---

## Validaciones del importador

- Al menos un campo `prompt_*` con valor no vacío por fila.
- `correct_option` o `correct_options_json` debe estar presente (no ambos vacíos).
- Los valores en `options_json`, `options_json_<lang>` y `correct_options_json` deben ser JSON válido.
- `image_path` debe ser una ruta relativa sin `..` ni prefijo `/`.
- Si se provee un directorio base (ZIP), el importador verifica que el archivo referenciado en `image_path` exista dentro del paquete.
- Si `cert_type` no coincide con ninguna certificación existente, la fila se omite con advertencia.

---

## Compatibilidad: variantes que acepta el importador

Además del formato exportado, el importador tolera estas variantes para compatibilidad con paquetes externos:

- `prompt` y `option_1..4` sin sufijo de idioma (se tratan como idioma base).
- `options_json` sin sufijo de idioma.
- `image_path` como columna adicional.
- `certification_slug` como alias de `cert_type`.

---

## Ejemplos de filas

### Pregunta MCQ estándar (dos idiomas)

```csv
cert_type,question_id,prompt_es,prompt_en,option_1_es,option_2_es,option_1_en,option_2_en,correct_option,content_hash,type,answer_payload_json,options_json,correct_options_json
curso-intro-laravel,1,"¿Qué es Laravel?","What is Laravel?","Framework PHP","CMS","PHP framework","CMS",1,abc123,mcq_2,{},,
```

### Pregunta con opciones en JSON y múltiples respuestas correctas

```csv
cert_type,question_id,prompt_es,prompt_en,option_1_es,option_2_es,option_1_en,option_2_en,correct_option,content_hash,type,answer_payload_json,options_json,correct_options_json
curso-intro-laravel,2,"¿Cuáles son ORMs de PHP?","Which are PHP ORMs?",,,,,,",[""Eloquent"",""Doctrine"",""Propel"",""jQuery""]","[1,2,3]"
```

> En este ejemplo `correct_option` se deja vacío y se usa `correct_options_json` con valor `[1,2,3]`.

### Pregunta true/false con image_path

```csv
cert_type,question_id,prompt_es,prompt_en,correct_option,content_hash,type,answer_payload_json,options_json,correct_options_json,image_path
curso-intro-laravel,3,"Laravel incluye un ORM llamado Eloquent.","Laravel includes an ORM called Eloquent.",1,def456,true_false,"{""value"": true}",,,"images/questions/q3-context.png"
```

> Para `true_false`, `correct_option: 1` representa "verdadero" y `correct_option: 2` representa "falso". Alternativamente, usar `answer_payload_json: {"value": true}`.

---

## Estructura recomendada del ZIP cuando hay imágenes

```
package.zip
├── manifest.json
├── questions.csv
├── template.html         # opcional
├── template.css          # opcional
├── assets/               # logo, firma, recursos del certificado
│   └── logo.png
└── images/
    └── questions/
        ├── q1.png
        └── q3-context.png
```

En `questions.csv`, `image_path` debe ser relativa a la raíz del ZIP: `images/questions/q1.png`.

---

## Nota sobre ejemplos de referencia

- `docs/fixtures/questions_example.csv` — ejemplo base con el contrato mínimo de columnas.
- `docs/fixtures/questions_types_example.csv` — variantes con `image_path` y tipos adicionales.
