# 🚀 Proceso de Release

---

## 1. Cuándo hacer un Release

| Trigger | Ejemplo |
|---------|--------|
| Nuevas funcionalidades | Se completaron features |
| Fixes | Corrección de bugs |
| Hotfix |修复 crítico en producción |

---

## 2. Checklist previa

- [ ] Todas las features en el environment correcto
- [ ] Code Review aprobado
- [ ] Tests passando (si hay)
- [ ] QA certificado en staging
- [ ] Backup de BD realizado
- [ ] Comunicación al equipo

---

## 3. Fase 1: Preparación

### Checklist de preparación

| # | Tarea | Responsable |
|---|-------|------------|
| 1 | Verificar que las PRs estén mergeadas | Dev |
| 2 | Verificartests en CI/CD | Dev |
| 3 | Certificate por QA | QA |
| 4 | Backup de BD | DBA |
| 5 | Notificar al equipo | Release Manager |

### Comunicación previa

```
📦 **Release programado**

Hora:
Ambiente:
Features:
Risks:

Equipo disponible para rollback:
```

---

## 4. Fase 2: Ejecución

### Steps del release

```bash
# 1. Checkout a main/master
git checkout main
git pull origin main

# 2. Mergear branch de release
git merge release/{version}

# 3. Tag (opcional)
git tag -a v1.0.0 -m "Release v1.0.0"

# 4. Build
./mvnw clean package -Dprofile=prod

# 5. Deploy (docker, k8s, etc.)
kubectl apply -f deployment.yaml
# o
docker-compose up -d
```

###Order de ejecución

| # | Step | Descripción |
|---|------|-----------|
| 1 | Build | Compilar el proyecto |
| 2 | Tests | Correr tests (si hay) |
| 3 | Migrate BD | Aplicar migraciones |
| 4 | Deploy | Subir a producción |
| 5 | Verify | Verificar que funciona |

---

## 5. Fase 3: Post-Release

### Checklist post-release

| # | Tarea | Responsable |
|---|-------|------------|
| 1 | Verificar/health check | DevOps |
| 2 | Tests de smoke | QA |
| 3 | Monitoreo | DevOps |
| 4 | Comunicación | Release Manager |

###/health check

```bash
# Verificar que la app responde
curl https://api.muta.com/health

# Verificar logs
kubectl logs -f deployment/{app}

# Verificar métricas
curl https://api.muta.com/actuator/metrics
```

### Comunicación post-release

```
✅ **Release completado**

Versión:
Hora inicio:
Hora fin:
Features desplegadas:
Incidentes:
```

---

## 6. Rollback (si aplica)

### Cuándo hacer rollback

| # | Condición |
|---|----------|
| 1 | Errores críticos en producción |
| 2 | Feature no funciona |
| 3 | Degradación de performa |

### Pasos de rollback

```bash
# Revertir a tag anterior
git revert HEAD
git push origin main

# O desplegar tag anterior
git checkout v1.0.0
./deploy.sh v1.0.0
```

---

## 7. Template de Release Doc

```markdown
# Release: {versión}

## metadata
| Campo | Valor |
|-------|-------|
| Fecha | {fecha} |
| Hora inicio | {hora} |
| Hora fin | {hora} |
| Responsable | {nombre} |

## Features
- [ ] Feature 1
- [ ] Feature 2

## Pre-check
- [ ] Tests passando
- [ ] QA aprobado
- [ ] Backup realizado

## Post-check
- [ ] Health OK
- [ ] Smoke test OK
- [ ] Monitoreo OK
```

---

## 8. Reglas obligatorias

| # | Regla |
|---|-------|
| 1 | **SIEMPRE** hacer backup antes |
| 2 | Communicarr al equipo antes |
| 3 | Verificar despuésde deploy |
| 4 | Tener plan de rollback listo |
| 5 | Comunicación post-release |

---