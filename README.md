# 🚀 Muta Backend Skill

> **Guía de desarrollo y estándares para el equipo de backend**

---

## 📋 ¿Qué es esto?

Este repositorio contiene la **guía oficial de desarrollo** del equipo backend de Muta. Aquí encontrarás todas las normas, patrones y workflows necesarios para mantener consistencia en nuestro código.

### Objetivos

| Objetivo | Descripción |
|----------|-------------|
| **Simplicidad** | Código fácil de entender y mantener |
| **Consistencia** | Estándar sólido que todos sigan |
| **Rapidez** | Estructura lite que no complique el desarrollo |
| **Claridad** | Guía clara para el equipo |

---

## 📁 Estructura del Proyecto

```
muta-skill-backend/
├── principles/              ← Principios y estándares base
│   ├── architecture.md      ← Arquitectura y capas
│   └── coding-standards.md ← Estándares de código
│
├── backend/                ← Configuración de backend
│   ├── springboot-guidelines.md
│   └── package-structure.md
│
├── workflows/              ← Guías de creación (14 archivos)
│   ├── create-new-endpoint.md   ← GUÍA PRINCIPAL
│   ├── create-controller.md
│   ├── create-service.md
│   ├── create-usecase.md
│   ├── create-repository.md
│   ├── create-entity.md
│   ├── create-enum.md
│   ├── create-exception.md
│   ├── create-dto.md
│   ├── create-test-controller.md
│   ├── create-test-service.md
│   ├── create-test-usecase.md
│   └── release-process.md
│
├── api/                   ← Estándares de API
│   └── rest-standards.md
│
└── SKILL.md               ← Este archivo
```

---

## 🏗️ Arquitectura

### Las 9 Capas del Sistema

| # | Capa | Descripción |
|---|------|-------------|
| 1 | Controller | Recibe HTTP, valida, delega al Adapter |
| 2 | Adapter | Traduce y decide Service o UseCase |
| 3 | Service | CRUD básico (extiende BaseService) |
| 4 | UseCase | Lógica compleja (extiende BaseUseCase) |
| 5 | Repository | Acceso a datos |
| 6 | Entity | Modelo de datos JPA |
| 7 | Enum | Catálogos (prefijo E: EEstatus, ETipo...) |
| 8 | Exception | Manejo de errores |
| 9 | Templates | Clases base para extender |

### Flujo de una Request

```
HTTP Request → Controller → Adapter → Service/UseCase → Repository → Entity → BD
                                    ↳ Response ← Adapter ← Service/UseCase ← Entity
```

### Regla Fundamental

> **UNA ENTITY = UN CONTROLLER**

Cada recurso/entidad tiene su propio Controller, Adapter, Service, UseCase y Repository. Aunque sea "sub" de otro recurso, si tiene lógica propia → separado.

---

## 📖 Documentación

### 📚 Principles (Obligatorio leer)

| Documento | Descripción |
|-----------|-------------|
| [architecture.md](principles/architecture.md) | Arquitectura completa, capas, flujo HTTP, reglas |
| [coding-standards.md](principles/coding-standards.md) | Nombrado, anotaciones, formato |

### ⚙️ Backend

| Documento | Descripción |
|-----------|-------------|
| [springboot-guidelines.md](backend/springboot-guidelines.md) | Application, properties, transacciones |
| [package-structure.md](backend/package-structure.md) | Estructura de paquetes Java |

### 🔄 Workflows (Guías de creación)

| Documento | Descripción |
|-----------|-------------|
| **[create-new-endpoint.md](workflows/create-new-endpoint.md)** | 🆕 **GUÍA PRINCIPAL** - Cómo crear un endpoint desde cero |
| [create-controller.md](workflows/create-controller.md) | Crear Controller |
| [create-adapter.md](workflows/create-adapter.md) | Crear Adapter |
| [create-service.md](workflows/create-service.md) | Crear Service |
| [create-usecase.md](workflows/create-usecase.md) | Crear UseCase |
| [create-repository.md](workflows/create-repository.md) | Crear Repository |
| [create-entity.md](workflows/create-entity.md) | Crear Entity |
| [create-enum.md](workflows/create-enum.md) | Crear Enum |
| [create-exception.md](workflows/create-exception.md) | Crear Exception |
| [create-dto.md](workflows/create-dto.md) | Crear DTOs |

### 🧪 Testing

| Documento | Descripción |
|-----------|-------------|
| [create-test-service.md](workflows/create-test-service.md) | Testing Service |
| [create-test-usecase.md](workflows/create-test-usecase.md) | Testing UseCase |
| [create-test-controller.md](workflows/create-test-controller.md) | Testing Controller |

### 🚀 Procesos

| Documento | Descripción |
|-----------|-------------|
| [release-process.md](workflows/release-process.md) | Proceso de release |

### 🌐 API

| Documento | Descripción |
|-----------|-------------|
| [rest-standards.md](api/rest-standards.md) | Estándares REST |

---

## ⚠️ Reglas No Negociables

| # | Regla |
|---|-------|
| 1 | Controller NO tiene lógica de negocio |
| 2 | SIEMPRE pasar por el Adapter |
| 3 | NUNCA exponer Entity en response |
| 4 | DTOs SIEMPRE como records, NO clases |
| 5 | Inyección por constructor con @RequiredArgsConstructor |
| 6 | Enums con prefijo E (EEstatus, ETipoDocumento) |
| 7 | Tests obligatorios (happy path + 1 caso de error) |

---

## 🆕 ¿Cómo crear un nuevo endpoint?

Este es el workflow más importante. Sigue estos pasos:

### 1. ANÁLISIS (Obligatorio)
Responde estas preguntas:
- ¿Cuál es el nombre del recurso?
- ¿Qué acción hace este endpoint?
- ¿Cuál es el HTTP method?
- ¿Es CRUD básico o lógica compleja?

### 2. CREACIÓN
Sigue la guía: **[create-new-endpoint.md](workflows/create-new-endpoint.md)**

### 3. TESTS
Crea tests unitarios para:
- Service
- UseCase (si aplica)
- Controller

---

## 👥 Equipo

| Rol | Responsabilidad |
|-----|----------------|
| Backend Devs | Desarrollo de features |
| Tech Lead | Revisión de código, arquitectura |
| QA | Testing y certificación |

---

## 📞 Contacto

Para dudas sobre estándares, consultar con el Tech Lead.

---

## 📝 Changelog

| Versión | Fecha | Descripción |
|---------|-------|-------------|
| 1.0.0 | 2026-04-24 | Versión inicial |

---

*Documento generado automáticamente*
*Última actualización: 2026-04-24*