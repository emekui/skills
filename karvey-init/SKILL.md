---
name: karvey-init
description: Initialize a new Karvey spec. Creates the directory structure, spec.json, and registers the Epic in ClickUp (or PLAN.md if not using ClickUp). Use after karvey-grill or when starting a new feature. Triggers include "karvey init", "iniciar spec", "nueva feature", "nuevo cambio".
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
argument-hint: <change-id> [--capability <nombre>]
---

# Karvey Init

## Propósito

Inicializar la estructura de una nueva especificación y registrar el Epic en el sistema de gestión (ClickUp o Markdown).

## Pasos de ejecución

### Paso 1 — Verificar contexto pre-spec

Si existe un resumen de `karvey-grill` en la conversación, usarlo para pre-poblar los campos.
Si no, pedir al usuario: "Describí brevemente el problema que resuelve este cambio."

### Paso 2 — Generar change-id

Si `$ARGUMENTS` incluye el change-id, usarlo. Si no, generarlo desde la descripción:
- Formato: `{add|fix|update|remove}-{nombre-descriptivo-url-safe}`
- Ejemplos: `add-call-transfer`, `fix-webhook-retry`, `update-tenant-config`
- Verificar que no exista en `spec/changes/`: `find spec/changes -maxdepth 1 -type d -name "{change-id}"`
- Si existe conflicto, agregar sufijo numérico: `add-call-transfer-2`

### Paso 3 — Preguntar sistema de gestión

```
¿Este proyecto usa ClickUp para gestión?
```

**Si SÍ (ClickUp):**
- Preguntar: ¿A qué proyecto/backlog pertenece? (obtener el `backlog_list_id` de ClickUp)
- Leer regla: `rules/clickup-protocol.md`

**Si NO (Markdown):**
- Se creará `PLAN.md` en el directorio del cambio
- No se requiere ninguna configuración adicional

### Paso 4 — Recopilar metadata

Preguntar (o inferir del contexto pre-spec):

1. **Capability**: dominio funcional al que pertenece (ej: `call-management`, `authentication`, `notifications`). Si no existe en `spec/specs/`, se creará.
2. **Security Tier**: 1-4. Leer `rules/security-tiers.md` para orientar al usuario.
3. **Capas involucradas**: BD / Backend / Frontend / Infra (puede ser múltiple)
4. **Descripción breve**: 1-2 líneas del problema que resuelve

### Paso 5 — Crear estructura de directorios

```bash
mkdir -p spec/changes/{change-id}/specs/{capability}
mkdir -p spec/specs/{capability}  # si no existe
```

Si `spec/specs/{capability}/spec.md` no existe, crearlo:
```markdown
# Spec: {Capability}

<!-- Living spec del capability {capability}. Se actualiza al archivar cada cambio. -->
```

### Paso 6 — Crear spec.json

Escribir `spec/changes/{change-id}/spec.json` con:
```json
{
  "change_id": "{change-id}",
  "capability": "{capability}",
  "description": "{descripción breve}",
  "layers": ["{BD|Backend|Frontend|Infra}"],
  "created_at": "{ISO timestamp}",
  "updated_at": "{ISO timestamp}",
  "language": "es",
  "management": "{clickup|markdown}",
  "security_tier": {1-4},
  "phase": "init",
  "clickup": {
    "epic_id": "",
    "feature_ids": [],
    "backlog_list_id": "{list_id o vacío}",
    "client_tag": "{tag o vacío}"
  },
  "approvals": {
    "requirements": { "generated": false, "approved": false },
    "mockup": { "generated": false, "approved": false },
    "design_graphic": { "generated": false, "approved": false },
    "architecture": { "generated": false, "approved": false },
    "tasks": { "generated": false, "approved": false }
  }
}
```

### Paso 7 — Crear proposal.md

Escribir `spec/changes/{change-id}/proposal.md`:
```markdown
# Propuesta: {change-id}

## Por qué
{problema que resuelve, quién lo tiene, impacto actual}

## Qué cambia
{descripción de lo que se modifica o agrega}

## Impacto
- Capas afectadas: {lista}
- Sistemas relacionados: {lista}
- Breaking changes: {Sí/No — descripción}

## Restricciones
- Security Tier: {N} — {justificación}
- Dependencias previas: {lista}

## Criterios de éxito
{cómo verificar que funciona}
```

### Paso 8A — Crear Epic en ClickUp (si management=clickup)

Leer credenciales de `.connections.json` (ver `rules/clickup-protocol.md`). Si no existe, crearlo y agregarlo al `.gitignore` antes de continuar.
Determinar el próximo número de Epic buscando en ClickUp:
```
clickup_search
  keywords: "E{1..99}"
  filters.location.categories: ["{backlog_list_id}"]
```

Crear Epic:
```
clickup_create_task
  name: "E{n} {Nombre del Epic}"
  list_id: "{backlog_list_id}"
  task_type: "Epic"
  description: (ver formato en reglas)
  tags: ["{client_tag}"]
```

Actualizar `spec.json` con `clickup.epic_id`.

Formato de descripción del Epic:
```
E{n}: {Nombre del Epic}

Definición:
{descripción de 2-3 párrafos: qué problema resuelve, para quién, por qué es importante}

Valor Estratégico:
{impacto en el negocio}

Security Tier: {N} — {justificación}

Decisiones de Diseño:
(pendiente — se completa en karvey-architecture)

Features:
(pendiente — se completa en karvey-requirements)

Realizado con Método Karvey
```

### Paso 8B — Crear PLAN.md (si management=markdown)

Escribir `spec/changes/{change-id}/PLAN.md`:
```markdown
# Plan: {change-id}

**Capability:** {capability} | **Security Tier:** {N} | **Capas:** {lista}
**Creado:** {fecha} | **Estado:** 🟡 En progreso

---

## Epic: {Nombre}

### Descripción
{problema, quién, impacto}

### Valor estratégico
{impacto en el negocio}

### Decisiones de diseño
| Tema | Decisión |
|------|----------|
| (pendiente — karvey-architecture) | |

---

## Features
(pendiente — karvey-requirements)

---

## Tasks
(pendiente — karvey-tasks)

---

## Estado de tareas
| Task | Estado | Tiempo estimado | Tiempo real |
|------|--------|----------------|-------------|
| (pendiente) | | | |

---

## Historial
| Fecha | Fase | Acción |
|-------|------|--------|
| {fecha} | init | Spec inicializada |
```

### Paso 8C — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar los documentos creados.
Si `spec/graphify-out/` no existe (primera vez en el proyecto), invocar `/graphify spec/` sin `--update`.

### Paso 9 — Output final

```
✅ Spec inicializada: spec/changes/{change-id}/

Archivos creados:
  - spec/changes/{change-id}/spec.json
  - spec/changes/{change-id}/proposal.md
  - spec/changes/{change-id}/PLAN.md (si markdown)
  - spec/specs/{capability}/spec.md (si nuevo capability)

Gestión: {ClickUp Epic E{n} creado | PLAN.md creado}
Security Tier: {N}

Siguiente paso:
/karvey-requirements {change-id}
```

## Safety

- Si `spec/changes/{change-id}` ya existe con `spec.json`, preguntar antes de sobrescribir
- Si ClickUp falla, ofrecer continuar en modo markdown como fallback
- Validar que el `change-id` sea URL-safe (solo letras minúsculas, números y guiones)
