# 📝 Estándares de Código

---

## 1.1 Nombrado

| Elemento | Ejemplo | Regla |
|----------|--------|-------|
| Archivos .java | `UserService.java` | PascalCase igual que la clase |
| Paquetes | `com.muta.common.service` | Todo lowercase |
| Clases | `UserService` | PascalCase |
| Métodos | `findByEmail()`, `createUser()` | camelCase |
| Variables | `userRepository`, `passwordEncoder` | camelCase |
| Constantes | `MAX_RETRY_COUNT` | UPPER_SNAKE_CASE |
| Enums | `EEstatusOrden`, `ETipoDocumento` | PascalCase con prefijo **E** |
| Records DTOs | `UserRequest`, `UserResponse` | PascalCase |

---

## 1.2 Anotaciones Obligatorias por Capa

| Capa | Anotaciones | Ejemplo |
|------|-----------|---------|
| **Entity** | `@Entity`, `@Table`, `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`, `@Builder` | |
| **Repository** | `@Repository` | |
| **Service** | `@Service`, `@Slf4j`, `@RequiredArgsConstructor` | |
| **UseCase** | `@Component`, `@Slf4j` | |
| **Adapter** | `@Service`, `@Slf4j`, `@RequiredArgsConstructor` | |
| **Controller** | `@RestController`, `@RequestMapping`, `@RequiredArgsConstructor`, `@Slf4j` | |
| **DTO Record Response** | `@Builder` | Obligatorio en todos los Response |

---

## 1.3 Estructura de Archivos

### Orden dentro de una clase Java

| Orden | Elemento | Ejemplo |
|-------|---------|---------|
| 1 | Package | `package com.muta.common.service;` |
| 2 | Imports | `import ...` |
| 3 | Anotación de clase | `@Service` |
| 4 | Nombrado de clase | `public class UserService` |
| 5 | Implements/Extends | `extends BaseService` |
| 6 | Anotaciones de clase | `@Slf4j @RequiredArgsConstructor` |
| 7 | Comentario Javadoc (opcional) | `/** Clase para... */` |
| 8 | Constantes | `private static final int MAX = 10;` |
| 9 | Campos (final) | `private final UserRepository repository;` |
| 10 | Constructor | (generado por @RequiredArgsConstructor) |
| 11 | Métodos públicos | `create()`, `findById()`, etc. |
| 12 | Métodos privados | `validate()`, `mapToEntity()`, etc. |

### Ejemplo

```java
package com.muta.common.service;

import com.muta.common.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
@Slf4j
public class UserService extends BaseService<UserEntity, UserRequest, UserResponse, UserRepository> {

    // Campos (si hay más)
    // private final PasswordEncoder passwordEncoder;

    // ============ Métodos públicos ============

    // create(), findById(), findAll(), update(), delete()

    // ============ Métodos privados ============

    private void validationCreate(UserRequest request) {
        // Validaciones específicas
    }
}
```

---

## 1.4 Inyección de Dependencias

### Regla: SIEMPRE por constructor

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
    private UserRepository userRepository;  // ¡PROHIBIDO!
}

// ❌ INCORRECTO: @Autowired en setter
@Service
public class UserService {
    private UserRepository userRepository;

    @Autowired  // ¡PROHIBIDO!
    public void setRepository(UserRepository repository) {
        this.userRepository = repository;
    }
}
```

### Por qué@RequiredArgsConstructor

| Beneficio | Descripción |
|----------|-------------|
| Inmutabilidad | Campos final No se pueden cambiar |
| Testabilidad | Easy de injectar en tests |
| Thread-safety | No hay estado compartido |
| Código limpio | No hay setters innecesarios |

---

## 1.5 Documentación Mínima

### Cuándo comentar

| Qué | Cuándo | Cómo |
|-----|-------|------|
| Clase | Si no es obvia su función | Javadoc: `/** Servicio para... */` |
| Método complejo | Si la lógica no es clara | Comentario: `// Validar que...` |
| Lógica compleja | Bloques de código largos | Separar en métodos privados |

### NO hacer

```java
// ❌ NO hacer: Comentarios innecesales
public User create(User user) {
    // Guardar usuario
    return repository.save(user);  // Guardar en BD
}

// ✅ CORRECTO: Comentario solo si aporta contexto
public User create(User user) {
    // Verificar que el email no existe antes de crear
    requireNotExists(user.getEmail());
    return repository.save(user);
}
```

### Regla de oro

> **Si necesitás comentar para entender qué hace tu código, probablemente el código necesita refactorizarse.**

---

## 1.6 Formato y Estilo

| Regla | Detalle |
|-------|---------|
| **Líneas por método** | Máximo 20-30 líneas |
| **Brace style** | K&R (misma línea) |
| **Indentación** | 4 espacios (no tabs) |
| **Ancho de línea** | Máximo 120 caracteres |
| **Líneas en blanco** | Máximo 1 línea entre métodos |
| **Orden de métodos** | Públicos primero, luego privados |

### Ejemplo de formato

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService extends BaseService<UserEntity, UserRequest, UserResponse, UserRepository> {

    public UserResponse create(UserRequest request) {
        validationCreate(request);
        UserEntity entity = mapToEntity(request);
        UserEntity saved = repository.save(entity);
        log.info("User created: {}", saved.getId());
        return toResponse(saved);
    }

    public UserResponse findById(UUID id) {
        return repository.findById(id)
            .map(this::toResponse)
            .orElseThrow(() -> new NotFoundException("User", id));
    }

    // Métodos privados debajo
    private void validationCreate(UserRequest request) {
        // ...
    }
}
```

---

## Resumen Rápido

| Regla | Obligatorio |
|-------|-----------|
| Nombrado | PascalCase clases, camelCase métodos/vars, E + nombre en Enums |
| Anotaciones | @Service @Slf4j @RequiredArgsConstructor en Service |
| Estructura | Paquete → Imports → Clase → Campos → Métodos |
| Inyección | SIEMPRE constructor con @RequiredArgsConstructor |
| Documentación | Solo si aporta, no comentarios triviales |
| Formato | 4 espacios, máx 120 chars, máx 30 líneas/método |

---