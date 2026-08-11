# Design Foundation — [Project Name]

<!-- Scaffold used by /new-project Gate 3 and by /implement-milestone for UI work.
     This file is the persistent, signed-off source of truth for the project's visual
     language. Every UI milestone derives from it and must not re-decide it.

     Fill every [bracketed] placeholder and delete these comment blocks. The token
     STRUCTURE below is fixed; the VALUES are this project's choices. Pick a coherent
     system and hold to it consistently. The numbers in the design-principles.md rule are
     defaults you may override here — but once set here, these values bind the project.
     Where an override is bounded, the rule states the bound alongside the default.

     The rendered companion to this file is docs/styleboard.html — regenerate it from
     these tokens whenever the language changes. The first UI milestone lifts the :root
     block below into the project's real token layer (Tailwind theme / globals.css). -->

## Visual direction

[One short paragraph: the chosen aesthetic direction and why it fits THIS product and
audience. Name the direction (editorial, brutalist, luxury, bento, Swiss, etc.), not a
vague "clean and modern". This is the decision design-quality.md is enforcing.]

## Colour

<!-- 60-30-10: a dominant surface, a secondary panel, a small accent share. -->

```css
:root {
  /* Dominant — surfaces and backgrounds */
  --color-bg:            #;
  --color-surface:       #;
  --color-surface-alt:   #;

  /* Secondary — cards, panels, borders */
  --color-panel:         #;
  --color-border:        #;
  --color-border-subtle: #;

  /* Accent — CTAs, highlights, active states */
  --color-accent:        #;
  --color-accent-hover:  #;
  --color-accent-text:   #; /* text on accent background */

  /* Semantic */
  --color-success:       #;
  --color-warning:       #;
  --color-error:         #;
  --color-info:          #;

  /* Text */
  --color-text-primary:   #;
  --color-text-secondary: #;
  --color-text-disabled:  #;
  --color-text-inverse:   #; /* text on dark/accent backgrounds */
}
```

## Typography

- **Display / heading font**: [font name] — [character notes; not a fallback default]
- **Body font**: [font name] — [character notes]
- **Mono / utility font** (if used): [font name]
- **Font source**: [system stack / self-hosted / CDN]
- **Type scale ratio**: [chosen ratio, e.g. 1.333] on a [base, e.g. 16px] base

| Token | Size | Weight | Use |
|---|---|---|---|
| `--text-xs`   | [px] | 400 | Caption, label |
| `--text-sm`   | [px] | 400 | Secondary body |
| `--text-base` | [px] | 400 | Primary body |
| `--text-lg`   | [px] | 400/500 | Lead / large body |
| `--text-xl`   | [px] | 600 | h3 / card title |
| `--text-2xl`  | [px] | 600/700 | h2 / section heading |
| `--text-3xl`  | [px] | 700 | h1 / page title |
| `--text-4xl`  | [px] | 700 | Display / hero |

## Spacing

<!-- Pick one base unit and scale, then use ONLY values from it. -->

```css
:root {
  --space-1:  [px];
  --space-2:  [px];
  --space-3:  [px];
  --space-4:  [px];
  --space-6:  [px];
  --space-8:  [px];
  --space-12: [px];
  --space-16: [px];
  --space-24: [px];
  --space-32: [px];
}
```

## Shape and elevation

- **Border radius convention**: [none / subtle / rounded / pill]
- **Shadow convention**: [flat / subtle depth / pronounced depth]

```css
:root {
  --radius-sm:   [px];
  --radius-md:   [px];
  --radius-lg:   [px];
  --radius-full: 9999px;

  --shadow-sm: [value];
  --shadow-md: [value];
  --shadow-lg: [value];
}
```

## Signature element

[The one element this product is remembered by — the deliberate risk from frontend-design.
Describe what it is, where it appears, and why it embodies this product. Spend boldness here
and keep everything around it quiet.]

## Motion and interaction

- **Motion posture**: [restrained / expressive] — where animation serves the product.
- **Interactive states**: describe the hover / focus / active / disabled treatment that all
  interactive elements share (colour shift, elevation, outline).

## Usage context

<!-- Carried forward from docs/PRD.md (new-project Gate 1). Restated here because every
     decision below depends on it. If it changes, change it in PRD.md first. -->

- **Operated on**: [device and screen — e.g. one person's laptop; a laptop plus projector when
  presenting; desktop and mobile]
- **Viewport range to build for**: [e.g. 1280-1920 desktop only; 360-1920 mobile through desktop]
- **Input devices assumed**: [mouse and keyboard / touch / both]
- **Presentation robustness**: [required — shown on projectors or client screens, so the layout
  must survive browser zoom and a resolution change / not required]

## Legibility defaults

<!-- Strong defaults from design-principles.md, with real reasons, not compliance obligations.
     Override deliberately and say why. Reference values on white: 4.5:1 is about #767676,
     3:1 about #959595. -->

- **Body text contrast**: [target, default 4.5:1] — [note any deliberate override and why]
- **Large text and UI objects**: [target, default 3:1]
- **Deliberately quiet text** (timestamps, captions, placeholder hints, disabled controls):
  [target if it sits below the default — this is the bounded override; primary and body text may
  not go below]
- **Visible focus**: [the `:focus-visible` treatment shared by all interactive elements]
- **Labels**: every input has a visible label; icon-only controls carry a name in the markup

## Interaction decisions

<!-- Answers, not requirements. Resolved during design (new-project Gate 3, or
     implement-milestone Step 5 per surface) and recorded here so they are not re-asked every
     milestone. "No, not worth it" is a valid answer — record it, do not leave it blank.
     Scoped by the usage context above. -->

- **Keyboard paths for frequent actions**: [per action — e.g. Delete key removes the selected
  item; Enter submits; Escape closes; arrow keys move within the list. Or: none, this surface is
  used occasionally and mouse-only is fine]
- **Focus on state change**: [where focus goes when a modal or panel opens, and where it returns
  on close]
- **Control sizing**: [which controls are used most and how they are sized and placed for it;
  any minimum this project sets]
- **Presentation robustness**: [if required above — how the layout holds up under zoom and a
  resolution change. Otherwise: not applicable]

## Avoid

[Design anti-patterns specific to this project or brand — things the user has explicitly
rejected. Carry over any "avoid" notes from the Gate 3 interview.]

## References

[Reference brands, products, or inspiration this draws from, if any.]
