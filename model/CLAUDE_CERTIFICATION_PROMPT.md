# Prompt para IA (Claude): Generar una Certificación de Examen Completa

## Objetivo

Proporcionar a una IA (por ejemplo, Claude) un prompt completo y estructurado para que genere una certificación de examen válida, verificable y lista para su emisión. El resultado debe incluir: el certificado (Markdown y HTML listo para PDF), los datos del examen, el informe de resultados, la plantilla de verificación (QR/JSON) y un desglose de calificación.

## Entradas requeridas (por parte del sistema/usuario)

- `exam_title`: Título del examen.
- `organization`: Nombre de la organización emisora.
- `candidate`: Objeto con `name`, `id`, `email` (opcional), `dob` (opcional).
- `date_taken`: Fecha del examen (YYYY-MM-DD).
- `questions`: Lista de preguntas (ver esquema más abajo).
- `passing_score`: Porcentaje mínimo o puntos necesarios para aprobar.
- `issuer`: Objeto con `name`, `title`, `signature_base64` (opcional).
- `verification_base_url`: (opcional) URL base para verificar certificados.

## Formato de salida esperado

La IA debe devolver un objeto con las siguientes claves. Cada campo debe estar claramente etiquetado y formateado en bloques de código (JSON o Markdown) para facilitar el post-procesado:

- `certificate_markdown`: Certificado completo en Markdown listo para conversión a PDF.
- `certificate_html`: HTML responsivo listo para generar PDF (incluye estilos mínimos inline).
- `certificate_json`: Objeto JSON con los datos del certificado y metadatos (`certificate_id`, `issued_at`, `expires_at` opcional, `hash`, `signature` opcional).
- `results`: Objeto con `raw_score`, `percentage`, `passed` (boolean), `per_question` (array de resultados por pregunta), `grading_notes`.
- `answer_key`: Lista de preguntas con respuestas correctas y explicaciones breves.
- `verification_qr_payload`: Texto corto (JSON compactado o URL) que debe codificarse en el QR del certificado.
- `assets`: Opcional — URLs o base64 (logos, firma, sello) que el sistema puede usar.

## Esquema de `question` (entrada)

Cada pregunta en `questions` debe tener este esquema mínimo:

- `id`: string único.
- `type`: `mcq` | `multiple_choice_multiple_answer` | `true_false` | `short_answer` | `essay` | `coding`.
- `stem`: Texto de la pregunta.
- `options`: (solo para MCQ) lista de opciones con `{id, text}`.
- `correct`: identificadores de la(s) respuesta(s) correctas.
- `points`: número de puntos asignados.
- `difficulty`: `easy` | `medium` | `hard` (opcional).
- `explanation`: explicación breve de la respuesta correcta (para `answer_key`).

## Reglas para generación de preguntas y respuestas

- Para preguntas objetivas (`mcq`, `true_false`) generar distractores plausibles y evitar opciones obviamente erróneas.
- Para preguntas de respuesta corta y ensayo, proveer una guía de calificación con criterios y ejemplos de respuestas para cada nivel (p.ej. 0-2-4-6 puntos).
- Para preguntas de `coding`, incluir un enunciado, criterios de entrada/salida, ejemplos y pruebas de evaluación (casos de test básicos y edge cases).
- Cada pregunta debe incluir `tags` (temas), y un `learning_outcome` breve.

## Rúbrica de calificación

- Calificación automática para tipos objetivos: sumar puntos por respuestas correctas.
- Para `essay` y `short_answer`, la IA debe devolver una rúbrica por criterio (p.ej. Relevancia: 0-3, Exactitud: 0-4, Claridad: 0-3) y una puntuación sugerida.
- Incluir `grading_notes` con justificación breve para las puntuaciones no triviales.

## Cálculo de resultados

- `raw_score`: suma de puntos obtenidos.
- `percentage`: raw_score / total_points \* 100 (redondear a 2 decimales).
- `passed`: `true` si `percentage >= passing_score`.
- `certificate_level`: `Distinction` / `Merit` / `Pass` / `Fail` basado en thresholds que la IA debe calcular o usar los proporcionados.

## Plantilla de certificado

El certificado debe incluir los siguientes elementos y etiquetas reemplazables:

- Header: Logo (`assets.logo`), `organization`, `exam_title`.
- Body: `candidate.name`, `candidate.id`, `date_taken`, `raw_score`/`percentage`, `passed`.
- Footer: `certificate_id` (UUID), `issued_at` (ISO 8601), `issuer.name` y `issuer.title`, `issuer.signature` (imagen base64 o URL), `verification_qr`.

## Generación del QR y verificación

- `verification_qr_payload` debe ser compacto y contener al menos `{certificate_id, candidate_id, hash}` o bien una URL de verificación corta como `{verification_base_url}/verify/{certificate_id}`.
- Incluir un `hash` (SHA256) del `certificate_json` como campo para verificación.

## Seguridad y privacidad

- No incluir datos sensibles innecesarios. Si se incluyen emails o DOB, indicar explícitamente que son opcionales.
- Si se solicita firma digital, devolver la firma sugerida en base64 y el método usado (p.ej. HMAC-SHA256 con clave externa — NO incluir claves privadas en la respuesta).

## Formato y etiquetas en la respuesta de la IA

Pedir a la IA que entregue su respuesta en un bloque JSON principal y además exporte `certificate_markdown` y `certificate_html` en bloques separados. Ejemplo de estructura de salida:

```json
{
    "certificate_json": {
        /* ... */
    },
    "certificate_markdown": "---\\n...\\n---",
    "certificate_html": "<!doctype html>...",
    "results": {
        /* ... */
    },
    "answer_key": [
        /* ... */
    ],
    "verification_qr_payload": "...",
    "assets": {
        /* ... */
    }
}
```

## Ejemplo mínimo de entrada (JSON)

```json
{
    "exam_title": "Certificación de Matemáticas Básicas",
    "organization": "Instituto Ejemplar",
    "candidate": { "name": "Ana Pérez", "id": "C-000123" },
    "date_taken": "2026-05-23",
    "passing_score": 70,
    "questions": [
        {
            "id": "q1",
            "type": "mcq",
            "stem": "¿Cuánto es 2+2?",
            "options": [
                { "id": "a", "text": "3" },
                { "id": "b", "text": "4" }
            ],
            "correct": "b",
            "points": 1
        }
    ],
    "issuer": { "name": "Dr. Juan Ruiz", "title": "Director" },
    "verification_base_url": "https://example.org/certs"
}
```

## Ejemplo mínimo de salida (resumen)

```json
{
    "certificate_json": {
        "certificate_id": "uuid-1234",
        "exam_title": "Certificación de Matemáticas Básicas",
        "candidate": { "name": "Ana Pérez", "id": "C-000123" },
        "raw_score": 1,
        "percentage": 100,
        "passed": true,
        "issued_at": "2026-05-23T14:12:00Z",
        "hash": "<sha256>"
    },
    "certificate_markdown": "# Certificado...",
    "verification_qr_payload": "https://example.org/certs/verify/uuid-1234"
}
```

## Instrucciones específicas para Claude (o IA similar)

1. Analiza la entrada JSON proporcionada y valida que todos los campos obligatorios estén presentes.
2. Genera el `answer_key` con explicaciones y, si procede, ejemplos de corrección.
3. Califica automáticamente las preguntas objetivas y genera sugerencias de calificación para ensayos.
4. Construye `certificate_markdown` siguiendo la plantilla arriba indicada; incluye `verification_qr` como texto que luego será convertido a un QR por el sistema.
5. Devuelve un `certificate_html` simple y responsivo para generar el PDF final.
6. Calcula `hash` (SHA256) del `certificate_json` y devuelve `verification_qr_payload`.
7. No inventes claves privadas ni firmes con claves incluidas en la petición.

## Copy-paste prompt final (plantilla)

Usa la siguiente plantilla completa como prompt para Claude. Reemplaza los contenidos entre `<< >>` con los datos reales.

"Produce una certificación de examen completa a partir del siguiente JSON: <<PEGA_AQUI_EL_JSON_DE_ENTRADA>>. Sigue las reglas descritas: devuelve un único objeto JSON con las claves `certificate_json`, `certificate_markdown`, `certificate_html`, `results`, `answer_key`, `verification_qr_payload` y `assets`. Asegúrate de incluir `hash` (SHA256) del `certificate_json`, un `certificate_id` único (UUID), y una `issued_at` en formato ISO 8601. Para `certificate_markdown` usa una presentación formal y clara, adecuada para impresión. Para `certificate_html` genera HTML minimalista y responsivo, listo para conversión a PDF."

## Consideraciones legales y de privacidad

- Informa si se incluyen datos personales y solicita confirmación de su almacenamiento/retención.
- Evita generar o almacenar claves privadas en la respuesta.

## Entrega y siguientes pasos

- Al usar este prompt con Claude, pega el JSON de entrada y solicita la respuesta en formato `application/json`.
- El sistema receptor debe:
    - Validar el `hash` del `certificate_json`.
    - Generar el QR a partir de `verification_qr_payload`.
    - Convertir `certificate_markdown` o `certificate_html` a PDF y almacenarlo junto al `certificate_json`.

---

Archivo creado para integrarlo en pipelines de emisión de certificados.
