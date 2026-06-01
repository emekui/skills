---
name: karvey-grill
description: Pre-spec interrogation. Interviews the engineer relentlessly about their problem before writing any spec. Use at the start of every new feature or initiative — before karvey-init. Triggers include "grill me", "entrevístame", "quiero especificar algo", "tengo una idea", "necesito una feature".
allowed-tools: Read, Bash, Glob, Grep
argument-hint: [descripción breve del problema o idea]
---

# Karvey Grill — Pre-Spec Interrogation

## Propósito

Antes de escribir una sola línea de spec, este skill te entrevista en profundidad hasta que el problema esté completamente comprendido. Una pregunta a la vez. Con recomendación incluida en cada una.

## Instrucciones de ejecución

### Paso 1 — Contexto inicial

Si el usuario dio una descripción en `$ARGUMENTS`, usarla como punto de partida.
Si no, hacer la primera pregunta abierta: "¿Qué problema querés resolver y para quién?"

Antes de preguntar algo que pueda responderse explorando el codebase, **explorar primero**:
- Buscar si ya existe funcionalidad similar (`grep`, `find`)
- Leer archivos de configuración o specs existentes en `spec/`
- Si encontrás la respuesta, presentarla como contexto y avanzar a la siguiente pregunta

### Paso 2 — Árbol de interrogación

Recorrer sistemáticamente estas ramas, **una pregunta a la vez**:

#### Rama A: El problema
1. ¿Quién tiene el problema? (persona/sistema/rol)
2. ¿Cuál es la situación actual que causa fricción?
3. ¿Qué tan frecuente ocurre? ¿Cuál es el impacto?
4. ¿Intentaron resolverlo antes? ¿Qué falló?

#### Rama B: El alcance
5. ¿Qué está DENTRO del alcance de esta iniciativa?
6. ¿Qué está FUERA del alcance explícitamente?
7. ¿Qué otros sistemas o componentes se ven afectados?
8. ¿Qué datos nuevos se necesitan? ¿Qué datos existentes se modifican?

#### Rama C: Restricciones técnicas
9. ¿En qué capa(s) opera esto: BD / Backend / Frontend / Infra?
10. ¿Hay integraciones externas involucradas?
11. ¿Qué nivel de seguridad requiere? (datos públicos / autenticado / datos sensibles / crítico con audit trail)
12. ¿Hay restricciones de performance o SLA?

#### Rama F: Stack tecnológico
> Antes de preguntar, explorar el codebase: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `pom.xml`, `Gemfile`, `Cargo.toml`, archivos de config de BD, etc. Solo preguntar lo que no se pueda inferir.

13. ¿Cuál es el lenguaje y framework del backend? (Node/Express, Python/FastAPI, Go, Java/Spring, Ruby/Rails, etc.)
14. ¿Qué base de datos usa y cómo accede a ella? (PostgreSQL/ORM, MySQL/queries, MongoDB, Oracle/SPs, Redis, etc.)
15. ¿El cambio tiene UI? ¿Con qué framework? (React, Vue, Angular, Svelte, server-rendered, no aplica, etc.)
16. ¿Qué protocolo de API usa? (REST, GraphQL, gRPC, mensajería, WebSockets, etc.)
17. ¿Cómo se versionan los releases? (package.json, pyproject.toml, archivo VERSION, git tags, ninguno, etc.)
18. ¿Hay convenciones de git establecidas? (Conventional Commits, gitflow, trunk-based, squash, etc.)

#### Rama D: Criterios de éxito
13. ¿Cómo sabremos que esto funciona correctamente?
14. ¿Cuáles son los casos de error críticos que deben manejarse?
15. ¿Qué NO debe cambiar del comportamiento actual?

#### Rama E: Dependencias y riesgos
16. ¿Qué debe estar listo ANTES de empezar esto?
17. ¿Qué bloquea este trabajo si no se resuelve primero?
18. ¿Cuál es el riesgo más grande de esta iniciativa?

### Paso 3 — Formato de cada pregunta

```
**Pregunta N/~18:** {pregunta clara y específica}

*Mi recomendación:* {tu respuesta sugerida basada en lo que ya sabés del contexto}
```

Adaptar las preguntas según las respuestas anteriores. Si una rama ya está clara, saltarla.
Si una respuesta genera sub-preguntas, profundizar antes de avanzar.

### Paso 4 — Síntesis final

Cuando el árbol esté cubierto (o el usuario indique que terminó), generar un resumen:

```markdown
## Resumen Pre-Spec: {nombre tentativo del cambio}

### El problema
{quién, situación actual, impacto}

### Lo que cambia
{qué entra en scope, qué queda fuera}

### Stack detectado
- Backend: {lenguaje/framework}
- Base de datos: {BD y patrón de acceso}
- Frontend: {framework o "no aplica"}
- API: {protocolo}
- Versionamiento: {mecanismo}
- Git: {convenciones}

### Restricciones clave
- Capas: {BD/Backend/Frontend/Infra}
- Seguridad: {nivel — justificación}
- Integraciones externas: {lista}
- Restricciones técnicas: {lista}

### Criterios de éxito
{cómo se verifica que funciona}

### Casos de error críticos
{lista}

### Riesgos principales
{lista}

### Dependencias previas
{qué debe existir antes}

### Change ID sugerido
`{add|fix|update|remove}-{nombre-descriptivo}` — {rationale}
```

### Paso 5 — Transición

Al terminar, indicar:
```
✅ Interrogación completa. Siguiente paso:
/karvey-init {change-id-sugerido}
```

## Reglas del entrevistador

- **Una pregunta a la vez**. Nunca hacer dos preguntas en el mismo mensaje.
- **Siempre incluir recomendación** en cada pregunta.
- **Explorar el codebase** antes de preguntar algo que pueda descubrirse solo.
- **No asumir** tecnología, escala ni usuarios sin confirmar.
- **No avanzar** a especificación hasta que el árbol esté cubierto.
- El tono es colaborativo, no inquisitorial. El objetivo es construir entendimiento compartido.
