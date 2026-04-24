# 🔌 Crear DTO

---

## 1. Cuándo crear un DTO

| Trigger | Ejemplo |
|---------|--------|
| Request HTTP | Recibir datos del cliente |
| Response HTTP | Devolver datos al cliente |
| Transferencia entre capas | Adapter → Service |

---

## 2. Checklist previa

- [ ] Se know los campos del Request
- [ ] Se know los campos del Response
- [ ] Se know si hay campos opcionales

---

## 3. Paso a Paso

### Paso 1:Crear Request (obligatorio)

```java
@Builder
public record {Modulo}Request(
    String nombre,
    String email,
    Integer edad
) {}
```

### Paso 2:Crear Response (obligatorio)

```java
@Builder
public record {Modulo}Response(
    UUID id,
    String nombre,
    String email,
    LocalDateTime createdAt
) {}
```

---

## 4. Templates

### 4.1 Request (Input)

```java
package com.muta.{modulo}.dto;

import lombok.Builder;
import lombok.Data;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

@Builder
public record {Modulo}Request(
    @NotBlank(message = "Nombre es obligatorio")
    @Size(max = 100)
    String nombre,

    @NotBlank(message = "Email es obligatorio")
    @Email(message = "Email inválido")
    String email,

    Integer edad
) {}

// Alternativa con @Data (si se necesita setters)
@Data
@Builder
public class {Modulo}Request {
    @NotBlank
    private String nombre;

    @Email
    private String email;

    private Integer edad;
}
```

### 4.2 Response (Output)

```java
package com.muta.{modulo}.dto;

import lombok.Builder;

import java.time.LocalDateTime;
import java.util.UUID;

@Builder
public record {Modulo}Response(
    UUID id,
    String nombre,
    String email,
    String estado,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) {}
```

### 4.3 GenericRequest

```java
package com.muta.common.dto;

import lombok.Builder;

@Builder
public record GenericRequest(
    String createdBy
) {}
```

### 4.4 GenericResponse

```java
package com.muta.common.dto;

import lombok.Builder;

import java.time.LocalDateTime;
import java.util.UUID;

@Builder
public record GenericResponse(
    UUID id,
    LocalDateTime createdAt,
    LocalDateTime updatedAt
) {}
```

### 4.5 ErrorResponse

```java
package com.muta.common.dto;

import lombok.Builder;

@Builder
public record ErrorResponse(
    String code,
    String message
) {}
```

### 4.6 PageResponse (para paginación)

```java
package com.muta.common.dto;

import lombok.Builder;

import java.util.List;

@Builder
public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean first,
    boolean last
) {}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** usar `record` (NO clases) |
| 2 | **SIEMPRE** usar `@Builder` en Response |
| 3 | Usar `@NotBlank`, `@Email`, `@Size` para validaciones |
| 4 | **NUNCA** exponer datos sensibles (password, hash) |
| 5 | Request puede tener validaciones |
| 6 | Response es solo lectura |

---

## 6. Ejemplo de uso

```java
// En Controller
public ResponseEntity<{Modulo}Response> create(
    @Valid @RequestBody {Modulo}Request request
) {
    {Modulo}Response response = adapter.create(request);
    return ResponseEntity.ok(response);
}

// En Service
@Override
protected {Modulo}Response toResponse({Modulo}Entity entity) {
    return {Modulo}Response.builder()
        .id(entity.getId())
        .nombre(entity.getNombre())
        .email(entity.getEmail())
        .createdAt(entity.getCreatedAt())
        .build();
}
```

---