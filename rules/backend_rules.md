# Backend Rules — Monorepo NestJS/TypeORM (sin CQRS)

## Introducción

Estas reglas establecen las convenciones y prácticas que debe seguir cualquier microservicio backend en el monorepo. Están diseñadas para un entorno de desarrollo con **NestJS 11**, **TypeORM 0.3** y **PostgreSQL 18**. Las instrucciones están actualizadas al **20 de octubre de 2025** y sustituyen a los lineamientos previos basados en CQRS. Se asume un modelo **monoinquilino** (una sola base de datos), pero se mantiene la preparación para evolucionar a multi‑tenant.

### Stack y versiones recomendadas

- **NestJS** 11.1.6 — Framework principal【937831791123850†L154-L156】.
- **TypeORM** 0.3.27 — ORM para la base de datos【252649584095210†L187-L234】.
- **PostgreSQL** 18 — Motor de base de datos recomendado【751745391940691†L34-L52】 (compatible con versiones 17 y 16【245916017138488†L530-L565】).
- **class-validator** 0.14.2 — Para validación de DTOs【632671327863386†L12-L27】.
- **class-transformer** 0.5.1 — Para transformación de datos【272920904669772†L185-L226】.
- **Swagger (@nestjs/swagger)** 11.2.1 — Documentación OpenAPI【271560829756094†L185-L232】.
- **Jest** 30 — Framework de testing【124000509919989†L38-L45】.

Consulte periódicamente las notas de lanzamiento de cada biblioteca y actualice estas versiones cuando sea necesario.

### Estructura de carpetas

Se recomienda una estructura de monorepo con separación clara entre microservicios y librerías compartidas. El esquema general es:

```
monorepo-root/
├── apps/
│   ├── <service-name>/
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── dto/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   ├── pipes/
│   │   │   ├── interceptors/
│   │   │   ├── filters/
│   │   │   ├── guards/
│   │   │   ├── config/
│   │   │   ├── main.ts
│   │   │   └── app.module.ts
│   │   ├── migrations/
│   │   ├── scripts/
│   │   ├── docs/
│   │   ├── test/
│   │   ├── package.json
│   │   └── .env
│   └── ...
├── libs/
│   ├── common/        # Decoradores, pipes, interceptores, guards y utilidades compartidas
│   ├── database/      # Configuración TypeORM común y factorización de conexiones
│   ├── logging/       # Servicio de logging con pino y GCP Logging
│   ├── auth/          # Guard y helpers de autenticación JWT
│   └── config/        # Lectura de variables de entorno y normalización
├── docs/
├── .cursor/
├── docker-compose.yml
└── package.json
```

- **`apps/`**: cada microservicio vive en su propia carpeta con su código fuente, migraciones, pruebas, scripts y configuración. No coloque lógica de negocio de un servicio en otro.
- **`libs/`**: contiene módulos compartidos reutilizables por cualquier microservicio (por ejemplo, logging, autenticación, conexión a la base de datos). Estas librerías no deben depender de código específico de un microservicio.
- **`.cursor/`**: carpeta de prompts y reglas para automatizar generación de código y PR.

### Multi‑tenant y modelo actual

La arquitectura está preparada para operar en un entorno multi‑tenant. Sin embargo, **actualmente se utiliza un único inquilino (monoinquilino)** alojado en una base de datos llamada `hero_main`. La migración a multi‑tenant se realiza añadiendo un campo `tenant_id` en cada tabla y una factoría de conexiones que genera un `DataSource` por tenant. Para el estado actual:

- Todas las tablas incluyen un campo `tenant_id` con valor por defecto `DEFAULT`.
- Los servicios no resuelven el tenant dinámicamente, pero mantienen la capacidad de hacerlo en el futuro.
- Los logs y métricas deben incluir el campo `tenantId` para facilitar la futura segregación.

### Políticas de migraciones

1. **Migraciones locales por servicio.** Cada microservicio gestiona sus migraciones en `apps/<service-name>/migrations`. No se permiten migraciones globales.
2. **Configuración independiente.** Cada microservicio define su `DataSource` en `apps/<service-name>/src/config`. Las variables de entorno (`DATABASE_HOST`, `DATABASE_NAME`, `DATABASE_USER`, etc.) deben encontrarse en su propio `.env`.
3. **Generación y ejecución.** Use scripts del `package.json` del microservicio:
   ```bash
   # Generar
   cd apps/<service-name>
   npm run typeorm:migration:generate -- -n create-users-table

   # Ejecutar
   npm run typeorm:migration:run
   ```
4. **Registro.** Documente cada migración en `apps/<service-name>/docs/migrations.md` indicando objetivo, autor y fecha.
5. **Revisión.** Antes de hacer merge a la rama principal, asegúrese de que la migración no afecta a otros servicios ni compromete el modelo multi‑tenant.

### Nombramiento y organización de archivos

- Utilice **nombres descriptivos en inglés**. Ejemplo: `UserController`, `CreateOrderService`.
- **Una clase por archivo**: cada controlador, servicio, entidad, repositorio, pipe o guard reside en su propio archivo.
- **PascalCase** para clases (`VisitService`), **camelCase** para métodos (`createVisit`) y variables (`visitRepository`), **kebab-case** para archivos (`visit.service.ts`), **UPPER_SNAKE_CASE** para constantes (`MAX_ITEMS`).
- **DTOs** en `src/dto` y terminan con `Dto` (`CreateVisitDto`). **Entidades** en `src/entities` anotadas con decoradores de TypeORM.
- Separe `pipes`, `filters`, `interceptors` y `guards` en carpetas dedicadas.

### DTOs y entidades

- Use `class-validator` para validar cada propiedad de entrada (`@IsString()`, `@IsUUID()`, etc.).
- Use `class-transformer` para transformar u ocultar campos (`@Expose()`, `@Exclude()`).
- Documente cada DTO con `@ApiProperty()` de Swagger para que la generación de OpenAPI sea completa.
- No reutilice entidades como DTOs: mantenga las clases de persistencia separadas de las de transporte.
- Incluya un campo `tenantId` en sus DTOs cuando sea necesario para segregación futura.

### Logging y observabilidad

- Utilice `pino` y su integración con NestJS (`nestjs-pino`) para todos los logs. Configure `pino-pretty` en desarrollo para una salida legible y JSON en producción para que Cloud Logging lo ingiera correctamente.
- Los logs deben incluir los campos:
  - `timestamp` — ISO 8601
  - `level` — nivel de severidad (`info`, `warn`, `error`)
  - `service` — nombre del microservicio
  - `tenantId` — identificador del inquilino (por defecto `DEFAULT`)
  - `correlationId` — identificador de correlación para rastreo entre servicios
  - `message` — descripción humana
  - `metadata` — objeto con información adicional (userId, requestId, etc.)
- En desarrollo, active un middleware que genere un `correlationId` por petición y simule un usuario de pruebas (`{ userId: 'test', roles: ['dev'] }`) con logs etiquetados `[TEST-MODE]`.
- En producción, desactive el modo de pruebas y emplee un guard de autenticación (JWT u OAuth2).

### Testing

- Use **Jest** para pruebas unitarias y **supertest** para pruebas E2E.
- **Pruebas unitarias:** mockee repositorios y servicios externos. Compruebe la lógica de controladores, servicios, pipes, etc.
- **Pruebas de integración:** levante un servicio de base de datos (p. ej., con Docker) y ejecute migraciones en una base `hero_test` para validar queries reales.
- **Pruebas E2E:** lance la aplicación en memoria con `supertest` y verifique las rutas HTTP, la validación de DTOs y los logs generados. Incluya cabeceras como `x-correlation-id` cuando corresponda.
- Implemente un pipeline de CI que ejecute `npm run test:cov` y exija al menos 85 % de cobertura.

### Seguridad y autenticación

- Aunque el proyecto es monoinquilino, todas las rutas deben estar listas para autenticación.
- Utilice JWT para autenticar usuarios. Cree un guard (`JwtAuthGuard`) en `libs/auth` que verifique el token y añada el objeto de usuario a la petición.
- Documente las rutas protegidas con `@ApiBearerAuth()` y añada la definición de seguridad en la configuración de Swagger.
- No registre en los logs contraseñas ni tokens. Si es necesario loguear objetos sensibles, excluya o enmascare los campos.

### Buenas prácticas de mantenimiento

- Siga las reglas de **Prettier** y **ESLint** incluidas en el monorepo. Ejecute `npm run lint` antes de hacer commits.
- Elimine archivos y código muerto. No suba a git archivos temporales, logs, ni claves secretas.
- Cada Pull Request debe incluir un checklist indicando que se respetan estas reglas, las migraciones y pruebas se ejecutan, y la documentación está actualizada.
- Proteja la rama `main`/`develop` con revisiones obligatorias.

### Despliegue y ejecución local

- Cree un `Dockerfile` en cada microservicio. Para levantar todos los servicios use `docker-compose up --build`.
- Defina scripts de `npm` como:
  - `start:dev`: ejecuta con `ts-node-dev` en desarrollo.
  - `build`: compila a JavaScript.
  - `start:prod`: ejecuta código compilado.
- Use archivos `.env.development`, `.env.staging` y `.env.production` para cada servicio. Nunca suba secretos al repositorio.
- Para iniciar un servicio local:
  ```bash
  cd apps/<service-name>
  npm install
  npm run start:dev
  ```

### Deploy en GCP (Cloud Run y Cloud SQL)

- Compile la imagen Docker y envíela a Google Container Registry/Artifact Registry.
- Despliegue con `gcloud run deploy` especificando región (p. ej., `us-east4`) y proyecto.
- Conecte la aplicación a Cloud SQL mediante el proxy o usando la VPC Serverless. Configure las variables de entorno necesarias (`DATABASE_URL`, `JWT_SECRET`, etc.) a través de Secret Manager.
- Mantenga todos los servicios en la misma región para evitar costes de egress.

---

Estas reglas deben mantenerse actualizadas y aplicarse a cada microservicio. El cumplimiento asegura un código consistente, fácil de escalar y preparado para futuros cambios como la habilitación de multi‑tenant o la separación de servicios en contenedores independientes.
