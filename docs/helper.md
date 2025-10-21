# Guía de Ayuda para Procedimientos y Código

Este documento sirve como **ayudante interactivo** para los desarrolladores y usuarios que trabajan con los prompts de IA y la arquitectura NestJS del monorepo. Aquí encontrarás consejos prácticos para utilizar las plantillas de prompts, una explicación de la estructura de código generada y referencias a la documentación oficial de NestJS.

## 1. Cómo solicitar un nuevo código de forma efectiva

Al pedirle a la IA que genere código (por ejemplo, un microservicio o un endpoint), proporciona una descripción básica de la funcionalidad y **deja que el agente te guíe**. La IA formulará preguntas de clarificación para recopilar toda la información necesaria antes de construir el prompt detallado. Estas son algunas categorías de preguntas que puedes esperar:

- **Propiedades y tipos**: ¿Qué campos componen la entidad o el DTO? ¿Qué tipos de datos (string, number, date, boolean) se usan?  
- **Validaciones y reglas de negocio**: ¿Existen restricciones como unicidad, rangos o formatos específicos? ¿Hay campos opcionales u obligatorios?  
- **Autenticación y autorización**: ¿El endpoint requiere autenticación con JWT? ¿Qué roles o permisos son necesarios?  
- **Persistencia y relaciones**: ¿Cómo se guardan los datos? ¿Qué entidades se relacionan? ¿Se necesitan transacciones?  
- **Eventos y side‐effects**: ¿Al crear o modificar datos se debe publicar un mensaje en Pub/Sub, invalidar caché o enviar un correo electrónico?  
- **Logs y métricas**: ¿Qué se debe registrar en los logs estructurados? ¿Se necesita capturar un `tenantId` y `correlationId`?

Responder a estas preguntas permitirá al agente generar un prompt completo y evitar ambigüedades en la implementación.

## 2. Comprender la estructura del código NestJS

La arquitectura recomendada sigue las mejores prácticas de NestJS. A continuación se describen los componentes principales y se citan extractos de la documentación oficial para mayor contexto:

### Controladores

Los **controladores** manejan las solicitudes HTTP y delegan la lógica al servicio correspondiente. Según la documentación oficial, un controlador recibe las peticiones entrantes, manipula las solicitudes y devuelve respuestas al cliente【235039593161354†L228-L235】. El decorador `@Controller('ruta')` define un prefijo para las rutas del controlador【235039593161354†L246-L253】.

Ejemplo:

```typescript
@ApiTags('users')
@ApiBearerAuth()
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get(':id')
  async findOne(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.findOne(id);
  }
}
```

### Servicios y Providers

Los **servicios** contienen la lógica de negocio y se inyectan en los controladores. NestJS los trata como *providers*, que se definen con el decorador `@Injectable()`【50502765477633†L220-L235】. Los servicios se inyectan mediante el constructor del controlador, lo que habilita la inversión de control (IoC)【50502765477633†L352-L357】.

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private usersRepository: Repository<User>,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const entity = this.usersRepository.create(dto);
    return this.usersRepository.save(entity);
  }

  async findOne(id: string): Promise<User> {
    return this.usersRepository.findOneBy({ id });
  }
}
```

### Módulos

Los **módulos** agrupan controladores, servicios y otros providers relacionados. Un módulo se declara con `@Module()` e incluye las propiedades `providers`, `controllers`, `imports` y `exports`【108285950678641†L220-L245】. Exportar un provider permite compartir la misma instancia en otros módulos【108285950678641†L319-L349】.

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### DTOs, Pipes y Validaciones

Las clases **DTO** (Data Transfer Objects) definen la forma de los datos de entrada y salida. Utilizan `class-validator` y `class-transformer` para aplicar restricciones y transformar tipos. Las **pipes** se ejecutan antes de que los controladores manejen la solicitud; sirven para validar o transformar datos【978693471184083†L226-L244】. Por ejemplo, `ParseIntPipe` convierte parámetros de cadena a enteros【978693471184083†L280-L297】.

```typescript
export class CreateUserDto {
  @IsNotEmpty()
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}
```

### Buenas prácticas de generación

- Usa la CLI de Nest para generar controladores, servicios y módulos (`nest generate controller users`, `nest generate service users`). Esto sigue las convenciones oficiales【235039593161354†L279-L280】【108285950678641†L273-L274】.  
- Mantén un archivo por entidad, DTO o servicio para una mejor organización.  
- Documenta cada endpoint con Swagger usando decoradores como `@ApiOperation()`, `@ApiResponse()`, etc.

## 3. Verificar y validar el código generado

Una vez que la IA genere el código (controlador, servicio, DTO, pruebas, etc.), sigue estos pasos para validarlo:

1. **Compila y ejecuta los tests** con `npm run test` y `npm run test:cov`. Asegúrate de que las pruebas unitarias y e2e cubren los casos más importantes, incluidos códigos de estado y estructura de respuestas.  
2. **Revisa la documentación Swagger** en `http://localhost:<puerto>/api/docs` para verificar que la API se documenta correctamente y que los modelos de entrada y salida son coherentes.  
3. **Prueba manualmente la API** con herramientas como Postman o cURL. Envía solicitudes con datos válidos e inválidos para observar las respuestas y los mensajes de error.  
4. **Valida los logs** generados con `pino-pretty` en desarrollo o en Cloud Logging en GCP. Verifica que incluyan campos como `tenantId`, `correlationId` y `userId`.  
5. **Revisa las migraciones** si se crearon nuevas tablas o columnas. Ejecute `typeorm migration:run` y comprueba que la base de datos refleja los cambios.

## 4. Responder a los usuarios finales

Es posible que los usuarios de la API (por ejemplo, consumidores de un servicio frontend) necesiten comprender cómo utilizar los nuevos endpoints. Para ello:

- **Documenta los contratos de respuesta**: especifica en los DTO de salida (`ResponseDto`) los campos devueltos y sus tipos.  
- **Incluye ejemplos de payload** en Swagger (`@ApiResponse({ schema: { example: { id: 'uuid', name: 'John Doe' } } })`).  
- **Asegura códigos de estado consistentes** (por ejemplo, `201 Created` para POST exitoso, `404 Not Found` para recursos inexistentes).  
- **Gestiona errores con excepciones de NestJS** (`HttpException`, `NotFoundException`, etc.) para enviar mensajes estandarizados.

## 5. Preguntas frecuentes y troubleshooting

- **¿Cómo añado un nuevo campo a un DTO y su validación?**  
  Simplemente edita la clase DTO, añade la propiedad y aplica los decoradores de validación necesarios. No olvides actualizar el Swagger con `@ApiProperty()`.  
- **¿Cómo integro un servicio externo?**  
  Crea un provider que encapsule la llamada al servicio externo (por ejemplo, una API REST). Inyecta este provider en tu servicio principal.  
- **¿Cómo publico un evento en Pub/Sub?**  
  Inyecta el cliente de Pub/Sub y llama a `publish()` dentro de tu servicio; asegúrate de manejar los errores.  
- **¿Por qué falla la migración?**  
  Verifica que la configuración de TypeORM apunte a la base de datos correcta y que `synchronize: false`. Si hay conflictos, revisa los nombres de columnas y tipos de datos.  
- **¿Cómo corrijo un error de validación?**  
  Revisa los decoradores en el DTO y ajusta los tipos; asegúrate de que los datos enviados cumplen las reglas de `class-validator`.

## 6. Referencias a la documentación oficial

Para más detalles, consulta las guías de NestJS:

- **Controllers**: responsabilidad de las rutas y delegación a servicios【235039593161354†L228-L235】.  
- **Modules**: definición de `@Module()` y cómo agrupar providers【108285950678641†L220-L245】.  
- **Providers & servicios**: inyección de dependencias y uso de `@Injectable()`【50502765477633†L220-L235】.  
- **Pipes y validación**: transformación y validación de datos de entrada【978693471184083†L226-L244】【978693471184083†L280-L297】.  
- **CLI**: generar scaffolds de controladores y servicios automáticamente【235039593161354†L279-L280】【108285950678641†L273-L274】.

La documentación oficial completa está disponible en [NestJS Docs](https://docs.nestjs.com/). Consulta siempre las secciones pertinentes cuando tengas dudas o necesites ampliar la información.
