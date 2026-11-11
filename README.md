# Design engineering pipeline

I'm Eldar Safarov, a product designer. For the last year my design systems haven't
stopped at Figma — components are generated against the repository's own conventions,
reviewed in Storybook, and shipped to production through pull requests.

This repository contains the rules that make that work.

---

## What's here

| File | What it is |
|---|---|
| [`skills/design-system-tokens.md`](skills/design-system-tokens.md) | Token architecture and design-system methodology: three-tier hierarchy, naming, variant conventions, audit steps |
| [`skills/html-to-figma.md`](skills/html-to-figma.md) | Converting markup into a production-grade Figma file: variables, components with variants, real vector icons, full Auto Layout |
| [`skills/component-to-production.md`](skills/component-to-production.md) | Shipping a component from Figma to production: branch, generated code, nine checks, Storybook anatomy page, pull request |

These are skills for Claude — written by me, used daily. They are not prompts.
They are the design system's rules in a form an agent can follow without me
standing over it.

---

## The pipeline

```
Figma (source of truth, read over MCP)
   ↓  agent generates component code against repo conventions
Storybook
   ↓  I review every state, behaviour, edge case
Pull request
   ↓  frontend engineer reviews the code
Production
```

Everything up to the pull request is automated. My work sits in three places:
the audit that teaches the agent the codebase, the rules in this repository,
and the review in Storybook.

---

## Where this runs

**B2B procurement platform** (Angular) — design system built from scratch:
token hierarchy, components with variants and full state coverage, documentation.
Around 70–80 components live in production and in active use. Two products share
one system through brand tokens.

**SaaS product** (React) — second design system, different stack, different
conventions, same method. Roughly a hundred system-level tasks closed in the
first month.

Alongside the pipeline, a frontend developer and I built an internal browser
extension for QA: switch it on and the live page shows which components already
come from the design system and which are still legacy. Design-system migration
became something we could measure instead of estimate.

---

## The part that actually matters

You cannot point an agent at a repository and start generating. It fails in the
most expensive way — code that looks right, passes visually, and breaks at review
or in CI.

What makes it work is the audit. Before any component: read the repository, the
existing system, the code style, how the team actually writes on both sides of the
stack. Run throwaway components through the pipeline. Then tune the rules to *that*
repository, because every team's conventions differ and a generic workflow produces
generic, foreign-looking code.

Once that groundwork is done, nothing falls over on CI and reviews go through clean.

That's the repeatable part — not a tool, but a method for making an existing
codebase workable by an agent.

---

## Elsewhere

Design portfolio: [behance.net/iameldarsafarov](https://www.behance.net/iameldarsafarov)
