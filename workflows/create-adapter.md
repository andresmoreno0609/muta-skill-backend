# 🔌 Crear Adapter

---

## 1. Cuándo crear un Adapter

| Trigger | Ejemplo |
|---------|--------|
| Neue Endpoint HTTP | Cuando hay un Controller que lo necesita |
| Traducción DTOs | Cuando hay Request → objeto interno |
| Decidir Service vs UseCase | Cuando la operación puede ser CRUD o compleja |
| Múltiples operaciones | Cuando se orquestan varios servicios |

---

## 2. Checklist previa

- [ ] El Service ya existe (para CRUD)
- [ ] El UseCase ya existe (para lógica compleja)
- [ ] Los DTOs (Request/Response) ya existen
- [ ] Se sabe qué operaciones van a Service y cuáles a UseCase

---

## 3. Paso a Paso

### Paso 1: Crear la Interfaz

```java
// Ubicación: src/main/java/com/muta/{modulo}/adapter/{Modulo}Adapter.java
public interface {Modulo}Adapter {
    {Modulo}Response create({Modulo}Request request);
    {Modulo}Response findById(UUID id);
    List<{Modulo}Response> findAll();
    {Modulo}Response update(UUID id, {Modulo}Request request);
    void delete(UUID id);
}
```

### Paso 2: Crear la Implementación

```java
// Ubicación: src/main/java/com/muta/{modulo}/adapter/{Modulo}AdapterImpl.java
@Service                        // @Service para que se injecte
@RequiredArgsConstructor
@Slf4j
public class {Modulo}AdapterImpl implements {Modulo}Adapter {

    private final {Modulo}Service service;
    private final {Accion}UseCase accionUseCase;  // si hay lógica compleja
}
```

### Paso 3: Definir los métodos

| Método | Lógica | Cuándo usar |
|--------|--------|-----------|
| create() | Si es simple → `service.create()`<br>Si es complejo → `useCase.execute()` | Operación de creación |
| findById() | `service.findById()` | Consulta por ID |
| findAll() | `service.findAll()` | Listar todos |
| update() | Si es simple → `service.update()`<br>Si es complejo → `useCase.execute()` | Actualización |
| delete() | `service.delete()` | Eliminación |

---

## 4. Template completo

```java
package com.muta.{modulo}.adapter;

import com.muta.{modulo}.dto.{Modulo}Request;
import com.muta.{modulo}.dto.{Modulo}Response;
import com.muta.{modulo}.service.{Modulo}Service;
import com.muta.{modulo}.usecase.{Accion}UseCase;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.UUID;

public interface {Modulo}Adapter {

    {Modulo}Response create({Modulo}Request request);

    {Modulo}Response findById(UUID id);

    List<{Modulo}Response> findAll();

    {Modulo}Response update(UUID id, {Modulo}Request request);

    void delete(UUID id);
}
```

```java
// Implementación
@Service
@RequiredArgsConstructor
@Slf4j
public class {Modulo}AdapterImpl implements {Modulo}Adapter {

    private final {Modulo}Service service;
    private final {Accion}UseCase accionUseCase;  // opcional, solo si hay lógica compleja

    @Override
    public {Modulo}Response create({Modulo}Request request) {
        // Decidir: Service o UseCase
        if (esOperacionSimple(request)) {
            return service.create(request);
        } else {
            return accionUseCase.execute(toUseCaseRequest(request));
        }
    }

    @Override
    public {Modulo}Response findById(UUID id) {
        return service.findById(id);
    }

    @Override
    public List<{Modulo}Response> findAll() {
        return service.findAll();
    }

    @Override
    public {Modulo}Response update(UUID id, {Modulo}Request request) {
        if (esOperacionSimple(request)) {
            return service.update(id, request);
        } else {
            return accionUseCase.execute(toUseCaseRequest(request));
        }
    }

    @Override
    public void delete(UUID id) {
        service.delete(id);
    }

    // ============ Helpers ============

    private boolean esOperacionSimple({Modulo}Request request) {
        // Lógica simple: solo create/update sin validaciones complejas
        return true; // o false según el caso
    }

    private {Accion}Request toUseCaseRequest({Modulo}Request request) {
        // Transformar Request a Request del UseCase
        return {Accion}Request.builder()
            .campo(request.campo())
            .build();
    }
}
```

---

## 5. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** crear Interfaz + Implementación |
| 2 | Usar `@Service` en la Impl, no en la interfaz |
| 3 | Usar `@RequiredArgsConstructor` para inyección |
| 4 | **NUNCA** lógica de negocio, solo traducción y decisión |
| 5 | Delegates al Service para CRUD básico |
| 6 | Delegates al UseCase para lógica compleja |
| 7 | Traducir Request → objeto interno |
| 8 | Traducir objeto interno → Response |

---

## 6. Decision: Service vs UseCase

| Operación | Usar |
|-----------|------|
| Create/Update/Delete básico | `service.create()` / `service.update()` / `service.delete()` |
| FindById, FindAll | `service.findById()` / `service.findAll()` |
| Validaciones simples | `service.create()` |
| Múltiples servicios | `useCase.execute()` |
| Lógica de negocio | `useCase.execute()` |
| Transacciones múltiples | `useCase.execute()` |

---