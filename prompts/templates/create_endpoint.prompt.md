---
prompt_id: "002_create_new_endpoint"
feature: "ADD_ENDPOINT"
autor: "your_name"
fecha: "2025-10-20"
version: 1
ejecutado: false
tags: ["endpoint", "api"]
---

## Instrucción original

Generar un nuevo endpoint `<HTTP_METHOD> /<resource>` para el microservicio `<service-name>`.

## Preguntas de clarificación

- ¿Cuál es el método HTTP? (GET, POST, PUT, DELETE, PATCH)
- ¿Qué datos recibe el endpoint y cómo deben validarse? (DTO)
- ¿Qué datos retorna? (estructura de respuesta)
- ¿Requiere autenticación? ¿Qué roles pueden acceder?
- ¿Debe registrarse en logs o métricas adicionales?

## Prompt detallado generado

Actualiza `apps/<service-name>/src/controllers/<resource>.controller.ts` implementando el endpoint `<HTTP_METHOD> /<resource>`. Crea un DTO con `class-validator` para las propiedades de entrada y un servicio con la lógica de negocio (`<Resource>Service.<method>`). Documenta el endpoint con decoradores Swagger (`@ApiOperation()`, `@ApiResponse()`). Incluye autorización con JWT si aplica (`@UseGuards(JwtAuthGuard)`). Asegura que el servicio interactúe con TypeORM de manera coherente. Añade pruebas unitarias y e2e para el nuevo endpoint.
