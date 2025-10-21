# Reglas de Workflow de Prompts y Branches

Este documento describe cómo generar, clarificar, versionar y ejecutar prompts de IA en el monorepo. Todas las acciones deben alinearse con la arquitectura de microservicios basada en NestJS/TypeORM y las reglas descritas en `backend_rules.md`.

## Estructura de directorios para prompts

Los prompts se organizan en la carpeta `.cursor/prompts`. Se recomienda la siguiente estructura:

```
.cursor/
  prompts/
    features/
      <feature-name>/
        <prompt-id>_<prompt-name>.prompt.md
    templates/
      ... (plantillas base)
  rules/
    backend_rules.md
    prompt_workflow.md
    migration_cloudrun.md
    multi_tenant.md
```

- **features/**: agrupa los prompts por funcionalidad o microservicio.
- **templates/**: contiene plantillas reutilizables para crear nuevos prompts (por ejemplo, crear un nuevo microservicio, un nuevo endpoint o una migración).
- **rules/**: define las reglas de backend, el workflow de prompts y otras guías auxiliares.

## Flujo de trabajo automatizado

Siga siempre este flujo antes de ejecutar cualquier prompt:

1. **Identificar el prompt**
   - Localice el archivo `.prompt.md` correspondiente dentro de `.cursor/prompts/features`.
   - Revise su metadata: `prompt_id`, `feature`, `version`, `ejecutado`.
   - Confirme que `ejecutado: false`.

2. **Crear una rama de desarrollo**
   - Trabaje siempre partiendo de `develop`.
   - Use nomenclatura `feature/<prompt-id>-<feature-name>`.
   - Ejemplo:
     ```bash
     git checkout develop
     git pull origin develop
     git checkout -b feature/002-create-visit
     ```

3. **Ejecutar el prompt**
   - Lea y entienda la instrucción detallada del prompt.
   - Aclare cualquier duda con preguntas (DTOs, validaciones, autenticación, logs, etc.).
   - Genere el código siguiendo las reglas de `backend_rules.md` (arquitectura, nombres, logs, pruebas, swagger, etc.).
   - Implemente tests unitarios y e2e.
   - Actualice la documentación Swagger.

4. **Commit y Push**
   - Haga commits claros y concisos usando conventional commits: `feat(<feature>): <descripción> según prompt <ID>`.
   - Ejemplo:
     ```bash
     git add .
     git commit -m "feat(visits): implementar GET /visits con filtros según prompt 002"
     git push --set-upstream origin feature/002-create-visit
     ```

5. **Actualizar estado del prompt**
   - Marque el prompt como `ejecutado: true` en el archivo de metadata (por ejemplo, `index.json` o en el propio `.prompt.md`).
   - Registre fecha de ejecución y versión final.

6. **Crear Pull Request**
   - Abra un PR hacia `develop`.
   - Incluya descripción del objetivo, resumen de la implementación, checklist de pruebas y documentación.
   - Espere revisiones y ajuste según comentarios.

## Clarificación y generación de prompts

Cuando un compañero solicita un nuevo prompt, la IA debe:

1. **Recibir la instrucción simple**, por ejemplo: “Generar endpoint POST /users para crear un usuario”.
2. **Hacer preguntas de clarificación** si la petición es ambigua:
   - ¿Cuáles son las propiedades del DTO?
   - ¿Requiere autenticación?
   - ¿Debe retornar un JSON con un formato específico?
   - ¿Se necesita registrar logs o métricas?
3. **Construir el prompt detallado** una vez aclarados los requisitos, siguiendo la estructura:
   ```md
   ---
   prompt_id: "003_create-user"
   feature: "USER_CREATION"
   autor: "nombre_apellido"
   fecha: "2025-10-20"
   version: 1
   ejecutado: false
   tags: ["endpoint", "create"]
   ---

   ## Instrucción original
   
   Generar endpoint POST /users para crear un usuario.

   ## Preguntas de clarificación
   
   - ¿Qué campos incluye el usuario? (name, email, password, …)
   - ¿Hay validaciones adicionales?
   - ¿Se debe enviar un email de bienvenida?

   ## Prompt detallado generado
   
   Implementa un controlador `UserController` con un método `createUser()` en NestJS 11. El método debe recibir un `CreateUserDto` con las propiedades `name`, `email` y `password`, validado por `class-validator`. Usa `bcrypt` para cifrar la contraseña, guarda el usuario en PostgreSQL mediante TypeORM y retorna el usuario creado sin el campo `password`. Documenta con Swagger y agrega tests unitarios y e2e. Incluye logs estructurados.
   ```

Esta estructura asegura que todos los prompts sean claros, reproducibles y trazables.

## Conexión con el backend

Los prompts siempre deben respetar las reglas definidas en `backend_rules.md`. En particular:

- No utilizar patrón CQRS; los controladores, servicios y repositorios deben ser coherentes con la arquitectura definida.
- Incluir pruebas unitarias y E2E.
- Generar logs y documentación Swagger.
- Mantener consistencia en el manejo de la base de datos (incluyendo campo `tenant_id` en DTOs cuando aplique).

## Tips para un flujo de trabajo eficiente

- Revise prompts existentes antes de crear uno nuevo; puede reutilizar plantillas o adaptar prompts pasados.
- Asegúrese de versionar los prompts cuando cambie algún detalle importante (por ejemplo, un nuevo campo en el DTO).
- Documente las decisiones y clarificaciones en el propio prompt para que otros desarrolladores comprendan el contexto.
- Automatice la creación de ramas y PRs mediante scripts o acciones de GitHub.

