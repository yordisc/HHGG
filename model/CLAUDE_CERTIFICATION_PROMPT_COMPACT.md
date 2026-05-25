Instrucciones compactas para Claude — Generar certificación completa

Pega el JSON de entrada y responde únicamente con `application/json` conteniendo las claves: `certificate_json`, `certificate_markdown`, `certificate_html`, `results`, `answer_key`, `verification_qr_payload`, `assets`.

Requisitos clave:

- Genera `certificate_id` (UUID) y `issued_at` (ISO 8601).
- Calcula `raw_score`, `percentage`, `passed` según `passing_score`.
- Incluye `hash` (SHA256) del `certificate_json` dentro de `certificate_json.hash`.
- `verification_qr_payload` debe ser una URL corta: `{verification_base_url}/verify/{certificate_id}` o un JSON compacto `{certificate_id, candidate_id, hash}`.
- `certificate_markdown`: formato formal listo para imprimir.
- `certificate_html`: HTML minimalista y responsivo.

Plantilla de uso (inserta el JSON de entrada en lugar de <<INPUT_JSON>>):

"Genera una certificación completa a partir de este JSON: <<INPUT_JSON>>. Devuelve únicamente un objeto JSON con las claves solicitadas y sin explicaciones adicionales."

Archivo de ejemplo: [docs/CLAUDE_CERTIFICATION_EXAMPLE_INPUT.json](docs/CLAUDE_CERTIFICATION_EXAMPLE_INPUT.json)
