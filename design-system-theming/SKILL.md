---
name: design-system-theming
description: >-
  Sets up and extends design-system theming from a styles package (primitive
  tokens, light/dark mixins, data-theme). Prefer explicit --padding-*,
  --margin-*, --border-width-*, and --gap-* over --space-*.
---

# Design system theming

Load this skill before creating, regenerating, or extending themes. Read the live styles files in the current workspace; do not invent token names.

## Architecture

Three layers. Keep them separate.

| Layer | Where | What |
| --- | --- | --- |
| Primitives | `variables/_*.scss` on `:root` | Raw scales. Same in every theme. |
| Semantic theme | `themes/_light.scss`, `themes/_dark.scss` mixins | Color roles that swap per theme. |
| Wiring | `themes/_index.scss` | Applies mixins to `:root`, `prefers-color-scheme`, `[data-theme]`. |
| Consumption | `components/_*.scss` and component SCSS | Only `var(--Token)`. No hex/rgb/px for themeable values. |

`style.scss` `@forward`s variables, then themes, then component partials. Consuming apps load that file via the project's style include path.

```text
styles/
├── style.scss                 # public entry
├── storybook.scss             # Storybook extras only
├── variables/                 # primitives
├── themes/
│   ├── _index.scss
│   ├── _light.scss            # @mixin theme-light
│   └── _dark.scss             # @mixin theme-dark
├── mixins/_viewports.scss
└── components/                # shared BEM partials, not theme definitions
```

## Explicit spacing (required)

Do **not** use `--space-*` for padding, margin, border, or gap. Generic space tokens hide intent.

| CSS property | Token family | Example |
| --- | --- | --- |
| `padding`, `padding-block`, `padding-inline`, `padding-*-start/end` | `--padding-{step}` | `padding-inline: var(--padding-300)` |
| `margin`, `margin-block`, `margin-inline`, `margin-*-start/end` | `--margin-{step}` | `margin-block: var(--margin-200)` |
| `border-width` / logical border width | `--border-width-{role}` | `border-width: var(--border-width-Default)` |
| `gap`, `row-gap`, `column-gap` | `--gap-{step}` | `gap: var(--gap-200)` |
| `border-radius` | `--radius-{size}` | `border-radius: var(--radius-sm)` |
| `border-color` | `--Border-{Family}-{Role}` | `border-color: var(--Border-Default-Default)` |

Existing `--space-*` is legacy. When setting up or generating theming:

1. Add primitive files if missing: `variables/_padding.scss`, `variables/_margin.scss`, `variables/_gap.scss`.
2. `@forward` them from `style.scss` next to the other variables.
3. Reuse the same numeric scale as `_space.scss` (`100` = `0.25rem`, `400` = `1rem`).
4. Do not add new `--space-*` steps or new `--space-*` usages in generated theme/setup work.
5. Zero is `--padding-None` / `--margin-None` / `--gap-None` (`0px`), not `--space-None`.

Scale (copy the steps you need; full table is in `_space.scss`):

```scss
:root {
  --padding-25: 0.0625rem;
  --padding-50: 0.125rem;
  --padding-100: 0.25rem;
  --padding-150: 0.375rem;
  --padding-200: 0.5rem;
  --padding-300: 0.75rem;
  --padding-400: 1rem;
  --padding-600: 1.5rem;
  --padding-800: 2rem;
  --padding-None: 0px;
}
```

Mirror the same steps for `--margin-*` and `--gap-*`. Do not alias them to `--space-*`.

Border width roles live in `variables/_border.scss`:

- `--border-width-Default` / `--border-width-Emphasise` — `1px`
- `--border-width-Focus` — `2px`
- `--border-width-Callout` — `4px`

Do not invent `--border-width-1` or raw `1px` borders.

## Generate themes

Semantic **color** tokens belong in the theme mixins, not on `:root`.

1. Read `themes/_light.scss` and `themes/_dark.scss` first. Those files are the contract: every token in light must exist in dark with the same name.
2. Map Figma / source variables to the existing families. Do not rename:

   - `--Background-{Family}-{Role}` (optional `-Hover`)
   - `--Text-{Family}-{Role}` (including `On-Default`, `On-Secondary`)
   - `--Border-{Family}-{Role}`
   - `--Icon-{Family}-{Role}`
   - `--Input-*`
   - `--Interaction-*`

   Families: `Default`, `Brand`, `Neutral`, `Disabled`, `Danger`, `Warning`, `Positive`, `Info`, `Utilities`, plus `Resizer` on borders.

3. Point semantic tokens at primitives (`var(--color-zinc-950)`, `var(--Primary)`), not at hardcoded hex. Brand aliases (`--Primary`, `--Primary-Alt`, `--Secondary`, `--Tertiary` and `*-Alt`) live in `variables/_colors.scss`.
4. Put new primitive swatches in `_colors.scss` only when no existing `--color-{hue}-{step}` fits. Keep `rgb(r g b)` / `rgba(r, g, b, n%)` as in that file.
5. Keep mixins as flat custom-property lists. No selectors inside `theme-light` / `theme-dark`.
6. After regenerating, diff light vs dark token **names**. Add any missing name to the other mixin. Delete orphans that no component reads only when the user asked for a cleanup.

`themes/_index.scss` must stay this shape:

```scss
:root { @include theme-light; }

@media (prefers-color-scheme: light) {
  :root { @include theme-light; color-scheme: light; }
}

@media (prefers-color-scheme: dark) {
  :root { @include theme-dark; color-scheme: dark; }
}

*[data-theme='dark'] { @include theme-dark; color-scheme: dark; }
*[data-theme='light'] { @include theme-light; color-scheme: light; }

html, body {
  color: var(--Text-Default-Default);
  background-color: var(--Background-Default-Default);
}
```

`[data-theme]` wins over `prefers-color-scheme` because it is more specific. Runtime toggle sets `data-theme` on the themed element. Theme names are `'light' | 'dark'` unless a new scheme is added.

## New theme or new consumer

**New color scheme (e.g. high-contrast):**

1. Add `themes/_<name>.scss` with `@mixin theme-<name>` declaring the **full** semantic set.
2. `@use` it from `_index.scss` and add `*[data-theme='<name>']`.
3. Extend any theme-name union or `['dark', 'light']` filters in the theme runtime.
4. Do not fork primitive files per theme.

**New consumer:**

1. Point the app's style include path at `styles/style.scss`.
2. Do not copy theme mixins into the app.
3. Scoped overrides: set tokens on a local `[data-theme]` or host, still using existing names.

## Consuming tokens

- Logical properties only (`padding-inline`, `margin-block`, `border-inline-end`).
- Colors: semantic tokens (`--Background-*`, `--Text-*`, `--Border-*`, `--Icon-*`). Never `--color-*` in component SCSS unless you are defining a semantic token.
- Padding / margin / gap: explicit families above. Never `--space-*`.
- Focus ring: `box-shadow: inset 0 0 0 var(--border-width-Focus) var(--Interaction-Focus);`
- Figma `space/200` → `--padding-200` / `--margin-200` / `--gap-200` by CSS property, not `--space-200`.
- Figma `background/default/tertiary` → `--Background-Default-Tertiary` (slash path, Pascal-Case token).

## Do not

- Put semantic colors on `:root` in `variables/`.
- Put padding/margin/gap/radius inside theme mixins (they are not theme-dependent).
- Hardcode `#`, `rgb()`, or raw `px` for values that have tokens.
- Generate a third copy of the color scale inside a component.
- Use physical `padding-left` / `margin-top` / `border-left`.

## Checklist

- [ ] Light and dark mixins have the same semantic token names
- [ ] New primitives live in `variables/` and are `@forward`ed from `style.scss`
- [ ] Padding/margin/border/gap use explicit families, not `--space-*`
- [ ] Consumers still load `styles/style.scss`
- [ ] Theme-name union / `[data-theme]` updated if a new scheme was added

## More detail

- Token families and Figma mapping: [token-map.md](token-map.md)
