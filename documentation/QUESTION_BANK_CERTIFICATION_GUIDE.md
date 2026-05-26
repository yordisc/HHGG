# Guia detallada: banco de preguntas por certificacion

## 1) Objetivo de esta guia

Este documento explica, paso a paso, como construir y mantener una base de datos de preguntas para una certificacion usando las herramientas actuales del proyecto.

Incluye:

- Flujos disponibles en admin (manual, constructor visual, CSV, ZIP de certificacion)
- Tipos de pregunta soportados hoy en backend
- Como se define la respuesta correcta
- Traducciones por idioma
- Campos avanzados (peso, muerte subita, imagen, explicacion)
- Validaciones reales del sistema
- Checklist de calidad del banco
- Gaps detectados para decidir futuras funcionalidades

## 2) Herramientas disponibles hoy

### 2.1 Gestion de preguntas (admin)

Pantalla principal:

- Ruta: /admin/questions
- Vista: resources/views/admin/questions/index.blade.php
- Controlador: app/Http/Controllers/Admin/QuestionAdminController.php

Desde aqui puedes:

- Crear pregunta individual
- Editar/eliminar/duplicar
- Importar CSV con vista previa
- Exportar CSV del banco actual
- Descargar plantilla CSV
- Abrir el constructor visual

### 2.2 Formulario manual de pregunta

- Vista crear: resources/views/admin/questions/create.blade.php
- Vista editar: resources/views/admin/questions/edit.blade.php

Permite configurar:

- Certificacion destino
- Tipo de pregunta
- Opciones y respuesta correcta
- Peso
- Modo de muerte subita
- Imagen
- Traducciones
- Estado activa/inactiva

### 2.3 Constructor visual

- Vista: resources/views/admin/questions/builder.blade.php
- Endpoint de guardado: admin.questions.store

Uso recomendado:

- Equipos de contenido no tecnico
- Carga rapida de preguntas con vista previa inmediata

Nota importante: el builder muestra mas tipos que los aceptados actualmente por backend (ver seccion 4 y seccion 10).

### 2.4 Importacion y exportacion CSV de preguntas

- Import validate preview: admin.questions.validate.csv
- Confirmacion de import: admin.questions.confirm.csv
- Export CSV: admin.questions.export.csv
- Plantilla CSV: admin.questions.template.csv

Controlador involucrado:

- app/Http/Controllers/Admin/QuestionAdminController.php
- app/Support/CsvValidator.php

### 2.5 Import/Export ZIP de certificacion completa

Si quieres mover curso completo (certificacion + banco + plantilla + assets):

- Import ZIP: /admin/certifications/import
- Export ZIP por certificacion: /admin/certifications/{certification}/export

Archivos clave:

- app/Http/Controllers/Admin/CertificationImportExportController.php
- app/Support/CertificationZipImporterService.php
- app/Support/CertificationZipExporterService.php

Estado actual:

- La importación pesada se prevalida y se despacha a cola.
- El ZIP exportado mantiene `assets/` como raíz de assets; el importador sigue aceptando paquetes antiguos con `slug/` o rutas absolutas históricas.
- Hay cobertura de round-trip completa para exportar, importar y restaurar assets/plantillas.

## 3) Modelo de datos de preguntas

### 3.1 Tabla questions (resumen funcional)

Campos principales:

- certification_id: relacion con certificacion
- prompt: enunciado base
- option_1..option_4: opciones base
- correct_option: entero 1..4
- options (JSON, opcional): array ordenado de opciones (nuevo, permite N opciones)
- correct_options (JSON, opcional): array de índices 1-based de las opciones correctas (permite múltiples correctas)
- type: tipo de pregunta
- weight: peso para scoring ponderado
- sudden_death_mode: regla de muerte subita
- explanation: explicacion opcional
- image_path: imagen opcional
- active: habilitada o no
- is_test_question: marca de pregunta de prueba

Modelo:

- app/Models/Question.php

### 3.2 Tabla question_translations

Campos principales:

- question_id
- language
- prompt
- option_1..option_4
- options (JSON, opcional): opciones por traducción (si existe se usa en preferencia al campo base)

Modelo:

- app/Models/QuestionTranslation.php

Comportamiento:

- Si falta traduccion de algun campo, el sistema puede usar fallback de la base.

## 4) Tipos de pregunta y respuestas (estado real)

## Tipos oficialmente soportados en backend

Definidos en enum (actualizado):

- mcq_2
- mcq_3
- mcq_4
- true_false

Archivo:

- app/Enums/QuestionType.php

### 4.1 MCQ-2

- Tipo: mcq_2
- Opciones utilizadas: option_1, option_2
- option_3 y option_4 se guardan como null en flujo builder/store
- Respuesta correcta: correct_option debe ser 1 o 2 en la practica

### 4.2 MCQ-4

- Tipo: mcq_4
- Opciones utilizadas: option_1..option_4
- Respuesta correcta: correct_option entre 1 y 4

### 4.3 Como se representa la respuesta correcta

- Campo legacy: `correct_option` (entero 1-based)
- Nuevo campo flexible: `correct_options` (JSON array de índices 1-based) para múltiples correctas
- Campo adicional: `answer_payload` (JSON nullable) para tipos que requieren estructura especial

Archivo de reglas:

- app/Http/Requests/UpdateQuestionRequest.php

Nota de compatibilidad: el sistema soporta varias formas de representación y mantiene compatibilidad:

- Legacy: `option_1..option_4` y `correct_option` (entero 1..4).
- Nuevo (flexible): `options` (JSON array de strings) y `correct_options` (JSON array de índices 1-based).
- `answer_payload`: campo JSON libre usado por tipos como `true_false`, `matching` o `fill_blank` para serializar datos específicos del tipo (ver ejemplos abajo).

El runtime y el importador/exportador consultan `options`/`correct_options`/`answer_payload` en prioridad cuando están presentes; si no, caen al formato legacy.

Ejemplos de `answer_payload` por tipo:

- `mcq_3` / `mcq_4` (opcional): `{"correct": [1]}` o `{"correct": [1,3]}` aunque normalmente `correct_options` es preferible.
- `true_false`: `{"value": true}` — cuando se necesita persistir el valor verdadero/falso en el payload.
- `matching`: `{"pairs": [["leftA","right1"],["leftB","right2"]]}` (ejemplo de estructura esperada).
- `fill_blank`: `{"blanks": ["respuesta1","respuesta2"]}` (puede incluir regex o flags de coincidencia).

CSV/ZIP import: el campo exportado/importado `answer_payload_json` contendrá la representación en JSON string (escapeada) y el importador lo parsea a `answer_payload` en BD.

## 5) Campos avanzados para evaluar mejor el banco

### 5.1 weight (ponderacion)

- Tipo decimal
- Usado por WeightedScoringService
- Permite que unas preguntas valgan mas que otras

### 5.2 sudden_death_mode

Enum actual:

- none
- fail_if_wrong
- pass_if_correct

Archivos:

- app/Enums/SuddenDeathMode.php
- app/Support/SuddenDeathRuleService.php

Notas:

- A nivel de `Question` existe `sudden_death_mode` que marca el comportamiento individual.
- A nivel de `Certification` ahora se puede configurar `settings.sudden_death` (objeto con `mode` y opcional `count`) para indicar cómo seleccionar qué preguntas marcadas como SD se aplican en cada intento (ej: `all`, `fixed_count`, `random_count`, `one`).
- `QuizRunner` usa esa configuración para marcar un subconjunto de preguntas SD para el intento, manteniendo `previewMode` sin aplicar la selección de SD.

### 5.3 explanation

- Texto opcional para justificar respuesta
- Util para revision/formacion

### 5.4 image_path

- Recurso visual opcional de la pregunta
- Admite carga de imagen en create/edit

Ejemplos y recomendaciones prácticas:

- En CSV/ZIP, la columna `image_path` debe contener una ruta relativa dentro del paquete, p. ej. `images/q1.png` o `assets/questions/q1.png`.
- Estructura recomendada del ZIP:

```
package.zip
├─ manifest.json
├─ questions.csv
├─ template.html
├─ template.css
├─ assets/
│  ├─ logo.png
│  └─ styles/
├─ images/
│  └─ questions/
│     └─ q1.png
└─ other-files
```

- En el CSV una fila podría incluir:

```
cert_type,question_id,prompt_en,option_1_en,option_2_en,image_path,correct_option
good-girl,,"What color is the sky?","Blue","Green","images/questions/q1.png",1
```

- El importador valida que la ruta relativa exista dentro del paquete antes de persistir la pregunta (ver `app/Support/QuestionsCsvValidator.php`). Si el paquete contiene `images/` o `assets/`, el importador copiará esos ficheros a `Certificates/<slug>/...` o a S3 según `filesystems`.

- Si subes la imagen desde el admin/builder, el fichero se almacena en disco público bajo `questions/` y el `image_path` apunta a `questions/<filename>` (ver `app/Http/Controllers/Admin/QuestionAdminController.php`). Para servir esas imágenes en local, asegúrate de tener creado el enlace de storage con `php artisan storage:link`.

- Buenas prácticas:
    - Usar nombres de fichero descriptivos y sin espacios: `q1-context.png`.
    - Mantener las imágenes optimizadas (webp/png/jpg) y < 200 KB cuando sea posible.
    - No usar rutas absolutas ni `..` dentro de `image_path`.

## 6) Flujo recomendado para crear banco de preguntas de una certificacion

### Paso 1: definir estrategia del banco

Antes de cargar preguntas, fijar:

- Numero objetivo de preguntas activas
- Distribucion por dificultad
- Peso por categoria (si usas scoring ponderado)
- Reglas de muerte subita (si aplica)
- Idiomas obligatorios

### Paso 2: crear lote inicial (manual o CSV)

Opciones:

- Manual (pocas preguntas): create/edit o builder
- Masivo (muchas preguntas): CSV con vista previa

### Paso 3: validar calidad minima

Checklist rapido:

- Sin preguntas duplicadas semanticamente
- Una sola correcta por pregunta
- Opciones claras y no ambiguas
- Enunciado corto y accionable
- Traducciones revisadas
- Porcentaje de activas suficiente para el examen configurado

### Paso 4: activar certificacion y probar

- Ejecutar un intento de quiz real
- Verificar randomizacion de preguntas/opciones
- Verificar scoring y resultado esperado

### Paso 5: versionar y exportar backup ZIP

- Exportar curso completo en ZIP
- Guardar version en repositorio interno de contenido

## 7) Contrato practico de CSV de preguntas

### 7.1 Columnas base para import/export legacy

El controlador de preguntas trabaja con:

- question_id
- language
- cert_type
- prompt
- option_1
- option_2
- option_3
- option_4
- correct_option
- active

### 7.2 Reglas operativas

- language = en crea/actualiza base de Question
- language != en actualiza/crea QuestionTranslation (requiere question_id valido)
- Si cert_type no existe, fila se omite
- correct_option fuera de rango genera error en validacion

### 7.3 Flujo seguro de carga CSV

1. Descargar plantilla
2. Completar datos
3. Subir para vista previa (validate)
4. Corregir errores y advertencias
5. Confirmar importacion

## 8) Como cargar curso completo con ZIP

Cuando ademas de preguntas necesitas mover:

- settings de certificacion
- template HTML/CSS
- assets del certificado

Usa modulo ZIP de certificaciones:

- Importa desde /admin/certifications/import
- Exporta desde accion de export por certificacion

Referencia recomendada:

- docs/IMPORT_EXPORT_SISTEMA.md
- docs/COURSE_ZIP_GUIDE.md

## 9) Auditoria funcional: que revisar al terminar un banco

### 9.1 Cobertura y distribucion

- Preguntas activas >= questions_required de la certificacion
- Balance de temas
- Balance de dificultad

### 9.2 Coherencia de respuestas

- Correct_option apunta a una opcion existente
- No hay opciones vacias en preguntas activas
- No hay opciones repetidas con distinto indice

### 9.3 Internacionalizacion

- Traducciones completas para idiomas obligatorios
- Terminologia consistente entre idiomas

### 9.4 Metrica de calidad sugerida

- % de preguntas con imagen util
- % de preguntas con explicacion
- % de preguntas con peso != 1.0
- % de preguntas con traduccion por idioma

## 10) Gaps detectados y funcionalidades candidatas

Esta seccion te sirve para decidir roadmap.

### 10.1 Desalineacion entre builder y backend

El builder UI muestra tipos:

- mcq_3
- true_false
- matching
- fill_blank

Pero validacion backend (QuestionType enum + request) solo acepta:

- mcq_2
- mcq_4

Impacto:

- El usuario puede seleccionar tipos en UI que luego no pasan validacion

Recomendacion:

- Opcion A: ocultar temporalmente esos tipos en builder
- Opcion B: implementar soporte real en backend (enum, validacion, persistencia, runner y scoring)

### 10.2 correct_option fijo 1..4

Para tipos no-MCQ (matching/fill_blank) no alcanza el esquema actual.

Recomendacion:

- Introducir estructura de respuesta por tipo (por ejemplo JSON answer_payload)

### 10.3 Falta versionado especifico de preguntas

Hoy hay versionado fuerte en certificaciones, pero no un historial completo por pregunta.

Recomendacion:

- Agregar audit/version por question para trazabilidad editorial

### 10.4 Falta lint semantico de distractores

No hay chequeo automatico de:

- opciones duplicadas
- longitud extrema
- patron de respuesta repetitivo

Recomendacion:

- agregar validador editorial previo a activar preguntas

## 11) Recomendacion de priorizacion

Prioridad alta:

1. Alinear builder con backend (evitar tipos no soportados)
2. Mejorar validaciones de calidad de opciones
3. Agregar reporte de salud del banco por certificacion

Prioridad media:

1. Soporte real para true_false
2. Versionado por pregunta
3. Herramienta de revision de traducciones incompletas

Prioridad baja:

1. Soporte matching/fill_blank
2. Editor enriquecido de explicaciones
3. Banco multimedia avanzado

## 12) Comandos utiles de validacion

- Suite completa:
  php artisan test

- Preguntas admin:
  php artisan test --filter=AdminQuestions

- Import/Export certificaciones:
  php artisan test --filter=AdminCertificationImportTest
  php artisan test --filter=AdminCertificationExportTest

- Contrato CSV:
  php artisan test --filter=AdminQuestionsCsvContractTest

---

Si quieres, en una segunda version puedo convertir esta guia en una checklist operativa por rol (Editor, Revisor, Admin) para estandarizar el proceso de construccion del banco de preguntas.
