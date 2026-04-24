# 🔌 Crear Repository

---

## 1. Cuándo crear un Repository

| Trigger | Ejemplo |
|---------|--------|
| Nueva Entity | Se creó una nueva entidad |
| Consultas específicas | Necesito findByEmail, findByEstado |
| Queries con JPQL | Consultation compleja con joins |
| Specifications | Búsqueda dinámica con filtros |

---

## 2. Checklist previa

- [ ] La Entity ya existe
- [ ] Se know los nombres de los campos en la Entity
- [ ] Se know los tipos de dato de cada campo

---

## 3. Paso a Paso

### Paso 1: Crear la Interfaz

```java
// Ubicación: src/main/java/com/muta/{modulo}/repository/{Modulo}Repository.java
@Repository
public interface {Modulo}Repository extends JpaRepository<{Modulo}Entity, UUID>, JpaSpecificationExecutor<{Modulo}Entity> {
    // Queries simples van aquí
}
```

### Paso 2: Agregar métodos de query

```java
// Por nombre de método (Spring Data)
Optional<{Modulo}Entity> findByEmail(String email);
boolean existsByEmail(String email);
List<{Modulo}Entity> findByEstado(String estado);
```

### Paso 3: Para queries complejas

```java
// Usar @Query con JPQL
@Query("SELECT e FROM {Modulo}Entity e WHERE e.campo = :campo AND e.estado = :estado")
List<{Modulo}Entity> findByCampoAndEstado(@Param("campo") String campo, @Param("estado") String estado);
```

---

## 4. Template completo

```java
package com.muta.{modulo}.repository;

import com.muta.{modulo}.entity.{Modulo}Entity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Repository
public interface {Modulo}Repository extends JpaRepository<{Modulo}Entity, UUID>, JpaSpecificationExecutor<{Modulo}Entity> {

    // ============ Query Methods ( Spring Data ) ============

    // findBy{campo}
    Optional<{Modulo}Entity> findByEmail(String email);

    // existsBy{campo}
    boolean existsByEmail(String email);

    // findBy{campo}{Campo2}
    List<{Modulo}Entity> findByEstadoAndActivo(String estado, Boolean activo);

    // countBy{campo}
    long countByEstado(String estado);

    // deleteBy{campo}
    void deleteByEmail(String email);

    // ============ @Query ( JPQL ) ============

    @Query("SELECT e FROM {Modulo}Entity e WHERE e.email = :email")
    Optional<{Modulo}Entity> buscarPorEmail(@Param("email") String email);

    @Query("SELECT e FROM {Modulo}Entity e WHERE e.estado = :estado AND e.activo = true")
    List<{Modulo}Entity> findActivosByEstado(@Param("estado") String estado);

    @Query("SELECT e FROM {Modulo}Entity e WHERE e.campo = :campo ORDER BY e.createdAt DESC")
    List<{Modulo}Entity> findByCampoOrdenado(@Param("campo") String campo);
}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** extender `JpaRepository<Entity, UUID>` |
| 2 | **SIEMPRE** agregar `JpaSpecificationExecutor<Entity>` |
| 3 | Usar `@Repository` |
| 4 | UUID como tipo de ID |
| 5 | Preferir Query Methods sobre @Query |
| 6 | Usar `@Query` solo cuando Query Methods no alcance |
| 7 | **NUNCA** lógica de negocio |

---

## 6. Tipos de Queries

### 6.1 Query Methods (recomendado)

```java
// Igual (=)
findByEmail(String email)

// Like
findByNombreContaining(String nombre)

// Mayor que
findByEdadGreaterThan(Integer edad)

// Enum
findByEstado(EEstatus estado)

// Múltiples campos
findByEstadoAndActivo(String estado, Boolean activo)

// Like ignore case
findByNombreContainingIgnoreCase(String nombre)
```

### 6.2 @Query con JPQL

```java
@Query("SELECT e FROM Entity e WHERE e.campo = :valor")
List<Entity> findConParametro(@Param("valor") String valor);

@Query("SELECT e FROM Entity e WHERE e.campo = :valor AND e.otro = :otro")
List<Entity> findConMultiples(@Param("valor") String v1, @Param("otro") String v2);
```

### 6.3 Specification (búsqueda dinámica)

```java
public Specification<{Modulo}Entity> hasEmail(String email) {
    return (root, query, cb) -> {
        if (email == null) return null;
        return cb.equal(root.get("email"), email);
    };
}
```

---

## 7. Métodos heredados de JpaRepository

| Método | Descripción |
|--------|-----------|
| `save(entity)` | Crear o actualizar |
| `findById(id)` | Buscar por ID |
| `existsById(id)` | Verificar si existe |
| `findAll()` | Listar todos |
| `deleteById(id)` | Eliminar por ID |
| `delete(entity)` | Eliminar entity |
| `count()` | Contar total |

---

## 8. JpaSpecificationExecutor

```java
// Para búsquedas dinámicas con filtros
public List<{Modulo}Entity> findAll(Specification<{Modulo}Entity> spec) {
    return repository.findAll(spec);
}

// Uso
Specification<{Modulo}> spec = Specifications
    .where(repo.hasEmail(email))
    .and(repo.hasEstado(estado));
```

---