---
name: karvey-mockup
description: Generate a navigable HTML mockup with 3 levels of depth from approved requirements. Iterate with user until approved, then advance to graphic design. Triggers include "karvey mockup", "generar mockup", "crear prototipo", "wireframe".
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
argument-hint: <change-id> [--iteration N]
---

# Karvey Mockup — Prototipo Navegable HTML

## Propósito

Generar un archivo HTML navegable con 3 niveles de profundidad antes de definir el diseño gráfico. El ingeniero navega el mockup en el browser, da feedback, y se itera hasta aprobación. Solo después se avanza a diseño gráfico y arquitectura.

## Niveles de navegación

- **Nivel 1 — App Shell**: Navegación principal (sidebar/topbar con secciones del producto)
- **Nivel 2 — Sección**: Vistas de cada sección (listados, formularios, dashboards)
- **Nivel 3 — Detalle**: Vistas de detalle (modales, paneles laterales, subvistas, wizards)

## Pasos de ejecución

### Paso 1 — Cargar contexto

Leer:
- `spec/changes/{change-id}/spec.json`
- `spec/changes/{change-id}/requirements.md`
- `spec/changes/{change-id}/proposal.md`

Verificar que `approvals.requirements.approved = true`. Si no, detener y pedir aprobar requirements primero.

**Verificar si aplica UI:** Leer `spec.json` → campo `layers`. Si solo incluye `[BD]` y/o `[Backend]` sin `[Frontend]`, preguntar al usuario: "Este cambio no parece tener interfaz de usuario. ¿Requiere mockup visual o pasamos directo a arquitectura?" Si no requiere, saltar a karvey-architecture.

Detectar iteración: si existe `spec/changes/{change-id}/mockup.html`, incrementar el número de iteración.

### Paso 2 — Mapear pantallas desde requirements

Construir el mapa de pantallas leyendo los requirements:

```
Nivel 1 (Navegación):
  - {Sección A} → requirements {N.N, N.N}
  - {Sección B} → requirements {N.N, N.N}

Nivel 2 (Vistas por sección):
  Sección A:
    - Vista Lista: {qué muestra}
    - Vista Formulario: {qué captura}
  Sección B:
    - Vista Dashboard: {qué muestra}

Nivel 3 (Detalles):
  - Modal confirmación
  - Panel detalle
  - Wizard paso a paso
```

### Paso 3 — Generar mockup.html

Generar un único archivo HTML autocontenido:

**Estructura del HTML:**
```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>MOCKUP — {change-id} — Iteración {N}</title>
  <!-- Tailwind CDN — solo para prototipado, no implica su uso en el proyecto real -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>/* estilos adicionales */</style>
</head>
<body>
  <!-- Banner de mockup siempre visible -->
  <!-- App Shell con navegación Nivel 1 -->
  <!-- Contenedor de vistas Nivel 2 (cambia con navegación) -->
  <!-- Overlays de Nivel 3 (modales, paneles) -->
  <!-- JavaScript de navegación -->
</body>
</html>
```

**Reglas de construcción:**

1. **Banner persistente** en top: `🔧 MOCKUP — {change-id} — Iteración {N} — {fecha}`
2. **Navegación Nivel 1**: sidebar o topbar con íconos de placeholder (círculos grises) y nombres de sección
3. **Vistas Nivel 2**: representar cada pantalla como una vista con:
   - Título de la pantalla
   - Tabla/lista con columnas reales (labels reales, datos de ejemplo coherentes con el dominio)
   - Botones con labels reales ("Transferir llamada", "Confirmar", no "Button 1")
   - Formularios con campos reales y tipos correctos
   - Estados de carga representados con skeletons grises
4. **Nivel 3**: modales y paneles implementados como overlays que se muestran/ocultan con JS puro
5. **Navegación JS**: `showView(viewId)`, `showModal(modalId)`, `closeModal(modalId)` — sin dependencias externas
6. **Paleta neutral**: grays únicamente. El color y el design system del proyecto real vendrán en karvey-design-graphic.
7. **Responsive básico**: funcionar en 1280px+ de ancho mínimo
8. **Datos de ejemplo**: usar datos ficticios pero coherentes con el dominio (nombres de tenant, tipos de llamada reales, etc.)

**Elementos prohibidos en el mockup:**
- Lorem ipsum (usar texto real del dominio)
- "Button 1", "Section A" (usar labels reales)
- Imágenes externas (usar divs con bg-gray-300)
- Frameworks JS pesados (solo Tailwind CDN y JS nativo)

### Paso 4 — Escribir archivo

```
spec/changes/{change-id}/mockup.html
```

Actualizar `spec.json`:
- `phase: "mockup-generated"`
- `approvals.mockup.generated: true`
- `updated_at: {timestamp}`

### Paso 4B — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar el `mockup.html` creado o modificado.
Si `spec/graphify-out/` no existe, invocar `/graphify spec/` sin `--update`.

### Paso 5 — Presentar al usuario

```
🖥️ Mockup generado — Iteración {N}

Archivo: spec/changes/{change-id}/mockup.html
Abrir con: open spec/changes/{change-id}/mockup.html

Pantallas incluidas ({N} total):
  Nivel 1 — Navegación:
    - {sección 1}
    - {sección 2}
  Nivel 2 — Vistas:
    - {sección 1}: {vista A}, {vista B}
    - {sección 2}: {vista C}
  Nivel 3 — Detalles:
    - {modal/panel 1}
    - {modal/panel 2}

¿Qué feedback tenés? Describí los cambios y genero la Iteración {N+1}.
O escribí "aprobado" para avanzar a diseño gráfico.
```

### Paso 6 — Ciclo de iteración

Si el usuario da feedback:
1. Leer el feedback detalladamente
2. Identificar qué pantallas/componentes cambiar
3. Editar `mockup.html` aplicando los cambios
4. Incrementar el número de iteración en el banner
5. Invocar `/graphify spec/ --update`
6. Volver al Paso 5

Si el usuario aprueba:
- Actualizar `spec.json`: `approvals.mockup.approved: true`
- Output:
```
✅ Mockup aprobado — Iteración {N}

Siguiente paso:
/karvey-design-graphic {change-id}
```

## Safety

- Verificar que requirements estén aprobados antes de generar
- Si el mockup tiene >20 pantallas, preguntar si dividir en módulos
- El archivo HTML debe abrir sin errores en browser moderno (Chrome/Safari/Firefox)
- No usar `document.write` ni `eval` en el JS generado
