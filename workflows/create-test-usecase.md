# 🧪 Testing UseCase

---

## 1. Qué testear en UseCase

| Fase | Qué verificar |
|------|---------------|
| `preConditions()` | Validaciones previas |
| `core()` | Lógica de negocio principal |
| `postConditions()` | Efectos secundarios |
| `execute()` | Flujo completo |

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
package com.muta.{modulo}.usecase;

import com.muta.{modulo}.usecase.dto.{Accion}Request;
import com.muta.{modulo}.usecase.dto.{Accion}Response;
import com.muta.{modulo}.service.{Modulo}Service;
import com.muta.common.exception.NotFoundException;
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

    // ============ PRE CONDITIONS - VALIDACIONES ============

    @Test
    void execute_deberiaLanzarValidationExceptionSiCampoInvalido() {
        // Given
        {Accion}Request invalidRequest = {Accion}Request.builder()
            .id(id)
            .campo(null)  // inválido - causa error en preConditions
            .build();

        // When / Then
        assertThrows(ValidationException.class, () -> useCase.execute(invalidRequest));
    }

    @Test
    void execute_deberiaLanzarNotFoundExceptionSiRecursoNoExiste() {
        // Given
        when(service.findById(any())).thenReturn(null);

        // When / Then
        assertThrows(NotFoundException.class, () -> useCase.execute(request));
    }

    @Test
    void preConditions_deberiaValidarEstado() {
        // Given - simulate error in preConditions
        {Accion}Request requestInactivo = {Accion}Request.builder()
            .id(id)
            .campo("test")
            .build();

        // When / Then - depends on your preConditions logic
        assertThrows(ValidationException.class, () -> useCase.execute(requestInactivo));
    }

    // ============ CORE - LÓGICA PRINCIPAL ============

    @Test
    void core_deberiaEjecutarLogica() {
        // Given
        when(service.update(any(), any())).thenReturn(null); // Mock response

        // When
        // Note: We test through execute() which calls core internally
        // Just verify the service method was called
        // In your actual test, you'd verify the specific logic

        // Then - this is just example pattern
        verify(service, never()).update(any(), any());  // Not called yet
    }

    @Test
    void execute_flujoCompleto_ejecutaCore() {
        // Given - prepare mocks for full flow
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any())).thenReturn(response);

        // When
        {Accion}Response result = useCase.execute(request);

        // Then
        assertNotNull(result);
    }

    // ============ POST CONDITIONS - EFECTOS SECUNDARIOS ============

    @Test
    void postConditions_deberiaEjecutarPostConditions() {
        // Given
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any())).thenReturn(response);

        // When
        useCase.execute(request);

        // Then - verify side effects (notification, logging, etc.)
        // depends on your postConditions implementation
    }

    // ============ FLUJO COMPLETO ============

    @Test
    void execute_flujoCompleto_deberiaRetornarResponse() {
        // Given - complete happy path
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any())).thenReturn(response);

        // When
        {Accion}Response result = useCase.execute(request);

        // Then
        assertNotNull(result);
        assertEquals(id, result.id());
    }

    @Test
    void execute_casoError_validacionFalla() {
        // Given - invalid input
        {Accion}Request invalidRequest = {Accion}Request.builder()
            .id(null)  // will trigger validation error
            .build();

        // When / Then
        assertThrows(ValidationException.class, () -> useCase.execute(invalidRequest));
        verify(service, never()).update(any(), any());  // Never reached
    }

    @Test
    void execute_casoError_recursoNoEncontrado() {
        // Given - resource doesn't exist
        when(service.findById(any())).thenReturn(null);

        // When / Then
        assertThrows(NotFoundException.class, () -> useCase.execute(request));
    }

    // ============ ERROR HANDLING ============

    @Test
    void execute_deberiaManejarExcepcionEnCore() {
        // Given
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any()))
            .thenThrow(new RuntimeException("Error de DB"));

        // When / Then
        assertThrows(RuntimeException.class, () -> useCase.execute(request));
    }

    @Test
    void execute_deberiaPropagarExcepcionSinManejar() {
        // Given - unexpected error in core
        when(service.findById(any())).thenReturn(null);
        when(service.update(any(), any()))
            .thenThrow(new IllegalStateException("Error inesperado"));

        // When / Then
        assertThrows(IllegalStateException.class, () -> useCase.execute(request));
    }
}
```

---

## 4. Reglas

| # | Regla |
|---|-------|
| 1 | Usar `@ExtendWith(MockitoExtension.class)` |
| 2 | Testear preConditions - validaciones previas |
| 3 | Testear core - lógica principal |
| 4 | Testear postConditions - efectos secundarios |
| 5 | Testear execute() - flujo completo |
| 6 | Testear casos de error |
| 7 | Verificar que el Service fue llamado correctamente |
| 8 | Verificar que no se llama servicio si preConditions falla |

---

## 5. Estructura de archivo test

```
src/test/java/com/muta/{modulo}/usecase/
└── {Accion}UseCaseTest.java
```

---