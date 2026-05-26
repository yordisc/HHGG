# Prompt para IA: Generar una Certificación de Examen Completa

## Objetivo

Proporcionar a una IA (por ejemplo, Claude) un prompt completo y estructurado para que genere una certificación de examen válida, verificable y lista para su emisión. El resultado debe incluir: el certificado (Markdown y HTML listo para PDF), los datos del examen, el informe de resultados, la plantilla de verificación y un desglose de calificación.

---

## Entradas requeridas

Pasa a la IA un único objeto JSON con esta estructura exacta:

```json
{
  "exam_title": "string — título del examen",
  "organization": "string — nombre de la organización emisora",
  "candidate": {
    "name": "string — nombre completo",
    "id": "string — identificador único del candidato",
    "email": "string — opcional",
    "dob": "string — opcional, formato YYYY-MM-DD"
  },
  "date_taken": "string — formato YYYY-MM-DD",
  "passing_score": "number — porcentaje mínimo para aprobar, ej: 70",
  "questions": [ /* ver esquema de question más abajo */ ],
  "issuer": {
    "name": "string",
    "title": "string",
    "signature_base64": "string — opcional, imagen en base64"
  },
  "verification_base_url": "string — opcional, ej: https://example.org/certs"
}
```

---

## Esquema de `question`

Cada elemento del array `questions` debe seguir este esquema:

```json
{
  "id": "string — identificador único dentro del examen, ej: 'q1'",
  "type": "string — uno de: mcq | true_false | short_answer | essay | coding",
  "stem": "string — enunciado completo de la pregunta",
  "options": [
    { "id": "string — ej: 'a'", "text": "string — texto de la opción" }
  ],
  "correct": "string o array — id(s) de la(s) respuesta(s) correcta(s), ej: 'b' o ['a','c']",
  "points": "number — puntos asignados a esta pregunta",
  "difficulty": "string — opcional, uno de: easy | medium | hard",
  "explanation": "string — justificación de la respuesta correcta, para el answer_key",
  "tags": ["string — temas o categorías"],
  "learning_outcome": "string — breve descripción del resultado de aprendizaje"
}
```

Nota: `options` solo aplica para los tipos `mcq`. Para `true_false`, `short_answer`, `essay` y `coding` se omite.

---

## Formato de salida esperado

La IA debe devolver **únicamente** un objeto JSON con estas claves. Sin texto adicional, sin bloques de código envolventes, sin explicaciones fuera del JSON.

```json
{
  "certificate_json": { },
  "certificate_markdown": "string",
  "certificate_html": "string",
  "results": { },
  "answer_key": [ ],
  "verification_qr_payload": "string",
  "assets": { }
}
```

Descripción de cada clave:

### `certificate_json`

```json
{
  "certificate_id": "string — UUID v4 generado por la IA, ej: '550e8400-e29b-41d4-a716-446655440000'",
  "exam_title": "string — igual al input",
  "organization": "string — igual al input",
  "candidate": {
    "name": "string",
    "id": "string"
  },
  "date_taken": "string — YYYY-MM-DD",
  "raw_score": "number — suma de puntos obtenidos",
  "total_points": "number — suma de todos los puntos posibles",
  "percentage": "number — raw_score / total_points * 100, redondeado a 2 decimales",
  "passed": "boolean",
  "certificate_level": "string — uno de: Distinction | Merit | Pass | Fail",
  "issued_at": "string — ISO 8601, ej: '2026-05-26T14:30:00Z'",
  "expires_at": "string — ISO 8601 opcional, null si no aplica",
  "issuer": {
    "name": "string",
    "title": "string"
  },
  "hash": "string — SHA-256 hex del objeto certificate_json con este campo en blanco (''), calculado después de construir el resto del objeto"
}
```

**Cómo calcular `hash`**: construir el objeto `certificate_json` completo con `hash: ""`, serializar a JSON con claves ordenadas alfabéticamente, aplicar SHA-256 y escribir el resultado hexadecimal en este campo.

### `certificate_markdown`

Cadena Markdown con esta estructura exacta:

```
# Certificado de [exam_title]

**Organización:** [organization]

---

Este certificado acredita que

## [candidate.name]

con identificación **[candidate.id]**

ha completado satisfactoriamente el examen **[exam_title]**
con una puntuación de **[percentage]%** ([raw_score]/[total_points] puntos).

**Resultado:** [certificate_level]
**Fecha:** [date_taken]
**Emitido:** [issued_at]

---

**[issuer.name]**
*[issuer.title]*
*[organization]*

---

ID de certificado: `[certificate_id]`
Verificación: [verification_qr_payload]
```

### `certificate_html`

HTML minimalista y responsivo, sin dependencias externas. Estilos inline o en un bloque `<style>` interno. Debe incluir todos los campos del certificado y un placeholder de texto `[QR: {verification_qr_payload}]` en el lugar donde iría el QR (el sistema receptor lo reemplazará por la imagen real).

### `results`

```json
{
  "raw_score": "number",
  "total_points": "number",
  "percentage": "number",
  "passed": "boolean",
  "certificate_level": "string",
  "per_question": [
    {
      "id": "string — id de la pregunta",
      "points_earned": "number",
      "points_possible": "number",
      "correct": "boolean"
    }
  ],
  "grading_notes": "string — justificación breve de puntuaciones no triviales, vacío si todas son automáticas"
}
```

**Umbrales de `certificate_level`:**
- `Distinction`: percentage >= 90
- `Merit`: percentage >= 75
- `Pass`: percentage >= passing_score
- `Fail`: percentage < passing_score

### `answer_key`

Array con una entrada por pregunta:

```json
[
  {
    "id": "string — id de la pregunta",
    "correct": "string o array — igual al campo correct del input",
    "explanation": "string — igual al campo explanation del input, o generado si no se proveyó",
    "rubric": "string — solo para essay y short_answer: criterios de puntuación con niveles explícitos"
  }
]
```

### `verification_qr_payload`

**Regla única:** si se recibe `verification_base_url` en el input, usar siempre esta forma:

```
{verification_base_url}/verify/{certificate_id}
```

Si **no** se recibe `verification_base_url`, usar este JSON compacto (sin espacios):

```
{"cid":"{certificate_id}","kid":"{candidate.id}","h":"{primeros 16 caracteres del hash}"}
```

No mezclar formatos. Un input con `verification_base_url` produce siempre una URL. Un input sin él produce siempre el JSON compacto.

### `assets`

Objeto con recursos opcionales. Si no se proveen en el input, devolver el objeto vacío `{}`. Si se proveen:

```json
{
  "logo": "string — URL o base64 con prefijo data:image/png;base64,...",
  "signature": "string — URL o base64 con prefijo data:image/png;base64,...",
  "seal": "string — URL o base64 con prefijo data:image/png;base64,..."
}
```

---

## Reglas de calificación

- **mcq / true_false**: calificación automática. `correct` en el input es el id (o array de ids) de la opción correcta.
- **short_answer**: proveer en `answer_key.rubric` una escala explícita, ej: `"0 puntos: sin respuesta o incorrecta. 1 punto: respuesta parcial. 2 puntos: respuesta completa y precisa."`.
- **essay**: proveer en `answer_key.rubric` criterios por dimensión, ej: `"Relevancia (0-3): ... Exactitud (0-4): ... Claridad (0-3): ..."`.
- **coding**: incluir en `answer_key.rubric` los casos de test básicos y edge cases que determinan la puntuación.

---

## Seguridad y privacidad

- No incluir `email` ni `dob` en `certificate_json` ni en `certificate_markdown` salvo que el sistema receptor lo requiera explícitamente.
- No generar ni incluir claves privadas en ninguna parte de la respuesta.
- Si se pide firma digital, devolver en `assets.signature` la representación base64 del campo de firma y anotar en `grading_notes` que la clave real debe aplicarse externamente.

---

## Ejemplo de entrada

```json
{
  "exam_title": "Certificación de Matemáticas Básicas",
  "organization": "Instituto Ejemplar",
  "candidate": { "name": "Ana Pérez", "id": "C-000123" },
  "date_taken": "2026-05-26",
  "passing_score": 70,
  "questions": [
    {
      "id": "q1",
      "type": "mcq",
      "stem": "¿Cuánto es 2 + 2?",
      "options": [
        { "id": "a", "text": "3" },
        { "id": "b", "text": "4" },
        { "id": "c", "text": "5" }
      ],
      "correct": "b",
      "points": 2,
      "difficulty": "easy",
      "explanation": "2 + 2 = 4 por definición de la suma en los naturales.",
      "tags": ["aritmética"],
      "learning_outcome": "Aplicar operaciones básicas de suma"
    },
    {
      "id": "q2",
      "type": "true_false",
      "stem": "El número π es un número racional.",
      "correct": "false",
      "points": 1,
      "difficulty": "medium",
      "explanation": "π es irracional; no puede expresarse como cociente de dos enteros.",
      "tags": ["números", "teoría"],
      "learning_outcome": "Distinguir entre números racionales e irracionales"
    }
  ],
  "issuer": { "name": "Dra. Laura Ruiz", "title": "Directora Académica" },
  "verification_base_url": "https://example.org/certs"
}
```

## Ejemplo de salida esperada

```json
{
  "certificate_json": {
    "certificate_id": "550e8400-e29b-41d4-a716-446655440000",
    "exam_title": "Certificación de Matemáticas Básicas",
    "organization": "Instituto Ejemplar",
    "candidate": { "name": "Ana Pérez", "id": "C-000123" },
    "date_taken": "2026-05-26",
    "raw_score": 3,
    "total_points": 3,
    "percentage": 100.00,
    "passed": true,
    "certificate_level": "Distinction",
    "issued_at": "2026-05-26T14:30:00Z",
    "expires_at": null,
    "issuer": { "name": "Dra. Laura Ruiz", "title": "Directora Académica" },
    "hash": "a3f1c2e4b5d6a7f8e9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2"
  },
  "certificate_markdown": "# Certificado de Certificación de Matemáticas Básicas\n\n**Organización:** Instituto Ejemplar\n\n---\n\nEste certificado acredita que\n\n## Ana Pérez\n\ncon identificación **C-000123**\n\nha completado satisfactoriamente el examen **Certificación de Matemáticas Básicas**\ncon una puntuación de **100%** (3/3 puntos).\n\n**Resultado:** Distinction\n**Fecha:** 2026-05-26\n**Emitido:** 2026-05-26T14:30:00Z\n\n---\n\n**Dra. Laura Ruiz**\n*Directora Académica*\n*Instituto Ejemplar*\n\n---\n\nID de certificado: `550e8400-e29b-41d4-a716-446655440000`\nVerificación: https://example.org/certs/verify/550e8400-e29b-41d4-a716-446655440000",
  "certificate_html": "<html><head><meta charset='UTF-8'><style>body{font-family:Georgia,serif;max-width:800px;margin:40px auto;padding:40px;border:3px double #333;} h1{text-align:center;} .candidate{text-align:center;font-size:1.5em;margin:20px 0;} .meta{margin:10px 0;} .footer{margin-top:40px;border-top:1px solid #ccc;padding-top:20px;font-size:0.85em;color:#555;}</style></head><body><h1>Certificación de Matemáticas Básicas</h1><p class='meta'><strong>Organización:</strong> Instituto Ejemplar</p><p>Este certificado acredita que</p><div class='candidate'><strong>Ana Pérez</strong><br><small>ID: C-000123</small></div><p>ha completado satisfactoriamente el examen con una puntuación de <strong>100%</strong> (3/3 puntos).</p><p><strong>Resultado:</strong> Distinction &nbsp;|&nbsp; <strong>Fecha:</strong> 2026-05-26</p><p>[QR: https://example.org/certs/verify/550e8400-e29b-41d4-a716-446655440000]</p><div class='footer'><p><strong>Dra. Laura Ruiz</strong> — Directora Académica, Instituto Ejemplar</p><p>ID: 550e8400-e29b-41d4-a716-446655440000</p></div></body></html>",
  "results": {
    "raw_score": 3,
    "total_points": 3,
    "percentage": 100.00,
    "passed": true,
    "certificate_level": "Distinction",
    "per_question": [
      { "id": "q1", "points_earned": 2, "points_possible": 2, "correct": true },
      { "id": "q2", "points_earned": 1, "points_possible": 1, "correct": true }
    ],
    "grading_notes": ""
  },
  "answer_key": [
    {
      "id": "q1",
      "correct": "b",
      "explanation": "2 + 2 = 4 por definición de la suma en los naturales.",
      "rubric": ""
    },
    {
      "id": "q2",
      "correct": "false",
      "explanation": "π es irracional; no puede expresarse como cociente de dos enteros.",
      "rubric": ""
    }
  ],
  "verification_qr_payload": "https://example.org/certs/verify/550e8400-e29b-41d4-a716-446655440000",
  "assets": {}
}
```

---

## Prompt listo para copiar y pegar

Copia el bloque siguiente y reemplaza `<<JSON_DE_ENTRADA>>` con tu objeto JSON de entrada:

```
Genera una certificación de examen completa a partir del siguiente JSON:

<<JSON_DE_ENTRADA>>

Devuelve únicamente un objeto JSON válido con las claves: certificate_json, certificate_markdown, certificate_html, results, answer_key, verification_qr_payload y assets. Sin texto adicional antes ni después del JSON. Sigue estrictamente el esquema de salida definido en el prompt de referencia.
```

---

## Pasos del sistema receptor

Una vez que la IA devuelve el JSON:

1. Parsear y validar que todas las claves requeridas están presentes.
2. Recalcular el hash SHA-256 del `certificate_json` (con `hash: ""`) y compararlo con `certificate_json.hash` para verificar integridad.
3. Generar la imagen QR a partir de `verification_qr_payload`.
4. Reemplazar `[QR: ...]` en `certificate_html` con la imagen QR generada.
5. Convertir `certificate_html` a PDF con DOMPDF o equivalente.
6. Almacenar `certificate_json` como registro de auditoría.
