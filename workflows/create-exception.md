# 🔌 Crear Exception

---

## 1. Cuándo crear una Exception

| Trigger | Ejemplo |
|---------|--------|
| Error específico | ValidationException para campo inválido |
| Error genérico de app | MutaException como base |
| Error personalizado | Recurso no encontrado |

---

## 2. Checklist previa

- [ ] No existe ya en common/exception/
- [ ] Se know el mensaje de error
- [ ] Se know el HTTP status

---

## 3. Paso a Paso

### Paso 1:Exception Base

```java
// common/exception/MutaException.java
public class MutaException extends RuntimeException {
    public MutaException(String message) {
        super(message);
    }

    public MutaException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Paso 2:Exception Específica

```java
@EqualsAndHashCode(callSuper = true)
public class NotFoundException extends MutaException {
    private final String recurso;
    private final Object valor;

    public NotFoundException(String recurso, Object valor) {
        super(String.format("%s no encontrado: %s", recurso, valor));
        this.recurso = recurso;
        this.valor = valor;
    }
}
```

---

## 4. Templates

### 4.1 Exception base

```java
package com.muta.common.exception;

public class MutaException extends RuntimeException {

    public MutaException(String message) {
        super(message);
    }

    public MutaException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 4.2 NotFoundException

```java
package com.muta.common.exception;

import lombok.EqualsAndHashCode;

@EqualsAndHashCode(callSuper = true)
public class NotFoundException extends MutaException {

    private final String recurso;
    private final Object valor;

    public NotFoundException(String recurso, Object valor) {
        super(String.format("%s no encontrado: %s", recurso, valor));
        this.recurso = recurso;
        this.valor = valor;
    }

    public String getRecurso() { return recurso; }
    public Object getValor() { return valor; }
}
```

### 4.3 ValidationException

```java
package com.muta.common.exception;

import lombok.EqualsAndHashCode;

@EqualsAndHashCode(callSuper = true)
public class ValidationException extends MutaException {

    private final String campo;

    public ValidationException(String message) {
        super(message);
        this.campo = null;
    }

    public ValidationException(String message, String campo) {
        super(message);
        this.campo = campo;
    }

    public String getCampo() { return campo; }
}
```

### 4.4 UnauthorizedException

```java
package com.muta.common.exception;

import lombok.EqualsAndHashCode;

@EqualsAndHashCode(callSuper = true)
public class UnauthorizedException extends MutaException {

    public UnauthorizedException(String message) {
        super(message);
    }
}
```

### 4.5 ForbiddenException

```java
package com.muta.common.exception;

import lombok.EqualsAndHashCode;

@EqualsAndHashCode(callSuper = true)
public class ForbiddenException extends MutaException {

    public ForbiddenException(String message) {
        super(message);
    }
}
```

### 4.6 ConflictException

```java
package com.muta.common.exception;

import lombok.EqualsAndHashCode;

@EqualsAndHashCode(callSuper = true)
public class ConflictException extends MutaException {

    public ConflictException(String message) {
        super(message);
    }
}
```

---

## 5. GlobalExceptionHandler

```java
package com.muta.config.exception;

import com.muta.common.exception.*;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(NotFoundException ex) {
        log.error("NotFound: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException ex) {
        log.error("Validation: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("VALIDATION_ERROR", ex.getMessage()));
    }

    @ExceptionHandler(UnauthorizedException.class)
    public ResponseEntity<ErrorResponse> handleUnauthorized(UnauthorizedException ex) {
        log.error("Unauthorized: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(new ErrorResponse("UNAUTHORIZED", ex.getMessage()));
    }

    @ExceptionHandler(ForbiddenException.class)
    public ResponseEntity<ErrorResponse> handleForbidden(ForbiddenException ex) {
        log.error("Forbidden: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(new ErrorResponse("FORBIDDEN", ex.getMessage()));
    }

    @ExceptionHandler(ConflictException.class)
    public ResponseEntity<ErrorResponse> handleConflict(ConflictException ex) {
        log.error("Conflict: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(new ErrorResponse("CONFLICT", ex.getMessage()));
    }

    @ExceptionHandler(MutaException.class)
    public ResponseEntity<ErrorResponse> handleMutaException(MutaException ex) {
        log.error("MutaException: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("ERROR", ex.getMessage()));
    }
}
```

### ErrorResponse DTO

```java
@Builder
public record ErrorResponse(
    String code,
    String message
) {}
```

---

## 6. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** extender de MutaException |
| 2 | Usar @EqualsAndHashCode(callSuper = true) |
| 3 | Agregar campos útiles (recurso, campo, etc.) |
| 4 | Usar el HTTP status correcto |
| 5 | Usar @RestControllerAdvicecentralizado |
| 6 | **NUNCA** exponer stack trace en producción |

---