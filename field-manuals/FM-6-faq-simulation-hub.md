# FM-6: FAQ & Simulation Hub
> Starfall Defence Corps — Field Manual

---

## Frequently Asked Questions

---

### 1. Why Ansible?

Ansible is agentless — nothing to install on target systems, nothing to maintain, nothing for the Voidborn to exploit. It speaks YAML, not a proprietary DSL. It is the dominant automation tool in defence and government environments for exactly these reasons. If you are hardening a fleet, Ansible is the weapon.

### 2. Do I need to know how to code?

No. YAML is structured data, not a programming language. If you can write a bulleted list with consistent indentation, you can write a playbook. Familiarity with the command line will help, but you do not need to be a developer.

### 3. Why Molecule?

"I ran it and it looked fine" is how General Snowflake operated, and we all know how that ended. Molecule gives you automated, repeatable proof that your infrastructure code works. It spins up containers, applies your role, runs assertions, and tears everything down. No human eyeballs required.

### 4. CIS vs STIG — what's the difference?

CIS benchmarks are industry consensus — developed by a global community, available to anyone, and the standard starting point for hardening. STIGs are DoD mandates — required for military and government systems, more prescriptive, and more painful. Defence organisations use STIGs. Everyone else starts with CIS Level 1 and works up from there.

### 5. Can I use this at work?

Everything in this curriculum uses open-source tools, publicly available benchmarks, and production-ready patterns. The techniques are the same ones used in real security operations. Swap "Voidborn" for your actual threat model and "fleet" for your actual inventory.

### 6. I'm not military — is this relevant?

The military framing is flavour. "Mission" instead of "lab exercise." "Fleet" instead of "server group." The skills underneath — hardening, compliance-as-code, automated testing, rolling deployments, drift detection — are universal. Every organisation with servers needs them.

### 7. How is this different from other Ansible courses?

Every example is security-focused. You will never "install nginx and serve a web page." Testing is core curriculum, not an afterthought. Git workflow is baked into every mission. You will learn to harden, verify, and automate — not just configure.

### 8. What tools do I need?

Docker Desktop, GNU Make, Python 3.10+, and Git. All free, all cross-platform. Windows users need WSL2. That is the entire toolchain. No paid licenses, no cloud accounts, no proprietary platforms.

### 9. How long does the full course take?

Module 1 (Missions 1.1–1.5): approximately 6–8 hours. Module 2 (Missions 2.1–2.4): approximately 8–12 hours. Gateway Simulation: 75 minutes. Master Simulation: 3.5 hours. Total curriculum: roughly 20–25 hours depending on your pace.

### 10. What rank do I earn?

You begin at **Cadet**. Completing Module 1 and the Gateway Simulation earns **Ensign**. Module 2 missions progress you through **Lieutenant JG** and **Lieutenant**. Completing the Master Simulation earns **Lt. Commander**. Each rank is earned, not given.

### 11. Can I skip ahead?

Each mission builds directly on skills from the previous one. Mission 1.4 assumes you can write a playbook (1.2) and manage services (1.3). The simulations test everything at once. Skipping missions leaves gaps, and the simulations will expose them.

### 12. What if ARIA fails my work?

Read the ARIA output carefully — it tells you exactly what failed and why. Fix the identified issues, run `make test` again, and resubmit. There is no penalty for retries. ARIA is not here to punish you. She is here to make sure your work is correct before it reaches production.

---

## Simulation Hub

---

All simulations are timed, proctored by ARIA, and designed to test accumulated skills under pressure. There is no partial credit. Every mission within a simulation must pass.

| Simulation | Codename | Time | Skills Tested | Rank Earned | Repo |
|------------|----------|------|---------------|-------------|------|
| Gateway | Operation: First Contact | 75 min | All Module 1 (inventory, playbooks, roles, Vault, multi-OS) | Ensign | `gateway-simulation` |
| Master | Operation: Iron Curtain | 3.5 hrs | All Module 2 (Molecule, CIS, rolling updates, CI/CD, incident response) | Lt. Commander | `master-simulation` |

---

### Gateway Simulation — Operation: First Contact

**What to expect**: Three missions in 75 minutes. You inherit a compromised forward observation post — three nodes, SSH wide open, firewalls down, insecure services running. You will assess, harden, and encrypt. No hints file. No hand-holding. Everything from Module 1, applied under the clock.

**Prerequisites**: Missions 1.1 through 1.5 completed. You must be comfortable with inventory, ad-hoc commands, playbooks, roles, handlers, templates, Vault, and multi-OS support.

**How to attempt**:
```bash
git clone https://github.com/starfall-defence-corps/gateway-simulation.git
cd gateway-simulation
make setup
source venv/bin/activate
# Read docs/BRIEFING.md, start your timer, begin.
```

**Performance tiers**:

| Total Time | Rating |
|------------|--------|
| Under 45 min | **Ace Cadet** |
| 45–55 min | **Distinguished** |
| 55–65 min | **Qualified** |
| 65–75 min | **Passed** |
| 75+ min | **RTB** (Return to Base — retry) |

---

### Master Simulation — Operation: Iron Curtain

**What to expect**: Four missions in 3.5 hours. You inherit General Snowflake's hand-built fleet — six nodes, no compliance baseline, no tests, no automation. You will assess all six against CIS benchmarks with Lynis, build a tagged hardening role with Molecule tests, construct a CI/CD pipeline with drift detection, and respond to a live incident. This is everything.

**Prerequisites**: All of Module 1, all of Module 2 (Missions 2.1 through 2.4), and the Gateway Simulation. You must be comfortable with Molecule, Testinfra, CIS controls, rolling updates, ansible-lint, CI pipelines, and incident investigation.

**How to attempt**:
```bash
git clone https://github.com/starfall-defence-corps/master-simulation.git
cd master-simulation
make setup
source venv/bin/activate
# Read docs/BRIEFING.md, start your timer, begin.
```

**Performance tiers**:

| Total Time | Rating |
|------------|--------|
| Under 2.5 hrs | **Outstanding** |
| 2.5–3 hrs | **Excellent** |
| 3–3.5 hrs | **Qualified** |
| 3.5–4 hrs | **Passed** |
| 4+ hrs | **Return to AIT** (retry) |

---

*SDC Cyber Command — 2187 — UNCLASSIFIED // Training Use Only*
