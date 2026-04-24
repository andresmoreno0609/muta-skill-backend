# 🏗️ Arquitectura - Resumen y Siguientes Pasos

---

## ✅ Lo que tenemos: principles/architecture.md

### Secciones completadas

| # | Sección | Contenido |
|---|---------|-----------|
| 1.1 | Visión General | Objetivos, tipo de arquitectura, beneficios |
| 1.2 | Descripción de Cada Capa | 9 capas con responsabilidades |
| 1.3 | Flujo de una Request | Diagrama visual del flujo HTTP |
| 1.4 | Service vs UseCase | Cuándo usar cada uno |
| 1.5 | Reglas No Negociables | 10 reglas obligatoria |
| 1.6 | Recomendaciones | 5 recomendaciones adicionales |
| 1.7 | Capa de Enums | Cómo crear catálogos |
| 1.8 | Capa de Excepciones | Sistema de errores |

---

## 📋 Las 9 Capas Definidas

| # | Capa |Descripción | Template/Base |
|---|------|-------------|--------------|
| 1 | Controller | Recibe HTTP, valida, delega | - |
| 2 | Adapter | Traduce y decide Service o UseCase | - |
| 3 | Service | CRUD básico | BaseService |
| 4 | UseCase | Lógica compleja | BaseUseCase |
| 5 | Repository | Acceso a BD | JpaRepository |
| 6 | Entity | Modelo de datos | - |
| 7 | Enum | Catálogos/valores fijos | CodeEnum |
| 8 | Exception | Manejo de errores | MutaException |
| 9 | Templates | Clases base | BaseService, BaseUseCase |

---

## ⏳ Lo que falta definir (próximos pasos)

| # | Archivo | Contenido |
|---|--------|-----------|
| 1 | principles/coding-standards.md | Estándares de código (nombrado, anotaciones, imports) |
| 2 | backend/package-structure.md | Estructura de paquetes detallada |
| 3 | api/rest-standards.md | Estándares REST (verbos, status codes) |
| 4 | workflows/create-endpoint.md | Flujo paso a paso para nuevos endpoints |
| 5 | workflows/create-entity.md | Flujo para nuevas entidades |
| 6 | database/migrations.md | Guías de migraciones |

---

## 🎯 Prioridad sugerida

| Orden | Archivo | Por qué |
|-------|--------|--------|
| 1 | coding-standards.md | Es la base para escribir código igual |
| 2 | workflows/create-endpoint.md | El flujo que más van a usar |
| 3 | rest-standards.md | Para mantener API consistente |
| 4 | create-entity.md | Cuando creen nuevas entidades |
| 5 | package-structure.md | Referencia de estructura |
| 6 | migrations.md | Para manejo de BD |

---

## 📁 Estructura actual

```
muta-skill-backend/
├── SKILL.md
└── principles/
    └── architecture.md   ← ✅ COMPLETO
```

---

## Acción siguiente

**¿Confirmás este resumen y arrancamos con el siguiente archivo?**

Recomendación: **Arrancar por coding-standards.md** para tener los estándares de código desde el inicio.

¿Qué preferís?