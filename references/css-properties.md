# TeamDesk CSS Custom Properties Reference (New UI v3)

All properties use the `--v3-` prefix and can be overridden in a CSS file added to Database Resources.

## Typography

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-font-face` | Primary font name | `"IBM Plex Sans"` |
| `--v3-monospace-font-face` | Monospace font | `"IBM Plex Mono"` |
| `--v3-font-family` | Primary + fallbacks | IBM Plex Sans + system |
| `--v3-monospace-font-family` | Mono + fallbacks | IBM Plex Mono + system |
| `--v3-font-pref-normal` | Base font size | `14px` |
| `--v3-font-pref-small` | Small font size | `12px` |
| `--v3-font-pref-large` | Large font size | `16px` |
| `--v3-h1-font-size` | Heading 1 | `2.25rem` |
| `--v3-h2-font-size` | Heading 2 | `2rem` |
| `--v3-h3-font-size` | Heading 3 | `1.75rem` |
| `--v3-h4-font-size` | Heading 4 | `1.5rem` |
| `--v3-h5-font-size` | Heading 5 | `1.25rem` |
| `--v3-h6-font-size` | Heading 6 | `1rem` |
| `--v3-small-font-size` | Small text | `0.85rem` |
| `--v3-input-font-size` | Input controls | `inherit` |
| `--v3-button-font-size` | Buttons | `inherit` |

## Color System (HSL-based)

All colors use H/S/L triplets for maximum flexibility.

### Background Colors

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-primary-back-color-h/s/l` | Main background | `0, 0%, 100%` (white) |
| `--v3-primary-back-color` | Calculated HSL | `hsl(h, s, l)` |
| `--v3-secondary-back-color-h/s/l` | Alt background | ±5% from primary |
| `--v3-secondary-back-color` | Calculated HSL | auto |

### Text Colors

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-primary-text-color-h/s/l` | Main text | `0, 0%, 15%` (dark) |
| `--v3-primary-text-color` | Calculated | auto |
| `--v3-secondary-text-color-h/s/l` | Muted text | ±20% from primary |
| `--v3-secondary-text-color` | Calculated | auto |

### Named Palette Colors

Available colors: `blue`, `red`, `orange`, `yellow`, `green`, `teal`, `navy`, `violet`, `purple`, `pink`, `black`, `white`, `gray`

Each has:
- `--v3-palette-{name}-color-h` (hue)
- `--v3-palette-{name}-color-s` (saturation)
- `--v3-palette-{name}-color-l` (lightness)
- `--v3-palette-{name}-color` (calculated HSL)

### Table Color

| Variable | Purpose |
|----------|---------|
| `--v3-current-table-hint-color-h/s/l` | Dynamic per-table color |
| `--v3-current-table-hint-color` | Calculated |

## Appearance

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-transition-time` | Animation duration | `300ms` |
| `--v3-border-radius` | Corner rounding | `6px` |
| `--v3-border-color` | Default border | auto |
| `--v3-link-text-color` | Link normal | `currentColor` |
| `--v3-link-text-color--hover` | Link hover | `currentColor` |
| `--v3-accent-color-h/s/l` | Focus/primary button | Palette blue |
| `--v3-focus-outline-color` | Focus ring | Accent 50% alpha |
| `--v3-focus-outline` | Focus box-shadow | `0 0 0 2px` accent |
| `--v3-body-back-color` | Page background | Primary bg |
| `--v3-body-text-color` | Page text | Primary text |
| `--v3-sidebar-width` | Nav panel width | `256px` |

## Input Controls

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-input-border-color` | Border color | `--v3-border-color` |
| `--v3-input-border-width` | Border thickness | `1px` |
| `--v3-input-border-radius` | Corner radius | `--v3-border-radius` |
| `--v3-input-text-color` | Input text | Primary text |
| `--v3-input-back-color` | Input background | Primary bg |
| `--v3-input-text-color--disabled` | Disabled text | Secondary text |
| `--v3-input-back-color--disabled` | Disabled bg | Secondary bg |
| `--v3-input-required-color` | Required indicator | Palette red |
| `--v3-input-invalid-color` | Invalid state | Required color |
| `--v3-input-text-color--placeholder` | Placeholder | Secondary text |

## Buttons

| Variable | Purpose | Default |
|----------|---------|---------|
| `--v3-button-border-color` | Border | `--v3-border-color` |
| `--v3-button-border-width` | Thickness | `0px` (borderless) |
| `--v3-button-border-radius` | Corners | `--v3-border-radius` |
| `--v3-button-back-color-h/s/l` | Standard button bg | Secondary bg |
| `--v3-button-back-color` | Calculated | auto |
| `--v3-button-default-back-color-h/s/l` | Primary button bg | Accent color |
| `--v3-button-default-back-color` | Calculated | auto |
| `--v3-button-text-color--disabled` | Disabled text | Secondary text |
| `--v3-button-back-color--disabled` | Disabled bg | Secondary bg |

> Button text color is auto-calculated (black/white) based on background lightness for contrast.

## Example: ForGreen Theme

```css
:root {
    /* Verde ForGreen como cor de destaque */
    --v3-accent-color-h: 142;
    --v3-accent-color-s: 60%;
    --v3-accent-color-l: 35%;

    /* Cantos mais arredondados */
    --v3-border-radius: 8px;

    /* Sidebar mais estreita */
    --v3-sidebar-width: 220px;
}
```

## Theme Support

TeamDesk supports Light, Dark, and OS Default themes via Quick Settings (footer).
Dark theme automatically inverts lightness values of all HSL-based properties.
