# verify-learning-skill

A realistic workspace used to regression-test `200166shang/learning-skill` behavior.

- `edge-model-deployment/` is the primary long-lived fixture.
- `cases/learning-workflow-regressions.md` describes learner-facing scenarios and observable invariants.

This repository is **not** a second source of truth for Skill contracts. Contract semantics live in `learning-skill`; these fixtures exist to reveal behavioral regressions, no-ops, stale caches, duplication, and UX failures during real use.

When changing `learning-skill`, prefer adding or updating a concrete regression case that demonstrates the observed failure before adding defensive workflow prose.
