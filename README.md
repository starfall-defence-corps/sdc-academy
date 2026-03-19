# Starfall Defence Corps Academy

> *Training the next generation of cyber defence operators through gamified, hands-on Ansible training.*

The Starfall Defence Corps Academy is a structured training programme that teaches Ansible automation through military-themed missions. Each mission presents a realistic scenario where you'll use Ansible to secure, configure, and manage fleet infrastructure.

## Missions

### Module 1 — Foundation

| Mission | Codename | Topic | Repo |
|---------|----------|-------|------|
| 1.1 | Fleet Census | Inventory, ad-hoc commands, fact gathering | [mission-1-1-fleet-census](https://github.com/starfall-defence-corps/mission-1-1-fleet-census) |
| 1.2 | Lock the Door | Playbook fundamentals, SSH hardening, handlers | [mission-1-2-lock-the-door](https://github.com/starfall-defence-corps/mission-1-2-lock-the-door) |

*More missions coming soon.*

## How to Enrol

1. Navigate to your assigned mission repo (see table above)
2. Click **Use this template** → **Create a new repository**
3. Clone your new repo locally
4. Follow the README in the mission repo

## Prerequisites

- **Docker Desktop** (running)
- **GNU Make**
- **Ansible** (`ansible-core`)
- **Python 3.10+**
- **Git**

## ARIA — Your Automated Reviewer

When you submit your work via Pull Request, **ARIA** (Automated Review & Intelligence Analyst) automatically reviews your submission:

1. Deterministic tests validate your work (pytest)
2. An LLM analyses your files for structure, security, and best practices
3. ARIA posts a qualitative review as a PR comment

Local testing (`make test`) runs pytest only — fast, offline, no API key needed.

## About

Built by the Starfall Defence Corps for cadets who learn by doing, not by reading.
