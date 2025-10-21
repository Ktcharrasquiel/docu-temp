---
prompt_id: "001_create_new_service"
feature: "NEW_SERVICE"
autor: "your_name"
fecha: "2025-10-20"
version: 1
ejecutado: false
tags: ["microservice", "scaffolding"]
---

## Instrucción original

Generar un microservicio nuevo llamado `<service-name>` en el monorepo.

## Preguntas de clarificación

- ¿Cuál es el propósito del microservicio y qué dominio cubre?
- ¿Qué endpoints iniciales se necesitan?
- ¿Existe un modelo de datos preliminar (entidades, relaciones)?
- ¿Requiere autenticación o autorización?
- ¿Debe incluir logging, métricas u otras integraciones (Pub/Sub, Redis)?
 - ¿El microservicio gestionará datos de salud o información sensible que deba cumplir con HIPAA? En caso afirmativo, especifique si se necesita enmascarar datos en logs o establecer controles de acceso adicionales.
 - ¿Se debe integrar análisis de calidad con SonarQube u otra herramienta de análisis estático en el pipeline CI/CD? Si es así, proporcione detalles sobre el servidor SonarQube o tokens.

## Prompt detallado generado

Crea una carpeta `apps/<service-name>` con la estructura estándar del monorepo. Configura `package.json` y `tsconfig.json` basados en los microservicios existentes. Implementa `app.module.ts` y `main.ts` inicializando NestJS 11. Genera el primer controlador y servicio según las respuestas a las preguntas de clarificación (por ejemplo, `UserController` y `UserService`). Crea entidades y DTOs validados con `class-validator`. Incluye la configuración de TypeORM, logs estructurados con pino y documentación Swagger. Prepara scripts de migraciones. Añade pruebas unitarias y e2e con Jest y Supertest.
