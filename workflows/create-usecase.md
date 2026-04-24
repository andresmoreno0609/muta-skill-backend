# 🔌 Crear UseCase

---

## 1. Cuándo crear un UseCase

| Trigger | Ejemplo |
|---------|--------|
| Lógica de negocio compleja | Proceso de checkout con validaciones |
| Múltiples operaciones | Orchestrar varios servicios |
| Transacciones múltiples | Transferencia entre cuentas |
| Validaciones específicas | Verificar stock antes de vender |
| Notificaciones post-operación | Enviar email después de compra |
| Orchestración de servicios | Generar factura completa |

### Señales de que necesitas un UseCase

- [ ] Hay más de 2 entidades involucradas
- [ ] Hay validaciones de negocio específicas
- [ ] Hay efectos secundarios (notificaciones, logs, eventos)
- [ ] La operación no es solo CRUD básico

---

## 2. Checklist previa

- [ ] Los Services relacionados ya existen
- [ ] Los DTOs específicos del UseCase existen
- [ ] Se know las pre-conditions necesarias
- [ ] Se know el flujo core
- [ ] Se know los post-conditions (si hay)

---

## 3. Paso a Paso

### Paso 1: Crear Request DTO específico

```java
// Ubicación: src/main/java/com/muta/{modulo}/usecase/dto/{Accion}Request.java
@Builder
public record {Accion}Request(
    UUID id,
    String campo,
    Integer cantidad
) {}
```

### Paso 2: Extender BaseUseCase

```java
// Ubicación: src/main/java/com/muta/{modulo}/usecase/{Accion}UseCase.java
@Component
@Slf4j
public class {Accion}UseCase extends BaseUseCase<{Accion}Request, {Accion}Response> {

    private final {Modulo}Service service;
    private final {Otro}Service otroService;
}
```

### Paso 3: Implementar los métodos

```java
// Pre-conditions: validaciones previas
@Override
protected void preConditions({Accion}Request request) {
    if (request.cantidad() <= 0) {
        throw new ValidationException("Cantidad debe ser mayor a 0");
    }
}

// Core: lógica principal
@Override
protected {Accion}Response core({Accion}Request request) {
    // Lógica aquí
    return response;
}

// Post-conditions: efectos secundarios (optional)
@Override
protected void postConditions({Accion}Response response) {
    // Notificaciones, logs, etc.
}
```

---

## 4. Template completo

```java
package com.muta.{modulo}.usecase;

import com.muta.common.usecase.BaseUseCase;
import com.muta.{modulo}.usecase.dto.{Accion}Request;
import com.muta.{modulo}.usecase.dto.{Accion}Response;
import com.muta.{modulo}.service.{Modulo}Service;
import com.muta.common.exception.NotFoundException;
import com.muta.common.exception.ValidationException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.util.UUID;

@Component
@Slf4j
public class {Accion}UseCase extends BaseUseCase<{Accion}Request, {Accion}Response> {

    private final {Modulo}Service service;

    // ============ Pre-conditions ============

    @Override
    protected void preConditions({Accion}Request request) {
        // Validaciones previas

        // Ejemplo: verificar que existe
        var entity = service.findById(request.id())
            .orElseThrow(() -> new NotFoundException("Recurso", request.id()));

        // Ejemplo: validar regla de negocio
        if (entity.getEstado() != EEstatus.ACTIVO) {
            throw new ValidationException("El recurso debe estar activo");
        }
    }

    // ============ Core ============

    @Override
    protected {Accion}Response core({Accion}Request request) {
        // Lógica principal
        // Orchestración de servicios, transacciones, etc.

        var entity = service.findById(request.id()).orElseThrow();

        // Procesar
        entity.setCampo(request.campo());
        service.update(entity);

        // Devolver respuesta
        return {Accion}Response.builder()
            .id(entity.getId())
            .build();
    }

    // ============ Post-conditions ============

    @Override
    protected void postConditions({Accion}Response response) {
        // Efectos secundarios

        // Ejemplo: enviar notificación
        // notificationService.send(...);

        // Ejemplo: crear log
        log.info("Proceso completado para: {}", response.id());
    }
}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** extender `BaseUseCase` |
| 2 | Usar `@Component`, `@Slf4j` |
| 3 | **Obligatorio** implementar `core(request)` |
| 4 | Usar `preConditions` para validaciones |
| 5 | Usar `postConditions` para efectos secundarios |
| 6 | **NUNCA** retornar Entity directamente |
| 7 | Usar transacciones cuando sea necesario |

---

## 6. Estructura de archivos

```
{modulo}/usecase/
├── {Accion}UseCase.java        ← UseCase principal
└── dto/
    ├── {Accion}Request.java  ← Input DTO
    └── {Accion}Response.java ← Output DTO
```

---

## 7. Flujo completo

```
1. Controller llama al Adapter
        ↓
2. Adapter decide: Service o UseCase
        ↓
3. UseCase.execute()
        ↓
   ├── preConditions() → validaciones
        ↓
   ├── core() → lógica principal
        ↓
   ├── postConditions() → efectos
        ↓
4. Retorna Response
```

---

## 8. Service vs UseCase (resumen)

| Escenario | Usar |
|---------|------|
| CRUD básico | `Service` (ya viene en BaseService) |
| Consulta simple | `Service` |
| Validaciones simples | `Service` |
| Lógica compleja | `UseCase` |
| Múltiples servicios | `UseCase` |
| Transacciones | `UseCase` |
| Notificaciones | `UseCase` (en postConditions) |

---