# Creación de un Nuevo Microservicio

Esta guía paso a paso explica cómo añadir un nuevo microservicio al monorepo Hero Med.

## 1. Planificación

Antes de comenzar, defina claramente:

- **Responsabilidad del servicio**: ¿qué dominio o función cubrirá?
- **End points**: rutas HTTP que expondrá.
- **Modelo de datos**: entidades y relaciones que utilizará.
- **Dependencias**: servicios o librerías compartidas que necesita.

## 2. Crear la estructura del servicio

En la raíz del repositorio, ejecute:

```bash
mkdir -p apps/new-service/src/{controllers,services,dto,entities,repositories,pipes,interceptors,filters,guards,config}
mkdir -p apps/new-service/migrations apps/new-service/scripts apps/new-service/docs apps/new-service/test
cd apps/new-service
```

Cree los archivos básicos:

- **`package.json`**: incluya scripts como:
  ```json
  {
    "name": "@hero/new-service",
    "version": "1.0.0",
    "main": "dist/main.js",
    "scripts": {
      "start:dev": "nest start --watch",
      "start:prod": "node dist/main.js",
      "build": "nest build",
      "test": "jest",
      "test:cov": "jest --coverage",
      "typeorm:migration:generate": "ts-node -r tsconfig-paths/register node_modules/typeorm/cli.js migration:generate",
      "typeorm:migration:run": "ts-node -r tsconfig-paths/register node_modules/typeorm/cli.js migration:run"
    },
    "dependencies": {
      "@nestjs/common": "^11.1.6",
      "@nestjs/core": "^11.1.6",
      "@nestjs/swagger": "^11.2.1",
      "typeorm": "^0.3.27",
      "pg": "^8.11.0",
      "class-validator": "^0.14.2",
      "class-transformer": "^0.5.1"
    },
    "devDependencies": {
      "@nestjs/testing": "^11.1.6",
      "jest": "^30.0.0",
      "supertest": "^6.3.3"
    }
  }
  ```
- **`tsconfig.json`**: extienda el tsconfig base del proyecto.
- **`src/app.module.ts`** y **`src/main.ts`**: configure el módulo raíz y la app de NestJS.

## 3. Configurar TypeORM

Cree `src/config/typeorm.config.ts`:

```typescript
import { DataSource } from 'typeorm';
import { User } from '../entities/user.entity';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DATABASE_HOST,
  port: +process.env.DATABASE_PORT,
  username: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  entities: [User],
  migrations: ['dist/migrations/*.js'],
  synchronize: false,
});
```

Luego, importe `AppDataSource` en `main.ts` antes de `app.listen()` para inicializar la conexión.

## 4. Crear entidades y DTOs

En `src/entities/` defina sus entidades con TypeORM. En `src/dto/`, defina las clases de entrada y salida usando `class-validator` y `class-transformer`.

## 5. Implementar controladores y servicios

- Defina un `Controller` para cada recurso (`user.controller.ts`, `order.controller.ts`, etc.).
- Cree servicios que encapsulen la lógica de negocio (`user.service.ts`).
- Inyecte repositorios de TypeORM con `@InjectRepository()`.

## 6. Configurar Swagger

En `main.ts` utilice `SwaggerModule` para exponer la documentación:

```typescript
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('New Service API')
  .setDescription('API del nuevo microservicio')
  .setVersion('1.0')
  .addBearerAuth()
  .build();
const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

Documente cada DTO y controlador con decoradores de Swagger (`@ApiTags()`, `@ApiOperation()`, `@ApiResponse()`).

## 7. Tests

Cree pruebas unitarias en `test/unit/` y pruebas E2E en `test/e2e/`. Use `jest` y `supertest` para levantar la aplicación y enviar peticiones HTTP.

## 8. Actualizar `docker-compose.yml`

Agregue el servicio a su archivo `docker-compose.yml`:

```yaml
  new-service:
    build:
      context: .
      dockerfile: apps/new-service/Dockerfile
    environment:
      - DATABASE_HOST=postgres
      - DATABASE_USER=postgres
      - DATABASE_PASSWORD=admin
      - DATABASE_NAME=hero_main
      - JWT_SECRET=local-secret
    depends_on:
      - postgres
    ports:
      - "4000:3000"
```

## 9. Registrar el prompt

Cree un prompt en `.cursor/prompts/features/new-service/001_create-new-service.prompt.md` siguiendo la plantilla de `prompt_workflow.md`. Incluya la instrucción original, preguntas de clarificación y el prompt detallado.

## 10. Crear rama y PR

- Cree una rama `feature/001-create-new-service`.
- Implemente el microservicio.
- Haga commit y push.
- Marque el prompt como ejecutado.
- Abra un Pull Request a `develop`.

