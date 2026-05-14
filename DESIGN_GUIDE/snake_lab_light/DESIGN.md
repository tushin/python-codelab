---
name: Snake Lab Light
colors:
  surface: '#f5fbef'
  surface-dim: '#d6dcd0'
  surface-bright: '#f5fbef'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#f0f6ea'
  surface-container: '#eaf0e4'
  surface-container-high: '#e4eade'
  surface-container-highest: '#dee4d9'
  on-surface: '#171d16'
  on-surface-variant: '#3f4a3c'
  inverse-surface: '#2c322a'
  inverse-on-surface: '#edf3e7'
  outline: '#6f7a6b'
  outline-variant: '#becab9'
  surface-tint: '#006e1c'
  primary: '#006e1c'
  on-primary: '#ffffff'
  primary-container: '#4caf50'
  on-primary-container: '#003c0b'
  inverse-primary: '#78dc77'
  secondary: '#545f73'
  on-secondary: '#ffffff'
  secondary-container: '#d5e0f8'
  on-secondary-container: '#586377'
  tertiary: '#505f76'
  on-tertiary: '#ffffff'
  tertiary-container: '#8c9cb5'
  on-tertiary-container: '#243448'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#94f990'
  primary-fixed-dim: '#78dc77'
  on-primary-fixed: '#002204'
  on-primary-fixed-variant: '#005313'
  secondary-fixed: '#d8e3fb'
  secondary-fixed-dim: '#bcc7de'
  on-secondary-fixed: '#111c2d'
  on-secondary-fixed-variant: '#3c475a'
  tertiary-fixed: '#d3e4fe'
  tertiary-fixed-dim: '#b7c8e1'
  on-tertiary-fixed: '#0b1c30'
  on-tertiary-fixed-variant: '#38485d'
  background: '#f5fbef'
  on-background: '#171d16'
  surface-variant: '#dee4d9'
typography:
  display-lg:
    fontFamily: Inter
    fontSize: 48px
    fontWeight: '700'
    lineHeight: 56px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Inter
    fontSize: 32px
    fontWeight: '600'
    lineHeight: 40px
    letterSpacing: -0.01em
  headline-lg-mobile:
    fontFamily: Inter
    fontSize: 28px
    fontWeight: '600'
    lineHeight: 36px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Inter
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
  body-lg:
    fontFamily: Inter
    fontSize: 18px
    fontWeight: '400'
    lineHeight: 28px
  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 24px
  body-sm:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  label-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '600'
    lineHeight: 20px
    letterSpacing: 0.05em
  label-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  base: 8px
  xs: 4px
  sm: 12px
  md: 16px
  lg: 24px
  xl: 32px
  gutter: 24px
  margin-mobile: 16px
  margin-desktop: 48px
---

## Brand & Style

This design system is built for a professional, academic environment that retains an energetic edge. The personality is precise and methodical, yet forward-moving—mimicking the clinical rigor of a high-tech laboratory. 

The style adheres to **Corporate / Modern** principles with a heavy emphasis on **Minimalism**. By utilizing generous whitespace and a restricted color palette, the system ensures that complex data and scientific insights remain the primary focus. The visual narrative is one of clarity and efficiency, utilizing high-contrast typography and "Snake Green" accents to guide the user through logical workflows.

## Colors

The palette is anchored by a pure white (`#FFFFFF`) foundation for primary content areas and a soft light gray (`#F8FAFC`) for background surfaces and structural offsets. 

The signature **Snake Green** (`#4CAF50`) is the sole driver of action and progression. It must be used purposefully for primary buttons, active navigation states, and success indicators. Typography and high-level iconography utilize a deep Slate (`#1E293B`) to ensure maximum WCAG AAA readability. Secondary information uses lighter grays to maintain a clear visual hierarchy without cluttering the interface.

## Typography

The design system exclusively utilizes **Inter** for its neutral, systematic, and highly legible characteristics. 

Headlines are set with tighter letter spacing and heavier weights to provide a sense of authority and structure. Body text prioritizes a comfortable 1.5x line height to facilitate long-form academic reading. Labels and captions utilize a slightly increased letter spacing and semi-bold weights to remain distinct from body content. All text color defaults to the primary dark slate to prevent eye strain against the white background.

## Layout & Spacing

The system follows a **Fluid Grid** model based on an 8px square baseline. 

- **Desktop:** 12-column grid with 24px gutters and 48px outer margins.
- **Tablet:** 8-column grid with 24px gutters and 32px outer margins.
- **Mobile:** 4-column grid with 16px gutters and 16px outer margins.

Spacing between related components (labels and inputs) should use `8px (base)`, while spacing between distinct sections should use `32px (xl)` to maintain an open, academic feel.

## Elevation & Depth

To maintain a crisp and professional look, this design system avoids heavy shadows and skeuomorphism. It utilizes **Tonal Layers** and **Low-Contrast Outlines**.

- **Level 0 (Base):** `#FFFFFF` or `#F8FAFC`.
- **Level 1 (Cards/Surface):** White background with a 1px border of `#E2E8F0`. No shadow.
- **Level 2 (Dropdowns/Modals):** White background with a subtle, highly diffused ambient shadow: `0 4px 12px rgba(30, 41, 59, 0.05)` and a 1px border.
- **Active State:** A 2px solid stroke of `#4CAF50` is used to indicate focus or selection, providing high-energy feedback.

## Shapes

In alignment with the "ROUND_EIGHT" requirement, the shape language is consistently **Rounded**. 

- **Standard Components:** Buttons, input fields, and alerts use a 0.5rem (8px) radius.
- **Containers:** Large cards and section blocks use a 1rem (16px) radius to soften the academic aesthetic.
- **Small Elements:** Tooltips and tags use a 0.25rem (4px) radius to maintain precision at smaller scales.

## Components

### Buttons
- **Primary:** Background `#4CAF50`, Text `#FFFFFF`, 8px corner radius. Heavy padding (12px 24px).
- **Secondary:** Transparent background, Border 1px `#E2E8F0`, Text `#1E293B`.
- **Ghost:** Transparent background, Text `#4CAF50`, for low-priority actions.

### Navigation (Codelab Header)
- Every sub-codelab page must expose a **Hub Back** button (`허브로 돌아가기`) in the header.
- Placement is fixed for consistency: **top-right action zone inside the sticky topbar**, directly above the progress area on desktop.
- On mobile, it remains in the same topbar action group and aligns left for readability.
- Style follows the shared secondary button language: surface background, 1px outline, full-pill radius, and green-tinted hover state.
- Implementation rule: render from shared layer (`src/codelabs/shared/app.js` + `src/codelabs/shared/styles.css`) instead of per-codelab overrides.

### Input Fields
- White background, 1px `#E2E8F0` border. On focus, the border transitions to `#4CAF50` with a 2px stroke. Placeholders use `#94A3B8`.

### Progress Bars
- Track: `#F1F5F9` (Light Gray).
- Fill: `#4CAF50` (Snake Green). For laboratory data processing, the fill may include a subtle pulse animation to signify activity.

### Chips & Tags
- For status indicators, use a light tint of the status color (e.g., Green tint for "Complete") with `#1E293B` text to ensure readability within the academic context.

### Cards
- Always white background. 1px border of `#E2E8F0`. Use `headline-md` for card titles.

### Lab-Specific Components
- **Data Traces:** High-contrast line graphs using Snake Green for the primary data line against a `#F8FAFC` grid background.
- **Status Monoliths:** Large, centered indicators for lab equipment status, using bold typography and green icons.
