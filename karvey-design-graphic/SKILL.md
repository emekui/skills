---
name: karvey-design-graphic
description: Define the graphic design specification for an approved mockup. Uses impeccable-inspired design laws to establish color system, typography, layout, and motion. Updates the mockup HTML with the visual system. Triggers include "karvey design-graphic", "diseño gráfico", "especificación visual", "sistema de diseño".
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
argument-hint: <change-id>
---

# Karvey Design Graphic

## Propósito

Con el mockup aprobado como wireframe estructural, definir la especificación completa del diseño visual: color, tipografía, layout, motion y micro-interacciones. Actualizar el mockup HTML con el sistema visual definido.

## Pasos de ejecución

### Paso 1 — Cargar contexto

Leer:
- `spec/changes/{change-id}/spec.json`
- `spec/changes/{change-id}/mockup.html`
- `spec/changes/{change-id}/proposal.md`

Verificar `approvals.mockup.approved = true`. Si no, detener.

Buscar si existe `PRODUCT.md` o `DESIGN.md` en el proyecto para entender el brand existente.

### Paso 2 — Identificar el registro de diseño

Determinar el tipo de producto:

**Producto empresarial (B2B, herramienta interna):**
- Paleta contenida, funcionalidad sobre expresividad
- Tipografía de alta legibilidad
- Motion mínimo y no distractor
- Densidad de información alta

**Producto de consumo (B2C, experiencia de usuario):**
- Paleta más expresiva y de marca
- Tipografía con más personalidad
- Motion como parte de la experiencia
- Densidad de información moderada

Inferir del `proposal.md` y `spec.json.capability`.

### Paso 3 — Definir sistema de color (OKLCH)

Elegir estrategia de paleta:

**Restrained (recomendado para B2B):**
- 1 color de acento, el resto neutros
- Acento: `oklch(55% 0.18 {hue})`
- Neutros: escala de grays `oklch(98% 0 0)` → `oklch(15% 0 0)`

**Committed:**
- 1 color primario + 1 de apoyo
- Primario: `oklch(50% 0.20 {hue})`
- Apoyo: `oklch(55% 0.15 {hue complementario})`

**Full palette:**
- Color primario, secundario, acento y semánticos
- Apropiado para productos con roles visuales diferenciados

Definir siempre:
- `--color-primary`: acción principal, CTA
- `--color-surface`: fondo de tarjetas/paneles
- `--color-background`: fondo de página
- `--color-border`: bordes sutiles
- `--color-text-primary`: texto principal
- `--color-text-secondary`: texto de apoyo
- `--color-semantic-success`: `oklch(62% 0.17 145)`
- `--color-semantic-error`: `oklch(55% 0.22 25)`
- `--color-semantic-warning`: `oklch(75% 0.18 80)`

### Paso 4 — Definir tipografía

Elegir 1-2 fuentes de Google Fonts (o system fonts para B2B):

**Para B2B empresarial:**
- Display/heading: Inter, DM Sans, o system-ui
- Body: misma familia, distintos pesos

**Para B2C:**
- Display: fuente con carácter (Fraunces, Instrument Serif, Plus Jakarta Sans)
- Body: fuente de alta legibilidad (Inter, DM Sans)

Escala tipográfica:
```
--text-xs:   0.75rem / 1rem
--text-sm:   0.875rem / 1.25rem
--text-base: 1rem / 1.5rem
--text-lg:   1.125rem / 1.75rem
--text-xl:   1.25rem / 1.75rem
--text-2xl:  1.5rem / 2rem
--text-3xl:  1.875rem / 2.25rem
```

### Paso 5 — Definir layout y spacing

Sistema de espaciado base 4px:
```
--space-1: 4px   --space-2: 8px   --space-3: 12px
--space-4: 16px  --space-6: 24px  --space-8: 32px
--space-12: 48px --space-16: 64px --space-24: 96px
```

Grid del producto:
- Sidebar: fijo 240px (o 64px colapsado)
- Content area: fluid con max-width según densidad
- Columnas de contenido: 12 columnas con gap de 24px

Border radius:
```
--radius-sm: 4px   --radius-md: 8px
--radius-lg: 12px  --radius-xl: 16px  --radius-full: 9999px
```

### Paso 6 — Definir motion

Para B2B: motion funcional, no decorativo
```css
--duration-fast: 100ms
--duration-base: 200ms
--duration-slow: 300ms
--ease-standard: cubic-bezier(0.4, 0, 0.2, 1)
--ease-enter: cubic-bezier(0, 0, 0.2, 1)
--ease-exit: cubic-bezier(0.4, 0, 1, 1)
```

Regla: usar `--duration-fast` para feedback, `--duration-base` para transiciones, `--duration-slow` para overlays.

### Paso 7 — Verificar anti-patrones prohibidos

Antes de escribir el design-spec, confirmar que el sistema NO incluye:
- ❌ Side-stripe borders decorativos (borde de color solo en un lado de tarjetas)
- ❌ Gradient text (texto con gradiente de color)
- ❌ Glassmorphism decorativo (blur/transparencia sin función)
- ❌ Card grids con todas las tarjetas idénticas sin variación de énfasis
- ❌ Hero metrics (número grande en el centro de una pantalla como único contenido)
- ❌ Fondos con patrones de ruido o textura excesiva
- ❌ Animaciones de más de 500ms en interacciones frecuentes

### Paso 8 — Escribir design-spec.md

```markdown
# Design Spec: {change-id}

## Registro de diseño
{B2B empresarial | B2C consumo} — {justificación}

## Sistema de color (OKLCH)
Estrategia: {Restrained | Committed | Full palette}

| Token | Valor OKLCH | Uso |
|-------|-------------|-----|
| --color-primary | oklch(...) | {uso} |
| --color-surface | oklch(...) | {uso} |
...

## Tipografía
| Token | Fuente | Peso | Uso |
|-------|--------|------|-----|
| Heading | {fuente} | 600-700 | Títulos de página y sección |
| Body | {fuente} | 400-500 | Texto de contenido |
| Caption | {fuente} | 400 | Metadatos, labels |

## Escala tipográfica
(tabla completa)

## Layout
- Sidebar: {fijo N px | colapsable | sin sidebar}
- Grid: {descripción}
- Max-width content: {N px}

## Spacing
(tabla base 4px)

## Border radius
(tabla)

## Motion
(tabla de variables)

## Componentes clave y su tratamiento visual
| Componente | Superficie | Borde | Shadow | Estado hover |
|------------|-----------|-------|--------|--------------|
| Botón primario | | | | |
| Botón secundario | | | | |
| Tarjeta de datos | | | | |
| Input de formulario | | | | |
| Row de tabla | | | | |
| Modal overlay | | | | |

## Anti-patrones evitados
(lista de los que se verificaron)
```

Escribir a `spec/changes/{change-id}/design-spec.md`.

### Paso 9 — Actualizar mockup.html con el sistema visual

Editar `mockup.html` para:
1. Agregar las CSS custom properties (`:root { --color-primary: ...; ... }`)
2. Reemplazar colores hardcodeados por las variables
3. Agregar la fuente vía `@import` de Google Fonts
4. Aplicar el sistema de motion a transiciones existentes
5. Actualizar el banner: `🎨 MOCKUP CON DISEÑO GRÁFICO — {change-id} — {fecha}`

### Paso 9B — Actualizar grafo de conocimiento

Invocar `/graphify spec/ --update` para reflejar `design-spec.md` y el `mockup.html` actualizado.
Si `spec/graphify-out/` no existe, invocar `/graphify spec/` sin `--update`.

### Paso 10 — Output

```
✅ Design spec generada

Archivos creados/actualizados:
  - spec/changes/{change-id}/design-spec.md
  - spec/changes/{change-id}/mockup.html (actualizado con sistema visual)

Sistema de diseño:
  - Registro: {B2B/B2C}
  - Estrategia de color: {nombre}
  - Tipografía: {fuente(s)}
  - Anti-patrones verificados: ✅

Actualizar spec.json: approvals.design_graphic = true

Siguiente paso:
/karvey-architecture {change-id}
```

Actualizar `spec.json`: `approvals.design_graphic.approved: true`, `phase: "design-graphic-approved"`.
