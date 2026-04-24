# 🌐 Estándares REST API

---

## 1. URL Conventions

### Estructura base

```
/api/v1/{recurso}
```

| Parte | Descripción |
|-------|-----------|
| `/api` | Prefijo fijo |
| `/v1` | Versión de la API |
| `{recurso}` | Nombre del recurso (plural, lowercase) |

### Ejemplos

| Recurso | URL |
|--------|-----|
| Usuario | `/api/v1/usuarios` |
| Orden | `/api/v1/ordenes` |
| Producto | `/api/v1/productos` |

### Reglas

| # | Regla |
|---|-------|
| 1 | Siempre plural |
| 2 | Todo lowercase |
| 3 | Separar palabras con guiones: `ordenes-entregas` |
| 4 | NUNCA usar espacios o caracteres especiales |

---

## 2. HTTP Methods

| Método | Descripción | Cuándo usar |
|--------|-----------|------------|
| **GET** | Leer | Obtener recursos |
| **POST** | Crear | Nuevos recursos |
| **PUT** | Reemplazar | Actualización completa |
| **PATCH** | Actualizar parcialmente | Actualización parcial |
| **DELETE** | Eliminar | Borrar recurso |

### Mapping CRUD

| Acción | Método | URL |
|--------|--------|-----|
| Listar todos | GET | `/api/v1/recurso` |
| Buscar por ID | GET | `/api/v1/recurso/{id}` |
| Crear | POST | `/api/v1/recurso` |
| Actualizar completo | PUT | `/api/v1/recurso/{id}` |
| Actualizar parcial | PATCH | `/api/v1/recurso/{id}` |
| Eliminar | DELETE | `/api/v1/recurso/{id}` |

---

## 3. HTTP Status Codes

### Códigos de éxito

| Status | Método | Descripción |
|--------|--------|-------------|
| **200** | GET, PUT, PATCH | OK |
| **201** | POST | Created (creado exitosamente) |
| **204** | DELETE | No content (sin respuesta) |

### Códigos de error del cliente

| Status | Descripción | Cuándo |
|--------|-----------|--------|
| **400** | Bad Request | Request inválido, validación falla |
| **401** | Unauthorized | No autenticado |
| **403** | Forbidden | No tiene permiso |
| **404** | Not Found | Recurso no existe |
| **409** | Conflict | Conflicto (ej: duplicado) |
| **422** | Unprocessable Entity | Validation business |

### Códigos de error del servidor

| Status | Descripción | Cuándo |
|--------|-----------|--------|
| **500** | Internal Server Error | Error inesperado |

### Tabla de referencia

| GET | POST | PUT | PATCH | DELETE |
|-----|------|-----|-------|--------|
| 200 | 201 | 200 | 200 | 204 |
| 404 | 400 | 404 | 404 | 404 |
| 500 | 400 | 400 | 400 | 500 |

---

## 4. Response Format

### Response exitoso

```json
{
  "data": {
    "id": "uuid",
    "nombre": "Juan",
    "email": "juan@email.com",
    "createdAt": "2026-04-24T10:00:00Z"
  }
}
```

### Response con lista

```json
{
  "data": [
    { "id": "1", "nombre": "Juan" },
    { "id": "2", "nombre": "Maria" }
  ],
  "meta": {
    "page": 0,
    "size": 10,
    "totalElements": 2,
    "totalPages": 1
  }
}
```

### Response sin contenido (204)

```
(empty body)
```

---

## 5. Error Format

### Estructura de error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email es obligatorio",
    "details": [
      { "field": "email", "message": "Email es obligatorio" }
    ]
  }
}
```

### Códigos de error

| Code | Descripción |
|------|-----------|
| `VALIDATION_ERROR` | Error de validación |
| `NOT_FOUND` | Recurso no encontrado |
| `UNAUTHORIZED` | No autorizado |
| `FORBIDDEN` | Sin permiso |
| `CONFLICT` | Conflicto (duplicado) |
| `INTERNAL_ERROR` | Error del servidor |

---

## 6. Versioning

### Estrategia

```
/api/v1/{recurso}
```

| Versión | URL |
|--------|-----|
| v1 | `/api/v1/` |
| v2 | `/api/v2/` |

### Reglas

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** incluir versión |
| 2 | Usar `v1` inicialmente |
| 3 | Incrementar solo si hay breaking changes |

---

## 7. Paginación

### Request

```
GET /api/v1/usuarios?page=0&size=10&sortBy=createdAt&direction=DESC
```

| Parámetro | Default | Descripción |
|-----------|---------|-----------|
| `page` | 0 | Página (0-indexed) |
| `size` | 10 | Elementos por página |
| `sortBy` | `createdAt` | Campo de ordenado |
| `direction` | `DESC` | ASC o DESC |

### Response

```json
{
  "data": [...],
  "meta": {
    "page": 0,
    "size": 10,
    "totalElements": 50,
    "totalPages": 5,
    "first": true,
    "last": false,
    "empty": false
  }
}
```

---

## 8. Filtering/Sorting

### Filtros

```
GET /api/v1/usuarios?estado=ACTIVO&emailContains=@email.com
```

| Filter | Ejemplo | Descripción |
|--------|---------|-------------|
| `={valor}` | `estado=ACTIVO` | Igual |
| `Contains={valor}` | `nombreContains=Juan` | Contiene |
| `StartsWith={valor}` | `nombreStartsWith=Ju` | Empieza con |
| `En={valor1,valor2}` | `estadoEn=ACTIVA,INACTIVA` | Uno de |
| `Gt={valor}` | `edadGt=18` | Mayor que |
| `Lt={valor}` | `edadLt=65` | Menor que |

### Sorting

```
GET /api/v1/usuarios?sortBy=nombre&direction=ASC
```

---

## 9. Ejemplos completos

### GET - Listar con filtros y paginación

```
GET /api/v1/usuarios?page=0&size=10&sortBy=createdAt&direction=DESC&estado=ACTIVO
```

### GET - Buscar por ID

```
GET /api/v1/usuarios/550e8400-e29b-41d4-a716-446655440000
```

### POST - Crear

```
POST /api/v1/usuarios
Content-Type: application/json

{
  "nombre": "Juan",
  "email": "juan@email.com"
}
```

### PUT - Actualizar

```
PUT /api/v1/usuarios/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "nombre": "Juan Actualizado",
  "email": "juan2@email.com"
}
```

### PATCH - Actualizar parcialmente

```
PATCH /api/v1/usuarios/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "nombre": "Juan"
}
```

### DELETE - Eliminar

```
DELETE /api/v1/usuarios/550e8400-e29b-41d4-a716-446655440000
```

---

## 10. Resumen rápido

| # | Regla |
|---|-------|
| 1 | URL: `/api/v1/{recurso plural}` |
| 2 | GET = leer, POST = crear, PUT = reemplazar, PATCH = parcial, DELETE = borrar |
| 3 | 200/201/204 para éxito |
| 4 | 400/401/403/404/409 para errores de cliente |
| 5 | Versionar siempre con `/v1/` |
| 6 | Usar paginación en listas |
| 7 | `data` wrapper en respuestas |

---