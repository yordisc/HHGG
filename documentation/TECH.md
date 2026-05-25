# Arquitectura y stack

## Stack principal

- Laravel 11
- Livewire 4
- PHP 8.4
- Tailwind CSS y Vite
- MySQL o PostgreSQL segun entorno
- `barryvdh/laravel-dompdf` para PDF

## Piezas clave

- Modelos para certificaciones, preguntas, traducciones, certificados, versiones y auditoria.
- Controladores publicos para home, busqueda, quiz, certificados y verificacion.
- Controladores admin para preguntas, usuarios, plantillas, import/export y dashboard.
- Servicios para scoring, elegibilidad, expulsion de datos caducados, versionado e importacion.
- Jobs y comandos para limpieza, purga y tareas programadas.

## Decisiones operativas

- En produccion la app corre con Docker, Nginx y PHP-FPM.
- Las migraciones se ejecutan fuera del contenedor web.
- La cola en produccion usa `sync` para evitar infraestructura adicional en el plan base.
- El scheduler puede ejecutarse por webhook protegido si no hay cron nativo.
- Redis se usa para cache y sesiones en produccion cuando esta disponible.

## Datos y mantenimiento

- Las certificaciones usan versionado para conservar historial y permitir rollback.
- La importacion y exportacion de paquetes ZIP esta cubierta por validacion previa y pruebas.
- El sistema maneja reglas de caducidad, retencion y purga automatica.
- Las preguntas soportan traducciones por locale y contratos CSV multilenguaje.

## Cobertura de pruebas

- Flujos de quiz y practica.
- Importacion y exportacion de certificaciones.
- Validacion de certificados y PDF.
- Versionado, reversa y eliminacion segura.
- Scheduler webhook y provisionamiento del admin principal.
