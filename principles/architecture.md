# 🏗️ Arquitectura y Responsabilidad de Cada Capa

---

## 1.1 Visión General

Este documento define la arquitectura del backend de Muta, diseñada con los siguientes principios:

### Objetivos

| Objetivo | Descripción |
|----------|-------------|
| **Simplicidad** | Código fácil de entender y mantener |
| **Consistencia** | Estándar sólido que todos sigan |
| **Rapidez** | Estructura lite que no complique el desarrollo |
| **Claridad** | Guía clara para el equipo |

### Tipo de Arquitectura

Utilizamos una **arquitectura hexagonal lite**, simplificada para nuestro contexto:

- Separación clara de responsabilidades por capa
- Flujo unidireccional y predecible
- Sin complejidad innecesaria de otros enfoques
- Fácil de onboarding para nuevos desarrolladores

### Por qué esta arquitectura

| Beneficio | Para qué sirve |
|-----------|---------------|
| Código legible | Cualquier dev puede entender el flujo |
| Estándar fijo | Todos escriben igual, sin discusiones |
| Fácil debugging | Sabés dónde está cada cosa |
| Escalable | Agregar features sin romper estructura |

---

| # | Capa | Descripción |
|---|------|-------------|
| 1 | Controller | Recibe HTTP, valida, delega al Adapter |
| 2 | Adapter | Traduce y decide Service o UseCase |
| 3 | Service | CRUD básico |
| 4 | UseCase | Lógica compleja |
| 5 | Repository | Acceso a BD |
| 6 | Entity | Modelo de datos |
| 7 | Enum | Catálogos/valores fijos |
| 8 | Exception | Manejo de errores |
| 9 | Templates | Clases base para extender |

## 1.3 Regla de Arquitectura: Una Entity = Un Controller

> **Regla: Cada recurso/entidad tiene su propio Controller, Adapter, Service, UseCase y Repository.**

| Entity | Controller | Endpoint |
|--------|------------|----------|
| User | UserController | `/api/v1/users` |
| UserPermission | UserPermissionController | `/api/v1/user-permissions` |
| Orden | OrdenController | `/api/v1/ordenes` |

**No importar si el recurso es "sub" de otro.** Si tiene lógica propia, va separado.

| Capa | Responsabilidad | Qué hace | Qué NO hace |
|------|----------------|----------|-------------|
| **Controller** | Recibir requests HTTP | • Recibe la petición<br>• Valida el DTO<br>• Delega al Adapter<br>• Devuelve HTTP response | • Lógica de negocio<br>• Consultas a BD<br>• Transformaciones complejas |
| **Adapter** | Coordinar y traducir | • Traduce DTOs<br>• Decide usar Service o UseCase<br>• Orchestación simple | • Lógica de negocio compleja<br>• Transacciones múltiples |
| **Service** | Operaciones CRUD básicas | • Create, Read, Update, Delete<br>• Consultas simples<br>• Búsquedas con filtros | • Lógica de negocio compleja<br>• Múltiples operaciones |
| **UseCase** | Lógica compleja | • Orchestración de servicios<br>• Transacciones múltiples<br>• Validaciones de negocio | • Cosas simples (para eso es Service) |
| **Repository** | Acceso a datos | • Consultas a la BD<br>• Queries definidos | • Lógica de negocio |
| **Entity** | Modelo de datos | • Define la tabla<br>• Validaciones de campo<br>• Enums internos | • Lógica de negocio |
| **Enum** | Catálogos de dominio | • Valores fijos (estados, tipos)<br>• Código + descripción<br>• Búsqueda por código | • Valores dinámicos<br>• Entidades |
| **Exception** | Manejo de errores | • Excepciones específicas por flujo<br>• Handler genérico<br>• Respuestas HTTP coherentes | • Excepciones no controladas |
| **Templates** | Clases base | • BaseService para CRUD<br>• BaseUseCase para lógica compleja<br>• Extender, no copiar | • Lógica personalizada (van en las capas acima) |

---

## 1.3 Flujo de una Request

```
HTTP Request
    │
    ▼
┌─────────────────────────────────────┐
│         Controller                  │ ◄── 1. Recibe request
│  - Define endpoint                  │     2. Valida DTO
│  - Documenta (OpenAPI)              │     3. Llama al Adapter
│  - Delega toda lógica               │     
└──────────────┬──────────────────────┘
                │ DTO Request
                ▼
┌─────────────────────────────────────┐
│         Adapter                     │ ◄── 1. Traduce DTO → objeto interno
│  - Interfaz + Implementación       │     2. Decide: Service o UseCase
│  - Translation layer              │     3. Traduce respuesta → DTO
└──────────────┬──────────────────────┘
                │
        ┌───────┴───────┐
        ▼               ▼
┌──────────────┐  ┌─────────────────┐
│   Service    │  │    UseCase      │ ◄── Service: operaciones básicas
│  (CRUD base) │  │ (lógica        │     UseCase: lógica compleja
│             │  │  compleja)    │
��──────┬───────┘  └────────┬────────┘
        │                    │
        ▼                    ▼
┌─────────────────────────────────────┐
│       Repository                    │ ◄── JpaRepository
│  - Acceso a datos                   │     + Query methods
└─────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────┐
│         Entity                       │ ◄── JPA Entity
└─────────────────────────────────────┘
```

---

## 1.4 Cuándo Usar Service vs UseCase

| Escenario | Usar | Ejemplo |
|-----------|------|--------|
| Crear, actualizar, eliminar registro | **Service** | Crear usuario |
| Consultar por ID, buscar todos | **Service** | Buscar usuario por email |
| Búsqueda con filtros y paginación | **Service** | Listar orders con filtros |
| Lógica de negocio compleja | **UseCase** | Proceso de checkout |
| Múltiples operaciones.transaccionales | **UseCase** | Transferir dinero |
| Orchestración de varios servicios | **UseCase** | Generar factura completa |
| Validaciones de negocio específicas | **UseCase** | Verificar stock antes de vender |
| Notificaciones post-operación | **UseCase** | Enviar email después de compra |

---

## 1.5 Reglas No Negociables

> ⚠️ Estas reglas **NO se pueden romper**. Si las rompen, el código no entra.

| # | Regla | Descripción | ¿Por qué? |
|---|-------|------------|-----------|
| 1 | **El Controller NO tiene lógica de negocio** | Solo recibe, valida, y devuelve | Mantiene el flujo limpio |
| 2 | **SIEMPRE se pasa por el Adapter** | Nunca llamar Service/UseCase directo desde Controller | Centraliza transformación |
| 3 | **NUNCA exponer Entity en response** | Siempre usar DTO Response | Seguridad de datos |
| 4 | **UseCase solo para lógica compleja** | No usar para CRUD básico | Evita sobreingeniería |
| 5 | **Service solo para CRUD básico** | No meter lógica de negocio | Mantiene separación |
| 6 | **Repository solo para acceso a datos** | Sin lógica de negocio | Separación de responsabilidades |
| 7 | **DTOs siempre inmutables** | Usar records | Previene estados inconsistentes |
| 8 | **Validar con @Valid en Controller** | Nunca confiar en request raw | Seguridad de entrada |
| 9 | **Inyección por constructor** | Usar @RequiredArgsConstructor, NUNCA @Autowired | Inmutabilidad y testabilidad |
| 10 | **DTOs como records** | NUNCA clases, siempre records | Inmutables por defecto |

### Inyección por Constructor (Obligatorio)

```java
// ✅ CORRECTO: Constructor con final fields
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
}

// ❌ INCORRECTO: @Autowired en campo
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // ¡ROMPIÓ LA REGLA!
}
```

### DTOs como Records (Obligatorio)

```java
// ✅ CORRECTO: Record con @Builder
@Builder
public record UserResponse(
    UUID id,
    String name,
    String email
) {}

// ❌ INCORRECTO: Clase POJO
public class UserResponse {  // ¡ROMPIÓ LA REGLA!
    private UUID id;
    private String name;
}
```

### Controller con lógica (Prohibido)

```java
// ❌ INCORRECTO: Controller con lógica
@PostMapping
public ResponseEntity<Entity> create(@RequestBody Entity entity) {
    entity.setCreatedAt(now());
    return repository.save(entity);  // ¡ROMPIÓ LA REGLA!
}

// ✅ CORRECTO: Controller delegando
@PostMapping
public ResponseEntity<Response> create(@Valid @RequestBody Request request) {
    Response response = adapter.create(request);  // Todo por el Adapter
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

### Exponer Entity en Response (Prohibido)

```java
// ❌ INCORRECTO: Retornar Entity directamente
@PostMapping
public ResponseEntity<Entity> create(@RequestBody Request request) {
    Entity entity = service.create(request);
    return ResponseEntity.ok(entity);  // ¡ROMPIÓ LA REGLA! Exposed Entity
}

// ✅ CORRECTO: Retornar DTO
@PostMapping
public ResponseEntity<Response> create(@RequestBody Request request) {
    Response response = service.create(request);  // Service convierte Entity → Response
    return ResponseEntity.ok(response);
}
```

### ¿Qué pasa si se rompe una regla?

| Consecuencia |
|--------------|
| Code Review ❌ |
| Retorno para corregir |
| Discussión一对一 con tech lead |

---

## 1.6 Recomendaciones Adicionales

| # | Recomendación | Descripción |
|---|--------------|-------------|
| 1 | **Usar UUID como ID primario** | Mejor que auto-increment para sistemas distribuidos |
| 2 | **Enums con EnumType.STRING** | Evita errores por índices |
| 3 | **Logs significativos** | No "entró/salió", incluir contexto |
| 4 | **Transacciones explícitas** | @Transactional mínimo necesario |
| 5 | **Separación de paquetes por dominio** | Agrupar por feature, no por tipo |

---

## 1.7 Capa de Enums (Catálogos)

> ⚠️ **REGLA: Todo valor de tabla que sea un catálogo (estados, tipos, categorías) debe ser un Enum en código, NO una entidad en BD.**

### Por qué usar Enums

| Cuando usas Enum | Cuando NO usas Enum |
|------------------|---------------------|
| Valores fijos y controlados | Valores dinámicos que pueden cambiar sin deploy |
| Lista cerrada de opciones | Cuando el usuario puede agregar opciones |
| Si cambia, requiere código | Si cambia por configuración |

### ¿Qué debe ser Enum?

| Si tenés... | Debe ser... |
|-------------|-------------|
| Estado de orden (PENDIENTE, COMPLETADO, CANCELADO) | ✅ Enum |
| Tipo de documento (FACTURA, BOLETA) | ✅ Enum |
| Rol de usuario (ADMIN, USER, GUEST) | ✅ Enum |
| Género (MASCULINO, FEMENINO, OTRO) | ✅ Enum |
| ~~Cliente~~ (puede crear/editar usuarios) | ❌ Entidad |
| ~~Producto~~ (catálogo dinámico) | ❌ Entidad |
| ~~Ciudad~~ (muchas ciudades) | ❌ Entidad |

### Estructura de Enums

```
src/main/java/com/muta/
├── common/
│   ├── entity/
│   ├── repository/
│   ├── service/
│   ├── dto/
│   └── enum/                    ← NUEVO: Capa de Enums
│       ├── GenericEnum.java    ← Interfaz base
│       ├── EstatusOrden.java
│       ├── TipoDocumento.java
│       ├── RolUsuario.java
│       └── ...
```

### Template de Enum

```java
public enum EstatusOrden implements CodeEnum {
    PENDIENTE("PEN", "Pendiente"),
    COMPLETADO("COM", "Completado"),
    CANCELADO("CAN", "Cancelado");

    private final String code;
    private final String descripcion;

    EstatusOrden(String code, String descripcion) {
        this.code = code;
        this.descripcion = descripcion;
    }

    public String getCode() {
        return code;
    }

    public String getDescripcion() {
        return descripcion;
    }

    public static EstatusOrden fromCode(String code) {
        return Arrays.stream(values())
            .filter(e -> e.code.equals(code))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Código no válido: " + code));
    }
}
```

### Interfaz base recomendada

```java
public interface CodeEnum {
    String getCode();
    String getDescripcion();
}
```

### Reglas para Enums

| # | Regla |
|---|-------|
| 1 | SIEMPRE usar `EnumType.STRING` en @Enumerated |
| 2 | NUNCA crear entidad para valores fijos |
| 3 | Incluir código y descripción legible |
| 4 | Implementar `fromCode()` para búsqueda por código |

---

## 1.8 Capa de Excepciones

> ⚠️ **REGLA: Cada flujo tiene sus propias excepciones específicas + un handler genérico para la aplicación.**

### Estructura de Excepciones

```
src/main/java/com/muta/
├── common/
│   ├── entity/                      ← JPA Entities
│   ├── repository/                  ← JpaRepository interfaces
│   ├── service/                     ← Services (CRUD base)
│   ├── dto/                         ← DTOs (records)
│   │   └── {modulo}/
│   │       ├── Request.java
│   │       └── Response.java
│   ├── enum/                       ← Catálogos/Enums
│   │   ├── CodeEnum.java
│   │   ├── EstatusOrden.java
│   │   ├── TipoDocumento.java
│   │   └── ...
│   └── exception/                  ← Excepciones
│       ├── MutaException.java
│       ├── NotFoundException.java
│       └── ValidationException.java
│
├── templates/                       ← Templates base (extender estos)
│   ├── BaseService.java            ← CRUD básico
│   └── BaseUseCase.java           ← Lógica compleja
│
├── controller/                      ← Controllers
│   └── {modulo}/
│       └── {Modulo}Controller.java
│
├── usecase/                         ← UseCases (lógica compleja)
│   └── {modulo}/
│       ├── {Accion}UseCase.java
│       └── dto/
│
├── adapter/                       ← Adapters
│   └── {modulo}/
│       ├── {Modulo}Adapter.java
│       └── {Modulo}AdapterImpl.java
│
└── config/
    └── exception/
        └── GlobalExceptionHandler.java
```

### Excepciones Genéricas (de la aplicación)

| Excepción | Cuándo usarla | HTTP Status |
|----------|--------------|-----------|
| `MutaException` | Error genérico de la app | 500 |
| `NotFoundException` | Recurso no encontrado | 404 |
| `ValidationException` | Error de validación | 400 |
| `UnauthorizedException` | No autorizado | 401 |
| `ForbiddenException` | Sin permiso | 403 |
| `ConflictException` | Conflicto (ej: email duplicado) | 409 |

### Template de excepción específica

```java
@EqualsAndHashCode(callSuper = true)
public class NotFoundException extends MutaException {
    private final String recurso;
    private final Object valor;

    public NotFoundException(String recurso, Object valor) {
        super(String.format("%s no encontrado con id: %s", recurso, valor));
        this.recurso = recurso;
        this.valor = valor;
    }

    public String getRecurso() {
        return recurso;
    }

    public Object getValor() {
        return valor;
    }
}
```

### Template de excepción base

```java
public class MutaException extends RuntimeException {
    public MutaException(String message) {
        super(message);
    }

    public MutaException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Global Exception Handler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(NotFoundException ex) {
        log.error("NotFound: {}", ex.getMessage());
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.error("Validation: {}", ex.getMessage());
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("VALIDATION_ERROR", ex.getMessage()));
    }

    @ExceptionHandler(MutaException.class)
    public ResponseEntity<ErrorResponse> handleMutaException(MutaException ex) {
        log.error("MutaException: {}", ex.getMessage());
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("ERROR", ex.getMessage()));
    }

    // Error response DTO
    @Builder
    public record ErrorResponse(
        String code,
        String message
    ) {}
}
```

### Reglas para Excepciones

| # | Regla |
|---|-------|
| 1 | NUNCA usar `Exception` directa, usar subclases |
| 2 | Cada flujo puede tener su excepción específica |
| 3 | NUNCA exponer stack trace en producción |
| 4 | Siempre incluir logging en el handler |
| 5 | Usar @RestControllerAdvicecentralizado |

---

*Contenido basado en principios de arquitectura hexagonal lite*