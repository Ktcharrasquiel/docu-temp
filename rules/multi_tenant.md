# Guía de Arquitectura Multi‑Tenant

Esta guía describe cómo preparar y evolucionar el backend para soportar múltiples inquilinos (tenants) de forma aislada. Aunque el proyecto actual se ejecuta en modo monoinquilino (una sola base de datos), se ha diseñado con la flexibilidad necesaria para pasar a multi‑tenant.

## Modelos de multi‑tenant

1. **Base de datos por inquilino (model DB-per-tenant)**
   - Cada cliente tiene su propia instancia o base de datos.
   - Máximo aislamiento y seguridad.
   - Mayor coste de gestión (múltiples conexiones, escalado complejo).

2. **Esquema por inquilino**
   - Una base de datos con múltiples esquemas (por ejemplo, `tenant_a`, `tenant_b`).
   - Aislamiento moderado y configuración centralizada.
   - Requiere que las migraciones se apliquen a cada esquema.

3. **Tabla compartida con columna `tenant_id`**
   - Todas las filas comparten las mismas tablas y se distinguen por `tenant_id`.
   - Económico y fácil de gestionar, pero requiere filtros en todas las consultas.

El monorepo está preparado para los tres modelos. Actualmente se emplea el modelo **monoinquilino** (una base de datos) con columna `tenant_id` para permitir la futura migración.

## Implementación recomendada (DB por inquilino)

- Mantenga un catálogo `tenants` en una base de control con la configuración de conexión para cada inquilino: host, puerto, base de datos, usuario y contraseña.
- Cree un servicio `TenantDatabaseService` que devuelva un `DataSource` de TypeORM en función del `tenantId` recibido en cada petición.
- Obtenga el `tenantId` de:
  - Encabezado HTTP `x-tenant-id`.
  - Reclamaciones del JWT (una vez habilitada la autenticación).
- Use un pool de conexiones por inquilino y ciérrelas cuando no se utilicen.

## Implementación actual (monoinquilino)

- Existe una única base de datos `hero_main`.
- Todas las tablas tienen un campo `tenant_id` con valor `DEFAULT`.
- El código mantiene la interfaz `TenantDatabaseService` para facilitar la transición a multi‑tenant. Por ahora, siempre devuelve la misma conexión.

## Migración de monoinquilino a multi‑tenant

1. Defina el catálogo `tenants` y registre al menos dos inquilinos nuevos.
2. Ajuste el middleware para extraer `x-tenant-id` y pasarlo al `TenantDatabaseService`.
3. Cree bases de datos separadas (o esquemas) para cada inquilino.
4. Ejecute las migraciones iniciales de cada microservicio en cada nueva base de datos/esquema.
5. Añada filtros `tenantId` a todos los repositorios y queries donde aplique.
6. Pruebe todas las rutas con distintos `tenantIds` para asegurar el aislamiento de datos.

## Logging multi‑tenant

- Incluya siempre `tenantId` en los logs para identificar de qué inquilino proviene cada evento.
- Configure Cloud Logging para crear *sinks* por inquilino y enviar logs a buckets o proyectos diferentes si es necesario.

## Consideraciones de seguridad

- Las credenciales de cada base de datos deben almacenarse en Secret Manager con permisos específicos por inquilino.
- Controle el acceso a los recursos compartidos (Pub/Sub, Cloud Storage) mediante etiquetas o prefijos con el `tenantId`.
- Revise las políticas de CORS y autenticación para que cada inquilino solo acceda a su información.

