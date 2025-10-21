# Guía de Arquitectura Multi‑Tenant

Esta guía describe cómo preparar y evolucionar el backend para soportar múltiples inquilinos (tenants) de forma aislada. Aunque el proyecto actual se ejecuta en modo monoinquilino (una sola base de datos), se ha diseñado con la flexibilidad necesaria para pasar a multi‑tenant.

## Modelo de multi‑tenant elegido

El proyecto Hero Med adoptará el modelo **base de datos por inquilino** (DB‑per‑tenant). Esto significa que cada cliente o institución contará con su propia base de datos o instancia de base de datos, ofreciendo el mayor nivel de aislamiento, seguridad y capacidad de personalización.

Aunque existen otros modelos, como el uso de esquemas por inquilino o tablas compartidas con un campo `tenant_id`, estos se consideran alternativas pero no forman parte del diseño objetivo.

En la fase actual, el entorno está configurado en modo **monoinquilino** utilizando una sola base de datos (`hero_main`). Sin embargo, el código y la infraestructura están listos para que, al registrar un nuevo inquilino, se aprovisione automáticamente una nueva base de datos (por ejemplo, `hero_tenant02`) y se configure la conexión adecuada a través del catálogo `tenants`.

## Implementación recomendada (DB por inquilino)

- Mantenga un catálogo `tenants` en una base de control con la configuración de conexión para cada inquilino: host, puerto, base de datos, usuario y contraseña.
- Cree un servicio `TenantDatabaseService` que devuelva un `DataSource` de TypeORM en función del `tenantId` recibido en cada petición.
- Obtenga el `tenantId` de:
  - Encabezado HTTP `x-tenant-id`.
  - Reclamaciones del JWT (una vez habilitada la autenticación).
- Use un pool de conexiones por inquilino y ciérrelas cuando no se utilicen.

## Implementación actual

Mientras no existan múltiples inquilinos registrados, el sistema opera con una sola base de datos llamada `hero_main`. En este modo:

- Solo se aprovisiona una base de datos física.
- Las tablas pueden incluir un campo `tenant_id` con valor predeterminado `DEFAULT` para compatibilidad futura, pero este no se utiliza para aislar datos.
- El servicio `TenantDatabaseService` devuelve siempre la misma conexión a `hero_main`.

Cuando se registra un nuevo inquilino, el automatismo de aprovisionamiento creará una nueva base de datos (por ejemplo, `hero_tenant2`) y actualizará el catálogo `tenants` con sus credenciales. A partir de ese momento, cada petición que incluya `x-tenant-id` utilizará su propia base de datos.

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

