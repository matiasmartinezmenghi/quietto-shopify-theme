# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

**Quietto** — ecommerce DTC para Argentina. Producto: entrenador ultrasónico para perros. Objetivo: validar demanda antes de importar stock.

### Oferta
- **Pack 1**: 1 dispositivo ultrasónico
- **Pack 2** (objetivo de ventas): 2 dispositivos + curso básico de adiestramiento incluido gratis
- **Upsell**: Si compra Pack 1 → ofrecer curso como upsell pago

### Posicionamiento
Marca moderna y confiable. Comunicar: ayuda conductual, entrenamiento, convivencia, tranquilidad, corrección de hábitos.
No comunicar: castigo, dolor, agresión, control extremo.
No hacer claims médicos. No prometer resultados garantizados. No decir "funciona en todos los perros".

### Tono de copy
Simple, emocional, moderno, confiable, directo, argentino neutro.

---

## Overview

This is a Shopify storefront theme based on Shopify's **Atelier v3.5.1**. There is no build step — all assets are deployed directly; Shopify handles minification server-side. Development uses the Shopify CLI.

## Development Commands

```bash
# Authenticate with Shopify
shopify auth login

# Push/pull theme files and start local dev server (hot-reload via Shopify CLI)
shopify theme dev --store <store-name>

# Push all theme files to a store
shopify theme push --store <store-name>

# Pull theme files from a store
shopify theme pull --store <store-name>

# Check Liquid syntax (requires Shopify CLI)
shopify theme check
```

There are no test suites, linting scripts, or package.json in this project.

## JavaScript Architecture

The theme uses **vanilla JS with custom Web Components** — no React, Vue, or bundler.

### Component System (`assets/component.js`)
All interactive UI is built on a `Component` base class extending `DeclarativeShadowElement`. Components use `ref` attributes in HTML for DOM queries (similar to Vue/React refs), with a `MutationObserver` that keeps refs live when the DOM changes.

```js
// Typical custom element pattern
class ProductFormComponent extends Component {
  connectedCallback() {
    this.refs.form.addEventListener('submit', this.onSubmit.bind(this));
  }
}
customElements.define('product-form-component', ProductFormComponent);
```

Key supporting files:
- **`assets/utilities.js`** — `yieldToMainThread()`, `isLowPowerDevice()`, `requestIdleCallback` shim, View Transitions API helpers
- **`assets/events.js`** — Custom event system (`ThemeEvents`, `CartAddEvent`, etc.) used for cross-component communication
- **`assets/morph.js`** — DOM diffing/morphing library used for efficient cart/section updates (similar to morphdom)
- **`assets/performance.js`** — Performance monitoring utilities

### Cart Integration
Cart mutations use the Shopify Ajax Cart API via `fetch()` to `/cart/add`, `/cart/change`, `/cart/update`. Results are diffed into the DOM using `morph.js`. Custom events (`CartAddEvent`, etc.) coordinate UI updates across components.

### Type Checking
`assets/jsconfig.json` enables strict JS type checking (`checkJs: true`, `strictNullChecks: true`, `noImplicitAny: true`, target ES2020). Use JSDoc comments for type annotations — there are no `.ts` files.

## Liquid & Template Architecture

### Directory roles
- **`sections/`** (41 files) — Reusable page sections configurable via the Shopify theme editor (hero, slideshow, product list, filters, etc.)
- **`blocks/`** (41 files) — Nested sub-components within sections; configured per-block in the theme editor
- **`snippets/`** (103 files) — Shared Liquid partials rendered with `{% render 'snippet-name' %}`
- **`templates/`** (13 JSON files) — Page-level templates that reference which sections appear on each page type
- **`layout/`** — `theme.liquid` is the root layout; `password.liquid` for store password pages

### Key Liquid patterns
- Sections use `content_for_blocks` (Shopify's proprietary block rendering API) — not standard Liquid
- Color schemes, typography, and spacing are driven by `config/settings_data.json` (written by the theme editor) and declared in `config/settings_schema.json`
- CSS custom properties bridge settings to styles (e.g., `--hover-lift-amount`, `--surface-transition-duration`)

## CSS Architecture

`assets/base.css` (~104KB) is the single stylesheet — no preprocessor. All theming uses CSS custom properties. Responsive design, animations, and component states are all handled via variable overrides rather than media-query duplication.

## Localization

47 language files live in `locales/`. The default locale is `locales/en.default.json`. Always add new user-facing strings to all locale files or use the `t:` Liquid filter with an existing key.

## Workflow Rules

**Antes de cambios importantes**, siempre explicar:
1. Estrategia
2. Archivos afectados
3. Impacto mobile
4. Impacto en conversión

**Restricciones operativas:**
- Nunca tocar el checkout
- Nunca modificar el live theme — siempre trabajar sobre **Quietto Dev**
- Nunca romper estructura Shopify
- Priorizar velocidad mobile y conversión sobre todo lo demás
- Evitar apps de terceros innecesarias

### Landing page
Inspiración: BarkNoMore, BarxBuddy, BarkBegone.
La landing debe incluir: sticky CTA mobile, bundles, CTAs repetidos, comparativas, problema/solución, social proof, FAQs.

### Estilo visual
Moderno, minimalista, estilo USA, clean, mobile first, alto contraste, CTAs claros, branding premium accesible.

---

## Non-Obvious Gotchas

- **Shadow DOM**: Many components render into shadow DOM; CSS from `base.css` does not pierce shadow boundaries. Component-scoped styles are inlined via JS using CSS custom property overrides on the host element.
- **No build pipeline**: Editing a file in `assets/` is the final artifact — there is nothing to compile or bundle before pushing.
- **`content_for_blocks`**: This Shopify-proprietary Liquid tag is not documented in standard Liquid references; refer to Shopify theme docs.
- **Mutation observer on refs**: Component refs update automatically when Shopify's theme editor injects/removes DOM nodes — do not cache refs across renders.
- **View Transitions API**: Navigation and cart updates use the View Transitions API where available; fallbacks exist for unsupported browsers via helpers in `utilities.js`.
