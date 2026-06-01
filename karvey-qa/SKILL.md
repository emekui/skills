---
name: karvey-qa
description: QA code review in 6 dimensions (Security, Errors, Consistency, Impact, Env vars, Versioning). Creates REVISION_PR document, ClickUp tasks or PLAN.md entries. Notifies Google Chat. Triggers include "karvey qa", "code review", "revisión de código", "QA".
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent
argument-hint: <change-id> [--source <branch>] [--target <branch>]
---

# Karvey QA

## Propósito

Revisión de código en 6 dimensiones post-implementación. Genera documento de revisión, crea subtareas en ClickUp o PLAN.md, y notifica al grupo de Google Chat del proyecto.

## Pasos de ejecución

### Paso 0 — Identificar ramas y stack

Leer `spec/changes/{change-id}/spec.json`.

Si el usuario no especificó ramas, preguntar: "¿Cuáles son las ramas a comparar? (source → target)"
Convención por defecto: `feature/{change-id}` → `dev`

Detectar stack del repo (ver `package.json`, `requirements.txt`, `pyproject.toml`).

Obtener diff:
```bash
git diff {target}...{source} --stat
git diff {target}...{source}
git log {target}...{source} --oneline
```

### Paso 1 — Análisis en 6 dimensiones

Despachar subagentes paralelos para las dimensiones 1–4, ejecutar 5–6 en contexto principal:

**Dimensión 1: Seguridad**
- Credenciales hardcodeadas (tokens, API keys, passwords)
- XSS: `v-html` sin sanitizar, `dangerouslySetInnerHTML`
- Auth solo en frontend sin enforcement en backend
- Datos personales reales en código (RUTs, emails, teléfonos)
- Validaciones de contexto de usuario ausentes en operaciones de datos o endpoints
- SQL dinámico sin sanitizar
- Stack traces expuestos al cliente

**Dimensión 2: Errores de código**
- Acceso a null/undefined sin guarda
- Memory leaks (listeners, intervals, subscriptions sin cleanup)
- Promesas no manejadas
- Firmas de funciones cambiadas sin actualizar callers
- Race conditions en async

**Dimensión 3: Consistencia**
- Typos en naming
- Mezcla de patrones en mismo módulo
- Código duplicado (3+ repeticiones que deberían ser helper)
- Axios directo saltando interceptores del apiService
- Tabs vs spaces

**Dimensión 4: Impacto en módulos existentes**
- Cambios en archivos compartidos (router, store raíz, apiService, componentes globales)
- Interfaces públicas modificadas sin actualizar consumidores
- Cambios de comportamiento implícitos (timeouts, guards, interceptores)

**Dimensión 5: Variables de entorno**
- Variables usadas en código pero no declaradas en Dockerfile/pipeline
- Variables sin fallback en algún ambiente
- Variables en `.env.example` pero no usadas

**Dimensión 6: Versionamiento**
- Archivo de versión del proyecto actualizado (`package.json`, `pyproject.toml`, `VERSION`, etc.)
- `CHANGELOG.md` con entrada para la versión actual
- Consistencia entre version file y CHANGELOG

### Paso 2 — Generar documento de revisión

Nombre del archivo: `REVISION_PR_{numero}_{YYYYMMDD}.md` en la raíz del repo.

Estructura:
```markdown
# Revisión de Código — {change-id}: {source} → {target}

## Información General
- Repositorio: {nombre}
- Stack: {stack}
- Rama source: {source}
- Rama target: {target}
- Fecha: {YYYY-MM-DD}
- Commits incluidos: {N}
- Archivos modificados: {N}

## Resumen Ejecutivo
{párrafo con hallazgos más importantes}

## Hallazgos por Dimensión

### 1. Seguridad
{hallazgos con: #N, archivo, línea ~NNN, severidad, descripción, código problemático, recomendación, instrucciones para IA}

### 2. Errores de código
...

### 3. Consistencia
...

### 4. Impacto en módulos existentes
...

### 5. Variables de entorno
{tabla de cruce + discrepancias}

### 6. Versionamiento
{verificación del archivo de versión del proyecto y CHANGELOG o equivalente}

## Tabla Resumen por Severidad
| Severidad | Cantidad |
|-----------|---------|
| Crítico | N |
| Alto | N |
| Medio | N |
| Bajo | N |

## Checklist antes del merge
- [ ] Todos los hallazgos críticos resueltos
- [ ] Todos los hallazgos altos resueltos
- [ ] Variables de entorno verificadas
- [ ] Tests pasan
- [ ] Build de producción exitoso

## Áreas que requieren testing manual
- {área}: {razón}
```

### Paso 3A — Crear tareas en ClickUp (si management=clickup)

Obtener sprint activo: `clickup_get_list` con nombre "Sprint XX".

Crear tarea padre:
```
clickup_create_task
  name: "QA Review {change-id} ({source} → {target})"
  list_id: "{sprint_list_id}"
  priority: "high"
  tags: ["{client_tag}"]
```

Por cada hallazgo crítico y alto, crear subtarea:
```
clickup_create_task
  name: "#{N} [{SEVERIDAD}] {archivo}: {descripción corta}"
  parent: {tarea_padre_id}
  priority: {urgent|high|normal|low}
  assignees: [{autor según git log del archivo}]
  time_estimate: {5-60 min en ms}
  markdown_description: (ver formato en QA_CODE_REVIEW_STANDARD)
```

Estimación de fixes:
- Fix simple (null check, typo): 5-10min
- Fix medio (agregar validación, cleanup): 15-20min
- Fix complejo (extraer helper, mover a env var): 30min
- Migración masiva: 45-60min

### Paso 3B — Actualizar PLAN.md (si management=markdown)

Agregar sección "QA Review" al final del PLAN.md con la lista de hallazgos y acciones pendientes.

### Paso 3C — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar el `REVISION_PR_{n}_{fecha}.md` generado.
Si `spec/graphify-out/` no existe, invocar `/graphify spec/` sin `--update`.

### Paso 4 — Notificar por Google Chat

Identificar el space del proyecto en la tabla de espacios conocidos.
Enviar resumen al grupo usando el protocolo de Google Chat del CLAUDE.md.

Formato del mensaje (Google Chat):
```
*QA Review — {change-id}*

*{source}* → *{target}*

*Hallazgos:*
- 🔴 Críticos: {N}
- 🟠 Altos: {N}
- 🟡 Medios: {N}
- ⚪ Bajos: {N}

*Áreas de testing manual:*
- {área 1}
- {área 2}

Documento completo: `REVISION_PR_{n}_{fecha}.md`
```

### Paso 5 — Output final

```
✅ QA Review completo

Hallazgos: {N} total ({críticos}, {altos}, {medios}, {bajos})
Documento: REVISION_PR_{n}_{fecha}.md

Gestión: {N subtareas creadas en ClickUp | PLAN.md actualizado}
Google Chat: notificación enviada a {grupo}

Siguiente paso (si hay hallazgos críticos/altos):
  Corregir → re-ejecutar /karvey-impl {change-id} → /karvey-test {change-id} → /karvey-qa {change-id}

Siguiente paso (si sin hallazgos bloqueantes):
  /karvey-archive {change-id}
```
