---
name: karvey-test
description: Execute unit tests and E2E tests after implementation. Generates test_plan.md and test_evidence.md with request/response/PASS/FAIL evidence. Triggers include "karvey test", "ejecutar tests", "pruebas", "testing", "evidencias".
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent
argument-hint: <change-id> [--e2e-only] [--unit-only]
---

# Karvey Test

## Propósito

Ejecutar el plan de pruebas completo post-implementación: tests unitarios por capa y pruebas E2E del flujo completo. Documentar evidencias en `docs/test_evidence.md`.

## Pasos de ejecución

### Paso 1 — Cargar contexto

Leer:
- `spec/changes/{change-id}/requirements.md`
- `spec/changes/{change-id}/architecture.md`
- `spec/changes/{change-id}/mockup.html` (para mapear flujos E2E)
- `spec/changes/{change-id}/tasks.md`

Detectar stack: `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `pom.xml`, `Gemfile`, `Cargo.toml` u otros archivos de configuración del proyecto. Identificar:
- Lenguaje y framework del backend
- Tipo de base de datos y patrón de acceso (ORM, SPs, queries directas)
- Framework de test unitario disponible
- Framework E2E disponible
- Protocolo de API (REST, GraphQL, gRPC, etc.)

### Paso 2 — Generar o actualizar test_plan.md

Si `docs/test_plan.md` no existe, crearlo. Si existe, agregar sección para este change-id.

**Estructura del test plan:**

```markdown
# Test Plan: {change-id}

## Scope
Requirements cubiertos: {lista}
Capas: {BD / Backend / Frontend}

## Tests Unitarios

### BD
| ID | SP / Función | Caso | Input | Expected |
|----|-------------|------|-------|----------|
| UT-BD-01 | `{schema}.{sp_nombre}` | Happy path | `@{contextKey}={valor}, @param=X` | Retorna {Y} |
| UT-BD-02 | `{schema}.{sp_nombre}` | Contexto inválido | `@{contextKey}={valor_inválido}` | Error controlado |
| UT-BD-03 | `{schema}.{sp_nombre}` | Input vacío | `@param=NULL` | Error controlado |

### Backend
| ID | Endpoint / Operación | Caso | Input | Expected |
|----|---------|------|---------|----------|
| UT-BE-01 | {operación} | Happy path | `{input válido}` | {respuesta esperada} |
| UT-BE-02 | {operación} | Sin auth | Sin credencial | Error de autenticación |
| UT-BE-03 | {operación} | Contexto incorrecto | `{input de otro usuario}` | Error de autorización |
| UT-BE-04 | {operación} | Input inválido | `{input incompleto}` | Error de validación |

### Frontend
| ID | Componente | Caso | Acción | Expected |
|----|-----------|------|--------|----------|
| UT-FE-01 | {Componente} | Render inicial | Montar | Muestra skeleton |
| UT-FE-02 | {Componente} | Datos cargados | API retorna datos | Muestra {N} items |
| UT-FE-03 | {Componente} | Error API | API retorna 500 | Muestra mensaje de error |
| UT-FE-04 | {Componente} | Sin datos | API retorna [] | Muestra estado vacío |

## Tests E2E

Flujos derivados de los 3 niveles del mockup:

### Flujo 1: {Nombre del flujo principal — Nivel 1→2→3}
| Paso | Acción | Expected |
|------|--------|----------|
| 1 | Navegar a {sección} | Vista de {Nivel 2} se carga |
| 2 | Hacer click en {elemento} | {Modal/Panel Nivel 3} se abre |
| 3 | Completar formulario con {datos válidos} | {resultado esperado} |
| 4 | Confirmar acción | {estado final} |

### Flujo 2: {Nombre del flujo de error}
...

### Flujo 3: {Flujo de permisos / seguridad}
...
```

### Paso 3 — Ejecutar tests unitarios

#### BD
Para cada SP / función, ejecutar directamente en la BD de dev usando la sintaxis del stack:
```sql
-- UT-BD-01: Happy path (adaptar sintaxis según BD: EXEC, CALL, SELECT, etc.)
{DB_CALL} {schema}.{sp_nombre} @{contextKey}={valor}, @param='valor_prueba'
-- Verificar resultado esperado
```

Registrar resultado en `docs/test_evidence.md`.

#### Backend
Usar `curl`, el test runner del proyecto, o el cliente apropiado según el protocolo:
```bash
# REST (adaptar método, URL y puerto según el stack)
curl -s -X POST "http://localhost:{puerto}/{endpoint}" \
  -H "Authorization: {mecanismo_auth}" \
  -H "Content-Type: application/json" \
  -d '{input_de_prueba}'

# GraphQL
curl -s -X POST "http://localhost:{puerto}/graphql" \
  -H "Content-Type: application/json" \
  -d '{"query": "{operacion(args) { campos }}"}'

# gRPC / otros: usar el cliente nativo del proyecto
```

Ejecutar con el test runner detectado:
- Python: `pytest`, `unittest`
- Node/JS: `jest`, `vitest`, `mocha`, `npm run test`
- Go: `go test ./...`
- Java: `mvn test`, `gradle test`
- Ruby: `rspec`, `rails test`
- Rust: `cargo test`
- Otro: usar el comando de test que exista en el proyecto (`Makefile`, `package.json scripts`, etc.)

Si no hay test runner configurado: verificación manual documentada como evidencia.

#### Frontend
Ejecutar con el test runner detectado del frontend:
- `npm run test`, `npx vitest run`, `jest`, `ng test`, etc.
Si no existe test runner frontend: verificación manual documentada como evidencia.

### Paso 4 — Ejecutar tests E2E

Detectar el framework E2E disponible en el proyecto:
- **Playwright**: `npx playwright test`
- **Cypress**: `npx cypress run`
- **Selenium**: ejecutar suite configurada
- **framework nativo** (Go, Java, etc.): usar el comando de integración del proyecto
- **Sin framework E2E**: ejecutar manualmente cada flujo navegando la app en dev

Para cada paso del flujo E2E, independientemente del método, documentar:
- URL o pantalla visitada
- Acción realizada
- Response/comportamiento observado
- PASS / FAIL

Para cada paso del flujo E2E documentar:
- URL visitada
- Acción realizada
- Response/comportamiento observado
- PASS / FAIL

### Paso 5 — Documentar evidencias

Escribir o actualizar `docs/test_evidence.md`:

```markdown
# Test Evidence: {change-id}

**Fecha:** {YYYY-MM-DD HH:MM}
**Ambiente:** dev / local
**Stack:** {stack detectado}

## Tests Unitarios — BD

### UT-BD-01: {nombre SP} — Happy path
**Resultado: ✅ PASS**

Request:
```sql
{DB_CALL} {schema}.{sp_nombre} @{contextKey}={valor}, @param='valor'
```

Response:
```
{resultado real — formato según el protocolo del proyecto (JSON, XML, binario, etc.)}
```

Observaciones: {si aplica}

---

### UT-BD-02: Contexto inválido
**Resultado: ✅ PASS**
...

## Tests Unitarios — Backend

### UT-BE-01: POST /endpoint — Happy path
**Resultado: ✅ PASS**

Request:
```
POST http://localhost:{puerto}/api/{endpoint}
Authorization: Bearer ***
Content-Type: application/json

{body}
```

Response:
```
HTTP/1.1 200 OK

{body de respuesta}
```

---

## Tests E2E

### Flujo 1: {Nombre}
**Resultado: ✅ PASS**

| Paso | Acción | Resultado | Estado |
|------|--------|-----------|--------|
| 1 | {acción} | {resultado} | ✅ |
| 2 | {acción} | {resultado} | ✅ |

---

## Resumen

| Categoría | Total | PASS | FAIL |
|-----------|-------|------|------|
| BD | {N} | {N} | {N} |
| Backend | {N} | {N} | {N} |
| Frontend | {N} | {N} | {N} |
| E2E | {N} | {N} | {N} |
| **Total** | **{N}** | **{N}** | **{N}** |
```

### Paso 5B — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar `test_plan.md` y `test_evidence.md`.
Si `spec/graphify-out/` no existe, invocar `/graphify spec/` sin `--update`.

### Paso 6 — Reportar al usuario

Si todos los tests PASS:
```
✅ Testing completo

Resultados:
  BD: {N}/{N} PASS
  Backend: {N}/{N} PASS
  Frontend: {N}/{N} PASS
  E2E: {N}/{N} PASS

Evidencias: docs/test_evidence.md

Siguiente paso:
/karvey-qa {change-id}
```

Si hay FAILs:
```
⚠️ {N} tests fallaron

FAILs:
  - {ID}: {descripción del fallo}
  - {ID}: {descripción del fallo}

Corregir y re-ejecutar: /karvey-test {change-id}
O avanzar con los fallos documentados (no recomendado): /karvey-qa {change-id}
```
