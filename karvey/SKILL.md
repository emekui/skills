---
name: karvey
description: Orquestador del Método Karvey — pipeline completo de spec-driven development. Muestra el estado del pipeline, guía qué skill ejecutar en cada fase, y sirve como punto de entrada del método. Triggers include "karvey", "método karvey", "pipeline karvey", "qué sigue en karvey", "iniciar proyecto karvey".
allowed-tools: Read, Bash, Glob, Grep
argument-hint: [<change-id>] [--phase <fase>]
---

# Karvey — Orquestador del Método

## Propósito

Punto de entrada al Método Karvey. Muestra el pipeline completo, el estado actual de un cambio específico, y guía al ingeniero hacia el siguiente skill a ejecutar.

## El Método Karvey

Karvey es un método de spec-driven development para proyectos empresariales. Combina:
- **Pre-spec interrogation** (estilo grill-me): descubrir qué se va a construir antes de especificar
- **EARS requirements + living specs** (estilo openspec/kiro): especificaciones formales acumulativas
- **Mockup navegable HTML**: validar UX antes de diseñar
- **Diseño gráfico sistémico** (estilo impeccable): sistema de colores OKLCH, tipografía, espaciado
- **Arquitectura empresarial**: seguridad por capas, Tiers 1–4, agnóstico de stack
- **Tasks 10–30 min con IA**: granularidad ejecutable por agente
- **Gestión ClickUp o Markdown**: bifurcación según el proyecto
- **Testing BD/Backend/Frontend + E2E**: evidencias documentadas
- **QA 6 dimensiones**: revisión de código post-implementación
- **Archive con merge de specs**: cierre del ciclo, living specs actualizadas

## Pipeline completo

```
FASE 0 ─── /karvey-grill          → Pre-spec: preguntas antes de especificar
FASE 1 ─── /karvey-init           → Crear change-id, spec.json, propuesta, Epic ClickUp
FASE 2 ─── /karvey-requirements   → EARS requirements, spec-delta, aprobación
FASE 3 ─── /karvey-mockup         → HTML navegable 3 niveles, iteración hasta aprobación
FASE 4 ─── /karvey-design-graphic → Sistema visual: colores OKLCH, tipografía, layout
FASE 5 ─── /karvey-architecture   → Arquitectura empresarial, Tiers seguridad, file structure
FASE 6 ─── /karvey-tasks          → Tasks 10–30 min, E{n}.F{n}.T{n}, sprint ClickUp
FASE 7 ─── /karvey-impl           → Implementación BD→Backend→Frontend, commits por task
FASE 8 ─── /karvey-test           → Unit tests + E2E, test_evidence.md con PASS/FAIL
FASE 9 ─── /karvey-qa             → QA 6D, REVISION_PR, Google Chat notification
FASE 10 ── /karvey-archive        → Merge spec-deltas en living specs, cerrar Epic
```

Vista de apoyo:
```
/karvey-context [--capability X] [--change Y]   → Dashboard del proyecto
```

## Ejecución según argumento

### Sin argumentos — Mostrar el pipeline y contexto del proyecto

Mostrar el pipeline de arriba, luego ejecutar los mismos pasos que `/karvey-context` para mostrar el estado general del proyecto (capabilities, cambios activos, archivados, sprint).

### Con `<change-id>` — Mostrar fase actual del cambio

```bash
cat spec/changes/{change-id}/spec.json 2>/dev/null || echo "No encontrado"
```

Leer `spec.json` y determinar la fase actual según `phase` y `approvals`:

| `phase` | `approvals` | Siguiente skill |
|---------|-------------|-----------------|
| `init` | requirements.generated=false | `/karvey-requirements {change-id}` |
| `requirements` | requirements.approved=false | Aprobar requirements con el usuario |
| `requirements` | mockup.generated=false | `/karvey-mockup {change-id}` |
| `mockup` | mockup.approved=false | Iterar mockup con el usuario |
| `mockup` | design_graphic.generated=false | `/karvey-design-graphic {change-id}` |
| `design_graphic` | design_graphic.approved=false | Revisar diseño con el usuario |
| `design_graphic` | architecture.generated=false | `/karvey-architecture {change-id}` |
| `architecture` | architecture.approved=false | Revisar arquitectura con el usuario |
| `architecture` | tasks.generated=false | `/karvey-tasks {change-id}` |
| `tasks` | tasks.approved=false | Revisar tasks con el usuario |
| `tasks` | tasks.approved=true | `/karvey-impl {change-id}` |
| `impl` | — | `/karvey-test {change-id}` |
| `test` | — | `/karvey-qa {change-id}` |
| `qa` | — | `/karvey-archive {change-id}` |

Mostrar al usuario:

```
📋 Karvey — Estado del cambio: {change-id}

Capability:    {capability}
Fase actual:   {phase}
Seguridad:     Tier {security_tier}
Gestión:       {clickup | markdown}

Aprobaciones:
  requirements   {✅ aprobado | ⏳ pendiente | ○ no generado}
  mockup         {✅ aprobado | ⏳ pendiente | ○ no generado}
  design_graphic {✅ aprobado | ⏳ pendiente | ○ no generado}
  architecture   {✅ aprobado | ⏳ pendiente | ○ no generado}
  tasks          {✅ aprobado | ⏳ pendiente | ○ no generado}

→ Siguiente paso: /{skill} {change-id}
```

### Con `--phase <fase>` — Descripción detallada de una fase

Mostrar la descripción expandida de la fase indicada (qué hace, qué produce, qué aprueba).

Fases válidas: `grill`, `init`, `requirements`, `mockup`, `design-graphic`, `architecture`, `tasks`, `impl`, `test`, `qa`, `archive`.

---

## Descripción de cada fase

### FASE 0: /karvey-grill
**Cuándo:** Antes de cualquier especificación. Si tienes solo una idea.
**Qué hace:** Hace preguntas en 5 ramas (problema, usuarios, datos, integraciones, restricciones) una por una, con recomendación incluida.
**Produce:** Síntesis estructurada lista para `karvey-init`.
**Aprueba:** El usuario decide si la síntesis es suficiente para iniciar.

### FASE 1: /karvey-init
**Cuándo:** Tienes claridad de qué se va a construir.
**Qué hace:** Genera `change-id`, crea `spec.json`, `proposal.md`. Si ClickUp: crea Epic. Si Markdown: crea `PLAN.md`.
**Produce:** `spec/changes/{change-id}/spec.json`, `proposal.md`, `PLAN.md` o Epic ClickUp.
**Aprueba:** El usuario confirma change-id y propuesta.
**Reglas:** `rules/clickup-protocol.md`, `rules/living-specs.md`

### FASE 2: /karvey-requirements
**Cuándo:** `spec.json` existe y `approvals.requirements.generated = false`.
**Qué hace:** Genera requirements en formato EARS, crea `requirements.md` y `spec-delta.md`. Actualiza living spec del capability si ya existe.
**Produce:** `spec/changes/{change-id}/requirements.md`, `spec-delta.md`.
**Aprueba:** Usuario revisa requirements antes de avanzar.
**Reglas:** `rules/ears-format.md`, `rules/living-specs.md`, `rules/security-tiers.md`

### FASE 3: /karvey-mockup
**Cuándo:** `approvals.requirements.approved = true`.
**Qué hace:** Genera HTML navegable con 3 niveles: App Shell → Section → Detail. Tailwind CDN, JS puro, paleta gris-only. Itera hasta aprobación.
**Produce:** `spec/changes/{change-id}/mockup.html`
**Aprueba:** Usuario navega el mockup y confirma los 3 niveles.

### FASE 4: /karvey-design-graphic
**Cuándo:** `approvals.mockup.approved = true`.
**Qué hace:** Detecta contexto B2B/B2C. Define sistema de colores OKLCH, tipografía, espaciado, motion. Verifica 7 anti-patrones. Actualiza `mockup.html` con el sistema visual.
**Produce:** `spec/changes/{change-id}/design-spec.md`, `mockup.html` actualizado.
**Aprueba:** Usuario valida el design system antes de arquitectura.

### FASE 5: /karvey-architecture
**Cuándo:** `approvals.design_graphic.approved = true`.
**Qué hace:** Define boundaries, controles de seguridad por capa según Tier, data flow, File Structure Plan (archivos a CREATE vs MODIFY), estrategia de observabilidad, decisiones arquitectónicas.
**Produce:** `spec/changes/{change-id}/architecture.md`
**Aprueba:** Usuario revisa arquitectura antes de generar tasks.
**Reglas:** `rules/security-tiers.md`

### FASE 6: /karvey-tasks
**Cuándo:** `approvals.architecture.approved = true`.
**Qué hace:** Descompone implementación en tasks de 10–30 min. Naming `E{n}.F{n}.T{n} [BD/Backend/Frontend]`. Si ClickUp: crea Features + Tasks con dependencias, time_estimate via REST, asignación a sprint. Si Markdown: genera checklist en `PLAN.md`.
**Produce:** `spec/changes/{change-id}/tasks.md`
**Aprueba:** Usuario valida tasks antes de implementar.
**Reglas:** `rules/clickup-protocol.md`

### FASE 7: /karvey-impl
**Cuándo:** `approvals.tasks.approved = true`.
**Qué hace:** Ejecuta tasks en orden BD→Backend→Frontend. Ciclo por task: read → code → test → commit. Actualiza ClickUp (tiempo real, status) o PLAN.md. Bump versión según el mecanismo del proyecto.
**Produce:** Código + 1 commit por task.
**Aprueba:** Implícito al completar todas las tasks.

### FASE 8: /karvey-test
**Cuándo:** Implementación completa.
**Qué hace:** Ejecuta tests unitarios (BD/Backend/Frontend) y E2E derivados de los 3 niveles del mockup. Documenta evidencias con request/response y PASS/FAIL.
**Produce:** `docs/test_plan.md`, `docs/test_evidence.md`
**Aprueba:** Tests PASS (o FAILs documentados) antes de QA.

### FASE 9: /karvey-qa
**Cuándo:** Tests completos.
**Qué hace:** Revisión en 6 dimensiones: Seguridad, Errores, Consistencia, Impacto, Env vars, Versionamiento. Crea `REVISION_PR_{n}_{date}.md`. Si ClickUp: subtareas por hallazgo. Notifica Google Chat.
**Produce:** `REVISION_PR_{n}_{date}.md`
**Aprueba:** Sin críticos/altos para avanzar a archive.

### FASE 10: /karvey-archive
**Cuándo:** Sin hallazgos bloqueantes en QA.
**Qué hace:** Verifica marker IMPLEMENTED. Mergea spec-deltas en living specs (ADDED/MODIFIED/REMOVED). Archiva directorio. Commits git. Cierra Epic en ClickUp o marca PLAN.md completo.
**Produce:** `spec/changes/archive/{date}-{change-id}/`, living specs actualizadas.
**Cierra:** El ciclo completo del cambio.

---

## Estructura de directorios del método

```
spec/
├── specs/                              ← Living specs (acumulativas por capability)
│   └── {capability}/
│       └── spec.md
└── changes/
    ├── {change-id}/                    ← Cambio activo
    │   ├── spec.json                   ← Metadatos, fase, aprobaciones
    │   ├── proposal.md                 ← Propuesta inicial
    │   ├── requirements.md             ← EARS requirements
    │   ├── spec-delta.md               ← Diff para living spec
    │   ├── mockup.html                 ← HTML navegable 3 niveles
    │   ├── design-spec.md              ← Sistema visual
    │   ├── architecture.md             ← Diseño técnico
    │   ├── tasks.md                    ← Breakdown de implementación
    │   ├── PLAN.md                     ← Solo si management=markdown
    │   └── IMPLEMENTED                 ← Marcador post-producción
    └── archive/
        └── {YYYY-MM-DD}-{change-id}/
```

## Reglas compartidas

| Archivo | Aplica en fases |
|---------|-----------------|
| `rules/clickup-protocol.md` | init, tasks, impl, qa, archive |
| `rules/ears-format.md` | requirements |
| `rules/security-tiers.md` | requirements, architecture |
| `rules/living-specs.md` | init, requirements, archive |

## Comandos de referencia rápida

```
/karvey                           → Pipeline + contexto del proyecto
/karvey <change-id>               → Estado y siguiente paso de un cambio
/karvey --phase <fase>            → Descripción detallada de una fase
/karvey-context                   → Dashboard: todos los cambios activos
/karvey-grill                     → Iniciar interrogación pre-spec
/karvey-init <change-id>          → Crear nuevo cambio
/karvey-requirements <change-id>  → Generar requirements EARS
/karvey-mockup <change-id>        → Generar mockup HTML navegable
/karvey-design-graphic <change-id>→ Aplicar sistema visual
/karvey-architecture <change-id>  → Diseñar arquitectura
/karvey-tasks <change-id>         → Generar tasks de implementación
/karvey-impl <change-id>          → Ejecutar implementación
/karvey-test <change-id>          → Ejecutar tests y documentar evidencias
/karvey-qa <change-id>            → QA 6 dimensiones
/karvey-archive <change-id>       → Cerrar ciclo y archivar
```
