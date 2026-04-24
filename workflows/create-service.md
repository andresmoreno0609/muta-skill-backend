# 🔌 Crear Service

---

## 1. Cuándo crear un Service

| Trigger | Ejemplo |
|---------|--------|
| Operaciones CRUD | Crear, leer, actualizar, eliminar |
| Consultas simples | findById, findAll, findBy... |
| Extender BaseService | Para obtener métodos CRUD automático |
| Busquedas con filtros | findAllWithFilters |

---

## 2. Checklist previa

- [ ] El BaseService está disponible en templates/
- [ ] La Entity ya existe
- [ ] El Repository ya existe
- [ ] Los DTOs (Request/Response) ya existen

---

## 3. Paso a Paso

### Paso 1: Extender BaseService

```java
// Ubicación: src/main/java/com/muta/{modulo}/service/{Modulo}Service.java
@Service
@RequiredArgsConstructor
@Slf4j
public class {Modulo}Service extends BaseService<{Modulo}Entity, {Modulo}Request, {Modulo}Response, {Modulo}Repository> {

    // Solo métodos específicos, los básicos ya están en BaseService
}
```

### Paso 2: Implementar los métodos abstractos

```java
// Obligatorio: retornar el repository
@Override
protected {Modulo}Repository getRepository() {
    return {modulo}Repository;
}

// Obligatorio: Entity → Response
@Override
protected {Modulo}Response toResponse({Modulo}Entity entity) {
    return {Modulo}Response.builder()
        .id(entity.getId())
        .campo(entity.getCampo())
        .build();
}

// Obligatorio: Request → Entity (create)
@Override
protected {Modulo}Entity toEntityCreate({Modulo}Request request) {
    return {Modulo}Entity.builder()
        .campo(request.campo())
        .build();
}

// Obligatorio: Request → Entity existente (update)
@Override
protected void toEntityUpdate({Modulo}Request request, {Modulo}Entity entity) {
    if (request.campo() != null) {
        entity.setCampo(request.campo());
    }
}
```

### Paso 3: Agregar métodos específicos (opcional)

```java
public List<{Modulo}Response> findByEstado(EEstatus estado) {
    return repository.findByEstado(estado).stream()
        .map(this::toResponse)
        .toList();
}
```

---

## 4. Template completo

```java
package com.muta.{modulo}.service;

import com.muta.common.service.BaseService;
import com.muta.{modulo}.dto.{Modulo}Request;
import com.muta.{modulo}.dto.{Modulo}Response;
import com.muta.{modulo}.entity.{Modulo}Entity;
import com.muta.{modulo}.repository.{Modulo}Repository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.UUID;

@Service
@RequiredArgsConstructor
@Slf4j
public class {Modulo}Service extends BaseService<{Modulo}Entity, {Modulo}Request, {Modulo}Response, {Modulo}Repository> {

    private final {Modulo}Repository repository;

    // ============ Implementar métodos obligatorios ============

    @Override
    protected {Modulo}Repository getRepository() {
        return repository;
    }

    @Override
    protected {Modulo}Response toResponse({Modulo}Entity entity) {
        return {Modulo}Response.builder()
            .id(entity.getId())
            .campo(entity.getCampo())
            .createdAt(entity.getCreatedAt())
            .updatedAt(entity.getUpdatedAt())
            .build();
    }

    @Override
    protected {Modulo}Entity toEntityCreate({Modulo}Request request) {
        return {Modulo}Entity.builder()
            .campo(request.campo())
            .build();
    }

    @Override
    protected void toEntityUpdate({Modulo}Request request, {Modulo}Entity entity) {
        if (request.campo() != null) {
            entity.setCampo(request.campo());
        }
        // Más campos...
    }

    // ============ Métodos específicos ============

    public List<{Modulo}Response> findByEstado(String estado) {
        return repository.findByEstado(estado).stream()
            .map(this::toResponse)
            .toList();
    }

    public boolean existsByCampo(String campo) {
        return repository.existsByCampo(campo);
    }
}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** extender `BaseService` |
| 2 | Usar `@Service`, `@RequiredArgsConstructor`, `@Slf4j` |
| 3 | **Obligatorio** implementar los 4 métodos |
| 4 | **NUNCA** lógica de negocio compleja |
| 5 | Usar `toResponse(entity)` para convertir |
| 6 | Usar `toEntityCreate(request)` para crear |
| 7 | Usar `toEntityUpdate(request, entity)` para actualizar |
| 8 | Delegar queries complejos al Repository |

---

## 6. Métodos disponibles de BaseService

| Método | Descripción |
|--------|-----------|
| `create(request)` | Crear registro |
| `findById(id)` | Buscar por ID (lanza NotFoundException si no existe) |
| `findAll()` | Listar todos |
| `existsById(id)` | Verificar si existe |
| `update(id, request)` | Actualizar registro |
| `delete(id)` | Eliminar registro |

---

## 7. Repository correspondiente

```java
package com.muta.{modulo}.repository;

import com.muta.{modulo}.entity.{Modulo}Entity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.UUID;

@Repository
public interface {Modulo}Repository extends JpaRepository<{Modulo}Entity, UUID>, JpaSpecificationExecutor<{Modulo}Entity> {

    List<{Modulo}Entity> findByEstado(String estado);

    boolean existsByCampo(String campo);
}
```

---