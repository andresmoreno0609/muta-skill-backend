# 🔌 Crear Enum

---

## 1. Cuándo crear un Enum

| Trigger | Ejemplo |
|---------|--------|
| Valores fijos | Estados (ACTIVO, INACTIVO) |
| Tipos | Tipos de documento (FACTURA, BOLETA) |
| Catálogos | Opciones que no cambian frecuentemente |
| Datos de tablas estáticas | Roles, permisos, categorías |

---

## 2. Checklist previa

- [ ] Se know los valores posibles
- [ ] Se know el código y descripción de cada valor
- [ ] Se sabe si aplica para una entidad específica o es global

---

## 3. Paso a Paso

### Paso 1:Crear interfaz base (opcional pero recomendado)

```java
public interface CodeEnum {
    String getCode();
    String getDescripcion();
}
```

### Paso 2:Crear el Enum

```java
public enum EEstatusOrden implements CodeEnum {
    PENDIENTE("PEN", "Pendiente"),
    COMPLETADO("COM", "Completado"),
    CANCELADO("CAN", "Cancelado");

    private final String code;
    private final String descripcion;

    EEstatusOrden(String code, String descripcion) {
        this.code = code;
        this.descripcion = descripcion;
    }

    public String getCode() { return code; }
    public String getDescripcion() { return descripcion; }

    public static EEstatusOrden fromCode(String code) {
        return Arrays.stream(values())
            .filter(e -> e.code.equals(code))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Código inválido: " + code));
    }
}
```

---

## 4. Template completo

```java
package com.muta.{modulo}.enum;

import lombok.Getter;

public enum EEstatus{Modulo} implements CodeEnum {

    // ============ Valores ============
    ACTIVO("ACT", "Activo"),
    INACTIVO("INA", "Inactivo"),
    ELIMINADO("ELI", "Eliminado");

    // ============ Código y Descripción ============
    private final String code;
    private final String descripcion;

    EEstatus{Modulo}(String code, String descripcion) {
        this.code = code;
        this.descripcion = descripcion;
    }

    @Override
    public String getCode() {
        return code;
    }

    @Override
    public String getDescripcion() {
        return descripcion;
    }

    // ============ Buscar por código ============
    public static EEstatus{Modulo} fromCode(String code) {
        if (code == null) {
            return null;
        }
        return Arrays.stream(values())
            .filter(e -> e.code.equalsIgnoreCase(code))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException(
                "Código inválido para EEstatus{Modulo}: " + code
            ));
    }

    // ============ Buscar por nombre ============
    public static EEstatus{Modulo} fromName(String name) {
        return valueOf(name);
    }
}
```

### Template con interfaz base

```java
package com.muta.common.enum;

public interface CodeEnum {
    String getCode();
    String getDescripcion();
}
```

---

## 5. Usar en Entity

```java
@Enumerated(EnumType.STRING)
@Column(name = "estado", length = 20)
private EEstatus{Modulo} estado;

// Getter
public EEstatus{Modulo} getEstado() {
    return estado;
}
```

---

## 6. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** prefijo `E` en el nombre |
| 2 | Usar `EnumType.STRING` en @Enumerated |
| 3 | Incluir código + descripción |
| 4 | Implementar `fromCode()` para búsqueda |
| 5 | **NUNCA** crear entidad para valores fijos |
| 6 | Usar en Entity con `@Enumerated(EnumType.STRING)` |

---