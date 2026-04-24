# 🧪 Testing - Guía de Pruebas Unitarias

---

## 1. Configuración de Testing

### Dependencias necesarias (pom.xml)

```xml
<!-- JUnit 5 -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- Mockito -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- MockMvc -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

###Estructura de tests

```
src/test/java/com/muta/{modulo}/
├── controller/
│   └── {Modulo}ControllerTest.java
├── service/
│   └── {Modulo}ServiceTest.java
└── usecase/
    └── {Accion}UseCaseTest.java
```

---

## 2. Testing Service

### Qué testear

| Método | Qué verificar |
|--------|-------------|
| `create()` | Se guarda correctamente |
| `findById()` | Retorna Response o lanza excepción |
| `findAll()` | Retorna lista |
| `update()` | Actualiza campos |
| `delete()` | Llama al repository |

### Template de test

```java
package com.muta.{modulo}.service;

import com.muta.{modulo}.dto.{Modulo}Request;
import com.muta.{modulo}.dto.{Modulo}Response;
import com.muta.{modulo}.entity.{Modulo}Entity;
import com.muta.{modulo}.repository.{Modulo}Repository;
import com.muta.common.exception.NotFoundException;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.List;
import java.util.Optional;
import java.util.UUID;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class {Modulo}ServiceTest {

    @Mock
    private {Modulo}Repository repository;

    @InjectMocks
    private {Modulo}Service service;

    private {Modulo}Entity entity;
    private {Modulo}Request request;
    private {Modulo}Response response;
    private UUID id;

    @BeforeEach
    void setUp() {
        id = UUID.randomUUID();
        entity = {Modulo}Entity.builder()
            .id(id)
            .nombre("Test")
            .build();
        request = {Modulo}Request.builder()
            .nombre("Test")
            .build();
        response = {Modulo}Response.builder()
            .id(id)
            .nombre("Test")
            .build();
    }

    // ============ CREATE ============

    @Test
    void create_deberiaGuardarEntity() {
        // Given
        when(repository.save(any({Modulo}Entity.class))).thenReturn(entity);

        // When
        {Modulo}Response result = service.create(request);

        // Then
        assertNotNull(result);
        verify(repository, times(1)).save(any({Modulo}Entity.class));
    }

    // ============ FIND BY ID ============

    @Test
    void findById_deberiaRetornarResponse() {
        // Given
        when(repository.findById(id)).thenReturn(Optional.of(entity));

        // When
        {Modulo}Response result = service.findById(id);

        // Then
        assertNotNull(result);
        assertEquals(id, result.id());
    }

    @Test
    void findById_deberiaLanzarNotFoundException() {
        // Given
        when(repository.findById(id)).thenReturn(Optional.empty());

        // When / Then
        assertThrows(NotFoundException.class, () -> service.findById(id));
    }

    // ============ FIND ALL ============

    @Test
    void findAll_deberiaRetornarLista() {
        // Given
        when(repository.findAll()).thenReturn(List.of(entity));

        // When
        List<{Modulo}Response> result = service.findAll();

        // Then
        assertEquals(1, result.size());
    }

    // ============ UPDATE ============

    @Test
    void update_deberiaActualizarEntity() {
        // Given
        when(repository.findById(id)).thenReturn(Optional.of(entity));
        when(repository.save(any({Modulo}Entity.class))).thenReturn(entity);

        // When
        {Modulo}Response result = service.update(id, request);

        // Then
        assertNotNull(result);
        verify(repository, times(1)).save(any({Modulo}Entity.class));
    }

    // ============ DELETE ============

    @Test
    void delete_deberiaEliminarEntity() {
        // Given
        when(repository.existsById(id)).thenReturn(true);
        doNothing().when(repository).deleteById(id);

        // When
        service.delete(id);

        // Then
        verify(repository, times(1)).deleteById(id);
    }
}
```

---

## 3. Testing UseCase

### Qué testear

| Método | Qué verificar |
|--------|-------------|
| `preConditions()` | Validaciones previas |
| `core()` | Lógica de negocio |
| `postConditions()` | Efectos secundarios |
| `execute()` | Flujo completo |

### Template de test

```java
package com.muta.{modulo}.usecase;

import com.muta.{modulo}.usecase.dto.{Accion}Request;
import com.muta.{modulo}.usecase.dto.{Accion}Response;
import com.muta.{modulo}.service.{Modulo}Service;
import com.muta.common.exception.ValidationException;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.UUID;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class {Accion}UseCaseTest {

    @Mock
    private {Modulo}Service service;

    @InjectMocks
    private {Accion}UseCase useCase;

    private {Accion}Request request;
    private {Accion}Response response;
    private UUID id;

    @BeforeEach
    void setUp() {
        id = UUID.randomUUID();
        request = {Accion}Request.builder()
            .id(id)
            .campo("test")
            .build();
        response = {Accion}Response.builder()
            .id(id)
            .build();
    }

    // ============ PRE CONDITIONS ============

    @Test
    void execute_deberiaLanzarValidationExceptionSiRequestInvalido() {
        // Given
        {Accion}Request invalidRequest = {Accion}Request.builder()
            .id(null)  // inválido
            .build();

        // When / Then
        assertThrows(ValidationException.class, () -> useCase.execute(invalidRequest));
    }

    // ============ CORE ============

    @Test
    void execute_deberiaEjecutarLogica() {
        // Given
        when(service.findById(any())).thenReturn(null); // Mock response
        when(service.update(any(), any())).thenReturn(response);

        // When
        {Accion}Response result = useCase.execute(request);

        // Then
        assertNotNull(result);
        verify(service, times(1)).update(any(), any());
    }

    // ============ POST CONDITIONS ============

    @Test
    void execute_deberiaEjecutarPostConditions() {
        // Given
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any())).thenReturn(response);

        // When
        useCase.execute(request);

        // Then - verificar efectos secundarios
        verify(service, times(1)).update(any(), any());
    }

    // ============ FLUJO COMPLETO ============

    @Test
    void execute_flujoCompleto() {
        // Given - todo mockeado
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any())).thenReturn(response);

        // When
        {Accion}Response result = useCase.execute(request);

        // Then
        assertNotNull(result);
        verify(service).findById(any());
        verify(service).update(any(), any());
    }
}
```

---

## 4. Testing Controller

### Qué testear

| Método | Qué verificar |
|--------|-------------|
| GET `/` | Retorna 200 + lista |
| GET `/{id}` | Retorna 200 o 404 |
| POST | Retorna 201 |
| PUT `/{id}` | Retorna 200 |
| DELETE `/{id}` | Retorna 204 |

### Template de test

```java
package com.muta.{modulo}.controller;

import com.muta.{modulo}.adapter.{Modulo}Adapter;
import com.muta.{modulo}.dto.{Modulo}Request;
import com.muta.{modulo}.dto.{Modulo}Response;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.List;
import java.util.UUID;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest({Modulo}Controller.class)
class {Modulo}ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private {Modulo}Adapter adapter;

    private {Modulo}Response response;
    private UUID id;
    private String requestJson;

    @BeforeEach
    void setUp() {
        id = UUID.randomUUID();
        response = {Modulo}Response.builder()
            .id(id)
            .nombre("Test")
            .build();
        requestJson = "{\"nombre\": \"Test\"}";
    }

    // ============ GET ALL ============

    @Test
    void findAll_deberiaRetornar200() throws Exception {
        // Given
        when(adapter.findAll()).thenReturn(List.of(response));

        // When / Then
        mockMvc.perform(get("/api/v1/{recurso}"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].id").value(id.toString()))
            .andExpect(jsonPath("$[0].nombre").value("Test"));
    }

    // ============ GET BY ID ============

    @Test
    void findById_deberiaRetornar200() throws Exception {
        // Given
        when(adapter.findById(id)).thenReturn(response);

        // When / Then
        mockMvc.perform(get("/api/v1/{recurso}/" + id))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(id.toString()));
    }

    @Test
    void findById_deberiaRetornar404() throws Exception {
        // Given
        when(adapter.findById(id)).thenReturn(null);

        // When / Then
        mockMvc.perform(get("/api/v1/{recurso}/" + id))
            .andExpect(status().isNotFound());
    }

    // ============ POST ============

    @Test
    void create_deberiaRetornar201() throws Exception {
        // Given
        when(adapter.create(any({Modulo}Request.class))).thenReturn(response);

        // When / Then
        mockMvc.perform(post("/api/v1/{recurso}")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(id.toString()));
    }

    // ============ PUT ============

    @Test
    void update_deberiaRetornar200() throws Exception {
        // Given
        when(adapter.update(eq(id), any({Modulo}Request.class))).thenReturn(response);

        // When / Then
        mockMvc.perform(put("/api/v1/{recurso}/" + id)
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(id.toString()));
    }

    // ============ DELETE ============

    @Test
    void delete_deberiaRetornar204() throws Exception {
        // Given
        doNothing().when(adapter).delete(id);

        // When / Then
        mockMvc.perform(delete("/api/v1/{recurso}/" + id))
            .andExpect(status().isNoContent());
    }
}
```

---

## 5. Reglas de Testing

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** testear el happy path |
| 2 | **SIEMPRE** testear casos de error (excepciones) |
| 3 | Usar `@Mock` para dependencias |
| 4 | Usar `@InjectMocks` para la clase a testear |
| 5 | Verificar con `verify()` las llamadas a mocks |
| 6 | NUNCA testear con datos reales en BD |
| 7 | Nombrar test descriptivamente: `metodo_deberiaHacerX()` |

---

## 6. Anotaciones útiles

| Anotación | Uso |
|-----------|-----|
| `@ExtendWith(MockitoExtension.class)` | Habilitar Mockito |
| `@Mock` | Crear mock automático |
| `@InjectMocks` | Inyectar mocks en clase |
| `@BeforeEach` | Ejecutar antes de cada test |
| `@Test` | Método de test |
| `@WebMvcTest` | Testear Controller web |

---

## 7. Checklist de test

| # | Capa | Qué testear |
|---|------|-------------|
| 1 | Service | CRUD básico, excepciones |
| 2 | UseCase | preConditions, core, postConditions |
| 3 | Controller | HTTP status, respuestas JSON |

---