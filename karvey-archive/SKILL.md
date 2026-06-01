---
name: karvey-archive
description: Archive a completed change: merge spec-deltas into living specs, move to archive, close Epic in ClickUp. Triggers include "karvey archive", "archivar", "cerrar epic", "merge specs".
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: <change-id>
---

# Karvey Archive

## Propósito

Completar el ciclo de vida del cambio: fusionar spec-deltas en las living specs, archivar el directorio del cambio, y cerrar el Epic en ClickUp o marcar completo en PLAN.md.

## Pasos de ejecución

### Paso 1 — Verificar completitud

Leer `spec/changes/{change-id}/spec.json`.

Verificar:
- [ ] `approvals.tasks.approved = true`
- [ ] Tests ejecutados (existe `docs/test_evidence.md` con entradas del change-id)
- [ ] QA review completado (existe `REVISION_PR_*_{fecha}.md`)
- [ ] No hay hallazgos críticos o altos pendientes

Si hay bloqueantes: reportar y detener.

Preguntar al usuario: "¿El cambio fue desplegado a producción? (S/N)"
Si sí: crear archivo `spec/changes/{change-id}/IMPLEMENTED`.
Si no: advertir que se archivará sin el marcador de producción.

### Paso 2 — Leer spec-deltas

Leer todos los archivos `spec/changes/{change-id}/specs/**/*.md`.

Para cada spec-delta, identificar las operaciones:
- `## ADDED Requirements` → append en living spec
- `## MODIFIED Requirements` → reemplazar bloque en living spec
- `## REMOVED Requirements` → eliminar bloque + dejar comentario deprecación

### Paso 3 — Mergear deltas en living specs

Para cada capability afectado, editar `spec/specs/{capability}/spec.md`:

**ADDED:** Agregar al final del archivo:
```bash
cat >> spec/specs/{capability}/spec.md << 'EOF'

{bloque del nuevo requirement completo}
EOF
```

**MODIFIED:** Localizar el requirement por nombre y reemplazar el bloque completo.
Búsqueda: `grep -n "### Requirement: {nombre}" spec/specs/{capability}/spec.md`
Reemplazar desde esa línea hasta el próximo `### Requirement:` o fin de archivo.

**REMOVED:** Localizar el bloque y eliminarlo, dejando comentario:
```markdown
<!-- Eliminado {fecha}: {razón del spec-delta} -->
```

### Paso 4 — Commit del merge de specs

```bash
git add spec/specs/
git commit -m "spec: merge deltas from {change-id}

- {capability}: ADDED {N} requirements, MODIFIED {N}, REMOVED {N}"
```

### Paso 5 — Archivar el directorio del cambio

```bash
TIMESTAMP=$(date +%Y-%m-%d)
mkdir -p spec/changes/archive
mv spec/changes/{change-id} spec/changes/archive/${TIMESTAMP}-{change-id}
```

Verificar:
```bash
ls -la spec/changes/archive/${TIMESTAMP}-{change-id}
# change-id ya no debe existir en spec/changes/
```

```bash
git add spec/changes/
git commit -m "chore: archive {change-id}

Spec deltas merged into living specs. Change archived."
```

### Paso 6A — Cerrar Epic en ClickUp (si management=clickup)

```
clickup_create_task_comment(epic_id,
  "✅ Epic completado y archivado.\n\nSpec deltas mergeados en: spec/specs/{capability}/spec.md\nArchivado en: spec/changes/archive/{fecha}-{change-id}\n\nRealizado con Método Karvey")
clickup_update_task(epic_id, status="complete")
```

### Paso 6B — Cerrar PLAN.md (si management=markdown)

Actualizar `spec/changes/archive/{fecha}-{change-id}/PLAN.md`:
- Estado general: `✅ Completado y archivado`
- Agregar entrada en historial: `| {fecha} | archive | Spec mergeada y archivada |`

### Paso 7 — Validar living specs

```bash
grep -n "### Requirement:" spec/specs/{capability}/spec.md
# Verificar que los requirements ADDED aparecen
# Verificar que los REMOVED ya no están
```

### Paso 7B — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar el merge de spec-deltas y el archivado.
El `--update` también elimina del grafo los nodos de documentos que fueron borrados (REMOVED requirements).

### Paso 8 — Output final

```
✅ Cambio archivado: {change-id}

Spec deltas mergeados:
  - spec/specs/{capability}/spec.md
    - ADDED: {N} requirements
    - MODIFIED: {N} requirements
    - REMOVED: {N} requirements

Archivado en: spec/changes/archive/{fecha}-{change-id}
IMPLEMENTED: {sí / no — sin marcar}

Gestión: {Epic E{n} cerrado en ClickUp | PLAN.md marcado completo}

Commits:
  - "spec: merge deltas from {change-id}"
  - "chore: archive {change-id}"

🏁 Ciclo completo del Método Karvey finalizado para {change-id}
```
