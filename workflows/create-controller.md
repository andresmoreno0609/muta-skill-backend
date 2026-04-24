# 🔌 Crear Controller

---

## 1. Cuándo crear un Controller

| Trigger | Ejemplo |
|---------|--------|
| Neue Endpoint HTTP | Crear `/api/v1/usuarios` |
| Exponer funcionalidad | Necesito que el frontend consuma un servicio |
| CRUD básico | Crear, leer, actualizar, eliminar un recurso |

---

## 2. Checklist previa

- [ ] El Adapter ya existe o se va a crear primero
- [ ] Los DTOs (Request/Response) ya existen
- [ ] El módulo/paquete está creado
- [ ] Se sabe qué verbos HTTP usar (GET, POST, PUT, DELETE)
- [ ] Se know los status codes (200, 201, 400, 404, 500)

---

## 3. Paso a Paso

### Paso 1: Crear el archivo

```java
// Ubicación: src/main/java/com/muta/{modulo}/controller/{Modulo}Controller.java
```

### Paso 2: Definir las anotaciones

```java
@RestController                           // Indica que es REST
@RequestMapping("/api/v1/{recurso}")   // URL base del endpoint
@RequiredArgsConstructor               // Inyección por constructor
@Slf4j                                   // Logging
public class UsuarioController {

    private final UsuarioAdapter usuarioAdapter;
}
```

### Paso 3: Definir los endpoints

| Verbo | Método | Descripción | Status |
|------|--------|------------|---------|
| GET | `findAll()` | Listar todos | 200 |
| GET | `findById(UUID id)` | Buscar por ID | 200 / 404 |
| POST | `create(Request)` | Crear | 201 |
| PUT | `update(UUID id, Request)` | Actualizar | 200 / 404 |
| DELETE | `delete(UUID id)` | Eliminar | 204 / 404 |

### Paso 4: Documentar con OpenAPI

```java
@PostMapping
@Operation(summary = "Crear {recurso}", description = "Crea un nuevo {recurso}")
@ApiResponse(responseCode = "201", description = "{recurso} creado")
@ApiResponse(responseCode = "400", description = "Request inválido")
public ResponseEntity<{Modulo}Response> create(
    @Valid @RequestBody {Modulo}Request request
) {
    {Modulo}Response response = adapter.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

---

## 4. Template completo

```java
package com.muta.{modulo}.controller;

import com.muta.{modulo}.adapter.{Modulo}Adapter;
import com.muta.{modulo}.dto.{Modulo}Request;
import com.muta.{modulo}.dto.{Modulo}Response;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.UUID;

@RestController
@RequestMapping("/api/v1/{recurso}")
@RequiredArgsConstructor
@Slf4j
public class {Modulo}Controller {

    private final {Modulo}Adapter adapter;

    // ============ CREATE ============

    @PostMapping
    @Operation(summary = "Crear {recurso}", description = "Crea un nuevo {recurso}")
    @ApiResponse(responseCode = "201", description = "{reculo} creado")
    @ApiResponse(responseCode = "400", description = "Request inválido")
    public ResponseEntity<{Modulo}Response> create(
            @Valid @RequestBody {Modulo}Request request
    ) {
        {Modulo}Response response = adapter.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    // ============ READ ============

    @GetMapping("/{id}")
    @Operation(summary = "Obtener {recurso} por ID")
    @ApiResponse(responseCode = "200", description = "{recurso} encontrado")
    @ApiResponse(responseCode = "404", description = "{recurso} no encontrado")
    public ResponseEntity<{Modulo}Response> findById(@PathVariable UUID id) {
        return ResponseEntity.ok(adapter.findById(id));
    }

    @GetMapping
    @Operation(summary = "Listar todos los {recurso}")
    @ApiResponse(responseCode = "200", description = "Lista de {recurso}")
    public ResponseEntity<List<{Modulo}Response>> findAll() {
        return ResponseEntity.ok(adapter.findAll());
    }

    // ============ UPDATE ============

    @PutMapping("/{id}")
    @Operation(summary = "Actualizar {recurso}")
    @ApiResponse(responseCode = "200", description = "{recurso} actualizado")
    @ApiResponse(responseCode = "400", description = "Request inválido")
    @ApiResponse(responseCode = "404", description = "{recurso} no encontrado")
    public ResponseEntity<{Modulo}Response> update(
            @PathVariable UUID id,
            @Valid @RequestBody {Modulo}Request request
    ) {
        {Modulo}Response response = adapter.update(id, request);
        return ResponseEntity.ok(response);
    }

    // ============ DELETE ============

    @DeleteMapping("/{id}")
    @Operation(summary = "Eliminar {recurso}")
    @ApiResponse(responseCode = "204", description = "{recurso} eliminado")
    @ApiResponse(responseCode = "404", description = "{recurso} no encontrado")
    public ResponseEntity<Void> delete(@PathVariable UUID id) {
        adapter.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE delegar al Adapter**, nunca al Service directamente |
| 2 | Usar `@Valid` en `@RequestBody` |
| 3 | Usar `@PathVariable` para IDs en URL |
| 4 | Documentar con `@Operation` y `@ApiResponse` |
| 5 | Retornar `ResponseEntity<>` paracontrolar HTTP status |
| 6 | **NUNCA** tener lógica de negocio |
| 7 | Usar `HttpStatus` correcto (200, 201, 204, 400, 404) |

---

## 6. HTTP Status Codes de referencia

| Status | Cuándo usar |
|--------|------------|
| 200 | OK (GET, PUT) |
| 201 | Created (POST) |
| 204 | No Content (DELETE) |
| 400 | Bad Request (validación falla) |
| 404 | Not Found (no existe) |
| 500 | Internal Server Error |

---

## 7. Regla: Una Entity = Un Controller

> ⚠️ **IMPORTANTE: Cada entidad tiene su propio Controller.**

| Entity | Controller | Endpoint |
|--------|------------|----------|
| User | UserController | `/api/v1/users` |
| UserPermission | UserPermissionController | `/api/v1/user-permissions` |
| Orden | OrdenController | `/api/v1/ordenes` |

**Aunque sea "sub" de otro recurso (como Permission para User), si tiene lógica propia → Controller separado.**

---