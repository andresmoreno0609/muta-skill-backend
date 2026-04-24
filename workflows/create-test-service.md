# 🧪 Testing Service

---

## 1. Qué testear en Service

| Método | Qué verificar |
|--------|---------------|
| `create()` | Se guarda correctamente en BD |
| `findById()` | Retorna Response o lanza NotFoundException |
| `findAll()` | Retorna lista de Responses |
| `update()` | Actualiza campos correctly |
| `delete()` | Elimina correctly |

---

## 2. Dependencias

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 3. Template

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

    @Test
    void create_deberiaRetornarResponse() {
        // Given
        when(repository.save(any({Modulo}Entity.class))).thenReturn(entity);

        // When
        {Modulo}Response result = service.create(request);

        // Then
        assertEquals(response.nombre(), result.nombre());
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

    @Test
    void findAll_deberiaRetornarListaVacia() {
        // Given
        when(repository.findAll()).thenReturn(List.of());

        // When
        List<{Modulo}Response> result = service.findAll();

        // Then
        assertTrue(result.isEmpty());
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

    @Test
    void update_deberiaLanzarNotFoundException() {
        // Given
        when(repository.findById(id)).thenReturn(Optional.empty());

        // When / Then
        assertThrows(NotFoundException.class, () -> service.update(id, request));
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

    @Test
    void delete_deberiaLanzarNotFoundException() {
        // Given
        when(repository.existsById(id)).thenReturn(false);

        // When / Then
        assertThrows(NotFoundException.class, () -> service.delete(id));
    }

    // ============ EXISTS BY ID ============

    @Test
    void existsById_deberiaRetornarTrue() {
        // Given
        when(repository.existsById(id)).thenReturn(true);

        // When
        boolean result = service.existsById(id);

        // Then
        assertTrue(result);
    }

    @Test
    void existsById_deberiaRetornarFalse() {
        // Given
        when(repository.existsById(id)).thenReturn(false);

        // When
        boolean result = service.existsById(id);

        // Then
        assertFalse(result);
    }
}
```

---

## 4. Reglas

| # | Regla |
|---|-------|
| 1 | Extender `BaseService` - tested indirectly |
| 2 | Usar `@Mock` para Repository |
| 3 | Usar `@InjectMocks` para Service |
| 4 | Testear happy path + casos de error |
| 5 | Verificar llamadas a repository con `verify()` |
| 6 | Lanzar excepciones en casos de error |

---