# Instalación del Entorno de Desarrollo (Windows, Mac y Linux)

Este documento explica cómo preparar el entorno de desarrollo para trabajar con el monorepo de Hero Med. Se muestran pasos específicos para Windows, pero los usuarios de Mac OS y Linux pueden adaptarlos fácilmente.

## Prerrequisitos

- **Git**: para clonar y trabajar con el repositorio.
- **Node.js** (versión LTS 20): para ejecutar NestJS.
- **Docker Desktop**: para ejecutar contenedores y `docker-compose`.
- **Google Cloud SDK** (`gcloud`): para desplegar servicios en Cloud Run y gestionar recursos de GCP.

### Instalación en Windows

1. **Instalar Chocolatey (si no lo tienes)**
   
   Abre PowerShell como Administrador y ejecuta:

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force;    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
   ```

2. **Instalar Node.js, Docker Desktop y Google Cloud SDK**

   ```powershell
   choco install nodejs-lts docker-desktop googlecloudsdk git -y
   ```

   Reinicia tu máquina si es necesario y abre Docker Desktop para finalizar la instalación.

3. **Configurar Google Cloud SDK**

   ```powershell
   gcloud auth login
   gcloud config set project <tu-project-id>
   gcloud config set compute/region us-east4
   gcloud config set compute/zone us-east4-b
   ```

### Instalación en Mac OS (Homebrew)

```bash
brew install node docker google-cloud-sdk git
brew install --cask docker

# Iniciar sesión y configurar proyecto
gcloud auth login
gcloud config set project <tu-project-id>
gcloud config set compute/region us-east4
gcloud config set compute/zone us-east4-b
```

### Instalación en Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install -y nodejs npm docker.io docker-compose git
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-442.0.0-linux-x86_64.tar.gz
sudo tar -xzf google-cloud-cli-*.tar.gz -C /usr/local
/usr/local/google-cloud-sdk/install.sh

# Configurar
gcloud init
```

## Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/hero-monorepo.git
cd hero-monorepo
```

## Instalar dependencias

Ejecute el siguiente comando en la raíz para instalar dependencias compartidas y en cada microservicio:

```bash
npm ci
npm run install:all  # Script opcional que recorre apps/* y ejecuta npm install
```

## Configurar variables de entorno

- Cada microservicio tiene un archivo `.env.example`. Copie este archivo a `.env.development` y ajuste los valores.
- Variables comunes: `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USER`, `DATABASE_PASSWORD`, `DATABASE_NAME`, `JWT_SECRET`.
- No suba archivos `.env` al repositorio.

## Levantar la base de datos local

Puede usar Postgres en Docker:

```bash
docker run --name hero-db -e POSTGRES_PASSWORD=admin -p 5432:5432 -d postgres:15
```

O, si usa Cloud SQL en desarrollo, instale el proxy:

```bash
cloud-sql-proxy.exe <project-id>:us-east4:hero-db --port 5432
```

## Ejecutar un microservicio en desarrollo

1. Navegue hasta la carpeta del microservicio:

   ```bash
   cd apps/visit-service
   npm run start:dev
   ```

2. Acceda a la API en `http://localhost:3000/` (o el puerto configurado) y la documentación Swagger en `http://localhost:3000/api/docs`.

## Levantar todos los microservicios con Docker

Para ejecutar toda la arquitectura (base de datos + servicios) en contenedores, utilice el archivo `docker-compose.yml` en la raíz:

```bash
docker-compose up --build
```

Esto levantará Postgres, Redis (si se utiliza) y cada microservicio con sus puertos mapeados.

## Siguientes pasos

- Ejecute `npm run test:cov` para asegurarse de que las pruebas funcionan.
- Siga las guías de `deployment.md` para desplegar en Cloud Run.
- Consulte `new_microservice.md` para crear un nuevo microservicio.

