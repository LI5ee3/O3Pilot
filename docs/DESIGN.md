---
version: alpha
name: O3Pilot-design-system
description: A quiet, data-first ecommerce intelligence workspace with restrained neutral surfaces, precise MiSans typography,
  efficient analytical density, a single Action Blue interaction accent, border-first depth, purpose-driven motion, and read-only-to-Ozon
  interaction cues.
colors:
  primary: '#0066CC'
  primary-hover: '#0071E3'
  primary-active: '#0055B3'
  primary-on-dark: '#2997FF'
  focus: '#0071E3'
  on-primary: '#FFFFFF'
  ink: '#1D1D1F'
  body: '#1D1D1F'
  muted: '#6E6E73'
  muted-soft: '#86868B'
  canvas: '#FFFFFF'
  canvas-secondary: '#F5F5F7'
  surface: '#FFFFFF'
  surface-subtle: '#FAFAFC'
  hairline: rgba(0,0,0,0.08)
  divider-soft: rgba(0,0,0,0.05)
  control-hover: rgba(0,0,0,0.04)
  selected-surface: rgba(0,102,204,0.08)
  disabled-text: rgba(29,29,31,0.38)
  disabled-surface: rgba(0,0,0,0.04)
  overlay: rgba(0,0,0,0.28)
  dark-ink: '#F5F5F7'
  dark-body: '#F5F5F7'
  dark-muted: '#A1A1A6'
  dark-muted-soft: '#8E8E93'
  dark-canvas: '#000000'
  dark-canvas-secondary: '#1D1D1F'
  dark-surface: '#1C1C1E'
  dark-surface-subtle: '#242426'
  dark-hairline: rgba(255,255,255,0.12)
  dark-divider-soft: rgba(255,255,255,0.08)
  dark-control-hover: rgba(255,255,255,0.06)
  dark-selected-surface: rgba(41,151,255,0.14)
  dark-disabled-text: rgba(245,245,247,0.38)
  dark-disabled-surface: rgba(255,255,255,0.06)
  dark-overlay: rgba(0,0,0,0.48)
  success: '#1F7A3D'
  success-subtle: rgba(31,122,61,0.10)
  success-border: rgba(31,122,61,0.28)
  warning: '#946200'
  warning-subtle: rgba(148,98,0,0.10)
  warning-border: rgba(148,98,0,0.28)
  danger: '#B42318'
  danger-subtle: rgba(180,35,24,0.10)
  danger-border: rgba(180,35,24,0.28)
  information: '#0066CC'
  information-subtle: rgba(0,102,204,0.08)
  information-border: rgba(0,102,204,0.24)
  neutral: '#6E6E73'
  neutral-subtle: rgba(110,110,115,0.08)
  neutral-border: rgba(110,110,115,0.22)
  success-dark: '#6CCB82'
  success-subtle-dark: rgba(108,203,130,0.14)
  success-border-dark: rgba(108,203,130,0.34)
  warning-dark: '#FFD166'
  warning-subtle-dark: rgba(255,209,102,0.14)
  warning-border-dark: rgba(255,209,102,0.34)
  danger-dark: '#FF7B72'
  danger-subtle-dark: rgba(255,123,114,0.14)
  danger-border-dark: rgba(255,123,114,0.34)
  information-dark: '#5AC8FA'
  information-subtle-dark: rgba(90,200,250,0.14)
  information-border-dark: rgba(90,200,250,0.34)
  neutral-dark: '#A1A1A6'
  neutral-subtle-dark: rgba(161,161,166,0.12)
  neutral-border-dark: rgba(161,161,166,0.28)
typography:
  page-title:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 28px
    fontWeight: 600
    lineHeight: 1.25
    letterSpacing: 0
  section-title:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 22px
    fontWeight: 600
    lineHeight: 1.3
    letterSpacing: 0
  panel-title:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 17px
    fontWeight: 600
    lineHeight: 1.35
    letterSpacing: 0
  body:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 15px
    fontWeight: 400
    lineHeight: 1.5
    letterSpacing: 0
  body-medium:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 15px
    fontWeight: 500
    lineHeight: 1.45
    letterSpacing: 0
  body-sm:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 13px
    fontWeight: 400
    lineHeight: 1.45
    letterSpacing: 0
  caption:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 12px
    fontWeight: 400
    lineHeight: 1.4
    letterSpacing: 0
  metric-xl:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 32px
    fontWeight: 600
    lineHeight: 1.15
    letterSpacing: 0
  metric-lg:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 24px
    fontWeight: 600
    lineHeight: 1.2
    letterSpacing: 0
  metric-md:
    fontFamily: "'MiSans Global', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    fontSize: 17px
    fontWeight: 600
    lineHeight: 1.25
    letterSpacing: 0
rounded:
  xs: 6px
  sm: 8px
  md: 10px
  lg: 12px
  xl: 16px
  pill: 999px
spacing:
  xxs: 4px
  xs: 8px
  sm: 12px
  base: 16px
  md: 20px
  lg: 24px
  xl: 32px
  xxl: 40px
  xxxl: 48px
  section: 64px
components:
  sidebar:
    backgroundColor: '{colors.canvas-secondary}'
    textColor: '{colors.ink}'
    width: 248px
    collapsedWidth: 72px
    narrowWidth: 60px
  sidebar-item:
    backgroundColor: transparent
    textColor: '{colors.muted}'
    typography: '{typography.body-medium}'
    rounded: '{rounded.sm}'
    height: 40px
    iconSize: 20px
  sidebar-item-active:
    backgroundColor: '{colors.selected-surface}'
    textColor: '{colors.ink}'
    typography: '{typography.body-medium}'
    rounded: '{rounded.sm}'
    height: 40px
  button-primary:
    backgroundColor: '{colors.primary}'
    textColor: '{colors.on-primary}'
    typography: '{typography.body-medium}'
    rounded: '{rounded.sm}'
    padding: 8px 16px
    height: 40px
  button-primary-active:
    backgroundColor: '{colors.primary-active}'
    textColor: '{colors.on-primary}'
    rounded: '{rounded.sm}'
  button-secondary:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    typography: '{typography.body-medium}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.sm}'
    padding: 8px 16px
    height: 40px
  button-ghost:
    backgroundColor: transparent
    textColor: '{colors.ink}'
    typography: '{typography.body-medium}'
    rounded: '{rounded.sm}'
    padding: 8px 12px
    height: 40px
  button-icon:
    backgroundColor: transparent
    textColor: '{colors.ink}'
    rounded: '{rounded.sm}'
    size: 40px
    iconSize: 18px
  text-input:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    typography: '{typography.body}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.sm}'
    padding: 8px 12px
    height: 40px
  search-input:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    typography: '{typography.body}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.sm}'
    padding: 8px 12px
    height: 40px
  select-control:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    typography: '{typography.body}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.sm}'
    padding: 8px 12px
    height: 40px
  bounded-surface:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.lg}'
    padding: 24px
  data-table:
    backgroundColor: '{colors.canvas}'
    textColor: '{colors.ink}'
    typography: '{typography.body-sm}'
    headerHeight: 40px
    rowHeight: 44px
    compactRowHeight: 36px
    horizontalPadding: 16px
    compactHorizontalPadding: 12px
  status-badge:
    backgroundColor: '{colors.neutral-subtle}'
    textColor: '{colors.neutral}'
    typography: '{typography.body-sm}'
    rounded: '{rounded.pill}'
    padding: 4px 8px
  tabs:
    backgroundColor: transparent
    textColor: '{colors.muted}'
    activeTextColor: '{colors.ink}'
    indicatorColor: '{colors.primary}'
    typography: '{typography.body-medium}'
    height: 40px
  filter-chip:
    backgroundColor: '{colors.surface-subtle}'
    textColor: '{colors.ink}'
    typography: '{typography.body-sm}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.pill}'
    padding: 4px 8px
  segmented-control:
    backgroundColor: '{colors.canvas-secondary}'
    textColor: '{colors.muted}'
    selectedBackgroundColor: '{colors.surface}'
    selectedTextColor: '{colors.ink}'
    typography: '{typography.body-sm}'
    rounded: '{rounded.md}'
    padding: 4px
  popover:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.lg}'
    padding: 16px
  drawer:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    rounded: '{rounded.xl}'
    width: 480px
    wideWidth: 640px
  dialog:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    rounded: '{rounded.xl}'
    padding: 24px
  toast:
    backgroundColor: '{colors.surface}'
    textColor: '{colors.ink}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.lg}'
    padding: 12px 16px
  skeleton:
    backgroundColor: '{colors.disabled-surface}'
    rounded: '{rounded.sm}'
  state-surface:
    backgroundColor: '{colors.surface-subtle}'
    textColor: '{colors.ink}'
    borderColor: '{colors.hairline}'
    rounded: '{rounded.lg}'
    padding: 24px
---

## Overview

O3Pilot is a **quiet, data-first ecommerce intelligence workspace**. It should feel closer to a focused native productivity application than a generic admin dashboard or a marketing site. The interface uses restrained neutral surfaces, precise MiSans typography, compact but breathable analytical layouts, and one controlled blue interaction accent.

Data is always louder than chrome. Dense comparison work is allowed to be genuinely dense, while explanation and diagnosis get more breathing room. Hierarchy comes from typography and spacing first, subtle surface contrast second, hairlines third, and shadow only when content actually rises above the document layer.

O3Pilot is permanently read-only toward Ozon. The interface may support inspection, explanation, simulation, export, local configuration, local drafts and other O3Pilot-owned state, but those surfaces must never visually imply that a recommendation or local edit has already been applied to Ozon.

**Key Characteristics:**
- Data before chrome; decoration is the lowest-priority layer.
- Neutral white/off-white and near-black surfaces with one interaction blue.
- Efficient density without tiny typography or visual congestion.
- Borders before shadows; ordinary data surfaces remain flat.
- Metrics are aligned information, not mandatory individual cards.
- Tabular numerals and deliberate alignment make comparison easy.
- Lucide is the single functional icon language; Morphicons is transition feedback only.
- Motion is short, purposeful, interruptible, and never triggered merely because data changed.
- Light/dark themes preserve semantic roles instead of mathematically inverting colors.
- Chinese, English, and Russian share one typography system.
- Data honesty is visual: Missing/Unknown/Unavailable/Partial/Stale never masquerade as real `0`, and estimate/fact, source, currency/unit, time basis and freshness stay discoverable when interpretation depends on them.

## Colors

### Interaction

`{colors.primary}` is the main interaction signal. Use it for links, primary actions, selection and focus-related affordances. Do **not** use Action Blue simply because a KPI, title, chart series, or card is important.

- `{colors.primary}` — primary interaction on light surfaces.
- `{colors.primary-hover}` — mouse hover where hover exists.
- `{colors.primary-active}` — pressed/active feedback.
- `{colors.primary-on-dark}` — high-visibility link/focus treatment on dark surfaces; it is not an automatic replacement for every solid primary button.
- `{colors.focus}` — keyboard focus root.

O3Pilot should feel visually calm when nothing requires attention:

```text
Normal state → neutral / quiet
Attention → semantic color when the upstream meaning supports it
```

### Neutral surfaces

Light mode is white-dominant with parchment-like secondary canvas. Dark mode uses true black for the outer canvas and near-black raised surfaces. Dark values are explicit `dark-*` tokens; never derive them by inversion.

Use surface contrast sparingly. A section does not need a card merely because it is a component. Prefer a continuous reading canvas for dense analytical work.

### Semantic visual color

`success`, `warning`, `danger`, `information`, and `neutral` are **visual roles**, not business-state definitions. The owning product/metric/security contract determines what a state means; DESIGN only determines how an already-known semantic role is rendered.

Rules:
- Color is never the only state channel; pair it with text, shape, icon, or another explicit cue.
- `VALID` or an ordinary normal condition must not flood the product with green.
- Up/down direction is not automatically good/bad: `↑ != success`, `↓ != danger`.
- Source/origin labels are not success/error labels.
- Chart series colors are separate from semantic status colors.

## Typography

All UI uses **MiSans Global** with the fallback stack encoded in the YAML typography tokens. The approved weight ladder is strictly **400 / 500 / 600 / 700**; do not synthesize other weights.

Use the named roles directly:
- `{typography.page-title}` for the page-level heading.
- `{typography.section-title}` for major content sections.
- `{typography.panel-title}` for bounded groups and compact panels.
- `{typography.body}` for normal UI copy.
- `{typography.body-medium}` for labels or controls needing modest emphasis.
- `{typography.body-sm}` for dense table/meta text.
- `{typography.caption}` for secondary metadata.
- `{typography.metric-xl}`, `{typography.metric-lg}`, `{typography.metric-md}` for prominent numerical values.

### Numbers

Comparable numbers use `font-variant-numeric: tabular-nums`. Numeric measures, money, ratios and percentages align right in tables. Identifiers align left and remain strings even when they contain only digits.

Never locale-format identifiers or strip leading zeroes. A visually truncated identifier must still expose and copy the complete original value.

### Truncation

Do not use ellipsis when it changes the meaning of a critical action, status, identifier, currency, unit, or error. Dense layouts may truncate low-risk descriptive text only when the complete value remains easily available and accessible.

## Spacing & Layout

O3Pilot uses a 4px-derived rhythm. Reuse `{spacing.*}` before inventing a local value.

**Core rhythm:**
- Desktop gutter: 32px.
- Compact gutter: 24px.
- Narrow gutter: 16px.
- Major section gap: 32px.
- Related section gap: 24px.
- Content group gap: 16px.
- Bounded surface padding: 24px; compact surface padding: 16px.
- Control group gap: 12px.
- Inline/icon gap: 8px.
- Metadata gap: 4px.

### Density

```text
Density follows task, not visual style.
```

Comparison and scanning may use high density. Overview/analysis surfaces are usually medium density. Explanation, diagnosis, and settings may open up when comprehension benefits. Never achieve density by shrinking the core body type or collapsing essential status/context.

### Shell grammar

The visual shell has a stable sidebar plus global/context chrome, page heading/context, and content canvas. DESIGN describes their appearance, not the product's page inventory or navigation business taxonomy.

Use `{component.sidebar}` as the persistent visual anchor:
- 248px expanded on desktop.
- 72px collapsed rail.
- 60px narrow rail.

The exact business navigation items come from product authority, not this file.

### Content widths

Use width as a reading strategy:
- **Data canvas:** fluid; no forced max-width for wide tables/analysis.
- **Standard content:** around 1200px max when constraint improves reading.
- **Reading/detail:** around 960px max when long-form comprehension benefits.

These are modes, not page-specific contracts.

## Depth & Elevation

O3Pilot does not use “everything is a card” dashboard styling.

```text
Spacing
> Background difference
> Hairline / divider
> Bounded surface
> Shadow
```

- Canvas, sections, normal panels, metric groups, and tables are elevation 0.
- Use `{component.bounded-surface}` only when a group needs a real visual boundary.
- Visible boxed nesting normally stops at two levels; nested surface radii stay coherent.
- Clickable/hover affordance appears only when real interaction exists. Ordinary cards and data surfaces do not lift, scale, or gain shadow merely on hover.
- Tooltip/popover/dropdown may use restrained elevation 1.
- Drawer or stronger overlay may use restrained elevation 2.
- Dialog/modal may use restrained elevation 3.
- Shadow never replaces surface hierarchy, scrim, or focus management.
- No glow and no decorative shadow on static data panels.

Exact shadow CSS is intentionally not frozen yet. If an implementation needs it, use the minimum restrained shadow that communicates actual visual overlap and keep it centralized.

## Iconography

**Lucide** is the only general-purpose functional icon family. Use outline icons with the standard stroke weight; do not mix filled and outline families for state.

Preferred icon sizes are 16, 18, 20, and 24px. Importance comes from placement, label, color, typography and surface hierarchy—not thicker stroke, fill, glow, or shadow.

Use icon + label when meaning is not instantly familiar. Icon-only controls are appropriate only for familiar, low-ambiguity utility actions and must have an accessible name and visible focus. Tooltip is supplementary; it is never the control's only accessible name. Status icons supplement a readable label/status and never replace it.

**Morphicons** is not a second static icon library. It is reserved for approved continuous-state feedback. The signature sidebar behavior is:

```text
Original icon
→ Check
→ Original icon
```

Total feedback is 180–220ms. Navigation occurs immediately and never waits for the morph. Repeated input may interrupt/cancel the previous animation. Under reduced motion, use a brief static Check/opacity confirmation rather than path morph displacement.

## Components

### Sidebar

`{component.sidebar}` and its item variants define appearance only. Business navigation groups and routes are supplied by the product.

Expanded state uses icon + label; collapsed/narrow uses an icon rail. Active state is a restrained selected surface and weight/ink change—never a thick colored rail plus saturated background plus shadow at the same time.

Desktop item height is 40px. Touch/coarse-pointer hit areas still meet the 44px accessibility target when applicable.

### Buttons

- `{component.button-primary}` is the dominant completion action for one action context. If no real primary action exists, do not manufacture a blue CTA.
- `{component.button-secondary}` supports the main task without competing with it.
- `{component.button-ghost}` is a low-emphasis contextual utility.
- `{component.button-icon}` is for familiar compact actions only.

Primary/secondary/ghost share the same shape grammar. Destructive intent changes semantic treatment, not corner geometry. Standard desktop height is 40px; 32px compact controls are allowed only for dense desktop utility contexts. Touch/coarse-pointer target is at least 44px.

Pressed feedback is immediate and subtle. Prefer background/border/opacity changes; do not make `scale(0.95)`, lift, bounce, or spring the default button feedback. For a copy action that needs acknowledgement, `Copy → Check` is an approved short local confirmation pattern.

### Inputs

`{component.text-input}`, `{component.search-input}`, and `{component.select-control}` share 40px height, 8px radius, hairline border and 15px body type.

Keep a visible label as the primary field description. Placeholder is hint/example, not the only label. Place helper/error text next to its field. Do not render ordinary read-only facts as disabled inputs merely to imitate an admin form.

Forms are single-column by default when that improves label/error/localization stability. Use multiple columns only for fields with a real semantic relationship. Checkbox/radio/switch and file-input keep native semantic behavior and the same focus/touch/spacing grammar; a progress indicator appears only when real progress meaning exists, never as generic activity decoration.

### Surfaces

`{component.bounded-surface}` is a boundary primitive, not a generic card template. Before adding it, ask whether spacing, section heading, subtle background or a divider already explains the grouping.

A metric is not automatically a card. A group of metrics can share one baseline or one bounded surface.

### Data tables

`{component.data-table}` is a first-class analytical surface:
- Header: 40px.
- Standard row: 44px.
- Compact row: 36px.
- Standard horizontal cell padding: 16px; compact: 12px.

Compact density never means tiny typography. Text/name/identifier align left; numeric/money/ratio align right. Keep currency/unit context visible. Preserve full identifier copy/accessibility even if rendered text truncates.

Wide tables may scroll horizontally. Do not convert every row to a stacked mobile card or shrink text to preserve desktop geometry.

Background refresh should preserve the user's reading position and visible data when possible rather than blanking the surface. Sticky headers/columns must not cover visible keyboard focus. Row/cell hover, cursor, disclosure, selection or click affordance appears only when that row/cell is actually interactive. Missing/Unknown/Unavailable/Partial/Stale treatment remains visible and is never fabricated as real `0`.

### Status badges

`{component.status-badge}` defaults to neutral. Swap to semantic foreground/subtle/border tokens only after the upstream meaning is known. A badge label remains readable without color.

### Tabs, filters, segmented controls

Use the visual primitives according to interaction semantics:
- **Tabs:** stable peer views.
- **Filters/chips:** subset of the same underlying dataset/view.
- **Segmented control:** a few mutually exclusive display modes.

Do not choose between them because one looks newer. Their query/state behavior belongs to application/product logic; DESIGN controls visual consistency and interaction feel.

### Overlays

- **Tooltip:** brief supplemental explanation; no meaningful workflow.
- **Popover:** anchored lightweight interaction; no blocking scrim.
- **Drawer:** secondary detail context; standard 480px, wide 640px. Do not turn it into a miniature multi-page application.
- **Dialog:** focused short blocking task. Large tables, multi-domain analysis, or long workflows belong elsewhere.
- **Toast:** transient confirmation, not persistent business truth. Prefer one sufficient feedback channel instead of stacking Toast + icon morph + inline success for the same action.

### Loading and state surfaces

`{component.skeleton}` represents known future geometry only. It does not invent fake KPIs/columns or imply values. Spinner is not decorative filler.

`{component.state-surface}` is the generic visual container for empty/error/missing/unknown/unavailable/not-ready/partial/stale presentation families. A strong structure is:

```text
optional semantic icon
Title
Reason / explanation
Relevant context
Recovery or next step when applicable
```

The owning contract determines the actual state meaning and recovery eligibility. DESIGN only keeps the visual language clear. Unknown vs zero, estimate/forecast vs actual/fact, source/origin, currency/unit, time basis, and freshness must remain distinguishable whenever they change interpretation.

## Data Visualization

Charts are **analytical evidence**, not dashboard filler.

```text
Question → chart type
Component slot ↛ chart
```

### Choose the simplest encoding

| Analytical question | Default presentation |
|---|---|
| One key value | Metric |
| Time trend | Line |
| Category comparison | Bar |
| Absolute composition | Stacked bar |
| Proportional composition | 100% stacked bar |
| Signed additive bridge | Waterfall |
| Ordered cohort conversion | Funnel |
| Time × dimension intensity | Heatmap |
| Precise multi-field comparison | Table |

Avoid default use of 3D, radar, gauge, decorative donut/pie, decorative gradient area, and dual Y-axis. Pie/donut is only acceptable for a true part-to-whole with few categories when precise ranking is not the primary task.

### Geometry rules

- Line charts use straight segments by default. Interpolation must not visually overshoot and invent values.
- Bar axes start at zero because bar length encodes magnitude.
- Multiple line series normally stay at five or fewer visible series. Use explicit selection, Top-N, small multiples or a table rather than silent hiding.
- If Top-N or series selection is active, show the scope explicitly.
- Unit, currency and date scope remain visible.

### Series colors

Light categorical sequence:

```text
#5E5CE6  #0086A0  #AF52DE  #A2845E  #667085  #C2417A
```

Dark:

```text
#7D7AFF  #64D2FF  #BF5AF2  #D4B483  #A1A1A6  #FF8AD8
```

Series color is separate from semantic status color. Forecast/comparison must also use non-color channels such as dash, label, boundary or annotation. Color cannot be the only way to identify a series.

### Accessibility and refresh

Give charts a title/context and an accessible exact-data representation when precision matters. Do not make every mark a keyboard stop. Ordinary refresh/filter/date changes do not replay decorative chart entrance animation.

Missing/Unknown/zero, estimate/forecast/actual/fact, Partial/Stale/Unavailable distinctions remain visible, but their definitions come from the owning data/metric contract, not DESIGN. Source/origin, currency/unit, time basis and freshness remain discoverable when they change interpretation. A chart follows the surrounding surface grammar instead of introducing a separate default “chart card” style.

## Motion & Interaction

```text
Motion must have a purpose.
User intent > animation completion.
Motion budget follows interaction frequency.
Data change != animation event.
Visual continuity < responsiveness.
Decorative motion is off by default.
```

### Timing language

- 80ms — instant feedback.
- 120ms — fast feedback / tooltip.
- 180ms — standard popover/dropdown/tab/toast transition.
- 220ms — drawer/dialog spatial transition.
- 280ms — slow ceiling for ordinary UI.
- Sidebar icon morph — 180–220ms total.

Target an immediate feel on 120Hz/high-refresh displays. Large tables/charts prioritize input, scroll and reading responsiveness over continuous layout animation; avoid motion that introduces visible input lag, layout thrash or reading instability.

Prefer `transform` and `opacity`. Avoid animating width/height/margin/padding/top/left for frequent interactions when that creates layout churn. If a continuous sidebar-width animation harms a large table/chart, switch layout immediately and animate only local label/icon opacity/transform.

All user-driven motion is interruptible. A new open/close/navigation intent supersedes the current animation rather than waiting in a queue.

Never use motion to hide latency. Do not ship `transition: all`, long dropdown animations, decorative full-page route slides, count-up/flip/bounce KPI refresh, or chart entrance animation on every data update.

### Reduced motion

Respect `prefers-reduced-motion`:
- No shimmer.
- No long spatial overlay travel.
- No decorative chart animation.
- No programmatic smooth scroll required to understand the UI.
- Replace path morph displacement with a brief static state change when appropriate.
- Keep focus and feedback meaning visible.

## Do's and Don'ts

### Do
- Let data dominate UI chrome.
- Use one interaction accent consistently.
- Reuse frozen tokens/components before local styling.
- Build hierarchy with spacing, surface difference and hairline before shadow.
- Keep normal states neutral and quiet.
- Use semantic color only when it communicates real meaning.
- Make numerical comparison easy with alignment and tabular numerals.
- Preserve full identifiers and exact context.
- Preserve unknown-vs-zero, estimate-vs-fact, source/currency/time-basis/freshness distinctions when they affect interpretation.
- Allow dense tables when scanning/comparison requires them.
- Keep motion short, purposeful, interruptible and optional.
- Test light/dark, keyboard, reduced motion, zoom/reflow and long localized labels.

### Don't
- Don't turn every section, KPI, or table row into a card.
- Don't add decorative gradients, glow, or default glassmorphism.
- Don't add a second general-purpose accent or functional icon family.
- Don't use green/red merely because a number moved up/down.
- Don't reuse danger/success colors as arbitrary chart series colors.
- Don't put shadows on ordinary data panels.
- Don't lift, scale, or add shadow to ordinary cards/data surfaces merely on hover.
- Don't use pill geometry for every control.
- Don't create giant marketing Hero patterns in analytical UI.
- Don't shrink text to preserve a desktop grid on narrow screens.
- Don't hide status, currency, unit, identifier, freshness, or error context that changes interpretation.
- Don't present Missing/Unknown/Unavailable/Partial/Stale as real `0`.
- Don't rely on color, tooltip, or motion as the only meaning channel.
- Don't make a local draft, simulation, or recommendation look as if it was applied to Ozon.

## Responsive Behavior

| Mode | Width | Primary layout behavior |
|---|---:|---|
| Full Desktop | ≥1440px | Full analytical canvas; sidebar may be expanded or collapsed |
| Desktop | 1200–1439px | Full product shell with 32px page gutters |
| Compact | 768–1199px | 72px sidebar rail by default; 24px gutters; reduce parallel layout |
| Narrow | <768px | Permanent 60px icon rail; 16px gutters; stack secondary layout |

`768px` belongs to Compact.

Responsive design preserves the task and semantics; it does not silently remove data. Prefer reducing parallel columns, prioritizing fields, horizontal scroll, or moving secondary detail into an appropriate detail surface before deleting context.

### Tables and charts

Tables stay tables when table semantics matter. Narrow layouts may preserve key identity columns plus horizontal scroll or detail access; do not force cardification.

Charts may reduce axis-tick density, move legends, stack vertically, or use an explicit series selector. Do not silently remove series, change the metric/date range, hide currency/unit, or introduce a dual axis merely because space is tight.

### Forms and touch

Forms can collapse from multiple columns to one. Standard controls remain 40px visually on desktop; touch/coarse-pointer hit areas are at least 44×44px. Width and input mode are separate concepts: Narrow does not always mean touch, and Desktop does not always mean mouse.

### Metrics

Metric groups may reduce from four columns to two or one. Keep primary value, unit/currency and important state discoverable; never preserve column count by shrinking primary values to micro typography.

## Accessibility

Target **WCAG 2.2 AA**.

```text
Semantic HTML first
Keyboard complete
Focus visible
State not color-only
Motion optional
```

- Use native semantic elements before recreating controls with ARIA.
- Use links for navigation and buttons for actions.
- Give primary content a meaningful landmark and heading structure.
- Never remove or clip focus rings.
- Hover-only affordances need keyboard/focus equivalents.
- Overlay open/close must move and restore focus coherently; Toast must not steal focus.
- Touch/coarse-pointer targets are at least 44×44px.
- At 200% browser zoom, critical text/actions remain usable; wide tables may scroll instead of shrinking type.
- Pair charts with accessible summary/exact data when the analytical task requires precision.
- Announce dynamic updates only when urgency/context requires it; routine refresh should not constantly interrupt screen readers.
- Reduced motion is required, not optional polish.

## Localization

Design from the start for:

```text
中文
English
Русский
```

MiSans Global is the unified UI family across Chinese, Latin, Cyrillic, numbers, common currency symbols, punctuation and units.

Allow roughly 30–50% label expansion when practical. Prefer grow, wrap, reflow, or full-width controls on narrow screens before shrinking fonts or ellipsizing critical actions/status/reasons.

Localized number/currency formatting changes presentation only, not the business value. Identifiers remain unformatted strings. In traceability/analytical contexts, avoid ambiguous all-numeric date formats when precision matters.

Visual text and accessible labels localize together. Technical codes can remain technical in diagnostic contexts, but ordinary UI should pair them with human-readable localized copy. The business terminology glossary itself is not owned by DESIGN.

## Iteration Guide

1. Reuse an existing color/type/spacing/radius token before creating another.
2. Reuse an existing component before creating a variant.
3. Page-specific styling does not automatically become a design token.
4. In YAML component definitions, use `{token.refs}` instead of repeating raw hex values.
5. Never introduce another general-purpose functional icon family.
6. Prefer spacing/surface/hairline before decoration/shadow.
7. New motion must explain feedback, state, or spatial continuity and remain interruptible.
8. Data refresh alone is not an animation trigger.
9. A new business state, metric, page, workflow, or feature must come from its owning product/data/security/deployment contract—not from DESIGN.
10. When uncertain, optimize for data clarity and scanning efficiency rather than visual novelty.
11. Dark-theme additions need explicit reviewed values; do not auto-invert.
12. If a style is used by only one page, keep it local until cross-page reuse proves it belongs here.

## Known Gaps

The following are intentionally **not** frozen by this version:
- Exact box-shadow CSS values for elevation 1/2/3.
- A universal max-width for every page.
- A universal `metric-card` component.
- Page-specific chart choices or dashboard compositions.
- Command-palette product availability.
- Exact business navigation groups/items.
- Domain-specific business-state → success/warning/danger mapping.
- Framework/library choices for motion, charts, tables, overlays, or form controls.
- Page-specific responsive column priority.
- Business terminology glossary ownership details.

A Known Gap is not permission to invent a conflicting local design language. Use the closest existing token/component; keep page-local styling local until reviewed and proven reusable.
