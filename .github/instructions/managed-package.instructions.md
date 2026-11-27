---
description: Package boundaries for a repository that extends the KimbleOne__ managed package.
applyTo: "**"
---

# Package boundaries

A design or change that breaks these rules is wrong, even if it works.

- **MP-EXT-001 — package behavior needs evidence.** When a task depends on how `KimbleOne__`
  behaves, base it on `docs/package-constraints.md`, vendor documentation for our package version,
  or an org observation. Never on memory or assumption. New save-time automation on a package
  object needs that evidence for the extension point it uses.
- **MP-DESIGN-001 — designs declare package impact.** A design that touches or depends on
  `KimbleOne__` components names each one, with its evidence, in its package impact section.
- **MP-MAP-001 — the org wins over docs.** If the org and `docs/` disagree, trust the org and
  propose a correction to `docs/`.

Org safety, confirmations, and trust rules are in `.github/copilot-instructions.md`.
