# Guía de Despliegue en Google Cloud Run

Este documento explica cómo desplegar los microservicios del monorepo en Cloud Run. Se asume que ya ha configurado el Google Cloud SDK y que está autenticado en el proyecto.

## 1. Construir y publicar la imagen Docker

Desde la raíz del monorepo, ejecute:

```bash
# Construir la imagen del microservicio (por ejemplo visit-service)
docker build -t gcr.io/<project-id>/visit-service:latest -f apps/visit-service/Dockerfile .

# Autenticar con Container Registry
gcloud auth configure-docker

# Subir la imagen
docker push gcr.io/<project-id>/visit-service:latest
```

Asegúrese de que cada microservicio tenga su propio `Dockerfile` que copie únicamente el código necesario y ejecute `npm run build`.

## 2. Desplegar en Cloud Run

Ejecute el comando de despliegue:

```bash
gcloud run deploy visit-service   --image gcr.io/<project-id>/visit-service:latest   --region us-east4   --platform managed   --allow-unauthenticated   --port 3000   --set-env-vars NODE_ENV=production,DATABASE_URL=postgresql://<user>:<password>@<host>:<port>/<db>,JWT_SECRET=<secret>
```

- Cambie `<project-id>` por el ID de su proyecto.
- Ajuste las variables de entorno según su configuración de Cloud SQL y secret manager.
- Use `--update-secrets` para apuntar a Secret Manager en lugar de pasar secretos directamente (recomendado).

## 3. Configurar Cloud SQL

Si su microservicio utiliza Cloud SQL (PostgreSQL), asegúrese de:

1. Crear una instancia de Cloud SQL y una base de datos (por ejemplo `hero_main`).
2. Conceder permiso al servicio de Cloud Run para acceder a la instancia (`Cloud SQL Client`).
3. Conectar con Cloud SQL Proxy automático de Cloud Run usando la opción `--add-cloudsql-instances`:
   ```bash
   --add-cloudsql-instances <project-id>:us-east4:hero-db
   ```
4. Configurar la cadena de conexión en `DATABASE_URL` con el formato adecuado.

## 4. Despliegue continuo (CI/CD)

Se recomienda automatizar los despliegues mediante Cloud Build o GitHub Actions. Un flujo típico incluye:

1. Ejecutar `npm ci` y `npm run test:cov`.
2. Construir la imagen Docker y etiquetarla con el commit (`:sha` o `:v1.0.0`).
3. Subir la imagen a Container Registry.
4. Ejecutar `gcloud run deploy`.

Ejemplo de configuración de Cloud Build (`cloudbuild.yaml`):

```yaml
steps:
  - name: node:20
    entrypoint: npm
    args: ['ci']

  - name: node:20
    entrypoint: npm
    args: ['run', 'test:cov']

  - name: gcr.io/cloud-builders/docker
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/visit-service:$COMMIT_SHA', '.']

  - name: gcr.io/cloud-builders/docker
    args: ['push', 'gcr.io/$PROJECT_ID/visit-service:$COMMIT_SHA']

  - name: gcr.io/cloud-builders/gcloud
    args:
      [
        'run', 'deploy', 'visit-service',
        '--image', 'gcr.io/$PROJECT_ID/visit-service:$COMMIT_SHA',
        '--region', 'us-east4',
        '--platform', 'managed',
        '--allow-unauthenticated',
        '--set-env-vars',
        'NODE_ENV=production,DATABASE_URL=postgresql://<user>:<password>@<host>:<port>/<db>,JWT_SECRET=<secret>'
      ]

images:
  - gcr.io/$PROJECT_ID/visit-service:$COMMIT_SHA
```

## 5. Verificar el despliegue

Una vez desplegado, Cloud Run proporcionará una URL pública. Puede probar su API accediendo a `https://visit-service-<hash>-uc.a.run.app/api/docs` para ver la documentación Swagger.

Consulte `logs` con:

```bash
gcloud logging read "resource.labels.service_name='visit-service'" --limit 20
```

