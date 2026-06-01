---
name: karvey-impl
description: Execute implementation tasks sequentially with ClickUp time tracking or PLAN.md updates. Read → execute → test → validate cycle per task. Triggers include "karvey impl", "implementar", "ejecutar tasks", "desarrollar".
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent
argument-hint: <change-id> [F{n}.T{n}] [--from F{n}.T{n}]
---

# Karvey Impl

## Propósito

Ejecutar las tasks de implementación en orden BD→Backend→Frontend. Ciclo por task: read → execute → test → validate. Actualizar ClickUp o PLAN.md en tiempo real.

## Pasos de ejecución

### Paso 1 — Cargar contexto

Leer:
- `spec/changes/{change-id}/spec.json`
- `spec/changes/{change-id}/tasks.md`
- `spec/changes/{change-id}/architecture.md`
- `spec/changes/{change-id}/requirements.md`

Verificar `approvals.tasks.approved = true`. Si no, detener.

Si se especifica una task específica (`F{n}.T{n}`): ejecutar solo esa.
Si se especifica `--from F{n}.T{n}`: comenzar desde esa task y continuar secuencialmente.
Si no se especifica: comenzar desde la primera task pendiente.

### Paso 2 — Seleccionar task a ejecutar

Identificar la próxima task pendiente respetando dependencias:
- No ejecutar [Backend] hasta que su [BD] dependiente esté completada
- No ejecutar [Frontend] hasta que su [Backend] dependiente esté completada
- Tareas marcadas `(P)` pueden ejecutarse en paralelo con subagentes

### Paso 3 — Iniciar task en gestión

**Si ClickUp:**
```
clickup_update_task(task_id, status="in progress")
clickup_start_time_tracking(task_id)
```

**Si Markdown:**
Editar `PLAN.md`, cambiar `⬜ pendiente` → `🔄 en progreso` para la task.

### Paso 4 — Ejecutar la task

Leer la descripción completa de la task y sus criterios de aceptación.
Ejecutar el trabajo técnico: crear/modificar archivos según el File Structure Plan.

**Reglas de ejecución:**
- Respetar el boundary de la task — no tocar código fuera de su scope
- Seguir los patrones del stack existente (leer archivos similares del codebase antes de escribir)
- No hardcodear secretos ni credenciales
- Validar contexto de usuario/autenticación en cada endpoint y acceso a datos, según el patrón del proyecto
- 1 commit por task con mensaje descriptivo, siguiendo las convenciones de git del proyecto

**Bump de versión (si el proyecto lo gestiona):**
Detectar el mecanismo de versionamiento leyendo `architecture.md` o explorando el proyecto:
- `package.json` → actualizar campo `version`
- `pyproject.toml` / `setup.py` → actualizar `version`
- Archivo `VERSION` → actualizar valor
- `git tags` → crear tag al final del Epic
- Si hay `CHANGELOG.md` o equivalente → agregar entrada con la versión, fecha y descripción
- Si el proyecto no tiene versionamiento → omitir este paso

### Paso 5 — Test inmediato

Ejecutar verificación antes de marcar como completada:

**Para tasks [BD]:**
- Ejecutar la query, SP, migración o función con datos de prueba
- Confirmar que retorna la estructura esperada y no produce errores

**Para tasks [Backend]:**
- Ejecutar el endpoint/función localmente o en dev si disponible
- Verificar respuesta correcta para input válido, error de auth sin credenciales, y error de validación con input inválido (según el protocolo del proyecto: HTTP status codes, errores GraphQL, etc.)

**Para tasks [Frontend]:**
- Verificar que el componente/vista renderiza sin errores
- Verificar estados: loading, error, vacío, con datos

Si el test falla: corregir en la misma task antes de avanzar.

### Paso 6 — Completar task en gestión

**Si ClickUp:**
```
clickup_stop_time_tracking()
clickup_create_task_comment(task_id,
  "✅ COMPLETADA\n\nRealizado:\n- {qué se hizo}\n\nArchivos:\n- {lista}\n\nResultado: OK")
clickup_update_task(task_id, status="listo! para pap")
```

Actualizar tiempo real via REST API:
```bash
curl -s -X PUT "https://api.clickup.com/api/v2/task/{TASK_ID}" \
  -H "Authorization: $API_KEY" -H "Content-Type: application/json" \
  -d '{"time_estimate": {tiempo_real_ms}}'
```

**Cascada de estados ClickUp:**
Cuando TODAS las tasks de un Feature están en "listo! para pap":
```
clickup_create_task_comment(feature_id, "Todas las tasks de {capa} completadas.")
# Solo cambiar Feature si TODAS las capas terminaron
clickup_update_task(feature_id, status="listo! para pap")  # si aplica
```

**Si Markdown:**
Editar `PLAN.md`:
- Cambiar `🔄 en progreso` → `✅ completado`
- Actualizar tiempo real en la tabla de estado
- Actualizar fecha en historial

### Paso 7 — Continuar con siguiente task

Repetir pasos 2–6 hasta que todas las tasks estén completadas.

Si hay tasks `(P)`: despachar subagentes paralelos para ejecutarlas simultáneamente.

### Paso 8 — Completar Epic

Cuando TODOS los Features estén en "listo! para pap":

**Si ClickUp:**
```
clickup_create_task_comment(epic_id, "Todos los Features completados. Epic listo para QA.")
clickup_update_task(epic_id, status="listo! para pap")
```

**Si Markdown:**
Actualizar `PLAN.md`: estado general `✅ Implementación completa`.

### Paso 9 — Output final

```
✅ Implementación completa

Tasks ejecutadas: {N}/{N}
Tiempo total estimado: {suma} | Tiempo real: {suma}

Archivos creados/modificados:
  BD: {lista}
  Backend: {lista}
  Frontend: {lista}

Commits realizados: {N}
Versión: {nueva versión}

Siguiente paso:
/karvey-test {change-id}
```

## Manejo de bloqueos

Si una task no puede completarse:

**Si ClickUp:**
```
clickup_stop_time_tracking()
clickup_create_task_comment(task_id, "BLOQUEADO: {descripción del bloqueo}\n\nNecesito: {qué se necesita para desbloquear}")
clickup_update_task(task_id, status="blocked")
```

**Si Markdown:**
Marcar `⛔ bloqueado` + nota en PLAN.md.

Reportar al usuario con el bloqueo específico y esperar desbloqueo.
