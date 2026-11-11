---
name: shain-dls
description: >
  Token-based design system skill for Figma and code handoff. Use whenever working with
  Figma components, design tokens, color variables, token migration, component creation,
  design-to-code handoff, or DS audits. Trigger on: design system, DS, DLS, tokens,
  figma variables, component spec, token naming, token hierarchy, which color to use,
  variant properties, theme config, dark mode. Even for "which color should I use" or
  "create a component" — use this skill.
---

# Shain Design Language System (DLS)

> **Note.** Token values and palette names in this document are examples from one
> project. The methodology — the three-tier hierarchy, naming rules, variant
> conventions and audit steps — is what transfers. In a new codebase, always use
> that project's own values and conventions.

A token-based design system for product UI work. Components use semantic tokens only —
no hardcoded colors, spacing, or typography. Figma is the source of truth; code is
generated or guided by Figma exports.

For the full Figma best practices reference, read `references/figma-best-practices.md`.
For guidance on evolving the token system, read `references/token-improvements.md`.

---

## 1. Architecture

### Three-tier token hierarchy

    Primitive → Semantic → Component-specific (optional)

    colors/blue/500          → #0097DB
    action/primary/default   → {colors/blue/500}
    button/bg/primary        → {action/primary/default}

- **Primitive** — raw palette values (`colors/blue/500`, `spacing/4`, `radius/md`).
Never referenced directly from components.
- **Semantic** — purpose-named aliases (`action/primary/default`, `text/primary`,
`bg/surface`). Components reference these.
- **Component-specific** — optional per-component tokens when a component needs its
own tuning knob (`button/bg/primary`). Use sparingly.

### Sync flow

    Figma Variables → figma-tokens.json → tokens/build.ts → tokens/dist/
                                                             ├── tokens.css
                                                             └── tokens.ts

### Project structure

    ├── docs/           Documentation
    ├── tokens/         figma-tokens.json → CSS vars + TS
    └── src/
        ├── components/ React components
        └── icons/      Figma → lucide-react catalog

---

## 2. Design principles

1. **Token-first** — All visual values come from design tokens. No hardcoded colors,
spacing, or typography in components.
2. **Semantic over raw** — Semantic token names describe purpose, not appearance.
Components reference `destructive/default`, not `red/500`.
3. **Figma as source of truth** — Tokens and component specs live in Figma. Code is
generated or guided by Figma exports.
4. **Mode-ready** — All semantic tokens resolve through modes (Light/Dark, density,
brand) so a single component definition adapts to any theme.

---

## 3. Token naming

### Pattern

    {category}/{purpose}/{state}

- **category** — `action`, `destructive`, `success`, `warning`, `info`, `text`, `bg`,
`border`, `icon`, `overlay`
- **purpose** — `primary`, `secondary`, `subtle`, `inverse`, `link`, `disabled`
- **state** — `default`, `hover`, `pressed`, `focus`, `disabled`, `selected`

### Examples

| Figma variable | CSS custom property |
| --- | --- |
| `action/primary/default` | `--action-primary-default` |
| `action/primary/hover` | `--action-primary-hover` |
| `destructive/default` | `--destructive-default` |
| `text/primary` | `--text-primary` |
| `bg/surface` | `--bg-surface` |
| `border/default` | `--border-default` |

### Rules

- Use lowercase, slashes in Figma, kebab-case as CSS vars.
- Do not encode appearance in the name (`red-hover` is wrong; `destructive-hover` is right).
- Stateful tokens always live at the third level (`.../hover`, `.../pressed`).
- A token that exists on one category should exist on comparable categories
(if `action/primary/hover` exists, so should `destructive/hover`).

---

## 4. Modes

### Built-in modes

- **Theme:** Light (default), Dark
- Optional: **Density** (Comfortable, Compact), **Brand** (if multi-brand)

### Rules

- Every semantic token defines a value for every mode. No mode may be empty.
- Components never branch on theme — they reference semantic tokens only.
- Primitive collections do not need modes; modes live on the semantic layer.

### Required dark-mode pairs (minimum)

| Semantic | Light | Dark |
| --- | --- | --- |
| `bg/canvas` | `#FFFFFF` | `#0B0D12` |
| `bg/surface` | `#F7F8FA` | `#141821` |
| `text/primary` | `#0F1420` | `#F2F4F8` |
| `text/secondary` | `#4A5261` | `#A7B0BF` |
| `border/default` | `#E4E7EC` | `#2A2F3A` |
| `action/primary/default` | `#0097DB` | `#3AB7F2` |

---

## 5. Component guidelines

### File layout

    src/components/[ComponentName]/
    ├── [ComponentName].tsx
    ├── [ComponentName].css
    └── index.ts

### CSS class naming (BEM-like)

- **Block:** `shain-[component]` (e.g. `shain-btn`, `shain-input`)
- **Element:** `shain-[component]__[element]` (e.g. `shain-btn__label`)
- **Modifier:** `shain-[component]--[modifier]` (e.g. `shain-btn--primary`, `shain-btn--sm`)

Multiple modifiers combine: `shain-btn shain-btn--primary shain-btn--md shain-btn--destructive`.

### Token usage in code

| Do | Don't |
| --- | --- |
| `background-color: var(--action-primary-default)` | `background-color: #0097DB` |
| `padding: var(--spacing-2) var(--spacing-3)` | `padding: 8px 12px` |
| `border-radius: var(--radius-md)` | `border-radius: 8px` |
| `color: var(--text-primary)` | `color: #0F1420` |

### Theme config pattern (variant styling)

For components with variants and states, use a config object instead of variant-specific CSS:

    enum ButtonVariant {
      Primary = "primary",
      Secondary = "secondary",
    }

    const theme = {
      [ButtonVariant.Primary]: {
        base:     ["--action-primary-default",  "--action-primary-default",  "--text-inverse"],
        hover:    ["--action-primary-hover",    "--action-primary-hover",    "--text-inverse"],
        pressed:  ["--action-primary-pressed",  "--action-primary-pressed",  "--text-inverse"],
        disabled: ["--action-disabled",         "--border-default",          "--text-disabled"],
      },
      [ButtonVariant.Secondary]: {
        base:     ["--bg-surface",              "--border-default",          "--action-primary-default"],
        hover:    ["--bg-surface-hover",        "--border-default",          "--action-primary-hover"],
        pressed:  ["--bg-surface-pressed",      "--border-default",          "--action-primary-pressed"],
        disabled: ["--bg-surface",              "--border-default",          "--text-disabled"],
      },
    };
    // Apply via CSS variables: --cmp-bg, --cmp-border, --cmp-color

### Component anatomy

**Interactive components** support: variants (primary, secondary, outline, ghost, link),
sizes (sm, md, lg, icon), states (default, hover, focus, pressed, disabled, loading).

**Non-interactive components** (e.g. Badge): no hover states or transitions, `cursor: default`.

### Variant properties in Figma

Keep variant properties to **2–3 maximum**. In exceptional cases 4 is acceptable with
justification. Use boolean properties for optional elements (icons, badges) and instance
swap for swappable elements.

    Good — 9 variants + flexible toggles:
      Variant: Size (sm, md, lg) × Type (primary, secondary, outline)
      Boolean: Show Leading Icon, Show Trailing Icon
      Instance swap: Icon

    Bad — 864 variants:
      Size (4) × Color (6) × Icon Position (3) × State (4) × Alignment (3)

### Props convention

- TypeScript interfaces with explicit types
- Support `className` and spread remaining HTML attributes
- `forwardRef` for form elements and focusable components
- Default values match the most common use case

### Icons

- Use [lucide-react](https://lucide.dev)
- Accept `LucideIcon` type for icon props
- Icon size: 16px inline with text; use tokens if a size scale exists
- Catalog: `src/icons/iconCatalog.ts` maps Figma names → Lucide components

---

## 6. Accessibility

1. **Focus** — `:focus-visible` for keyboard focus; avoid focus ring on mouse click.
2. **Disabled** — `disabled` on interactive elements; `aria-disabled` only as fallback.
3. **Loading** — `aria-busy="true"` when appropriate.
4. **Labels** — All form controls have associated labels (visible or `aria-label`).
5. **Keyboard** — Tab, Enter, Space, Escape for dropdowns/modals.
6. **Touch targets** — Minimum 44×44px.
7. **Contrast** — 4.5:1 for text, 3:1 for UI elements, verified in both Light and Dark.

---

## 7. Typography

- **Font family:** sans-serif (configurable per brand)
- **Weights:** 400 (normal), 600 (medium), 700 (bold)
- **Sizes:** `text/xs` (12px) through `text/3xl` (32px)
- **Line heights:** `leading/1` (16px) through `leading/6` (40px)

---

## 8. Spacing & Radius

**Spacing scale:**

| Token | Value |
| --- | --- |
| `spacing/0` | 0px |
| `spacing/0.5` | 2px |
| `spacing/1` | 4px |
| `spacing/2` | 8px |
| `spacing/3` | 12px |
| `spacing/4` | 16px |
| `spacing/5` | 20px |
| `spacing/6` | 24px |

**Radius scale:**

| Token | Value |
| --- | --- |
| `radius/none` | 0px |
| `radius/xs` | 2px |
| `radius/sm` | 4px |
| `radius/md` | 8px |
| `radius/lg` | 12px |
| `radius/xl` | 24px |
| `radius/full` | 9999px |

---

## 9. Anti-patterns

1. **Absolute positioning instead of auto layout** — Always use auto layout with spacing tokens.
2. **Too many variants** — 100+ variants means booleans and instance swaps are underused.
3. **Hardcoded values** — Any raw hex, px value, or magic number should be a token reference.
4. **Appearance-based names** — `red-hover`, `blue-bg` leak visuals into purpose names.
5. **Components referencing primitives** — Primitives are internal; components reference semantic tokens.
6. **Unnamed layers** — "Frame 123" is unacceptable. Name every layer by purpose.
7. **No component docs** — Every component needs a doc from the template.
8. **Mixing presentation and content** — Separate structural layers from content layers.
9. **Componentizing everything** — Only reusable, consistent, interactive, or coded elements.
10. **Ignoring accessibility in Figma** — 44×44px targets, contrast ratios, focus states.
11. **Deep nesting** — Maximum 3 levels (e.g., Card → Button → Icon).
12. **Detached instances** — Always use main component references inside other components.

---

## 10. Checklists

### Figma-side (before handoff)

**Structure:** auto layout, descriptive layer names, ≤3 nesting levels, main component refs,
proper constraints.

**Properties:** 2–3 variant props, booleans for optional elements, text props for content,
instance swap for icons, sensible defaults.

**Tokens:** all fills use semantic variables (no hex), spacing variables, text styles, effect
styles, no magic numbers. No primitive references from component layers.

**States:** default, hover, pressed/active, disabled, focus (keyboard).

**Testing:** works at different sizes, short/long content, no overflow,
Light and Dark mode both pass contrast checks.

### Code-side (new components)

- All colors: `var(--semantic-token-name)` — never primitives
- Variant colors via theme config
- All spacing: `var(--spacing-N)`
- All typography: `var(--text-*)` / `var(--leading-*)`
- Border radius: `var(--radius-*)`
- CSS classes: `shain-[component]` naming
- TypeScript typed props
- `forwardRef` for focusable elements
- `:focus-visible` styles
- Component doc from template

---

## 11. AI workflow for component generation

When generating components from Figma:

1. **Read component spec** — Variants, sizes, states from Figma.
2. **Map to semantic tokens** — Every fill, stroke, text, padding, radius, gap maps to a
semantic token (never a primitive).
3. **Use the theme config pattern** for variants with multiple states.
4. **Follow component guidelines** — BEM naming (`shain-*`), forwardRef, accessibility.
5. **Verify both modes** — Component must pass contrast in Light and Dark.
6. **Add changelog entry** to the component doc.

---

## Reference files

Read these when you need deeper context:

| File | When to read |
| --- | --- |
| `references/figma-best-practices.md` | Building/reviewing Figma components, design tokens architecture, platform-specific guidelines (iOS/Android/Web), export settings |
| `references/token-improvements.md` | Planning token system changes, adding new tokens, restructuring collections, brand expansion |
