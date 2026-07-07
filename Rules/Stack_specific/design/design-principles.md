# Web Design Principles — Craft Floor

> Framework-agnostic web design rule. Applies to every web frontend stack (React, Vue,
> Angular, Nuxt, plain web) — provisioned per project into `.claude/rules/design/`.
> This rule is the measurable companion to [design-quality.md](design-quality.md): that rule
> governs aesthetics and distinctiveness (the taste ceiling); this one governs usability craft
> (the floor every direction must clear).

This rule states the usability-craft standard for web UI: layout, typography, colour,
hierarchy, interaction states, forms, navigation, data visualisation, and accessibility.
For the procedure that applies it while designing, use the Global `frontend-design` skill;
for aesthetic direction and anti-template policy, follow [design-quality.md](design-quality.md).

## Scope

This rule governs **user-facing UI surfaces only**. It does not bind backend, API, domain,
data, or infrastructure code, and backend reviewers do not apply it. A milestone that ships
no user-facing UI is out of scope here entirely.

## How to read the numbers in this rule

Most concrete numbers below are **recommended defaults**, not universal law. Their origin is
particular design systems and conventions; they are sound starting points, not commandments.
A project may choose its own values — a different spacing base, a different type ratio — and
record them in its `DESIGN.md`. Once chosen there, those project values are what binds, and
they must be applied **consistently**. The discipline is "pick a system and hold to it," not
"use exactly these numbers."

Two categories are exceptions and are stated as **firm requirements**, marked `[STANDARD]`:

1. **Accessibility** — WCAG 2.2 AA thresholds (contrast, tap-target, visible focus, reduced
   motion, never-colour-alone). A project does not get to opt out of accessibility.
2. **Consistency itself** — whatever scale a project picks, ad-hoc off-scale values are a
   defect regardless of the specific numbers.

Everything unmarked is a default: honour it unless the project's `DESIGN.md` deliberately
overrides it.

## Cognitive Load and Attention

- **One primary action per region.** At most one primary-styled CTA per screen region;
  competing actions become secondary (outline/ghost) or tertiary (text). [Hick's Law]
- **Purpose readable fast.** A user should extract a screen's primary purpose at a glance; if
  not, reduce content or sharpen hierarchy before adding anything.
- **Chunk to working-memory limits.** Group content so a region reads as a handful of
  super-chunks, not a dozen loose atoms — use proximity, cards, or dividers. (Default target:
  5-7 chunks per region.) [Miller; Cowan]
- **Signal over noise.** Every non-functional visual element is a deletion candidate. If
  removing it does not reduce comprehension, remove it.

## Visual Hierarchy

- **Scale contrast is the primary tool.** Make dominant elements clearly larger than
  secondary ones — differences of only 10-20% read as "equal" and fail. (Default: primary
  heading around 2x body; subheading 1.25-1.5x body.)
- **Weight before size for inline emphasis.** Use a heavier weight for emphasis and a regular
  weight for body; avoid ultra-thin weights for body text — they vanish on low-contrast
  screens and for low-vision users.
- **One focal point per region.** Exactly one element dominates each region, by size,
  contrast, colour, or isolation. Squint-test: what do you see first?
- **Put primary content and CTAs where the eye lands first** (top band / reading-start),
  following the surface's reading pattern (F for text-dense, Z for marketing). Critical
  actions are not buried below the fold on first load.

## Gestalt Grouping

- **Proximity encodes relationship.** Related items sit closer together than unrelated ones;
  the gap between groups should be clearly larger than the gap within a group (a ~2x
  relationship is a good default).
- **Similarity encodes role.** Elements with the same function share one visual treatment.
  Never give two different roles the same full style, and never split one role into two styles
  for decoration.
- **Common region for weak proximity.** When proximity alone is ambiguous, group with a card,
  border, or background — but avoid stacking border AND shadow unless depth is intentional.
- **Figure/ground stays separable.** Content keeps sufficient contrast against its background
  in every state; text over imagery needs a scrim, shadow, or solid panel to stay legible.

## Typography

- **One modular type scale.** Pick a single ratio and derive every size from a base; do not
  introduce ad-hoc sizes. (Common ratios: 1.2, 1.25, 1.333, 1.414, 1.5, 1.618. A reasonable
  default is 1.333 on a 16px base — but the project's chosen scale in `DESIGN.md` wins.)
- **Limit sizes and fonts.** Keep the number of distinct type sizes and font families small
  and deliberate (a common ceiling: ~4 sizes per screen, ~6 per app, 2 families). Fewer,
  well-chosen roles read as intentional.
- **Readable line settings.** Body line-height in the loose range (roughly 1.4-1.6) and
  display/heading tighter (roughly 1.05-1.2); cap body line length so lines do not run
  edge-to-edge (around 65-75 characters is a durable default).
- **Body size stays legible.** Keep body text at a comfortable reading size on each device;
  do not shrink body copy to fit more in.

## Colour

- **Distribute, do not scatter.** A dominant surface colour, a secondary panel/card colour,
  and a small accent share (the 60-30-10 idea) keeps attention where it belongs. If the accent
  is everywhere, it stops being an accent — one accent moment per region is the default.
- **Define colours as tokens.** All colours live as CSS custom properties (or the framework's
  token layer), never as scattered hard-coded hex in component styles. This is what makes a
  theme swap or a `DESIGN.md` change a one-place edit.
- **`[STANDARD]` Meaning is never carried by hue alone.** Pair any colour-coded distinction
  with an icon, label, shape, or weight. [WCAG 1.4.1]
- **`[STANDARD]` Contrast meets WCAG 2.2 AA.** Body text >= 4.5:1 against its background;
  large text (>= 24px regular or >= 18.66px bold) and UI/graphical objects >= 3:1. Aim higher
  (7:1) where practical for resilience. [WCAG 1.4.3, 1.4.11]

## Layout and Grid

- **Spacing comes from one scale.** Pick a base unit and a spacing scale and use only values
  from it; ad-hoc off-scale values (the classic `13px`, `22px`, `37px`) are a defect. `[STANDARD:
  consistency]` The specific scale is the project's choice (an 8px-based scale such as
  4/8/12/16/24/32/48/64/96/128 is a common default); consistency to whatever is chosen is the
  requirement.
- **Layout snaps to a grid.** Use a consistent column grid with regular gutters; major blocks
  align to it rather than drifting to arbitrary widths.
- **Content never touches the viewport edge** without padding; safe-zone margins scale up from
  mobile to desktop, or content is centred within a max width.
- **Responsive by default.** Layout works from small mobile widths up through desktop with no
  horizontal scroll; design mobile-first and add breakpoints upward.

## Components and Interaction States

- **Every interactive element has designed states:** default, hover, active/pressed, focus,
  and disabled. An undesigned state is an unfinished component.
- **`[STANDARD]` Focus is always visible.** Every interactive element has a visible
  `:focus-visible` style with sufficient contrast against its surroundings. Never remove the
  focus outline without an equivalent replacement. [WCAG 2.4.7]
- **`[STANDARD]` Adequate tap targets.** Interactive elements meet at least a 44x44px hit area
  (expand with padding if the visual is smaller). [WCAG 2.5.8]
- **Disabled state is legible as disabled** (reduced opacity plus `cursor: not-allowed`), not
  just greyed text with no affordance.
- **Async actions give feedback.** Any action with perceptible delay shows a loading state and
  prevents double-submit; completion shows explicit success or inline error. For content-heavy
  areas prefer skeletons that mirror the final layout over spinners; spinners suit short,
  single-value actions.
- **Empty and error states are designed**, never a raw "No data" or a silent failure. An empty
  state invites the next action; an error says what happened and what to do about it.
- **Destructive actions are protected.** Never fire an irreversible action on a single
  unguarded click — confirm first, or offer undo.
- **Consistency across screens.** An action that looks a certain way behaves the same way
  everywhere; the same action keeps the same label through a whole flow.

## Forms

- **Single-column by default.** Multi-column only for short, logically paired fields (e.g.
  city + postcode).
- **Labels above fields, always visible.** Never use placeholder text as the label — it
  disappears on input and fails accessibility.
- **Validate on blur, not on keystroke.** Showing errors mid-typing is hostile; blur is early
  enough to catch problems before submit. Real-time meters (password strength) may run
  alongside, not instead of, blur validation.
- **State required vs optional explicitly**, with a stated convention; never leave the user
  guessing.
- **Match input type to data** (`email`, `tel`, `number`, `date`) so the right keyboard and
  native validation engage. For a small set of mutually exclusive options, prefer radios or a
  segmented control over a dropdown.
- **Confirm submission.** After submit, show a dedicated success state and say what happens
  next — not just a cleared form.

## Navigation

- **Keep primary navigation small** and group overflow into secondary navigation rather than
  crowding the top level. (A common ceiling is ~7 items.)
- **Always show location.** The current section has a distinct active state; deep pages show
  breadcrumbs; the page title matches the nav label that leads to it.
- **Navigation is stable.** Primary navigation keeps its position across screens; users build
  a spatial model and moving it forces relearning.
- **Prefer label plus icon.** Icon-only navigation is acceptable only for universally
  understood glyphs in space-constrained contexts.

## Data Visualisation

- **Maximise data-ink.** Most of a chart's ink should encode data: no 3D, no decorative
  gradients on data, no chartjunk, no chart border.
- **Limit series** so a single chart stays readable (around five is a practical ceiling);
  beyond that, aggregate, filter, or use small multiples.
- **Restrained gridlines**, low-opacity and horizontal, only where they aid reading; prefer
  direct labels over a detached legend.
- **`[STANDARD]` Series are distinguishable without colour** — pair colour with direct labels
  or patterns for colour-blind safety. [WCAG 1.4.1]
- **Titles state the takeaway**, not just the variable ("Q3 revenue grew 22%", not
  "Q3 revenue").

## Accessibility `[STANDARD]`

These are firm requirements, not defaults. Target WCAG 2.2 AA.

- **Semantic HTML.** Use the right elements (`nav`, `main`, `button`, `label`, headings in
  order); do not rebuild interactive semantics on top of `div`s.
- **Labels and names.** Every input has an associated label; icon-only controls have an
  accessible name.
- **Keyboard operable.** All actions are reachable and operable by keyboard with a logical tab
  order; nothing is mouse-only.
- **Visible focus** on every interactive element (see Components).
- **Contrast** meets AA (see Colour).
- **Reduced motion respected.** Non-essential animation is wrapped in
  `prefers-reduced-motion`; avoid autoplay motion and rapid flashing. [WCAG 2.3.3]
- **Never colour alone** to convey meaning (see Colour).

The Global `accessibility` skill is the procedure for meeting and auditing these.

## Nielsen's Heuristics — Secondary Audit Layer

After the visual and interaction rules above pass, run Nielsen's ten heuristics as a
second-pass usability audit. They are qualitative checks, not numeric thresholds:

1. Visibility of system status — the UI always shows what is happening.
2. Match to the real world — plain language and familiar concepts, not system jargon.
3. User control and freedom — reachable exits, cancel, and undo.
4. Consistency and standards — same thing looks and reads the same everywhere.
5. Error prevention — design out the error before writing the error message.
6. Recognition over recall — options are visible, not memorised.
7. Flexibility and efficiency — accelerators for experts without blocking novices.
8. Aesthetic and minimalist design — every element earns its place.
9. Help users recover from errors — plain language, the cause, and the fix.
10. Help and documentation — contextual, inline, at the moment of need.

## Review Severity

| Level | Example | Action |
|---|---|---|
| CRITICAL | Any `[STANDARD]` accessibility failure (contrast below AA, no visible focus, keyboard trap, meaning by colour alone) | BLOCK — fix before merge |
| HIGH | Missing empty/error/loading state; destructive action with no guard; placeholder-as-label; off-scale spacing throughout | Fix before merge |
| MEDIUM | Weak hierarchy; inconsistent component states; too many type sizes | Fix when reasonable |
| LOW | Minor rhythm or alignment polish | Note, optional |

## Related rules

- [design-quality.md](design-quality.md) — aesthetics, anti-template policy, style directions.
