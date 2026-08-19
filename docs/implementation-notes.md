# Implementation Notes (internal)

> Maintainer-facing notes moved out of [COURSE_OUTLINE.md](../COURSE_OUTLINE.md). Not course material.

## Repo Structure

```
starfall-defence-corps/                    # GitHub org
├── sdc-academy/                           # Hub: README, COURSE_OUTLINE, field manuals
├── aria/                                  # Shared ARIA GitHub Action
├── mission-1-1-fleet-census/              # Template repos ↓
├── mission-1-2-lock-the-door/
├── mission-1-3-clean-sweep/
├── mission-1-4-many-ships/
├── mission-1-5-clean-house/
├── gateway-simulation/
├── mission-2-1-weapon-handling-test/
├── mission-2-2-compliance-as-code/
├── mission-2-3-fleet-sync/
├── mission-2-4-defence-in-depth/
└── master-simulation/
```

Each mission repo: `.github/workflows/` (ARIA CI), `docs/` (BRIEFING/EXERCISES/HINTS), `CHECKLIST.md`, `Makefile` (setup/test/reset/destroy), `molecule/` (Testinfra checks + ARIA reporter), `scripts/`, `workspace/` (student working dir), `README.md`. All 11 are flagged as GitHub template repos.

## ARIA

**Current**: shared GitHub Action (`starfall-defence-corps/aria@main`) referenced by every mission CI workflow. PR review needs an `ANTHROPIC_API_KEY` secret in the student's repo; without it the review step skips gracefully. Local `make test` (pytest + in-character reporter) needs no key.

**Planned — GitHub App**:

```
GitHub App (installed on the org)
  → Receives webhook on PR events across student repos
  → Backend service (hosted, e.g., Railway/Fly.io/VPS)
  → Reads PR via GitHub API
  → Analyses: lint output, Molecule results, diff review
  → Calls LLM API (key stored in backend, students never see it)
  → Posts review comment on PR via GitHub API

Cost control:
  - Max 5 ARIA reviews per mission per student
  - Lightweight model for lint-level checks
  - Full model for substantive architecture review
  - Rate limiting per GitHub user
```

## Progression Tracking (planned)

- GitHub org project board: columns = ranks
- Card per student, moved on mission completion
- **Tracking trigger**: ARIA marks mission as passed → webhook → project board update
  - Alternative: student self-reports by adding label to PR (honour system)
  - Alternative: instructor reviews and moves card manually (B2B cohort model)
- B2B dashboard: completion rates, time-per-mission, common failures

## Codespaces (planned — spike in sdc-academy#40)

Draft devcontainer for the labs (needs validation against the privileged/systemd Docker Compose setup):

```json
{
  "name": "Starfall Academy Lab",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/python:1": { "version": "3.11" }
  },
  "postCreateCommand": "pip install ansible molecule molecule-docker ansible-lint testinfra yamllint",
  "customizations": {
    "vscode": {
      "extensions": ["redhat.ansible", "redhat.vscode-yaml"]
    }
  }
}
```

## Build Sequence

Phases 1–3 (Missions 1.1–2.4 + both simulations + ARIA action) are **built and validated** (see sdc-academy issues #28–#39). Remaining:

- **Modules 3 & 4**: MOS guides (2–3 complete, rest outlined), CIS mapping — Field Manual FM-1..6 already seeded
- **Final Exercise**: Operation: Enduring Shield repo
- **Terraform/Hetzner** automation for advanced missions
- **ARIA GitHub App** + progression tracking
- **Beta**: 5–10 users (military + civilian), timing calibration from real data
- **Improvement backlog**: sdc-academy issues #40–#56
