# Pattern — Documentation-deliverable modules (Build-to-Teach) via Nexus EPAV

> Adapted from johnmartin-marasigan/node-js-training-rnd, the reference output Ms. Jem shared.

**Discovered in:** M01 (Dev Environment, Git & GitHub)
**Context:** A Build-to-Teach module where the submission scope is **documentation** (a learning
materials guide + a module write-up), while the module's lab describes a hands-on exercise.

## Problem

The module `brief.md` defines a lab and checkpoints (e.g. a merged PR with a resolved conflict), but
the actual submission scope is the documentation produced by Nexus EPAV. Treating the lab as a
completion requirement (a) creates scope creep and (b) pushes toward claiming unverified work.

## Pattern

1. **Separate deliverables by scope.** State explicitly, up front, which artifacts are the submission
   deliverables (the docs) and which existing artifacts are *supplementary evidence* (the real repo /
   PR).
2. **Describe supplementary evidence at its verified status.** Fetch or inspect the real artifact and
   record its true state (e.g. "PR #1 — Draft, not merged, 0 reviews; branch history shows no merge
   conflict"). Do not upgrade status.
3. **Keep the lab's techniques as concepts, not tasks.** Branching, merge conflicts, and pull requests
   stay in the learning materials as explanatory content; the write-up does not instruct the author to
   perform them.
4. **Never fabricate.** Use real facts where available and an explicit pending marker otherwise.
5. **Preserve the template contract.** Keep the write-up template's headings in the same order so the
   checkpoint reviewer and later modules can rely on the shape.
6. **Distinguish the two checkpoint sets.** Present "documentation deliverables" and "original lab
   checkpoints" as separate tables so a reader never confuses a teaching artifact with a completed
   exercise.

## Why it works

It keeps the submission both *comprehensive* and *honest*: the teaching artifact fully covers the
module's topic scope, while the status of any real artifact is stated as observed. Reviewers can judge
the documentation on its own merits without being misled about lab completion.

## Applies to

Any stage where a Nexus EPAV run produces docs rather than a running system (M01, and write-up-only
tasks generally). Complements the "preserve originals, add separate files" convention.
