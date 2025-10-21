# Integración de SonarQube para Calidad de Código

SonarQube es una plataforma de análisis estático que ayuda a detectar bugs, vulnerabilidades y code smells en el código fuente. Integrarlo en el flujo de CI/CD del monorepo garantiza que los microservicios mantengan un nivel de calidad alto y uniforme.

## 1. Configuración inicial

Hay dos opciones principales:

1. **SonarQube autohospedado**: instale un servidor SonarQube (Community, Developer o Enterprise) en una máquina o clúster. Puede utilizar una imagen Docker (`sonarqube:community`).
2. **SonarCloud**: servicio gestionado por Sonar que ofrece todas las funcionalidades y requiere menos mantenimiento. Requiere crear una organización y un token de acceso.

En ambos casos necesitará un **token de análisis** para autenticar el análisis.

## 2. Preparar el proyecto

En la raíz del monorepo, cree un archivo `sonar-project.properties` o configure las propiedades mediante variables de entorno. Un ejemplo básico:

```
sonar.projectKey=hero-monorepo
sonar.projectName=Hero Monorepo
sonar.sources=apps,libs
sonar.exclusions=**/test/**,**/*.spec.ts
sonar.ts.tslintpath=node_modules/tslint
sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.tests=apps/**/test
sonar.test.inclusions=**/*.spec.ts,**/*.test.ts
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

Ajuste las rutas según la estructura y la configuración de su proyecto. El análisis de cobertura puede integrarse utilizando el reporte `lcov.info` generado por Jest (`npm run test:cov`).

## 3. Integración en el pipeline CI/CD

Ejecute el análisis como parte de su pipeline de CI (GitHub Actions, GitLab CI, Cloud Build). Ejemplo con **GitHub Actions**:

```yaml
name: SonarQube
on: [push]
jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run test:cov
      - uses: sonarsource/sonarqube-scan-action@v2
        with:
          projectBaseDir: .
          args: >
            -Dsonar.projectKey=hero-monorepo
            -Dsonar.organization=tu-organizacion
            -Dsonar.sources=.
            -Dsonar.host.url=https://sonarcloud.io
            -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

Para **Cloud Build**, utilice un paso que instale `sonar-scanner-cli` y ejecute `sonar-scanner` con los parámetros adecuados.

## 4. Quality Gates y perfiles

Configure **Quality Gates** en el servidor SonarQube para definir umbrales mínimos (por ejemplo, cobertura > 85 %, cero bugs críticos, deuda técnica menor a cierto porcentaje). Si el análisis no alcanza el gate, falle el pipeline para obligar a corregir problemas antes de fusionar el código.

Utilice los **Quality Profiles** para adaptar las reglas a TypeScript/NestJS. Puede activar reglas específicas (por ejemplo, evitar `console.log`, detectar SQL injection) y desactivar las que no apliquen a su proyecto.

## 5. Recomendaciones

- Ejecute SonarQube en cada PR para detectar problemas tempranamente.
- Configure exclusiones en `sonar-project.properties` para carpetas generadas, archivos de pruebas y migraciones.
- Revise los reportes y asigne responsables para corregir vulnerabilidades y code smells.
- Integre la verificación de Quality Gate con el flujo de revisión de PR. No se debería aprobar un PR si el análisis no pasa.

Más información: consulte la documentación oficial de SonarQube para proyectos TypeScript y Node.js.