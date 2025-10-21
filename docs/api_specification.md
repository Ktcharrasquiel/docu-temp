# Guía de Especificación de APIs

Diseñar y documentar una API de forma correcta es fundamental para la interoperabilidad y el mantenimiento de los microservicios. Esta guía resume las mejores prácticas para generar y mantener una especificación OpenAPI (Swagger) en el contexto del monorepo.

## 1. Importancia de la especificación de API

Una especificación de API proporciona un contrato entre el servidor y los consumidores (frontends, integraciones externas o incluso otros microservicios). Definir de manera explícita los endpoints, parámetros, tipos de respuesta y códigos de estado evita ambigüedades y facilita la generación de SDKs, documentación y pruebas automatizadas.

## 2. Uso de Swagger/OpenAPI

NestJS incluye una integración oficial con Swagger a través de `@nestjs/swagger`, que permite generar automáticamente una especificación OpenAPI a partir de los decoradores aplicados en controladores, métodos y DTOs. Para habilitarlo:

1. Instale la dependencia: `npm install @nestjs/swagger swagger-ui-express`.
2. En el archivo `main.ts`, configure el módulo de Swagger:

```typescript
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('API Hero')
  .setDescription('Documentación de la API del microservicio')
  .setVersion('1.0')
  .addBearerAuth()
  .build();
const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

3. Anote los controladores y métodos con decoradores como `@ApiTags()`, `@ApiOperation()`, `@ApiResponse()`, `@ApiBody()` y `@ApiParam()`. Documente cada DTO con `@ApiProperty()`.

## 3. Versionado de la API

Las APIs evolucionan con el tiempo; para evitar que los cambios rompan a los consumidores existentes, utilice versionado. Algunas pautas:

- Prefije las rutas con un identificador de versión (`/api/v1/users`, `/api/v2/users`).
- Mantenga varios módulos de controladores si conviven versiones diferentes.
- Actualice la versión en el `DocumentBuilder` de Swagger.
- Deprecate explícitamente los endpoints antiguos en la documentación (`@ApiDeprecated()`).

## 4. Convenciones de diseño

- Utilice **HTTP verbs** de acuerdo con su semántica: `GET` para recuperar recursos, `POST` para crear, `PUT` para reemplazar, `PATCH` para actualizar parcialmente y `DELETE` para eliminar.
- Use **nombres de recursos en plural** (`/users`, `/orders`) y **subrecursos** para relaciones (`/users/{userId}/visits`).
- Devuelva **códigos de estado estándar** (`200 OK`, `201 Created`, `204 No Content`, `400 Bad Request`, `404 Not Found`, etc.).
- Para errores, exponga una estructura de respuesta consistente (por ejemplo, `{ statusCode: 400, message: 'Invalid email', error: 'Bad Request' }`).
- Use **tipos estrictos** en DTOs para entradas y salidas; evite campos ambiguos.

## 5. Documentación interactiva

La integración de Swagger genera automáticamente una interfaz interactiva (`/api/docs`) donde los desarrolladores pueden probar los endpoints. Mantenga esta interfaz actualizada, incluyendo ejemplos de solicitudes y respuestas mediante los campos `example` en los DTOs y decoradores de Swagger.

## 6. Herramientas y validación

- Valide la consistencia de la especificación con herramientas como **Swagger Editor** o **Speccy**.
- Genere clientes (SDKs) a partir del OpenAPI usando **OpenAPI Generator** para facilitar la integración con otras aplicaciones.
- Integre la revisión de la especificación en el pipeline de CI para detectar cambios no documentados.

## 7. Referencias

Consulte la documentación oficial de NestJS para más detalles sobre la generación de Swagger y OpenAPI【235039593161354†L246-L253】 y explore la guía de OpenAPI para conocer todas las posibilidades de la especificación.