# Carpeta `.cursor`

Esta carpeta contiene reglas, plantillas de prompts y documentación auxiliar para el proyecto Hero Med. Su objetivo es proporcionar a la IA (Cursor, ChatGPT u otras herramientas) la información necesaria para generar código y documentación coherente con la arquitectura y las prácticas del equipo.

## Estructura

```
.cursor/
├── rules/
│   ├── backend_rules.md          # Reglas de arquitectura y desarrollo backend
│   ├── prompt_workflow.md        # Flujo de generación y ejecución de prompts
│   ├── migration_cloudrun.md     # Guía para migrar servicios a Cloud Run
│   └── multi_tenant.md           # Guía para soportar múltiples inquilinos
├── prompts/
│   ├── templates/                # Plantillas base para prompts
│   │   ├── create_microservice.prompt.md
│   │   ├── create_endpoint.prompt.md
│   │   └── update_schema.prompt.md
│   └── index.json                # Ejemplo de índice de prompts
├── docs/
│   ├── installation.md           # Cómo instalar las herramientas
│   ├── local_setup.md            # Cómo levantar el entorno local
│   ├── deployment.md             # Cómo desplegar en Cloud Run
│   ├── new_microservice.md       # Cómo crear un nuevo microservicio
│   ├── architecture.md           # Descripción y diagramas de la arquitectura
│   └── diagrams/
│       ├── architecture_diagram.png
│       └── monorepo_structure.png
└── README.md                     # Este archivo
```

## Uso

1. Lea `rules/backend_rules.md` para conocer las normas de desarrollo backend.
2. Lea `rules/prompt_workflow.md` antes de crear o ejecutar cualquier prompt.
3. Cuando necesite crear un nuevo microservicio o endpoint, utilice las plantillas en `prompts/templates/` como punto de partida.
4. Consulte `docs/` para instalación, despliegue y ampliación de la arquitectura.

## Contribuciones

Mantenga esta carpeta actualizada. Si añade nuevas reglas o plantillas, recuerde documentarlas y ajustar los diagramas en `docs/architecture.md`.

