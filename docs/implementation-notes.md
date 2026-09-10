# Implementation Notes (internal)

> Maintainer-facing notes moved out of [COURSE_OUTLINE.md](../COURSE_OUTLINE.md). Not course material.

## Repo Structure

```
starfall-defence-corps/                    # GitHub org
├── sdc-academy/                           # Hub: README, COURSE_OUTLINE, field manuals
├── aria/                                  # Shared ARIA GitHub Action
├── mission-0-reporting-for-duty/          # Template repos ↓
├── mission-1-1-fleet-census/
├── mission-1-2-lock-the-door/
├── mission-1-3-clean-sweep/
├── mission-1-4-many-ships/
├── mission-1-5-clean-house/
├── mission-1-6-inventory-from-nothing/
├── gateway-simulation/
├── mission-2-1-weapon-handling-test/
├── mission-2-2-compliance-as-code/
├── mission-2-3-fleet-sync/
├── mission-2-4-defence-in-depth/
├── mission-2-5-noise-storm/
├── mission-2-6-counterattack/
├── master-simulation/
├── mission-3-4-eyes-everywhere/           # MOS electives
└── mission-3-5-battle-rattle/
```

Each mission repo: `.github/workflows/` (ARIA CI), `docs/` (BRIEFING/EXERCISES/HINTS), `CHECKLIST.md`, `Makefile` (setup/test/reset/destroy/doctor/submit), `molecule/` (Testinfra checks + ARIA reporter), `scripts/`, `workspace/` (student working dir), `README.md`. All 17 are flagged as GitHub template repos.

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

## Codespaces (SHIPPED — validated in sdc-academy#40, rolled out in #59)

All 17 mission repos carry this devcontainer. Validated on a real 4-core codespace: labs run **unchanged** under the DinD VM (systemd `running`, graceful test fail, clean destroy); cold provision ~5 min, `make setup` ~40s. Do **not** add the `sshd` feature — it collides with lab port 2222. Prebuilds don't transfer to student template copies, so the ~5 min first boot is per-student and one-time.

Canonical config: [`mission-1-1-fleet-census/.devcontainer/devcontainer.json`](https://github.com/starfall-defence-corps/mission-1-1-fleet-census/blob/main/.devcontainer/devcontainer.json) (identical across the repos).

## Build Sequence

Phases 1–3 (Missions 0–2.6 + both simulations + ARIA action) are **built and validated** (see sdc-academy issues #28–#39). MOS-4 (`mission-3-4-eyes-everywhere`) and MOS-5 (`mission-3-5-battle-rattle`) are shipped and playable. Remaining:

- **Modules 3 & 4**: MOS-1/2/3/6 (need VM infra beyond the current lab), CIS mapping — Field Manual FM-1..6 already seeded
- **Final Exercise**: Operation: Enduring Shield repo
- **Terraform/Hetzner** automation for advanced missions
- **ARIA GitHub App** + progression tracking
- **Beta**: 5–10 users (military + civilian), timing calibration from real data
- **Improvement backlog**: sdc-academy issues #40–#56
