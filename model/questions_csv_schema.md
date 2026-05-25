# Esquema de `questions.csv`

Este documento describe el CSV que exporta e importa el sistema de certificaciones dentro del ZIP. El flujo real está implementado en `app/Support/CertificationZipExporterService.php`, `app/Support/CertificationZipImporterService.php` y validado por `app/Support/QuestionsCsvValidator.php`.

El contrato es multilenguaje primero: el exportador genera columnas por idioma y el importador acepta tanto ese formato como algunas variantes de compatibilidad.

## Encabezado exportado

El exportador genera estas columnas, en este orden:

1. `cert_type`
2. `question_id`
3. `prompt_<lang>` para cada idioma detectado, por ejemplo `prompt_es`, `prompt_en`
4. `option_1_<lang>`, `option_2_<lang>`, `option_3_<lang>`, `option_4_<lang>` para cada idioma detectado
5. `options_json_<lang>` para cada idioma detectado
6. `correct_option`
7. `content_hash`
8. `type`
9. `answer_payload_json`
10. `options_json`
11. `correct_options_json`

## Campos clave

- `cert_type`: identifica la certificación a la que pertenece la fila.
- `question_id`: identificador de la pregunta dentro del paquete; el importador lo usa como clave de origen para emparejar filas.
- `prompt_<lang>`: enunciado de la pregunta para cada idioma disponible.
- `option_X_<lang>`: opciones traducidas, opcionales si se usa `options_json_<lang>`.
- `options_json_<lang>`: JSON array con las opciones de ese idioma; si existe, el importador lo prefiere para esa traducción.
- `correct_option`: índice 1-based para la respuesta correcta de una única opción correcta.
- `correct_options_json`: JSON array con índices 1-based cuando hay varias respuestas correctas.
- `content_hash`: ayuda a flujos de importación inteligente y deduplicación.
- `type`: tipo de pregunta, por ejemplo `single_choice`, `multiple_choice`, `textarea`, `boolean` o `number`.
- `answer_payload_json`: payload JSON opcional para tipos que necesiten más estructura.
- `options_json`: opciones por defecto cuando no se usa una versión por idioma.

## Compatibilidad que acepta el importador

Además del formato exportado, el importador también tolera estas variantes:

- `prompt` y `option_1..4` sin sufijo de idioma.
- `options_json` sin sufijo de idioma.
- `correct_options_json` cuando hay varias respuestas correctas.
- `image_path` como ruta relativa dentro del ZIP.

Si detecta columnas `prompt_<lang>`, el validador exige que exista al menos un `prompt_*` con valor y que la fila incluya `correct_option` o `correct_options_json`.

## Validaciones relevantes

- Los valores JSON en `options_json`, `options_json_<lang>` y `correct_options_json` deben ser válidos.
- `image_path` debe ser relativa y no puede contener `..` ni empezar por `/`.
- Si se pasa un directorio base, el importador comprueba que el archivo exista dentro del paquete.
- Los tipos aceptados por el validador incluyen `mcq_2`, `mcq_3`, `mcq_4`, `true_false`, `single_choice`, `multiple_choice`, `textarea`, `boolean` y `number`.

## Ejemplo mínimo de fila exportada

```csv
cert_type,question_id,prompt_es,prompt_en,option_1_es,option_2_es,option_1_en,option_2_en,correct_option,content_hash,type,answer_payload_json,options_json,correct_options_json
curso-intro-laravel,1,"¿Qué es Laravel?","What is Laravel?","Framework PHP","CMS","PHP framework","CMS",1,abc123,single_choice,"{}","[\"Framework PHP\",\"CMS\"]","[1]"
```

## Nota sobre ejemplos

`docs/examples/questions.example.csv` ahora refleja el contrato base completo que exporta el sistema. Para ver variantes más amplias con `image_path` y tipos adicionales, usa `docs/fixtures/questions_types_example.csv`.
