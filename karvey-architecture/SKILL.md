---
name: karvey-architecture
description: Generate enterprise architecture design with explicit security controls, component boundaries, and integration patterns. Use after karvey-design-graphic. Triggers include "karvey architecture", "diseño técnico", "arquitectura", "diseño de sistema".
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent, WebSearch, AskUserQuestion
argument-hint: <change-id> [-y]
---

# Karvey Architecture

## Propósito

Generar el diseño técnico de arquitectura empresarial: componentes, boundaries, integraciones, controles de seguridad por tier, observabilidad, y estructura de archivos concreta por capa.

## Pasos de ejecución

### Paso 1 — Cargar contexto

Leer en paralelo:
- `spec/changes/{change-id}/spec.json` (security_tier, layers, capability)
- `spec/changes/{change-id}/requirements.md`
- `spec/changes/{change-id}/design-spec.md`
- `rules/security-tiers.md`
- Steering del proyecto: `product.md`, `tech.md` o equivalentes si existen

Verificar `approvals.design_graphic.approved = true`. Si no, detener.

### Paso 2 — Discovery de arquitectura

**Para features que extienden el sistema existente (brownfield):**

Despachar subagentes para explorar en paralelo:
- Subagente A: `grep -r` de patrones existentes relacionados con el capability
- Subagente B: leer endpoints/SPs existentes del área afectada

Identificar:
- Patrones de código existentes a seguir (naming, estructura de funciones, manejo de errores)
- Acceso a datos existente y sus firmas (SPs, queries, ORM models, repositorios, etc.)
- Endpoints / operaciones de API existentes y sus contratos
- Integraciones externas que ya existen

**Para features completamente nuevos (greenfield):**
- Definir los patrones nuevos coherentes con el stack existente

### Paso 3 — Clasificar complejidad arquitectónica

| Tipo | Criterio | Discovery |
|------|----------|-----------|
| Simple addition | Solo agrega CRUD o UI sin lógica nueva | Mínimo |
| Extension | Extiende sistema existente con lógica nueva | Medio |
| New capability | Capability nuevo, sin base existente | Completo |
| Complex integration | Integración con sistema externo o multi-capa | Exhaustivo |

### Paso 4 — Generar borrador de architecture.md

Estructura obligatoria:

```markdown
# Architecture: {change-id}

## Resumen
{1 párrafo: qué resuelve y cómo a alto nivel}

## Boundary del sistema
**Esta spec es dueña de:**
- {componente/endpoint/SP 1}
- {componente/endpoint/SP 2}

**Esta spec NO toca:**
- {componente existente que se mantiene}
- {funcionalidad adyacente fuera de scope}

**Cambios que requieren revalidación de este diseño:**
- {condición 1 que haría obsoleto este diseño}

## Componentes y responsabilidades

### Capa BD
| Componente | Tipo | Responsabilidad | Security Tier |
|------------|------|----------------|---------------|
| `{schema}.{sp_nombre}` | SP / función nueva | {qué hace} | Tier {N} |
| `{tabla}` | Tabla modificada | {qué columna/índice} | Tier {N} |

**Controles de seguridad BD:**
- Validación de contexto de usuario en cada query/SP: SÍ/NO
- Parámetros tipados (no SQL/query dinámico): SÍ/NO
- Logging de operaciones críticas: SÍ/NO

### Capa Backend
| Componente | Tipo | Responsabilidad | Security Tier |
|------------|------|----------------|---------------|
| `{backend_path}/{nombre}` | Endpoint / función nueva | {qué hace} | Tier {N} |
| `{módulo}` | Módulo modificado | {qué cambia} | Tier {N} |

**Controles de seguridad Backend:**
- Autenticación requerida: Tier {N} → {mecanismo: JWT, session, API key, etc.}
- Validación de contexto de usuario por request: SÍ/NO
- Sanitización de inputs en boundaries: SÍ/NO
- Manejo de errores sin exponer stack traces: SÍ/NO
- Variables sensibles via secrets manager: SÍ/NO

### Capa Frontend
| Componente | Tipo | Responsabilidad |
|------------|------|----------------|
| `{componente}` | Componente nuevo | {qué renderiza} |
| `{state_path}/{store}` | Store nuevo/modificado | {qué estado maneja} |
| `{api_layer}/{servicio}` | Servicio API | {qué endpoints consume} |

**Controles de seguridad Frontend:**
- Auth checks en rutas: Solo frontend como UX, enforcement en backend
- No exponer datos de otros tenants en store
- Sanitización de outputs dinámicos (no v-html sin sanitizar)

### Integraciones externas
| Sistema | Dirección | Protocolo | Auth | Timeout |
|---------|-----------|-----------|------|---------|
| {sistema} | inbound/outbound | REST/WebSocket | {mecanismo} | {ms} |

## Flujo de datos principal

```
{Actor} → {Frontend} → {API Gateway / BFF} → {Backend} → {BD / Servicio}
                                                    ↓
                                         {Sistema externo}
```

Describir el flujo paso a paso para el caso de uso principal:
1. {paso 1: qué hace el actor}
2. {paso 2: cómo responde el frontend}
3. {paso N: qué persiste en BD}

## Puntos de control de seguridad

| Punto | Tier | Control |
|-------|------|---------|
| Entrada al endpoint | {N} | Validar auth token, extraer identidad del usuario |
| Llamada a BD / servicio | {N} | Pasar contexto de usuario, no trustar input |
| Respuesta al cliente | {N} | No filtrar datos de otros usuarios/tenants |
| Logging | {N} | No loggear PII ni tokens |

## Plan de archivos (concreto)

### Archivos a CREAR
| Archivo | Capa | Responsabilidad |
|---------|------|----------------|
| `{backend_path}/{nombre}` | Backend | {descripción} |
| `{frontend_path}/{nombre}` | Frontend | {descripción} |

### Archivos a MODIFICAR
| Archivo | Capa | Cambio |
|---------|------|--------|
| `{backend_path}/{existente}` | Backend | Agregar {qué} |

### Archivos de BD
| Archivo | Tipo | Cambio |
|---------|------|--------|
| `{db_path}/{sp_nombre}.sql` | SP / migración nueva | {descripción} |

## Estrategia de observabilidad

- Logging estructurado en: {puntos de log}
- Métricas a trackear: {lista}
- Alertas recomendadas: {lista}
- Trazabilidad de request: {correlationId, userId/contextKey en cada log}

## Decisiones arquitectónicas

| Decisión | Alternativa considerada | Por qué se eligió esta |
|----------|------------------------|------------------------|
| {decisión 1} | {alternativa} | {razón} |

## Riesgos y mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|-----------|
| {riesgo} | Alta/Media/Baja | Alto/Medio/Bajo | {cómo se mitiga} |
```

### Paso 5 — Review gate

Verificar antes de escribir:
- [ ] Todos los requirements tienen al menos un componente que los implementa
- [ ] El boundary está explícitamente definido
- [ ] Cada componente tiene su Security Tier declarado
- [ ] No hay componentes con responsabilidad vaga ("helper", "utils" sin descripción)
- [ ] El plan de archivos es concreto (paths reales, no "crear archivo para X")
- [ ] Los controles de seguridad cubren el Tier declarado en spec.json
- [ ] Hay estrategia de observabilidad

Si hay issues: corregir y re-verificar. Máximo 2 iteraciones.

### Paso 6 — Escribir architecture.md

```
spec/changes/{change-id}/architecture.md
```

Actualizar `spec.json`:
- `phase: "architecture-generated"`
- `approvals.architecture.generated: true`

### Paso 6B — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar el `architecture.md` creado.
Si `spec/graphify-out/` no existe, invocar `/graphify spec/` sin `--update`.

### Paso 7 — Presentar para aprobación

Si flag `-y`: auto-aprobar.
Si no: presentar resumen y pedir aprobación.

Al aprobar: `approvals.architecture.approved: true`, `phase: "architecture-approved"`.

```
✅ Architecture aprobada

Siguiente paso:
/karvey-tasks {change-id}
```
