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


## Parámetros y criterios obligatorios en los prompts

Para mantener la trazabilidad y la coherencia, cada prompt debe incluir la siguiente metadata en su cabecera YAML:

- `version`: versión del prompt. Incremente este número cuando cambie la especificación.  
- `autor`: nombre y apellido de quien define el prompt.  
- `fecha`: fecha de creación o última actualización del prompt.  
- `ejecutado`: indica si el prompt ya fue implementado (`true` o `false`).  
- `tags`: lista de etiquetas descriptivas.  
- `pipeline`: identificador del pipeline de CI/CD asociado (si aplica).

Además, incluya en cada prompt una sección `## Documentación` donde especifique qué archivos de documentación deberán actualizarse (Swagger, `README.md`, notas de migración, etc.).

## Controles obligatorios antes de finalizar un prompt

Antes de marcar un prompt como ejecutado y crear el Pull Request, el equipo debe asegurarse de cumplir con los siguientes pasos:

1. **Validar pruebas y cobertura**. Ejecute `npm run test:cov` en la raíz del proyecto para correr todas las pruebas unitarias y E2E. Asegúrese de que todas las pruebas pasan y la cobertura cumple con el umbral definido (por ejemplo, 85 %).  
2. **Actualizar documentación**. Genere o actualice la especificación Swagger del microservicio y cualquier documento técnico relacionado.  
3. **Commit y mensajes claros**. Use conventional commits (`feat`, `fix`, `docs`, etc.) con una descripción breve y referencia al `prompt_id`.  
4. **Crear Pull Request**. Incluya en la descripción el objetivo del prompt, el resumen de cambios, las instrucciones para probar la implementación y un checklist de pruebas.  
5. **Confirmación del usuario**. Antes de fusionar el PR, confirme con la persona que solicitó el prompt que todas sus respuestas y requisitos se han incorporado correctamente.  

Estos controles son obligatorios para garantizar que los prompts se integran de forma segura y ordenada en el código base.

## Referencias y buenas prácticas de NestJS (documentación oficial)

Esta guía de workflow se nutre de las recomendaciones de la documentación oficial de NestJS. Para garantizar que todos los prompts y microservicios se adhieran a los principios del framework, considera las siguientes prácticas:

- **Controladores**: Los controladores son responsables de gestionar las solicitudes entrantes y enviar respuestas. Se definen con el decorador `@Controller()` y sus métodos se anotan con decoradores HTTP como `@Get()`, `@Post()`, etc. La documentación oficial indica que su propósito es manejar solicitudes específicas y delegar la lógica compleja a los servicios【235039593161354†L228-L235】. También se recomienda agrupar rutas relacionadas mediante un prefijo en `@Controller()`【235039593161354†L246-L253】.

- **Servicios y proveedores**: La lógica de negocio debe encapsularse en servicios u otros proveedores anotados con `@Injectable()`. Estos proveedores son la unidad básica de inyección de dependencias en Nest y pueden ser servicios, repositorios o fábricas. Los controladores deben delegar tareas complejas a los proveedores, mientras que el contenedor de Nest se encarga de instanciarlos y resolver sus dependencias【50502765477633†L220-L235】. Para inyectar un servicio en un controlador basta con declararlo en el constructor utilizando el modificador `private`【50502765477633†L352-L357】.

- **Módulos**: Los módulos agrupan componentes relacionados. Cada aplicación Nest tiene al menos un módulo raíz, pero se recomienda crear módulos de característica para organizar mejor el código. El decorador `@Module()` acepta las propiedades `providers`, `controllers`, `imports` y `exports` para declarar los componentes asociados【108285950678641†L220-L245】. Encapsular y exportar servicios mediante módulos permite compartir una misma instancia entre distintos módulos y facilita la reutilización【108285950678641†L319-L349】.

- **Pipes**: Las pipes se utilizan para transformar y validar datos antes de que lleguen al método del controlador. Existen dos casos de uso típicos: transformar datos de entrada al tipo deseado y validar los datos, lanzando una excepción si son incorrectos【978693471184083†L226-L244】. Nest proporciona pipes integradas como `ValidationPipe` y `ParseIntPipe`, que pueden vincularse a parámetros de rutas para asegurarse de que los valores cumplen las expectativas【978693471184083†L280-L297】. Cuando una pipe lanza una excepción, el método del controlador no se ejecuta【978693471184083†L246-L251】.

- **Herramientas CLI**: La CLI de NestJS facilita la generación de clases básicas. Se recomienda ejecutar comandos como `nest g controller cats`, `nest g service cats` o `nest g module cats` para crear controladores, servicios y módulos siguiendo las convenciones del framework【235039593161354†L279-L280】【108285950678641†L273-L274】.

Estas prácticas oficiales deben reflejarse en la generación y ejecución de prompts. Asegúrate de que cada prompt respete esta estructura: los controladores reciben las peticiones y delegan la lógica a los servicios, las entidades se agrupan en módulos coherentes y las entradas se validan mediante pipes.
