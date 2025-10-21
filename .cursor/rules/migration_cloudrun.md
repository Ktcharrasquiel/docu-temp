# Guía de Migración de Microservicios a Cloud Run Independiente

Esta guía explica cómo extraer un microservicio desde el monorepo y desplegarlo como un contenedor independiente en Google Cloud Run. El objetivo es permitir escalar servicios de forma individual sin afectar al resto del proyecto.

## Requisitos previos

- Estructura de monorepo siguiendo las reglas de `backend_rules.md`.
- El microservicio a migrar no debe depender directamente del código de otro microservicio; solo puede usar librerías compartidas (`libs/`).
- Tener instalado Google Cloud SDK (`gcloud`) y estar autenticado en el proyecto correcto.
- Disponer de acceso a Container Registry o Artifact Registry.

## Pasos para migrar un microservicio

### 1. Revisar dependencias

Ejecute un análisis de dependencias para asegurarse de que el microservicio (`apps/<service>`) no importa código de otros servicios. Verifique que solo use módulos de `libs/`.

### 2. Ajustar la configuración

- Revise el `package.json` del microservicio y asegúrese de que los scripts (`start:prod`, `start:dev`, `build`) funcionan de forma independiente.
- Si utiliza variables de entorno comunes, cámbielas a variables específicas del servicio o añada prefijos para evitar conflictos.

### 3. Crear una imagen Docker aislada

En el directorio raíz del monorepo, cree un `Dockerfile` específico para el servicio (si no existe):

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY apps/<service> ./apps/<service>
COPY libs ./libs
RUN npm run build --workspace=@hero/<service>
CMD ["node", "dist/apps/<service>/main.js"]
```

Compile la imagen y súbala al registro:

```bash
docker build -t gcr.io/<project-id>/<service>:v1 .
gcloud auth configure-docker
docker push gcr.io/<project-id>/<service>:v1
```

### 4. Desplegar en Cloud Run

Despliegue la imagen en Cloud Run especificando la región y las variables de entorno:

```bash
gcloud run deploy <service>   --image gcr.io/<project-id>/<service>:v1   --region us-east4   --platform managed   --allow-unauthenticated   --set-env-vars DATABASE_URL=postgresql://... ,JWT_SECRET=... ,LOG_LEVEL=info
```

### 5. Actualizar referencias

- Actualice el `frontend` o los clientes para que consuman la nueva URL del servicio (`https://<service>-<hash>-uc.a.run.app`).
- Si otros microservicios enviaban peticiones internas a este servicio dentro del monorepo, cambie las llamadas al nuevo endpoint de Cloud Run.

### 6. Automatizar despliegues

Cree un pipeline de CI/CD (GitHub Actions o Cloud Build) dedicado al microservicio que ejecute:

1. Instalación y compilación (`npm ci`, `npm run build`).
2. Pruebas unitarias y e2e.
3. Construcción de la imagen Docker y push al registro.
4. Despliegue en Cloud Run.

### 7. Retirar el servicio del monorepo (opcional)

Una vez comprobado que el servicio independiente funciona correctamente y que sus dependencias están resueltas:

- Elimine la carpeta `apps/<service>` del monorepo o conviértala en un submódulo git.
- Mantenga las librerías compartidas (`libs/`) en un repositorio aparte o publíquelas como paquetes privados (`@hero/common`).

## Consideraciones

- Asegúrese de que todas las llamadas entre servicios pasen por HTTP(s) o Pub/Sub; nunca por imports directos.
- Verifique las políticas de IAM y los secretos: cada servicio puede tener su propia identidad de servicio y sus propios permisos.
- Centralice logs y métricas en Cloud Logging y Cloud Monitoring para mantener visibilidad tras la migración.

