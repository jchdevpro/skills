# Token map

Read the SCSS files listed here when you need exact names. This file is a map, not a dump.

## Primitive files (`variables/`)

| File | Tokens | Theme-dependent? |
| --- | --- | --- |
| `_colors.scss` | `--Primary` / `*-Alt`, `--color-{hue}-{step}` | No |
| `_padding.scss` | `--padding-{step}`, `--padding-None` | No — **add if missing** |
| `_margin.scss` | `--margin-{step}`, `--margin-None` | No — **add if missing** |
| `_gap.scss` | `--gap-{step}`, `--gap-None` | No — **add if missing** |
| `_border.scss` | `--border-width-Default`, `Emphasise`, `Focus`, `Callout` | No |
| `_border-radius.scss` | `--radius-xs` … `--radius-4xl`, `--radius-full` | No |
| `_typography.scss` | `--Font-Family-*`, `--Font-Size-*`, `--Font-Weight-*` | No |
| `_shadow.scss` | `--shadow-2xs` … `--shadow-2xl` | No |
| `_breakpoints.scss` | `--Desktop-Breakpoint-In-Pixels`, Tablet, Mobile | No |
| `_space.scss` | `--space-*` | Legacy. Do not extend. |

Hue steps in `_colors.scss` are typically `50`–`950` plus extras (`indigo-25`, `indigo-975`, `white/black` alpha ramps). Brand aliases sit at the top of that file.

## Semantic color families (theme mixins)

Declared only inside `@mixin theme-light` / `@mixin theme-dark`.

Prefix → typical roles:

- `--Background-{Brand|Default|Neutral|Disabled|Danger|Warning|Positive|Info|Utilities}-{Default|Hover|Secondary|Tertiary|Quaternary|…}`
- `--Text-{…}-{Default|Secondary|Tertiary|On-Default|On-Secondary|On-Tertiary|On-Disabled}`
- `--Border-{…}-{Default|Secondary|Tertiary}` plus `Resizer` / `Utilities`
- `--Icon-{…}` — usually mirrors the matching `--Text-*` token
- `--Input-Bg`, `--Input-Bg-Disabled`, `--Input-Bg-checkbox-radio`, `--Input-Border`, `--Input-Border-Active`, `--Input-Border-Hover`
- `--Interaction-Focus`, `--Interaction-Link-Primary`, `--Interaction-Link-Secondary`, `--Interaction-Selected-bg`, `--Interaction-Selected-bg-secondary`, `--Interaction-Selected-border`

When generating from Figma, convert path segments to Pascal-Case and join with `-`:

| Figma | CSS |
| --- | --- |
| `background/default/secondary` | `--Background-Default-Secondary` |
| `text/brand/on-default` | `--Text-Brand-On-Default` |
| `border/danger/default` | `--Border-Danger-Default` |
| `icon/disabled/on-disabled` | `--Icon-Disabled-On-Disabled` |
| `space/300` used as padding | `--padding-300` |
| `space/300` used as margin | `--margin-300` |
| `space/300` used as gap | `--gap-300` |
| `radius/sm` | `--radius-sm` |
| `font/size/text-s` | `--Font-Size-Text-S` |
| `shadow/xl` | `--shadow-xl` |

## Numeric scale

`{step}` is hundredths of a rem-quarter: `400` = `1rem` = `16px` at default root.

`rem = step / 400`. Copy unused steps from `_space.scss` into padding/margin/gap when a Figma value needs them.

Common steps: `25 50 75 100 150 200 250 300 400 500 600 800 1000 1200 1600`. Zero: `None`.

## Runtime

- Theme runtime observes `data-theme` and `prefers-color-scheme`.
- Storybook should set `document.body` `data-theme` from its preview config.
- Overlays that must match a surface set `[attr.data-theme]` on the host.

## Viewport mixins

`mixins/_viewports.scss`: `@include desktop()` ≥ `768px`, `@include mobile()` ≤ `768px`. Prefer these over ad-hoc media queries.
