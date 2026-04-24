# 📦 Estructura de Paquetes

---

## 1. Estructura General

```
src/main/java/com/muta/
├── common/                    ← Compartido globalmente
├── templates/                ← Templates base
├── {modulo}/                ← Por dominio (feature)
├── config/                  ← Configuración
└── MutaApplication.java    ← Main
```

---

## 2. Paquete Common

> Todo lo que se comparte entre módulos va en **common**.

```
common/
├── entity/               ← Entities JPA compartidas
│   ├── BaseEntity.java   ← Entidad con campos comunes
│   └── auditable/
│
├── repository/           ← JpaRepository compartidas
│   └── BaseRepository.java
│
├── service/             ← Services base (templates)
│   └── BaseService.java
│
├── dto/                  ← DTOs genéricos
│   ├── GenericRequest.java
│   ├── GenericResponse.java
│   └── ErrorResponse.java
│
├── enum/                 ← Enums globales (E*)
│   ├── CodeEnum.java     ← Interfaz base
│   ├── EEstatusRegistro.java
│   └── ETipoDocumento.java
│
├── exception/            ← Excepciones compartidas
│   ├── MutaException.java
│   ├── NotFoundException.java
│   ├── ValidationException.java
│   └── UnauthorizedException.java
│
└── config/              ← Configuraciones comunes
    └── ...
```

---

## 3. Templates

> Clases base para extender, no para instanciar directamente.

```
templates/
├── BaseService.java      ← Extender para CRUD
└── BaseUseCase.java     ← Extender para lógica compleja
```

### Cómo usar

```java
// ✅ CORRECTO: Extender
@Service
@Slf4j
@RequiredArgsConstructor
public class UsuarioService extends BaseService<UsuarioEntity, UsuarioRequest, UsuarioResponse, UsuarioRepository> {
    // Solo sobreescribir métodos específicos
}

// ❌ INCORRECTO: Usar directamente
UsuarioService service = new UsuarioService();  // No!
```

---

## 4. Paquete por Dominio

> Cada **módulo/feature** tiene su propio paquete.

### Ejemplo: módulo de Usuarios

```
usuarios/
├── controller/
│   └── UsuarioController.java
│
├── service/
│   └── UsuarioService.java   ← Extiende BaseService
│
├── adapter/
│   ├── UsuarioAdapter.java    ← Interfaz
│   └── UsuarioAdapterImpl.java ← Impl
│
├── usecase/
│   ├── CrearUsuarioUseCase.java
│   └── CambiarPasswordUseCase.java
│
├── dto/
│   ├── UsuarioRequest.java    ← Record
│   └── UsuarioResponse.java ← Record
│
├── entity/
│   └── UsuarioEntity.java
│
├── repository/
│   └── UsuarioRepository.java
│
└── enum/
    └── EEstatusUsuario.java
```

### Ejemplo: módulo de Órdenes

```
ordenes/
├── controller/
│   └── OrdenController.java
│
├── service/
│   ├── OrdenService.java
│   └── OrdenItemService.java
│
├── adapter/
│   ├── OrdenAdapter.java
│   └── OrdenAdapterImpl.java
│
├── usecase/
│   ├── CrearOrdenUseCase.java
│   ├── CompletarOrdenUseCase.java
│   └── CancelarOrdenUseCase.java
│
├── dto/
│   ├── CrearOrdenRequest.java
│   ├── OrdenResponse.java
│   └── OrdenItemResponse.java
│
├── entity/
│   ├── OrdenEntity.java
│   └── OrdenItemEntity.java
│
├── repository/
│   ├── OrdenRepository.java
│   ��── OrdenItemRepository.java
│
└── enum/
    ├── EEstatusOrden.java
    └── ETipoOrden.java
```

---

## 5. Paquete Config

> Configuración centralizada de la aplicación.

```
config/
├── exception/
│   └── GlobalExceptionHandler.java  ← @RestControllerAdvice
│
├── security/
│   └── SecurityConfig.java          ← Spring Security
│
├── web/
│   └── WebConfig.java              ← CORS, filtros
│
└── database/
    └── DatabaseConfig.java         ← DataSource, pooling
```

---

## 6. Reglas de Ubicación

| Tipo de clase | Dónde va | Ejemplo |
|--------------|---------|---------|
| Entity JPA | `common/entity/` o `{modulo}/entity/` | `UsuarioEntity` |
| Repository | `common/repository/` o `{modulo}/repository/` | `UsuarioRepository` |
| Service (CRUD) | `{modulo}/service/` | `UsuarioService` |
| UseCase | `{modulo}/usecase/` | `CrearUsuarioUseCase` |
| Adapter | `{modulo}/adapter/` | `UsuarioAdapterImpl` |
| Controller | `{modulo}/controller/` | `UsuarioController` |
| DTO Request | `{modulo}/dto/` | `UsuarioRequest` |
| DTO Response | `{modulo}/dto/` | `UsuarioResponse` |
| Enum | `common/enum/` o `{modulo}/enum/` | `EEstatusUsuario` |
| Excepción | `common/exception/` | `NotFoundException` |

---

## 7. Reglas de Organización

| # | Regla | Ejemplo |
|---|-------|-------|
| 1 | **1 módulo = 1 paquete** | `ordenes/` no mezclado con `usuarios/` |
| 2 | **Todo junto por módulo** | Controller, Service, Entity en su carpeta |
| 3 | **Common solo si se reutiliza** | Si solo lo usa 1 módulo, va en el módulo |
| 4 | **Enum global en common** | Estados que se usan en múltiples módulos |
| 5 | **Enum local en módulo** | Estados específicos de un módulo |
| 6 | **1 Entity = 1 Controller** | UserController + UserPermissionController separados |

---

## 8. Regla: Una Entity = Un Controller

> ⚠️ **Cada entidad tiene su propio Controller, Adapter, Service, UseCase y Repository.**

| Entity | Controller | Endpoint |
|--------|------------|----------|
| User | UserController | `/api/v1/users` |
| UserPermission | UserPermissionController | `/api/v1/user-permissions` |
| Orden | OrdenController | `/api/v1/ordenes` |

**Aunque sea "sub" de otro recurso, si tiene lógica propia → Controller separado.**

---

## Resumen visuel

```
com/muta/
├── common/           ← Compartido (entities, repositories, services, dtos, enums, exceptions)
├── templates/         ← Templates base
├── usuarios/         ← Módulo 1: todo junto
│   ├── controller/
│   ├── service/
│   ├── adapter/
│   ├── usecase/
│   ├── dto/
│   ├── entity/
│   ├── repository/
│   └── enum/
├── ordenes/           ← Módulo 2: todo junto
│   ├── controller/
│   ├── service/
│   └── ...
├── config/           ← Configuración
│   ├── exception/
│   └── security/
└── MutaApplication.java
```

---

## Checklist rápido

| Pregunta | Respuesta |
|----------|-----------|
| ¿Nueva Entity? | Va en `entity/` de su módulo |
| ¿Se reutiliza en otros? | Va en `common/entity/` |
| ¿Nuevo Service? | Va en `service/` de su módulo |
| ¿UseCase? | Va en `usecase/` de su módulo |
| ¿Enum nuevo? | `E` + nombre en `enum/` |

---