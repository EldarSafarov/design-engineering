---
name: html-to-figma
description: >
  Convert an HTML/CSS mockup into a production-grade Figma file — variables,
  components with variants, Auto Layout, pixel-accurate to the source. Use whenever
  the user provides HTML, CSS, a live URL, or a screenshot of a coded page and wants
  it rebuilt in Figma. Trigger on: html to figma, вёрстка в фигму, перенеси макет
  в figma, собери макет из html, recreate this page in figma, code to design,
  reverse handoff, HTML → Figma, пересобрать в компоненты, разобрать вёрстку
  на компоненты. Also use for live design tests where speed and structure matter.
---

# HTML → Figma

Rebuild a coded interface in Figma as a **system**, not a picture: variables first,
components second, screen last. Runs through the Figma Bridge plugin (WebSocket).

---

## 0. Prime directive — fidelity over taste

**Every value comes from the source. Nothing is invented, nothing is "improved".**

| Do | Never |
| --- | --- |
| Read the actual hex from CSS | Substitute a palette from another design system |
| Keep the source's spacing scale, even if irregular | Round 14px to 16px "because 8pt grid" |
| Reproduce the exact type sizes and line-heights | Swap the typeface for a "better" one |
| Flag a mismatch out loud and ask | Silently fix what looks wrong |

If a value looks like a mistake, **say so and reproduce it anyway**. The user decides.
Fidelity is the thing being tested; opinions come after the rebuild is done.

**Never load another design system's tokens into this task.** Methodology travels
between projects; values do not.

---

## 1. Phases

Always run in this order. Never start with the screen.

    AUDIT → TOKENS → COMPONENTS → ASSEMBLY → VERIFY

Announce the phase you're entering. Stop after AUDIT and wait for confirmation
unless the user explicitly asked for a non-stop run.

---

## 2. AUDIT — read before you build

Parse the HTML/CSS and report, without touching Figma:

1. **Colors** — every unique value, deduplicated, with where each is used
   (`#1C1B18` — body text, active doc title). Note values already declared as
   CSS custom properties: those are the author's own token layer, mirror it.
2. **Typography** — families, sizes, weights, line-heights, letter-spacing,
   text-transform. Group into a style list.
3. **Spacing & radius** — the actual scale in use, in source order.
4. **Component candidates** — repeated elements, with their variants visible in
   the markup (a button appearing as default / primary / disabled = one component,
   three variants).
5. **Layout skeleton** — the frame tree you intend to build, matching DOM structure.

If the source uses CSS custom properties (`--paper`, `--ink`), **adopt those names**
as the primitive layer. The author already made naming decisions; overriding them
is noise.

---

## 3. TOKENS — two collections, always

**Primitives** — raw values from source. Names mirror the source's own vocabulary
if it has one, otherwise `colors/*`, `spacing/*`, `radius/*`.

**Semantic** — purpose aliases pointing at primitives:

    {category}/{purpose}/{state}

    category  text · bg · border · icon · action · accent · overlay
    purpose   primary · secondary · subtle · inverse · disabled
    state     default · hover · pressed · focus · selected · disabled

Rules:

- Never bind a component directly to a primitive.
- A state token only exists if that state exists in the source. Do not invent hover
  states the CSS doesn't define.
- Text styles: create for every distinct combination found, named by role
  (`body/base`, `heading/h1`, `label/caps`), not by appearance.

Batch variable creation (`figma_batch_create_variables`) rather than one call per
value — this is the single biggest time saver.

---

## 4. COMPONENTS — bottom-up

Build atoms before composites. For each component:

- **Auto Layout always.** Padding, gap and alignment mirror the CSS box model.
  Hug vs Fill mirrors the source's sizing behaviour.
- **Every visual value bound to a variable.** No raw hex, no raw numbers.
- **States as variants** via component properties, not as separate frames.
  Only the states present in the source.
- **Booleans for optional elements** (leading icon, meta label), instance swap for
  swappable ones.
- Keep variant properties to 2–3. Combinatorial explosion is a failure, not thoroughness.

Naming: `Component / Subcomponent` matching the source's own semantics
(`Doc Row`, `Toolbar Button`) — not `Frame 47`.

---

## 5. ASSEMBLY — screen from instances

- Frame tree mirrors the DOM tree. Sidebar / main / inspector become sibling frames
  in a horizontal Auto Layout, exactly as the flex container implies.
- Use **instances**, never detached copies. If the screen contains a raw shape that
  should have been a component, that's a bug — go back to phase 4.
- Fixed widths only where CSS fixes them; everything else fills or hugs.

---

## 6. VERIFY — the loop that matters

1. `figma_take_screenshot` of the result.
2. Compare against the source side by side: spacing, weights, color, alignment.
3. Report differences **as a list, honestly** — including ones the user might not spot.
4. Fix, re-screenshot, repeat. Two iterations maximum, then hand back with the
   remaining deltas named.

Also run a token audit: any node with a hardcoded fill or a detached text style is
a defect. `figma_lint_design` if available.

---

## 7. Live-test mode

When the user says this is a timed test or gives a deadline, switch behaviour:

**Budget for 30 minutes**

| Phase | Minutes |
| --- | --- |
| Audit | 2 |
| Tokens | 5 |
| Components | 10 |
| Assembly | 8 |
| Verify | 5 |

- Skip the confirmation stop after AUDIT — report and continue immediately.
- Narrate decisions in one short line each; the observer is grading reasoning,
  not silence.
- If time runs short: **cut scope, never structure.** Three correct components
  with variants and bound tokens beat ten loose frames. Say out loud what you're
  cutting and why.
- Missing fonts: substitute the nearest available, state it once, move on. Never
  stall on font installation.

---

## 8. Working in someone else's file

If the target file already has a design system:

1. Read what exists first — `figma_get_variables`, `figma_search_components`,
   `figma_get_design_system_summary`.
2. Map source values onto **their** tokens where a match exists. Only create new
   tokens for genuinely new values, and name them in their convention.
3. Extend their components rather than building parallel ones.
4. Report at the end: what was reused, what was added, what conflicted.

Never impose an external naming scheme on an existing system.

---

## 9. Failure modes

| Symptom | Cause | Fix |
| --- | --- | --- |
| Layout collapses on resize | Hug/Fill guessed instead of read | Re-read the flex/grid rules |
| Colors "close but off" | Alpha or overlay ignored | Check rgba() and layered backgrounds |
| Component count explodes | State modelled as separate component | Merge into variants |
| Text reflows differently | line-height set as multiplier vs px | Match the CSS unit exactly |
| Model starts redesigning | Prime directive not restated | Restate §0 and rerun the phase |
