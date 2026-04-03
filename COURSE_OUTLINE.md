# Starfall Defence Corps Academy
## Blue Team Ansible Automation Training

> *"If you have to SSH into a box, you've already lost."*

---

## The Premise

The year is 2187. Humanity's interstellar colonies are defended by the **Starfall Defence Corps** — an elite cyber defence force that keeps fleet systems hardened against the **Voidborn**, an enemy faction that exploits every misconfiguration, every unpatched service, every hardcoded credential.

Your weapon system? **ANSIBLE** — the Automated Network for Secure Infrastructure, Baseline Lockdown & Enforcement.

Your ship AI? **ARIA** (Automated Review & Intelligence Analyst) — she reviews every PR submission, catches what you missed, and delivers feedback with the warmth of an automated email from HR. Helpful. Thorough. Will not tell you "good job" unless she means it.

You are a new cadet at the Academy. Your fleet is only as secure as its weakest node. Time to earn your rank.

---

## Who This Is For

**Primary**: Military cyber defence teams (Blue Team analysts, SOC operators, sysadmins in uniform). You've seen what happens when hardening is done by hand. You're tired of it.

**Secondary**: Anyone in IT security who wants to automate hardening, compliance, and detection — and learn Ansible properly in the process, not through another "install nginx" tutorial.

**Prerequisites**:
- Basic Linux CLI (you can `ls`, `cd`, `grep`, `ssh` without Googling)
- A GitHub account
- A tolerance for having your PRs reviewed by an AI that doesn't do praise

---

## A Note on Idempotency and Mission Command

Ansible's core principle is **idempotency**: you declare the desired end state, not the steps to get there. "SSH must be configured like this" — not "open this file, find this line, change it to this."

If you've served in any military that uses **Mission Command** (Auftragstaktik), this will feel familiar. You define the intent and the desired outcome. You don't prescribe every step. Ansible is Mission Command for infrastructure:

> *"The SSH service will be hardened to these specifications"* — not *"Open /etc/ssh/sshd_config, go to line 47, change PermitRootLogin to no, save, restart sshd"*

You define WHAT. Ansible determines HOW. If it's already done, it does nothing. Run it ten times, same result. That's idempotency.

---

## The Technical Stack

| Tool | Role in Academy | Why |
|------|----------------|-----|
| GitHub | Mission repos, PRs, code review, progression tracking | Real-world workflow from day 1 |
| GitHub Actions | CI pipeline — runs on every PR | Automated testing before students fully understand it |
| GitHub Codespaces | Zero-setup lab environment | One-click start, any device, 60hrs/month free |
| Docker | Target containers for Molecule testing | Consistent across all platforms including Apple Silicon |
| Molecule + Testinfra | Role testing — Weapon Handling Test | You don't deploy what you haven't tested |
| ansible-lint | Code quality enforcement | Inline PR annotations teach style naturally |
| Lynis / OpenSCAP | Security scoring with delta from baseline | Measurable hardening progress |
| Elastic Stack | Fleet-wide logging, detection, alerting | Industry-standard SIEM, core to MOS 4 |
| pfSense | Firewall management and automation | Real network security, core to MOS 2 |
| ARIA (GitHub App + LLM) | Automated qualitative PR review in character | Feedback worth reading |
| Terraform + Hetzner | Cloud VMs for advanced missions (Lieutenant+) | Real multi-host when Docker isn't enough |

---

## Student Workflow: Fork → Branch → PR → ARIA

Every mission follows the same real-world Git workflow:

```
1. Student FORKS the mission template repo to their own GitHub account
2. Student opens a CODESPACE on their fork (or clones locally)
3. Student creates a BRANCH for their work
4. Student writes code, commits, pushes to their fork
5. Student opens a PR within their own fork (branch → main)
6. CI runs automatically (GitHub Actions: lint + Molecule)
7. ARIA reviews the PR (triggered by GitHub App webhook)
8. Student addresses feedback, pushes fixes
9. When CI passes and ARIA approves → mission complete
```

**Why fork, not branch on the org repo?**
- Students can't interfere with each other's work
- Template repo stays clean — never polluted with student code
- Each student has their own full copy to experiment with
- PRs within the fork keep review history private to the student
- Mirrors real open-source contribution workflow

**The PR is the submission.** It's never merged to the org's template repo. The student's fork is their portfolio — every mission PR with CI results and ARIA feedback preserved.

### ARIA Architecture

ARIA runs as a **GitHub App** installed on the Academy's GitHub organisation:

```
Student opens PR on their fork
  → GitHub sends webhook to ARIA backend (hosted service)
  → ARIA backend:
      1. Reads PR diff and file contents via GitHub API
      2. Runs structured analysis (lint results, test results, security checks)
      3. Sends to LLM API with system prompt (Starfall lore + Ansible expertise)
      4. Posts review comment on the PR via GitHub API
  → Student sees ARIA's feedback as a PR comment
```

**API key handling**: The LLM API key lives in the ARIA backend service — students never see it, never need it. The GitHub App authenticates via its own installation token. One deployment serves all students across all forks.

**Cost control**: Rate limiting per student (e.g., max 5 ARIA reviews per mission), review only on PR open/update events, lightweight LLM model for initial lint-level feedback, full model for substantive review.

---

## Rank Progression (Starfleet)

| Rank | Module | Earned By | Scaffolding |
|------|--------|-----------|-------------|
| **Cadet** | 1.1–1.2 | Enrolling | Full: inventory provided, skeleton playbook, all hints, tests visible |
| **Ensign** | 1.3–1.5 + Gateway | Passing Gateway Simulation | Reduced: inventory provided, minimal skeleton, some hints |
| **Lieutenant JG** | 2.1–2.2 | Completing WHT + Compliance | Minimal: blank playbook, no hints, tests visible |
| **Lieutenant** | 2.3–2.4 | Completing fleet ops + pipeline | Mission briefing + pass/fail only |
| **Lt. Commander** | Master Sim | Passing Operation: Iron Curtain | Scenario description only — full autonomy |
| **Commander** | Module 3 | Completing 2+ MOS specializations | Mission briefing only — design own approach |
| **Captain** | Final Exercise | Passing Operation: Enduring Shield | Red team scenario — write own requirements, then implement |

*Fleet Captain through Fleet Admiral: reserved for instructors, mentors, and course contributors.*

### Scaffolding Principle

At Cadet, you fill in blanks. By Captain, you're writing the requirements yourself. This mirrors professional development: juniors follow procedures, seniors design them.

---

## Spaced Repetition: Every Skill Builds on the Last

**Every new mission reuses skills from previous missions.** Nothing learned once and forgotten.

| Mission | New Skill | Reuses From |
|---------|-----------|-------------|
| 1.1 Fleet Census | Inventory, ad-hoc, basic facts | — (first mission) |
| 1.2 Lock the Door | Playbooks, modules, tasks, handlers | 1.1 inventory |
| 1.3 Clean Sweep | Services, packages, file management | 1.1 inventory, 1.2 playbook structure |
| 1.4 One Playbook Many Ships | Variables, facts, templates, conditionals | 1.1–1.3 everything (new OS families) |
| 1.5 Clean House | Roles, Vault, Git workflow | Restructures ALL 1.2–1.4 into roles |
| Gateway | All Module 1 combined | Everything 1.1–1.5 under pressure |
| 2.1 WHT | Molecule testing | Tests the roles from 1.5 |
| 2.2 Baseline | CIS benchmarks, compliance | Extends roles from 1.5 with new controls |
| 2.3 Fleet Sync | Multi-host orchestration | Deploys roles from 1.5/2.2 across fleet |
| 2.4 Defence in Depth | CI/CD automation | Automates testing from 2.1 + deployment |
| Master Sim | Everything | All skills, all tools, under time pressure |

*By the Master Simulation, students have written the same core tasks (SSH hardening, service management, file permissions) in 5+ different contexts.*

---

## Ansible Facts: Know Your Fleet

Ansible automatically gathers **facts** about every host — OS, IPs, memory, CPU, disks, network interfaces. This is your reconnaissance.

```yaml
ansible_os_family: "Debian"          # or "RedHat"
ansible_distribution: "Ubuntu"        # or "Rocky"
ansible_distribution_version: "22.04"
ansible_default_ipv4:
  address: "10.0.1.5"
  interface: "eth0"
ansible_hostname: "web-01"
ansible_memtotal_mb: 4096
ansible_processor_vcpus: 2
```

**Taught progressively**:
- **1.1**: Basic inspection — `ansible -m setup hostname`
- **1.4**: Conditionals — `when: ansible_os_family == "Debian"`
- **1.4**: Templates — `{{ ansible_default_ipv4.address }}`
- **2.2**: Custom facts — `.fact` files on hosts
- **2.3**: Fact caching — `fact_caching = jsonfile` for fleet performance

---

## The Villain Gallery

The Voidborn's agents — each a real-world anti-pattern:

| Villain | Anti-Pattern | First Appears |
|---------|-------------|---------------|
| **Agent Chmod-777** | "Just give it all the permissions, it works now" | 1.1 |
| **The SSH Root Fairy** | Leaves root login enabled everywhere | 1.2 |
| **Corporal Copy-Paste** | Copies configs from forums without reading them | 1.4 |
| **Colonel Hardcoded-Password** | Secrets in plaintext, passwords in repos | 1.5 |
| **Private YOLO-Deploy** | Pushes to prod untested. Friday. 16:59. | 2.1 |
| **Captain Unpatched** | "If it works, don't update it" (last patched: 2019) | 2.2 |
| **The Phantom Logstash** | Broke the Elastic Stack, left the fleet blind | MOS 4 |
| **General Snowflake** | Every server hand-configured. "Documentation? It's in my head." | Master Sim |

*General Snowflake is the final boss. The General's motto: "But this server is special." No. It isn't.*

---

# Module 1: Basic Training (Foundation)

> **Rank**: Cadet → Ensign
> **Time**: 12–15 hours
> **Lab**: Docker + Molecule via GitHub Codespaces
> **Concludes with**: Gateway Simulation — "Operation: First Contact"

## 1.1 Reporting for Duty (SSH, Inventory & Ad-Hoc Commands)

**Rank**: Cadet
**Villain**: Agent Chmod-777

### What You'll Learn
- How Ansible connects (SSH-based, agentless — nothing to install on targets)
- Writing inventory files — fleet asset registry
- Grouping hosts logically
- Running ad-hoc commands
- Basic facts gathering — `ansible -m setup`
- What idempotency means

### Briefing
*"Cadet, the fleet's asset registry is chaos. Half our ships aren't catalogued. Agent Chmod-777 has been leaving permissions set to 777 — anyone can read, write, execute anything. Find them. Fix them."*

### Content
1. **Guide**: "Reporting for Duty — Your First Connection"
   - What Ansible is (and isn't)
   - Agentless: SSH, nothing on targets
   - Inventory: hosts, groups, children, variables
   - Ad-hoc: `ansible all -m ping`, `ansible web -m shell -a "uptime"`
   - Facts: `ansible hostname -m setup`
   - `ansible.cfg` configuration
   - Common mistakes: SSH key permissions

2. **Practice Field**: "The Asset Registry"
   - 3 Docker containers as fleet nodes
   - Write inventory, verify connectivity
   - Run `setup`, inspect facts
   - Fix broken permissions
   - Backup for reset

3. **Mission**: "Fleet Census"
   - Fork → Codespace → branch → inventory → ad-hoc verify → PR
   - CI validates inventory + connectivity + permissions
   - ARIA reviews

4. **Checklist**:
   - [ ] Read: Reporting for Duty
   - [ ] Practice: Asset Registry (all 3 nodes connected)
   - [ ] Practice: `ansible -m setup` — identify OS family per node
   - [ ] Mission: Fleet Census PR — CI passing
   - [ ] ARIA feedback addressed

---

## 1.2 Your First Operations Order (Playbook Fundamentals)

**Rank**: Cadet
**Villain**: The SSH Root Fairy
**Builds on**: 1.1 inventory

### What You'll Learn
- Playbook structure (YAML, plays, tasks)
- Core modules: `lineinfile`, `copy`, `file`, `user`
- Task ordering, handler notification
- Output: `changed`, `ok`, `failed`, `skipped`
- `--check` (dry run) and `--diff`

### Briefing
*"The SSH Root Fairy has visited again — root login enabled on every node. The digital equivalent of leaving your front door open with a sign saying 'COME IN.' Write your first OPORD (playbook) to lock down SSH fleet-wide."*

### Content
1. **Guide**: "Writing Your First OPORD"
   - Playbook anatomy, YAML survival guide
   - Module docs, `ansible-doc`
   - Handlers, `--check`, `--diff`
   - Common mistakes

2. **Practice Field**: "The YAML Minefield"
   - 10 broken YAML snippets, progressive difficulty

3. **Mission**: "Operation: Lock the Door"
   - **Uses 1.1 inventory**
   - Skeleton provided
   - Disable root login, password auth, set LoginGraceTime
   - Molecule verifies, ARIA reviews

4. **Checklist**:
   - [ ] Read: Writing Your First OPORD
   - [ ] Practice: YAML Minefield (all 10)
   - [ ] Mission: Lock the Door — CI passing
   - [ ] Understand: `changed` vs `ok`
   - [ ] Can explain: why handlers exist

---

## 1.3 Managing the Fleet (Services, Packages & Config Files)

**Rank**: Cadet → Ensign candidate
**Builds on**: 1.1 inventory + 1.2 playbook structure

### What You'll Learn
- Service management: `service` / `systemd`
- Package management: `apt`, `dnf`
- Copying config files: `copy` module
- File permissions/ownership: `file` module
- Combining modules in one playbook

### Briefing
*"Half the fleet runs services nobody asked for. FTP. Telnet. The firewall? Not running. Not enabled. Fix this."*

### Content
1. **Guide**: "Managing Your Fleet's Services"
   - `state: started` vs `restarted` (idempotency)
   - `state: present` vs `absent` vs `latest`
   - File management, `copy` module

2. **Practice Field**: "The Service Parade"
   - 3 containers with issues
   - Ensure firewall, remove telnet/ftp
   - Copy firewall rules, set permissions

3. **Mission**: "Operation: Clean Sweep"
   - **Uses 1.1 inventory, extends 1.2 patterns**
   - Remove unnecessary services/packages
   - Firewall configured and running
   - Permissions on critical files
   - Hardened `sysctl.conf` deployed

4. **Checklist**:
   - [ ] Read: Managing Your Fleet's Services
   - [ ] Practice: Service Parade (all 3 fixed)
   - [ ] Mission: Clean Sweep — CI passing
   - [ ] Can do: install + enable + start in one playbook
   - [ ] Understand: `started` is idempotent, `restarted` is not

---

## 1.4 Adapting to Conditions (Variables, Facts, Templates & Conditionals)

**Rank**: Ensign candidate
**Villain**: Corporal Copy-Paste
**Builds on**: 1.1–1.3 (making them flexible)

### What You'll Learn
- Variables (playbook, inventory, group_vars, host_vars)
- Variable precedence
- Deep facts usage in conditionals and templates
- Jinja2 templates
- `when` conditionals
- Registered variables

### Briefing
*"Fleet runs mixed OS — Ubuntu and Rocky. Corporal Copy-Paste has written 47 separate playbooks. No more. One playbook. All ships."*

### Content
1. **Guide**: "Adapting to the Fleet"
   - Variable hierarchy and precedence (the 5 levels that matter)
   - Facts: `ansible_os_family`, `ansible_distribution`, `ansible_default_ipv4`
   - Custom facts
   - Jinja2: `{{ }}`, `{% if %}`, `{% for %}`, filters
   - Template module, conditionals, registered variables

2. **Practice Field**: "The Template Forge"
   - Template `sshd_config` with variables
   - Template firewall (ufw vs firewalld)
   - Precedence debugging exercises

3. **Mission**: "Operation: One Playbook, Many Ships"
   - **Reuses 1.1 inventory** (with OS group_vars)
   - **Extends 1.2 + 1.3 tasks** (now templated)
   - Both OS families, Molecule on both
   - Minimal skeleton

4. **Checklist**:
   - [ ] Read: Adapting to the Fleet
   - [ ] Practice: Template Forge
   - [ ] Mission: One Playbook Many Ships — passes both OS
   - [ ] Can write: `when` with `ansible_os_family`
   - [ ] Can explain: 5 precedence levels that matter

---

## 1.5 Standard Operating Procedures (Roles, Vault & Git Workflow)

**Rank**: Ensign candidate
**Villain**: Colonel Hardcoded-Password
**Builds on**: ALL of 1.1–1.4, restructured

### What You'll Learn
- Ansible roles (tasks, handlers, templates, defaults, vars, meta)
- `ansible-galaxy init`
- Ansible Vault — the **Crypto Cell** (encrypted secrets storage)
- Full Git workflow: branch → commit → PR → review → merge
- `.gitignore` done right

### Briefing
*"Colonel Hardcoded-Password embedded database credentials on three fleet nodes. Plaintext. On the filesystem. This ends now. Everything you've built — SSH hardening, services, templates — restructure into a proper SOP (role). Secrets go in the Crypto Cell (Vault). Everything through change authorisation (PRs)."*

### Content
1. **Guide**: "Building Your SOPs"
   - Role directory structure
   - `defaults/` vs `vars/`
   - `ansible-galaxy init`
   - **Ansible Vault — the Crypto Cell**: `ansible-vault encrypt`, `create`, vault password files
   - Git workflow, `.gitignore`

2. **Practice Field**: "The Crypto Cell"
   - Create vault-encrypted variables
   - Decrypt, edit, re-encrypt
   - Reference vault vars in playbooks
   - Detective exercise: find Colonel Hardcoded-Password's plaintext credentials on the fleet filesystem
     - `ansible all -m shell -a "cat /opt/fleet-db-creds.txt"` — why plaintext secrets on disk are a compromise waiting to happen

3. **Mission**: "Operation: Clean House"
   - **Convert 1.2–1.4 playbooks into a role** (same tasks, proper structure)
   - Vault for sensitive values
   - Full Git workflow
   - No skeleton — `ansible-galaxy init`

4. **Checklist**:
   - [ ] Read: Building Your SOPs
   - [ ] Practice: Crypto Cell (vault create, encrypt, decrypt)
   - [ ] Practice: Found the Colonel's secret in git history
   - [ ] Mission: Clean House — role structure, vault, CI green
   - [ ] Can do: branch → PR workflow without instructions

---

## Gateway Simulation: "Operation: First Contact"

> **The Voidborn have compromised a forward observation post. Three nodes exposed. 75 minutes. Everything you've learned.**

**Prerequisites**: All Module 1 PRs complete
**Format**: Fork template → branch → PR → CI must pass

### Mission 1: Reconnaissance (20 min)
- 3 containers with misconfigurations
- Write inventory with group_vars
- Ad-hoc + facts to assess state
- Document in `RECON.md`

### Mission 2: Hardening (30 min)
- Ansible role: SSH, services, permissions, templates, conditionals
- Handles both OS families
- Molecule passes on all nodes

### Mission 3: Secure & Submit (25 min)
- Vault-encrypt sensitive values
- No plaintext secrets (including git history)
- PR with description, CI passes, ARIA approves

### Performance Tiers
| Time | Rating |
|------|--------|
| Under 45 min | Ace Cadet |
| 45–55 min | Distinguished |
| 55–65 min | Qualified |
| 65–75 min | Passed |
| 75+ min | RTB — retry |

> **Rank Earned**: Ensign
> **Badge**: Starfall Basic Training

---

# Module 2: Advanced Individual Training (Proficiency)

> **Rank**: Lieutenant JG → Lieutenant → Lt. Commander
> **Time**: 16–20 hours
> **Lab**: Codespaces + Terraform VMs for 2.3+
> **Concludes with**: Master Simulation — "Operation: Iron Curtain"

## 2.1 Weapon Handling Test (Molecule Deep Dive)

**Rank**: Lieutenant JG (blank playbook, no hints, tests visible)
**Villain**: Private YOLO-Deploy
**Builds on**: Role from 1.5

### What You'll Learn
- Writing Molecule scenarios from scratch
- Testinfra test cases
- Multiple platforms
- Molecule lifecycle: create → converge → verify → destroy
- Test-driven infrastructure: tests FIRST
- Molecule in GitHub Actions

### Briefing
*"Private YOLO-Deploy pushed a 'hardening' role to production. No tests. No review. Friday. 16:59. Three nodes down. 'I ran it locally and it worked.' This is why we have the Weapon Handling Test. You don't deploy until Molecule confirms it works."*

### Content
1. **Guide**: "Weapon Handling Test — Prove It Before You Deploy It"
   - Molecule architecture, `molecule.yml` from scratch
   - Testinfra: `host.file()`, `host.service()`, `host.socket()`, `host.package()`
   - Multiple scenarios, Red-Green-Refactor, CI integration

2. **Obstacle Course 1**: "The WHT Range"

   > START YOUR TIMER

   **Mission 1**: 5 Testinfra tests given → write the role that passes them
   **Mission 2**: Working untested role → write tests that catch the planted bug

   > STOP YOUR TIMER

   | Time | Rating |
   |------|--------|
   | Under 30 min | Sharpshooter |
   | 30–40 min | Marksman |
   | 40–50 min | Qualified |
   | 50–60 min | Basic |
   | 60+ min | Weapon Returned — retry |

   > **Badge**: WHT — Passed

3. **Mission**: "Operation: Test Everything"
   - **Write Molecule tests for your 1.5 role**
   - SSH, services, packages, permissions, firewall
   - Multi-platform, no skeleton

---

## 2.2 Compliance as Code (CIS Benchmarks & Security Baselines)

**Rank**: Lieutenant JG
**Villain**: Captain Unpatched
**Builds on**: Tested role from 2.1

### What CIS and STIG Are

**CIS Benchmarks** (Center for Internet Security): industry-consensus security baselines. Level 1 = basic hardening. Level 2 = stricter, may limit functionality. Free to download.

**STIGs** (Security Technical Implementation Guides): U.S. Department of Defense equivalent. More detailed, more prescriptive, required for military systems. Same concept as CIS, stricter.

Both: **documented, measurable, repeatable security configuration that can be audited.** Exactly what Ansible is for.

### Content
1. **Guide**: "The Fleet Hardening Standard"
   - CIS sections, levels, STIG overview
   - Mapping controls to tasks (worked example)
   - Tags: `--tags "ssh,filesystem"`
   - OpenSCAP scanning and delta reporting

2. **Practice Field**: "The Compliance Dashboard"
   - Unhardened vs hardened containers, OpenSCAP delta
   - Fix 5 controls, rescan, measure improvement

3. **Obstacle Course 2**: "The Compliance Gauntlet"

   > START YOUR TIMER

   **Mission 1**: 10 CIS controls → Ansible tasks with tags
   **Mission 2**: OpenSCAP before/after — 80%+ improvement

   > STOP YOUR TIMER

   | Time | Rating |
   |------|--------|
   | Under 35 min | Gold Standard |
   | 35–45 min | Compliant |
   | 45–55 min | Improving |
   | 55–70 min | Needs Work |
   | 70+ min | Audit Failed — retry |

   > **Badge**: Compliance Officer

4. **Mission**: "Operation: Baseline"
   - **Extend 2.1 role** with CIS Level 1 (spaced repetition)
   - Filesystem, auth, network, logging, SSH
   - Molecule-tested, OpenSCAP as CI artifact, tagged

---

## 2.3 Fleet-Wide Operations (Multi-Host & Orchestration)

**Rank**: Lieutenant (mission briefing + pass/fail only)
**Builds on**: CIS-hardened, tested role → deployed across real fleet

### What You'll Learn
- Complex inventories, dynamic plugins
- `delegate_to`, `run_once`, `serial`
- Rolling updates, zero downtime
- `ansible-pull`: autonomous hardening
- Error handling: `block/rescue/always`
- Fact caching, performance tuning

### Content
1. **Guide**: "Commanding the Fleet"
   - Dynamic inventory, delegation, `serial`, `ansible-pull`
   - Error handling, fact caching, performance

2. **Adventure Simulation**: "The Rolling Hardening"
   - 4 servers behind LB, CIS updates, zero downtime
   - Phase 1: Manual rolling
   - Phase 2: Automate with `serial` + delegation
   - Phase 3: One node fails mid-roll — handle it

3. **Mission**: "Operation: Fleet Sync"
   - Terraform VMs (4+ on Hetzner)
   - Dynamic inventory, rolling hardening
   - **Same role from Module 1, extended through 2.1–2.2**

---

## 2.4 The Automated Defence Line (CI/CD Pipelines)

**Rank**: Lieutenant
**Villain**: Private YOLO-Deploy (final confrontation)
**Builds on**: Everything — entire workflow automated

### What You'll Learn
- GitHub Actions pipeline for Ansible
- Stages: lint → test → scan → deploy
- Branch protection, required checks
- Approval gates, scheduled drift detection

### Content
1. **Guide**: "Building the Automated Defence Line"
   - GitHub Actions, matrix testing
   - Branch protection, scheduled runs
   - CI secrets, notifications

2. **Obstacle Course 3**: "Operation: Pipeline"

   > START YOUR TIMER

   **Mission 1**: Actions workflow — lint, Molecule 2 OS, OpenSCAP artifact
   **Mission 2**: Branch protection, required review, weekly scan

   > STOP YOUR TIMER

   | Time | Rating |
   |------|--------|
   | Under 30 min | Elite Ops |
   | 30–40 min | Combat Ready |
   | 40–55 min | Field Ready |
   | 55–70 min | Basic |
   | 70+ min | YOLO-Deploy Wins Again — retry |

   > **Badge**: Pipeline Operator

3. **Mission**: "Operation: Defence in Depth"
   - Full pipeline for CIS role
   - Lint, multi-OS Molecule, OpenSCAP, deploy with approval
   - Branch protection, weekly drift detection
   - ARIA reviews pipeline design

---

## Master Simulation: "Operation: Iron Curtain"

> **General Snowflake. 6 servers, every one different. Hand-built. Undocumented. Your mission: uniform, tested, automated compliance.**

**Prerequisites**: All Module 2 complete
**Format**: 6 Terraform VMs (Ubuntu + Rocky). 3.5 hours. One repo. One pipeline.

### Mission 1: Assessment (40 min)
- 6 VMs, mixed OS, different misconfigurations
- OpenSCAP baseline, inventory, facts, `ASSESSMENT.md`

### Mission 2: Remediation (75 min)
- Roles → CIS Level 1 on all 6
- Variables, templates, conditionals, Vault
- Molecule on both OS families
- OpenSCAP 60%+ improvement

### Mission 3: Automation (60 min)
- Full CI/CD pipeline
- Branch protection, drift detection
- PR with delta report

### Mission 4: Incident (35 min — surprise after Mission 3)
- One VM "compromised" (config altered)
- Detect, identify, remediate, document in `INCIDENT.md`

| Total Time | Rating |
|------------|--------|
| Under 2.5 hrs | Outstanding |
| 2.5–3 hrs | Excellent |
| 3–3.5 hrs | Qualified |
| 3.5–4 hrs | Passed |
| 4+ hrs | Return to AIT — retry |

> **Rank Earned**: Lt. Commander
> **Badge**: Iron Curtain — Master Operator

---

# Module 3: MOS Specialization

> Available from Ensign. Mission briefing only. 2+ MOS = Commander rank.

## MOS 1: Windows Hardening
- WinRM, Kerberos/CredSSP/NTLM
- `win_feature`, `win_service`, `win_regedit`, `win_security_policy`
- CIS Windows Server, AD hardening
- Group Policy vs Ansible
- Molecule with Windows containers

## MOS 2: Network & Firewall Automation
- `ios_config`, `nxos_config`, `eos_config`, `junos_config`
- **pfSense**: `pfsensible.core` — rules, aliases, VLANs
- Network inventory, config backup/restore
- ACL hardening, compliance

## MOS 3: Container & Kubernetes Security
- CIS Docker Benchmark
- K8s hardening (kubeadm, RBAC, network policies)
- Trivy scanning, Falco runtime security
- Pod security standards

## MOS 4: Detection & Monitoring (Elastic Stack Focus)

**Villain**: The Phantom Logstash — *broke the Elastic Stack and left the fleet blind*

### Deployment
- Elasticsearch cluster: nodes, shards, security, TLS
- Kibana deployment and configuration
- Filebeat/Metricbeat agent rollout fleet-wide
- Detection rule deployment and lifecycle
- Index lifecycle management

### Hardening the Stack Itself
This is critical and often overlooked. During a real incident, the red team went after Elastic first — broke it, left the fleet with no visibility. You harden the monitoring before you trust it.

- **Elasticsearch security**: TLS between nodes, authentication enabled, minimal roles
- **Kibana hardening**: TLS, authentication, session timeouts, space-based access control
- **Filebeat security**: encrypted transport, certificate pinning, output authentication
- **Network isolation**: Elastic cluster on dedicated VLAN/subnet, access restricted to collectors and analysts
- **Backup and recovery**: Snapshot lifecycle, repository configuration, tested restoration
- **Monitoring the monitor**: Elasticsearch cluster health checks via Ansible, alerts if Elastic goes down — because you can't detect an intrusion if your SIEM is offline
- **Molecule tests**: verify TLS is enforced, default passwords changed, unnecessary plugins removed, health endpoint responding

### Detection
- Auditd rules via Ansible
- Centralised logging (rsyslog/syslog-ng)
- Wazuh as complement
- Alert routing

## MOS 5: Incident Response Automation
- Forensic collection (memory, disk, logs)
- Containment: isolation, credential rotation
- IOC scanning fleet-wide
- IR runbooks: human decides, Ansible executes
- Integration: TheHive, Shuffle, Elastic SIEM

## MOS 6: Cloud Infrastructure Hardening
- AWS: Security Groups, IAM, CloudTrail, GuardDuty
- Azure: NSGs, Key Vault, Defender
- Cloud CIS Benchmarks
- Dynamic inventory, Terraform + Ansible

---

# Module 4: Field Manual (Always Available)

## FM-1: Tool Comparison
| Tool | Purpose | Best For |
|------|---------|----------|
| Ansible | Config management, hardening | Agentless, multi-OS |
| Terraform | Infrastructure provisioning | Cloud resources |
| Molecule | Role testing | Verification |
| OpenSCAP | Compliance scanning | CIS/STIG evidence |
| Elastic Stack | SIEM, logging, detection | Fleet visibility |
| pfSense | Firewall | Network perimeter |
| Wazuh | HIDS + SIEM | Host detection |

## FM-2: Module Reference (Blue Team)
Searchable by category. Each card: FQCN, example, security use case, mistakes.

## FM-3: CIS → Ansible Mapping
Ubuntu 22.04 + Rocky 9. Control → Module → Example → Level.

## FM-4: Simulation Hub
All missions with replay and personal best tracking.

## FM-5: Glossary
- **Idempotency**: Run twice = same result. The entire point.
- **Facts**: What Ansible discovers about a host. Not opinions.
- **Handlers**: Tasks that run only when notified.
- **Crypto Cell**: Ansible Vault — encrypted secrets storage.
- **Molecule**: Your role's Weapon Handling Test.
- **CIS Benchmark**: Industry security baseline. The minimum.
- **STIG**: DoD security baseline. CIS but stricter.
- **Drift**: Actual ≠ declared state. Why we schedule compliance runs.
- **Snowflake server**: Hand-configured, undocumented, irreplaceable.
- **Infrastructure as Code**: In files, in git. Not in someone's head.
- **Mission Command**: Define objective, let executor determine how. `state: present` is Mission Command for servers.

## FM-6: FAQ
- **Why Ansible?** Agentless, YAML, dominant in defence/government.
- **Need to code?** No. YAML + CLI.
- **Why Molecule?** "I ran it and it looked fine" is how General Snowflake won.
- **CIS vs STIG?** CIS = industry. STIG = DoD. Defence = STIG. Everyone else = CIS L1.
- **Use at work?** Open-source tools, public benchmarks, production patterns.
- **Not military?** Framing is flavour. "Mission" instead of "exercise."
- **How different?** Every example is security. Testing is core. Git workflow baked in. Not "install nginx."

---

# Final Exercise: "Operation: Enduring Shield"

> Available after Lt. Commander. Pass → Captain.

### Setup
6 nodes, running 48 hours. Red team has planted:
- 3 backdoor accounts
- 2 modified SSH configs
- 1 unauthorised service
- 1 firewall rule change
- Tampered log config
- Breadcrumbs in logs

### Mission
Using **only playbooks/roles from the course**:
1. **Detect**: Compliance + monitoring automation → find anomalies
2. **Contain**: Isolate, disable accounts, block services
3. **Remediate**: Restore CIS baseline
4. **Report**: PR with incident report, delta, evidence

**Time**: 2.5 hours. Scored on detection + containment + remediation + report.

> **Rank Earned**: Captain
> **Badge**: Starfall Defence Corps — Captain's Commission
> *"You detect, contain, and remediate through code. General Snowflake's fleet fears you."*

---

# Implementation

## Repo Structure
```
starfall-academy/                          # GitHub org
├── mission-1-1-fleet-census/              # Template repos
├── mission-1-2-lock-the-door/
├── mission-1-3-clean-sweep/
├── mission-1-4-one-playbook-many-ships/
├── mission-1-5-clean-house/
├── gateway-operation-first-contact/
├── mission-2-1-test-everything/
├── mission-2-2-baseline/
├── mission-2-3-fleet-sync/
├── mission-2-4-defence-in-depth/
├── master-operation-iron-curtain/
└── final-operation-enduring-shield/
```

Each: `.github/workflows/ci.yml`, `.devcontainer/`, `BRIEFING.md`, `HINTS.md` (rank-dependent), `molecule/`, `README.md`.

## ARIA as GitHub App
```
Architecture:
  GitHub App (installed on starfall-academy org)
    → Receives webhook on PR events across student forks
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

## Progression Tracking
- GitHub org project board: columns = ranks
- Card per student, moved on mission completion
- **Tracking trigger**: ARIA marks mission as passed → webhook → project board update
  - Alternative: student self-reports by adding label to PR (honour system)
  - Alternative: instructor reviews and moves card manually (B2B cohort model)
- B2B dashboard: completion rates, time-per-mission, common failures

## Codespace Config
```json
{
  "name": "Starfall Academy Lab",
  "image": "ghcr.io/starfall-academy/lab:latest",
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

---

# Build Sequence

## Phase 1: Proof of Concept (Week 1–2)
- [ ] GitHub org
- [ ] Devcontainer image
- [ ] Mission 1.1 — full workflow: fork → Codespace → PR → CI → ARIA
- [ ] ARIA GitHub App prototype
- [ ] Validate on Mac, Windows, browser-only

## Phase 2: Module 1 (Weeks 3–5)
- [ ] Missions 1.2–1.5
- [ ] Gateway Simulation
- [ ] All guides, practice fields

## Phase 3: Module 2 (Weeks 6–8)
- [ ] Missions 2.1–2.4
- [ ] Master Simulation
- [ ] Terraform/Hetzner automation
- [ ] Obstacle course timing calibration

## Phase 4: Modules 3 & 4 (Week 9)
- [ ] MOS guides (2–3 complete, rest outlined)
- [ ] Field Manual seeded
- [ ] CIS mapping, FAQ, Glossary

## Phase 5: Final + Polish (Week 10–11)
- [ ] Operation: Enduring Shield
- [ ] ARIA tuning
- [ ] Beta: 5–10 users (military + civilian)
- [ ] Timing calibration from real data

---

> *"Automate the boring. Fortify the critical. Test everything. Trust nothing."*
> *— Starfall Defence Corps Academy*
