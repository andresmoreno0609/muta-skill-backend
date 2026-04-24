# 🚀 Guía Completa: Crear un Nuevo Endpoint

> **Este documento guía al desarrollador desde que se pide un nuevo endpoint hasta que está listo en producción.**

---

## ⚠️ IMPORTANTE: Antes de empezar

```
❌ NO EMPECER a programar hasta completar la Parte 1 - Análisis
✅ SíEMPRE seguir este proceso para evitar reprocesos
```

---

## Parte 1: ANÁLISIS Y DISEÑO (Obligatorio)

### Paso 1: Questions iniciales

Antes de escribir código, responde estas preguntas:

| # | Pregunta | Tu respuesta |
|---|---------|--------------|
| 1 | ¿Cuál es el nombre del recurso? | _ |
| 2 | ¿Qué acción hace este endpoint? | _ |
| 3 | ¿Cuál es el HTTP method? | GET / POST / PUT / PATCH / DELETE |
| 4 | ¿Qué entidad usa? | _ |
| 5 | ¿Es CRUD básico o lógica compleja? | _ |
| 6 | ¿Quién lo va a consumir? (frontend, sistema externo) | _ |

### Passo 2: Datos de Entrada (Request)

Completa la tabla:

| # | Campo | Tipo | Obligatorio? | Validaciones |
|---|------|-----|-------------|--------------|
| 1 | _ | String | ✅ / ❌ | _ |
| 2 | _ | Integer | ✅ / ❌ | _ |
| 3 | _ | UUID | ✅ / ❌ | _ |
| 4 | _ | Enum | ✅ / ❌ | _ |

**Ejemplo:**
| # | Campo | Tipo | Obligatorio? | Validaciones |
|---|------|-----|-------------|--------------|
| 1 | nombre | String | ✅ | @NotBlank, @Size(max=100) |
| 2 | email | String | ✅ | @NotBlank, @Email |
| 3 | edad | Integer | ❌ | @Min(18), @Max(99) |

### Paso 3: Datos de Salida (Response)

| # | Campo | Tipo | Descripción |
|---|------|-----|-------------|
| 1 | id | UUID | Identificador |
| 2 | nombre | String | Nombre del recurso |
| 3 | createdAt | LocalDateTime | Fecha de creación |

### Paso 4: Definir el proceso (Core)

| # | Pregunta | Respuesta |
|---|---------|----------|
| 1 | ¿Es solo CREATE/READ/UPDATE/DELETE básico? | Sí → usar Service |
| 2 | ¿Hay validaciones de negocio? | Sí → needed UseCase |
| 3 | ¿Hay múltiples operaciones? | Sí → usar UseCase |
| 4 | ¿Hay efectos secundarios? (notificaciones, etc) | Sí → needed UseCase |

---

## Parte 2: CHECKLIST DE CREACIÓN

### Checklist de-archivos-a-crear

| # | Archivo | Crear? | Ubicación |
|---|---------|--------|-----------|
| 1 | Entity | ⬜ / ✅ | `{modulo}/entity/` |
| 2 | Repository | ⬜ / ✅ | `{modulo}/repository/` |
| 3 | Request DTO | ⬜ / ✅ | `{modulo}/dto/` |
| 4 | Response DTO | ⬜ / ✅ | `{modulo}/dto/` |
| 5 | Enum (si aplica) | ⬜ / ✅ | `{modulo}/enum/` |
| 6 | Exception (si aplica) | ⬜ / ✅ | `common/exception/` |
| 7 | Service | ⬜ / ✅ | `{modulo}/service/` |
| 8 | UseCase (si aplica) | ⬜ / ✅ | `{modulo}/usecase/` |
| 9 | Adapter | ⬜ / ✅ | `{modulo}/adapter/` |
| 10 | Controller | ⬜ / ✅ | `{modulo}/controller/` |
| 11 | Test Service | ⬜ / ✅ | `{modulo}/service/` |
| 12 | Test UseCase (si aplica) | ⬜ / ✅ | `{modulo}/usecase/` |
| 13 | Test Controller | ⬜ / ✅ | `{modulo}/controller/` |

---

## Parte 3: FLUJO DE CREACIÓN (Paso a paso)

### Step 1: Entity + Repository

```java
// 1.1 Crear Entity
// archivo: src/main/java/com/muta/{modulo}/entity/{Modulo}Entity.java
@Entity
@Table(name = "{tabla}")
@Getter @Setter
@NoArgsConstructor @AllArgsConstructor @Builder
public class {Modulo}Entity {
    @Id @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
    // campos...
}

// 1.2 Crear Repository
// archivo: src/main/java/com/muta/{modulo}/repository/{Modulo}Repository.java
@Repository
public interface {Modulo}Repository extends JpaRepository<{Modulo}Entity, UUID>, 
    JpaSpecificationExecutor<{Modulo}Entity> {
    // query methods...
}
```

### Paso 2: DTOs

```java
// 2.1 Request
// archivo: src/main/java/com/muta/{modulo}/dto/{Modulo}Request.java
@Builder
public record {Modulo}Request(
    @NotBlank @Size(max = 100) String campo1,
    Integer campo2
) {}

// 2.2 Response
// archivo: src/main/java/com/muta/{modulo}/dto/{Modulo}Response.java
@Builder
public record {Modulo}Response(
    UUID id,
    String campo1,
    Integer campo2,
    LocalDateTime createdAt
) {}
```

### Paso 3: Service

```java
// archivo: src/main/java/com/muta/{modulo}/service/{Modulo}Service.java
@Service @RequiredArgsConstructor @Slf4j
public class {Modulo}Service extends BaseService<{Modulo}Entity, {Modulo}Request, 
    {Modulo}Response, {Modulo}Repository> {
    
    @Override protected {Modulo}Repository getRepository() { return repository; }
    
    @Override protected {Modulo}Response toResponse({Modulo}Entity e) { 
        return {Modulo}Response.builder().id(e.getId())...build(); 
    }
    
    @Override protected {Modulo}Entity toEntityCreate({Modulo}Request r) { 
        return {Modulo}Entity.builder()...build(); 
    }
    
    @Override protected void toEntityUpdate({Modulo}Request r, {Modulo}Entity e) {
        // map fields
    }
}
```

### Paso 4: UseCase (solo si aplica)

```java
// archivo: src/main/java/com/muta/{modulo}/usecase/{Accion}UseCase.java
@Component @Slf4j
public class {Accion}UseCase extends BaseUseCase<{Accion}Request, {Accion}Response> {
    private final {Modulo}Service service;
    
    @Override protected void preConditions({Accion}Request r) {
        // validaciones previas
    }
    
    @Override protected {Accion}Response core({Accion}Request r) {
        // lógica principal
    }
    
    @Override protected void postConditions({Accion}Response r) {
        // efectos secundarios
    }
}
```

### Paso 5: Adapter

```java
// 5.1 Interfaz
// archivo: src/main/java/com/muta/{modulo}/adapter/{Modulo}Adapter.java
public interface {Modulo}Adapter {
    {Modulo}Response create({Modulo}Request r);
    {Modulo}Response findById(UUID id);
    List<{Modulo}Response> findAll();
    {Modulo}Response update(UUID id, {Modulo}Request r);
    void delete(UUID id);
}

// 5.2 Implementación
@Service @RequiredArgsConstructor @Slf4j
public class {Modulo}AdapterImpl implements {Modulo}Adapter {
    private final {Modulo}Service service;
    
    @Override
    public {Modulo}Response create({Modulo}Request r) {
        return service.create(r);  // o useCase.execute()
    }
    //... implement todos los métodos
}
```

### Paso 6: Controller

```java
// archivo: src/main/java/com/muta/{modulo}/controller/{Modulo}Controller.java
@RestController
@RequestMapping("/api/v1/{recurso}")
@RequiredArgsConstructor @Slf4j
public class {Modulo}Controller {
    private final {Modulo}Adapter adapter;
    
    @GetMapping
    public ResponseEntity<List<{Modulo}Response>> findAll() {
        return ResponseEntity.ok(adapter.findAll());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<{Modulo}Response> findById(@PathVariable UUID id) {
        return ResponseEntity.ok(adapter.findById(id));
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public ResponseEntity<{Modulo}Response> create(
            @Valid @RequestBody {Modulo}Request request) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(adapter.create(request));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<{Modulo}Response> update(
            @PathVariable UUID id,
            @Valid @RequestBody {Modulo}Request request) {
        return ResponseEntity.ok(adapter.update(id, request));
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public ResponseEntity<Void> delete(@PathVariable UUID id) {
        adapter.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Parte 4: TESTS (Obligatorio)

### Test Service

```java
// archivo: src/test/java/com/muta/{modulo}/service/{Modulo}ServiceTest.java
@ExtendWith(MockitoExtension.class)
class {Modulo}ServiceTest {
    @Mock private {Modulo}Repository repository;
    @InjectMocks private {Modulo}Service service;
    
    @Test
    void create_deberiaGuardarEntity() {
        when(repository.save(any())).thenReturn(entity);
        {Modulo}Response result = service.create(request);
        assertNotNull(result);
        verify(repository).save(any());
    }
}
```

### Test UseCase (si aplica)

```java
// archivo: src/test/java/com/muta/{modulo}/usecase/{Accion}UseCaseTest.java
@ExtendWith(MockitoExtension.class)
class {Accion}UseCaseTest {
    @Mock private {Modulo}Service service;
    @InjectMocks private {Accion}UseCase useCase;
    
    @Test
    void execute_deberiaEjecutarCore() {
        // test flow
    }
}
```

### Test Controller

```java
// archivo: src/test/java/com/muta/{modulo}/controller/{Modulo}ControllerTest.java
@WebMvcTest({Modulo}Controller.class)
class {Modulo}ControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean {Modulo}Adapter adapter;
    
    @Test
    void create_deberiaRetornar201() throws Exception {
        when(adapter.create(any())).thenReturn(response);
        mockMvc.perform(post("/api/v1/{recurso}")
            .contentType(APPLICATION_JSON)
            .content(json))
            .andExpect(status().isCreated());
    }
}
```

---

## Parte 5: REGLAS OBLIGATORIAS

| # | Regla | Descripción |
|---|-------|-------------|
| 1 | Análisis primero | NO programar sin completar Parte 1 |
| 2 | Una Entity = Un Controller | Aunque sea "sub" recurso |
| 3 | CRUD básico = Service | Solo extender BaseService |
| 4 | Lógica compleja = UseCase | Con preConditions, core, postConditions |
| 5 | Adapter + Interfaz | Siempre crear los dos |
| 6 | Records para DTOs | NUNCA clases |
| 7 | Tests obligatorios | Al menos happy path + 1 caso de error |
| 8 | Enums con prefijo E | EEstatus, ETipo |
| 9 | @Valid en Controller | Siempre para Request |
| 10 | @RequiredArgsConstructor | Para inyección |

---

## Parte 6: CHEAT SHEET

### Mapping HTTP → Código

| HTTP Method | Método Service | Método Controller |
|------------|----------------|-------------------|
| GET | `findAll()` | `@GetMapping` |
| GET `/{id}` | `findById()` | `@GetMapping("/{id}")` |
| POST | `create()` | `@PostMapping` |
| PUT | `update()` | `@PutMapping("/{id}")` |
| DELETE | `delete()` | `@DeleteMapping("/{id}")` |

### Cuándo usar cada capa

| Escenario | Usar |
|----------|------|
| CREATE/READ/UPDATE/DELETE básico | Service (extends BaseService) |
| Con validaciones de negocio | Service + validaciones |
| Múltiples operaciones | UseCase |
| Notificaciones post-op | UseCase (postConditions) |
| Adaptar formato DTO | Adapter |

---

## 📋 RESUMEN DEL PROCESO

```
1. ANALISÍS (Parte 1)
   └─→ Responder preguntas
   └→ Definir Request/Response
   └→ Definir Service vs UseCase

2. CHECKLIST (Parte 2)
   └→ Marcar qué archivos crear

3. IMPLEMENTACIÓN (Parte 3)
   └→ Entity + Repository
   └→ DTOs (Request + Response)
   └→ Service o UseCase
   └→ Adapter (Interfaz + Impl)
   └→ Controller

4. TESTS (Parte 4)
   └→ ServiceTest
   └→ UseCaseTest (si aplica)
   └→ ControllerTest
```

---

## ⚠️ ERRORES COMUNES (evitar)

| # | Error | consequence | Solución |
|---|-------|-------------|---------|
| 1 | Empezar a programar sin análisis | Rework | Completar siempre Parte 1 |
| 2 | No crear Adapter | Rompe arquitectura | Siempre Adapter + Interfaz |
| 3 | Poner lógica en Controller | Code review rejected | Delegar a Adapter/Service |
| 4 | No hacer tests | Bugs en producción | Al menos happy path + 1 error |
| 5 | Usar clase en vez de record DTO | Architecture violated | Usar siempre records |
| 6 | Enum sin prefijo E | Inconsistencia | Usar EEstatus, ETipo |

---