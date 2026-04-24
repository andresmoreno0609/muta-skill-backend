# 🔌 Crear Entity

---

## 1. Cuándo crear una Entity

| Trigger | Ejemplo |
|---------|--------|
| Nueva tabla en BD | Se necesita guardar un nuevo recurso |
| Mapping de tabla | Mapear una tabla existente |
| Definir estructura de datos | La entitydefine la tabla |

---

## 2. Checklist previa

- [ ] Se know el nombre de la tabla en BD
- [ ] Se know los nombres de las columnas
- [ ] Se know los tipos de dato
- [ ] Se know las relaciones con otras entidades

---

## 3. Paso a Paso

### Paso 1:Crear la Entity

```java
@Entity
@Table(name = "nombre_tabla")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class {Modulo}Entity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;
}
```

### Paso 2: Agregar campos

```java
@Column(name = "nombre", nullable = false, length = 100)
private String nombre;

// Enum
@Enumerated(EnumType.STRING)
private EEstatusModulo estado;

// Timestamps
@Column(name = "created_at", updatable = false)
private LocalDateTime createdAt;
```

### Paso 3: Agregar callbacks

```java
@PrePersist
protected void onCreate() {
    createdAt = LocalDateTime.now();
}

@PreUpdate
protected void onUpdate() {
    updatedAt = LocalDateTime.now();
}
```

---

## 4. Template completo

```java
package com.muta.{modulo}.entity;

import com.muta.{modulo}.enum.EEstatus{Modulo};
import lombok.*;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.UUID;

@Entity
@Table(
    name = "{tabla}",
    indexes = {
        @Index(name = "idx_{campo}", columnList = "{campo}"),
        @Index(name = "idx_estado", columnList = "estado")
    }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class {Modulo}Entity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    @Column(name = "id", updatable = false)
    private UUID id;

    @Column(name = "nombre", nullable = false, length = 100)
    private String nombre;

    @Column(name = "email", unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    @Column(name = "estado", length = 20)
    private EEstatus{Modulo} estado;

    @Column(name = "descripcion", columnDefinition = "TEXT")
    private String descripcion;

    @Column(name = "activo")
    private Boolean activo;

    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    // ============ Callbacks ============

    @PrePersist
    protected void onCreate() {
        if (id == null) {
            id = UUID.randomUUID();
        }
        if (activo == null) {
            activo = true;
        }
        if (estado == null) {
            estado = EEstatus{Modulo}.ACTIVO;
        }
    }

    @PreUpdate
    protected void onUpdate() {
        // Lógica antes de actualizar
    }
}
```

---

## 5. Tipos de Columnas

| Tipo Java | Columna BD | Anotación |
|----------|-----------|----------|
| String | VARCHAR | `@Column(length = 100)` |
| String large | TEXT | `@Column(columnDefinition = "TEXT")` |
| Integer | INT | `@Column` |
| Long | BIGINT | `@Column` |
| BigDecimal | DECIMAL | `@Column(precision = 10, scale = 2)` |
| LocalDateTime | TIMESTAMP | `@Column` |
| Boolean | BOOLEAN | `@Column` |
| Enum | VARCHAR | `@Enumerated(EnumType.STRING)` |
| UUID | UUID | `@GeneratedValue` |

---

## 6. Relaciones

```java
// ManyToOne
@ManyToOne
@JoinColumn(name = "usuario_id")
private UsuarioEntity usuario;

// OneToMany
@OneToMany(mappedBy = "{modulo}")
private List<{Item}Entity> items;

// ManyToMany
@ManyToMany
@JoinTable(
    name = "{tabla}_rel",
    joinColumns = @JoinColumn(name = "{modulo}_id"),
    inverseJoinColumns = @JoinColumn(name = "otro_id")
)
private Set<OtroEntity> otros;
```

---

## 7. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** usar `UUID` como ID |
| 2 | Usar `@Enumerated(EnumType.STRING)` para enums |
| 3 | Usar `@Column` para nombrar columnas |
| 4 | Agregar `@PrePersist` para timestamps |
| 5 | **NUNCA** lógica de negocio |
| 6 | Usar índices si se queryea por ese campo |

---