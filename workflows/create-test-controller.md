# 🧪 Testing Controller

---

## 1. Qué testear en Controller

| Método HTTP | HTTP Status | Qué verificar |
|------------|------------|--------------|
| GET `/` | 200 | Retorna lista de recursos |
| GET `/{id}` | 200 | Retorna recurso encontrado |
| GET `/{id}` | 404 | Recurso no encontrado |
| POST | 201 | Recurso creado |
| PUT `/{id}` | 200 | Recurso actualizado |
| DELETE `/{id}` | 204 | Recurso eliminado |

---

## 2. Dependencias

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 3. Template

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

## 4. Reglas

| # | Regla |
|---|-------|
| 1 | Usar `@WebMvcTest` |
| 2 | Mockear el Adapter con `@MockBean` |
| 3 | Testear happy path + errores |
| 4 | Verificar status codes correctos |
| 5 | Verificar estructura JSON |

---