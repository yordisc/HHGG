# Prompt compacto para IA — Generar certificación completa

Pega el JSON de entrada y responde **únicamente** con un objeto JSON válido que contenga estas claves:
`certificate_json`, `certificate_markdown`, `certificate_html`, `results`, `answer_key`, `verification_qr_payload`, `assets`.

Sin texto antes ni después del JSON.

---

## Requisitos clave

- Genera `certificate_id` como UUID v4.
- Genera `issued_at` en formato ISO 8601, ej: `"2026-05-26T14:30:00Z"`.
- Calcula `raw_score`, `total_points`, `percentage` (2 decimales) y `passed` según `passing_score`.
- `certificate_level`: `Distinction` (≥90%) · `Merit` (≥75%) · `Pass` (≥passing_score) · `Fail`.
- `hash` en `certificate_json`: SHA-256 hex del objeto con `hash: ""`, claves ordenadas alfabéticamente.
- `certificate_markdown`: estructura formal con nombre del candidato, puntuación, emisor e ID de certificado.
- `certificate_html`: HTML minimalista con estilos inline, sin dependencias externas. Incluir `[QR: {verification_qr_payload}]` donde irá el código QR.
- `verification_qr_payload`:
  - Si el input incluye `verification_base_url` → usar `{verification_base_url}/verify/{certificate_id}`
  - Si no incluye `verification_base_url` → usar `{"cid":"{certificate_id}","kid":"{candidate.id}","h":"{primeros 16 chars del hash}"}`
- `assets`: objeto vacío `{}` si no se proveen recursos en el input.

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
      "explanation": "2 + 2 = 4 por definición de la suma en los naturales."
    },
    {
      "id": "q2",
      "type": "true_false",
      "stem": "El número π es un número racional.",
      "correct": "false",
      "points": 1,
      "explanation": "π es irracional; no puede expresarse como cociente de dos enteros."
    }
  ],
  "issuer": { "name": "Dra. Laura Ruiz", "title": "Directora Académica" },
  "verification_base_url": "https://example.org/certs"
}
```

## Ejemplo de salida esperada (fragmento)

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
  "verification_qr_payload": "https://example.org/certs/verify/550e8400-e29b-41d4-a716-446655440000",
  "assets": {}
}
```

---

## Prompt listo para copiar y pegar

```
Genera una certificación de examen completa a partir del siguiente JSON:

<<JSON_DE_ENTRADA>>

Devuelve únicamente un objeto JSON válido con las claves: certificate_json, certificate_markdown, certificate_html, results, answer_key, verification_qr_payload y assets. Sin texto adicional antes ni después del JSON.
```
