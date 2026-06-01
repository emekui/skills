---
name: karvey-context
description: Dashboard of the current project spec state: capabilities, active changes, archived changes, sprint status. Use at any point in the Karvey pipeline. Triggers include "karvey context", "estado del proyecto", "qué specs hay", "cambios activos".
allowed-tools: Read, Bash, Glob, Grep
argument-hint: [--capability <nombre>] [--change <change-id>]
---

# Karvey Context

## Propósito

Vista rápida del estado completo del proyecto: capabilities documentados, cambios en curso, archivados, sprint actual y cobertura de specs.

## Pasos de ejecución

### Si se especifica --capability

Mostrar detalle del capability:
```bash
cat spec/specs/{capability}/spec.md
grep -c "### Requirement:" spec/specs/{capability}/spec.md
grep -c "#### Scenario:" spec/specs/{capability}/spec.md
```

### Si se especifica --change

Mostrar detalle del cambio:
```bash
cat spec/changes/{change-id}/spec.json
cat spec/changes/{change-id}/proposal.md
# Estado de aprobaciones
```

### Si no se especifica nada — Dashboard completo

```bash
# Capabilities
find spec/specs -mindepth 1 -maxdepth 1 -type d 2>/dev/null

# Requirements por capability
for cap in spec/specs/*/; do
  name=$(basename "$cap")
  reqs=$(grep -c "### Requirement:" "$cap/spec.md" 2>/dev/null || echo "0")
  echo "$name: $reqs requirements"
done

# Cambios activos
find spec/changes -maxdepth 1 -type d -not -path "spec/changes" -not -path "*/archive" 2>/dev/null

# Cambios archivados
ls spec/changes/archive/ 2>/dev/null | wc -l
```

Mostrar al usuario:

```
📊 Karvey Context — {fecha}

CAPABILITIES ({N} total)
━━━━━━━━━━━━━━━━━━━━━━━━
{capability-1}: {N} requirements, {N} scenarios
{capability-2}: {N} requirements, {N} scenarios

CAMBIOS ACTIVOS ({N})
━━━━━━━━━━━━━━━━━━━━
{change-id-1}
  Fase: {phase}
  Capability: {capability}
  Security Tier: {N}
  Gestión: {ClickUp Epic E{n} | Markdown}
  Aprobaciones: requirements={✅|⬜} mockup={✅|⬜} design={✅|⬜} arch={✅|⬜} tasks={✅|⬜}

{change-id-2}
  ...

CAMBIOS ARCHIVADOS: {N}
Último archivado: {fecha-change-id}

SPRINT ACTIVO
━━━━━━━━━━━━
{verificar con clickup_get_workspace_hierarchy o indicar "no aplica (markdown)"}
```

### Para el sprint activo (si ClickUp disponible)

```
clickup_get_workspace_hierarchy
  max_depth: 2
```

Buscar el folder "Dev Sprints Metodo Karvey" y el sprint activo.

Mostrar:
```
Sprint activo: Sprint {N} (hasta {fecha})
Tasks del sprint: {total} | En progreso: {N} | Listo para PAP: {N} | Bloqueado: {N}
```
