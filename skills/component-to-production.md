---
name: component-to-production
description: >-
  Ship a design-system component from Figma to production: branch → generated code →
  automated checks → tests → Storybook anatomy page → commit → pull request → review.
  Use on every component deploy ("ship this component", "push to git", "new ds component",
  "move it into code").
---

# component-to-production

The deploy half of the pipeline. The design side decides what a component is;
this skill decides when it is allowed to exist in the repository.

**Roles.** The agent writes the code and runs its own checks. The human runs the
test suite and Storybook, reviews the result, and performs the commit and push.
Nothing enters the repository without an explicit go-ahead from the human.

> **Note.** This document is written generically. Paths, ticket prefixes, package
> names and CI specifics are placeholders — in a real repository they are filled in
> during the audit step below, and the skill is tuned to that codebase.

---

## 0. Audit before the first component

This is the step people skip, and skipping it is why generated code fails review.

Before any component is written:

- Read the repository. Framework and version, state management, styling approach,
  folder conventions, barrel vs direct imports, how existing components are structured.
- Read the existing design system, if there is one. What is a wrapper over a vendor
  library and what is native. How tokens are consumed in code.
- Read the Figma source. Which file is the source of truth, how component sets are
  named, which node IDs map to which components.
- Run two or three throwaway components through the pipeline and have them reviewed
  by a frontend engineer. Fix the rules, not the components.
- Only then write the project-specific version of this skill.

Generic rules produce generic code: syntactically fine, structurally foreign,
rejected at review. The audit is what makes generated output indistinguishable
from what the team writes by hand.

---

## 1. Ticket first

One ticket covers design and code. Record:

- **Goal** — what the component is for.
- **Design** — link to the Figma component set.
- **Scope** — what is in code, what is out.
- **Acceptance criteria** — Storybook matches Figma; compiler, linter and tests
  clean; frontend review passed; PR merged into the working branch.

Move it to In Progress at the start, Resolved after merge. If the workflow requires
extra fields to start work, fill them from a neighbouring ticket rather than guessing.

---

## 2. Branch

One branch per component, named after the ticket. If a component depends on another
that is still unmerged, stack the branch on top of it and rebase with
`git rebase --onto` rather than a plain rebase or merge — otherwise git sees the same
change as two different commits and the pull request fills with someone else's commits.

Symptom that it was forgotten: the pull request reports a conflict and lists commits
that are already merged. Same cure.

Rebasing a stacked branch before pushing is the agent's default responsibility, not
something to do after the conflict appears.

---

## 3. Spec from Figma

Read the component from Figma directly through the plugin bridge, not from a
description or a screenshot. Extract:

- Every variant and every component property.
- Every state: default, hover, pressed, focus, disabled, loading, error, empty.
- Exact geometry, spacing and typography, expressed as tokens — never as raw values.
- Which parts are slots the consumer fills, and which are fixed.

Node IDs go stale when sets are rebuilt. Verify them against the sync file rather
than trusting an ID remembered from a previous session.

---

## 4. Code

Write the component against the repository's conventions, not against a generic
style guide. Hard rules:

- Semantic tokens only. A hardcoded colour, spacing or font size is a defect.
- Every state from the spec is implemented, including the ones that are hard to
  reach by clicking.
- The public surface — inputs, outputs, slots — is minimal and documented.
- If the component needs something *from* its consumer (wiring, projected content,
  routing), that is part of the contract and must be written down. A contract that
  only exists in the author's head is lost on the next change.

---

## 5. Nine checks before the human sees it

The agent runs all of these and reports results. Any failure is fixed before handoff.

1. Styles compile.
2. Types compile.
3. Linter clean.
4. Unit tests pass.
5. No hardcoded values — every colour, spacing and type value resolves to a token.
6. All states from the spec exist and are reachable in Storybook.
7. Accessibility: roles, labels, focus order, keyboard operation.
8. Responsive behaviour and long-string handling, including the project's
   localisation set.
9. Nothing unrelated to this component changed in the diff.

---

## 6. Storybook anatomy page

Mandatory for every component. Not a demo — a reference page that shows:

- Every variant side by side.
- Every state.
- The anatomy: what each part is called and which token drives it.
- Edge cases: longest realistic string, empty content, overflow.

If a reviewer has to open Figma to understand the component, the page is incomplete.

---

## 7. Documentation and sync

- Component README: selector, wrapper or native, key geometry, inputs and outputs,
  behaviour notes, runtime prerequisites. A missing README means the task is not done.
- Sync file: record `component → figma node → synced at`.
- When a Figma component that already exists in code changes, a ticket is opened
  immediately. Drift between design and code is the failure mode this whole pipeline
  exists to prevent.

---

## 8. Commit, pull request, review

- Commit messages in English, prefixed by type and ticket.
- Pull request description: what, details (Figma set, tokens, states, tests), checks run.
- Reviewer is a frontend engineer, never the author.
- Every review comment is addressed; reply in the thread with the fixing commit and resolve.
- After merge, close the ticket.

---

## 9. Failure modes worth knowing

| Symptom | Cause | Fix |
|---|---|---|
| Self-merge guard red | Guardrail waiting for a non-author approval | Get the approval, retry the job |
| PR shows conflict and foreign commits | Stacked branch after the parent was squash-merged | `rebase --onto` + force-push with lease |
| Pipeline fails with no jobs / unresolved includes | Pusher lacks access to shared CI templates | Grant read access, or have someone with access retry |
| Generated code passes checks but fails review | Audit step was skipped or rules are stale | Re-audit, update the project skill, not the component |
| Editor flags code samples inside markdown | Linter treating documentation as source | Ignore for documentation only, never for real code |
