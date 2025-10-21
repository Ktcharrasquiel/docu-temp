# Configuración Local de Desarrollo

Esta guía explica cómo levantar los servicios y dependencias en un entorno local para desarrollo y pruebas.

## Base de datos

### Opción 1: Postgres en Docker

Ejecute un contenedor de Postgres con los parámetros adecuados:

```bash
docker run --name hero-db -e POSTGRES_PASSWORD=admin -e POSTGRES_USER=postgres -e POSTGRES_DB=hero_main -p 5432:5432 -d postgres:15
```

Esto creará una base de datos `hero_main` con el usuario `postgres` y contraseña `admin`.

### Opción 2: Cloud SQL Proxy (usando la instancia remota)

Si necesita conectarse a una instancia de Cloud SQL real desde su máquina local, descargue el proxy e inicie:

```bash
cloud-sql-proxy.exe <project>:us-east4:hero-db --port 5432
```

Luego, configure `DATABASE_HOST=localhost` y `DATABASE_PORT=5432` en su archivo `.env`.

## Variables de entorno

Cree un archivo `.env.development` en la carpeta de cada servicio (`apps/<service>`) basándose en `.env.example`. Defina, entre otras:

```env
NODE_ENV=development
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=admin
DATABASE_NAME=hero_main
JWT_SECRET=local-secret
```

## Ejecución de servicios

Para correr un servicio aislado:

```bash
cd apps/visit-service
npm run start:dev
```

Esto iniciará el servicio en el puerto definido en `main.ts` (por defecto 3000). Repita con otros microservicios.

## Ejecución con Docker Compose

Para levantar todos los servicios y dependencias de forma orquestada:

```bash
docker-compose up --build
```

El archivo `docker-compose.yml` define los servicios de base de datos y microservicios. Asegúrese de que los puertos no colisionen.

## Pruebas

Desde la raíz del proyecto, ejecute:

```bash
npm run test:cov
```

Esto lanzará todas las pruebas unitarias y E2E de cada microservicio. Puede ejecutar pruebas individuales navegando a `apps/<service>`.

## Logging local

Los servicios utilizan `pino-pretty` en desarrollo, por lo que la salida de logs se verá legible en la consola. Cada petición incluirá un `correlationId` y, si no hay autenticación, un usuario de pruebas. No olvide desactivar el modo de pruebas al desplegar.

## Interacción con GCP en local

- Use `gcloud` para autenticar y configurar el proyecto.
- Puede listar logs en GCP con:
  ```bash
  gcloud logging read "resource.type=cloud_run_revision AND labels.service_name=<service>" --limit 10
  ```
- Para emular Pub/Sub localmente, utilice el emulador:
  ```bash
  gcloud beta emulators pubsub start --host-port=localhost:8085
  $(gcloud beta emulators pubsub env-init)
  ```

