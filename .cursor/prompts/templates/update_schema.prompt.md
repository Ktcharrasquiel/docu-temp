---
prompt_id: "003_update_database_schema"
feature: "UPDATE_SCHEMA"
autor: "your_name"
fecha: "2025-10-20"
version: 1
ejecutado: false
tags: ["database", "migration"]
---

## Instrucción original

Actualizar el esquema de base de datos para agregar o modificar una tabla/columna.

## Preguntas de clarificación

- ¿Qué entidad o tabla necesita cambios?
- ¿Cuáles son los nuevos campos o modificaciones? (tipo, longitud, nulos)
- ¿Hay datos existentes que deban migrarse o transformarse?
- ¿La migración afecta a múltiples microservicios?
- ¿Se requiere migrar datos por tenant o globalmente?

## Prompt detallado generado

Crea una nueva migración en `apps/<service-name>/migrations` usando TypeORM CLI con un nombre descriptivo. Modifica la entidad correspondiente en `src/entities/`. Ajusta DTOs y validaciones si es necesario. Si los cambios afectan a varios microservicios (p. ej., modificando la entidad `User`), coordina las migraciones en cada servicio. Incluye instrucciones en `docs/migrations.md` sobre cómo ejecutar la migración y cómo revertirla. Añade pruebas que aseguren que las nuevas columnas se guardan y se leen correctamente.
