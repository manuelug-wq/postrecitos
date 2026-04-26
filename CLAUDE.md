# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Postrecitos** is a single-page web app (SPA) for dessert recipe cost calculation, targeting small business operators. It has zero build dependencies — the entire application lives in `index.html` (HTML + CSS + JS).

## Running the App

There is no build step. Serve `index.html` directly:

```bash
# Local dev
python -m http.server 8000

# Docker (production-like, serves on port 80)
docker build -t postrecitos .
docker run -p 8080:80 postrecitos
```

There are no tests, no linter, and no package manager. No `npm install` is needed.

## Architecture

The entire application is a single `index.html` file organized in three sequential sections:

1. **CSS** (`<style>` block): full design system using CSS custom properties
2. **HTML**: DOM skeletons for every screen and modal — no JS-generated markup
3. **JavaScript** (`<script>` block at the bottom): all app logic

### Screen / State Flow

```
PIN screen → Product selector → Main app (tab-based)
```

- **PIN screen** (`.pin-screen`): numeric keypad, hardcoded PIN `5050`
- **Product selector** (`.prod-screen`): list of saved recipes; create/delete products
- **Main app** (`.app-scroll`): 5-tab navigation driven by showing/hiding `#tab-*` divs

Tabs: `#tab-home`, `#tab-calcular`, `#tab-compras`, `#tab-receta`, `#tab-historial`, `#tab-config`

Screens are shown/hidden by toggling `.active` on their root element or via `display` style. Modals use an `.open` class on `.modal-overlay`.

### Data Layer

All persistence is `localStorage`. Keys:

| Key | Contents |
|-----|----------|
| `ptc_productos` | JSON array of product objects |
| `ptc_hist` | JSON array of history entries (max 10) |
| `ptc_activo` | ID string of the currently selected product |

**Product shape:**
```js
{
  id: string,          // e.g. 'tiramisu' or 'prod_1234567890'
  nombre: string,
  emoji: string,
  base: number,        // base portion count
  ings: [{
    nombre: string,
    u: string,         // unit: 'g','kg','ml','L','unidades','tazas','cdas'
    cantBase: number,  // quantity for `base` portions
    costoU: number,    // cost per unit
    paqC: number,      // units per package
    paqN: string,      // package label
    paqP: number       // package price
  }]
}
```

**History entry shape:**
```js
{ fecha, producto, porciones, costo, pv, ingreso, util }
```

Global JS state variables: `pinVal`, `productos`, `historial`, `prodActivo` (ID), `editIngIdx`.

### Calculation Logic

- **Scale factor:** `factor = requestedPortions / product.base`
- **Ingredient cost:** `scaledQty = ing.cantBase * factor`, `cost = scaledQty * ing.costoU`
- **Shopping list:** `packages = ceil(scaledQty / ing.paqC)`, `cost = packages * ing.paqP`
- **Profit:** `util = ingreso - costo` (revenue entered by user minus calculated cost)

## Naming Conventions

CSS classes and HTML IDs use kebab-case. JS uses camelCase. Several DOM ID prefixes are non-obvious abbreviations:

| Prefix | Meaning |
|--------|---------|
| `kp-` | Keypad (PIN / portion input) |
| `cfg-` | Config tab fields |
| `ing-` | Ingredient modal fields |
| `porc` | Porciones (portions) |
| `pv-` | Precio de venta (sale price) |

## Design System

CSS variables (defined on `:root`):

- Colors: `--espresso`, `--caramel`, `--gold`, `--cream`, `--bg`, `--surface`
- `--radius: 18px` used consistently for cards and buttons
- Fonts: `Cormorant Garamond` (headings) + `Outfit` (body), loaded from Google Fonts
- Layout constrained to `max-width: 430px`, mobile-first

## Default Data

A built-in "Tiramisú Clásico" recipe (15 base portions, 11 ingredients) is injected when `ptc_productos` is absent from localStorage. It serves as the reference example for ingredient structure.
