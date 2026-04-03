# Starfall Defence Corps Academy

> *Training the next generation of cyber defence operators through gamified, hands-on Ansible training.*

The Starfall Defence Corps Academy is a structured training programme that teaches Ansible automation through military-themed missions. Each mission presents a realistic scenario where you'll use Ansible to secure, configure, and manage fleet infrastructure.

## Missions

### Module 1 — Foundation

| Mission | Codename | Topic | Repo |
|---------|----------|-------|------|
| 1.1 | Fleet Census | Inventory, ad-hoc commands, fact gathering | [mission-1-1-fleet-census](https://github.com/starfall-defence-corps/mission-1-1-fleet-census) |
| 1.2 | Lock the Door | Playbook fundamentals, SSH hardening, handlers | [mission-1-2-lock-the-door](https://github.com/starfall-defence-corps/mission-1-2-lock-the-door) |
| 1.3 | Clean Sweep | Service hardening, firewall, sysctl, file permissions | [mission-1-3-clean-sweep](https://github.com/starfall-defence-corps/mission-1-3-clean-sweep) |
| 1.4 | Many Ships | Variables, Jinja2 templates, conditionals, multi-OS | [mission-1-4-many-ships](https://github.com/starfall-defence-corps/mission-1-4-many-ships) |
| 1.5 | Clean House | Roles, Ansible Vault, secrets management | [mission-1-5-clean-house](https://github.com/starfall-defence-corps/mission-1-5-clean-house) |
| **Gateway** | **Operation: First Contact** | **Capstone assessment — all Module 1 skills, 75 min** | [gateway-simulation](https://github.com/starfall-defence-corps/gateway-simulation) |

## How to Enrol

1. Navigate to your assigned mission repo (see table above)
2. Click **Use this template** > **Create a new repository** (this creates your own copy)
3. Name your repo (e.g., `mission-1-1-fleet-census`), set it to **Public**
4. Clone your new repo locally and follow the README inside

> **Important**: Use **"Use this template"**, not "Fork". Templates give you a clean copy with no link back to the original.

## Prerequisites

- **Docker Desktop** (running)
- **GNU Make**
- **Ansible** (`ansible-core`)
- **Python 3.10+** (with `python3-venv` on Debian/Ubuntu)
- **Git**

> **Windows users**: Install [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) and run all commands from your WSL terminal. Docker Desktop should use the WSL2 backend.

## ARIA — Your Automated Reviewer

**ARIA** (Automated Review & Intelligence Analyst) reviews your work in two ways:

**Locally** — run `make test` to get instant pass/fail verification. No API key needed. Works offline.

**On Pull Request** — when you push your work and open a PR to `main`, ARIA reads your files and posts a qualitative code review as a PR comment (structure, security, recommendations).

To enable PR reviews, add an API key to your repo:
1. Get a key from [platform.claude.com](https://platform.claude.com/)
2. In your repo: **Settings** > **Secrets and variables** > **Actions** > **New repository secret**
3. Name: `ANTHROPIC_API_KEY`, Value: your key

If no key is configured, ARIA skips the PR review — local testing still works.

## About

Built by the Starfall Defence Corps for cadets who learn by doing, not by reading.
