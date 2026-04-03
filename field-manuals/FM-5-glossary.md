# FM-5: Glossary
> Starfall Defence Corps -- Field Manual

Classification: UNCLASSIFIED // Training Use Only
Revision: 1.0 | Date: 2187.04.03

---

Combined technical and lore glossary for the SDC Academy curriculum. Entries are alphabetical. Where a term maps to an in-universe equivalent, both are noted.

---

## A

**Ad-hoc command** -- A one-off Ansible command run directly from the command line without writing a playbook. Uses the `ansible` binary with `-m` (module) and `-a` (arguments). Useful for quick inspections and emergency fixes, but not repeatable or auditable the way playbooks are. -- *First introduced: Mission 1.1*

**ansible.cfg** -- The Ansible configuration file. Controls defaults such as inventory path, remote user, privilege escalation, and SSH options. Ansible searches for it in the current directory, the home directory, or `/etc/ansible/`. Each mission workspace ships one pre-configured for the lab environment.

**Ansible Vault** -- Ansible's built-in encryption system for protecting sensitive data (passwords, API keys, certificates). Files are encrypted with AES-256 using a vault password. In SDC lore, the Vault is referred to as the **Crypto Cell**. -- *First introduced: Mission 1.5*

**ansible-lint** -- A static analysis tool that checks playbooks, roles, and task files for style violations, anti-patterns, and potential errors. Enforces best practices such as using FQCNs and naming every task. Central to CI/CD pipeline enforcement. -- *First introduced: Mission 2.4*

**ARIA (Automated Review & Intelligence Analyst)** -- The SDC Academy's AI-powered code reviewer. ARIA runs as a GitHub Action on student pull requests, analysing playbooks against mission-specific criteria and returning a structured debrief. Personality: formal, terse, military-analytical. Never praises without evidence.

## B

**Become** -- Ansible's privilege escalation mechanism. When `become: true` is set, tasks execute as a privileged user (default: root) via `sudo`. Configured per play, per task, or globally in `ansible.cfg`. See also: **Privilege escalation**.

**Block / Rescue / Always** -- Ansible's error-handling structure, analogous to try/catch/finally. `block` wraps a set of tasks, `rescue` runs if any task in the block fails, and `always` runs regardless of outcome. Essential for safe rolling updates. -- *First introduced: Mission 2.3*

## C

**Cadet / Ensign / Lieutenant JG / Lieutenant / Lt. Commander** -- SDC Academy rank progression. Cadets begin at Mission 1.1. Promotion occurs upon completing mission milestones: Ensign (after 1.2), Lieutenant JG (after 1.5), Lieutenant (after the Gateway Simulation), Lt. Commander (after the Master Simulation).

**Captain Unpatched** -- Lore villain. A negligent commanding officer whose ships run outdated software with known CVEs. Represents the threat of unpatched systems and ignored compliance baselines. -- *First introduced: Mission 2.2*

**CI/CD Pipeline** -- Continuous Integration / Continuous Delivery. An automated workflow (typically GitHub Actions) that runs linting, syntax checks, and Molecule tests on every push or pull request. Catches configuration errors before they reach production. -- *First introduced: Mission 2.4*

**CIS Benchmark** -- A set of security configuration guidelines published by the Center for Internet Security. Provides prescriptive, consensus-based hardening standards for operating systems, services, and applications. SDC missions implement a subset of CIS controls via Ansible. -- *First introduced: Mission 2.2*

**Collection** -- A distribution format for Ansible content (modules, roles, plugins). Academy missions rely on `ansible.posix` (for `sysctl`, `firewalld`, `authorized_key`) and `community.general` (for `ufw` and others). Installed via `ansible-galaxy collection install`.

**Colonel Hardcoded-Password** -- Lore villain. An officer who stores credentials in plaintext across every ship in the fleet. Represents secrets committed to version control and unencrypted variable files. -- *First introduced: Mission 1.5*

**Conditionals / when clause** -- Ansible's mechanism for running tasks only when a condition is true. Uses Jinja2 expressions evaluated at runtime (e.g., `when: ansible_os_family == "Debian"`). Critical for writing multi-OS playbooks. -- *First introduced: Mission 1.4*

**Configuration drift** -- The gradual divergence of a system's actual state from its desired state, caused by manual changes, ad-hoc fixes, or missed automation runs. Detected by scheduled Molecule tests or periodic playbook runs with `--check --diff`. -- *First introduced: Mission 2.4*

**Converge** -- A Molecule test phase that applies the role or playbook under test to the target container. This is where Ansible actually runs. Preceded by `create` (spin up containers) and followed by `verify` (run assertions). -- *First introduced: Mission 2.1*

**Corporal Copy-Paste** -- Lore villain. A technician who duplicates configuration blocks across ships instead of using variables and templates. Represents hardcoded values, copy-paste errors, and lack of abstraction. -- *First introduced: Mission 1.4*

**Crypto Cell** -- SDC lore term for Ansible Vault. The secure compartment aboard each vessel where encrypted credentials and classified configuration data are stored. See: **Ansible Vault**. -- *First introduced: Mission 1.5*

## D

**delegate_to** -- An Ansible task directive that executes the task on a different host than the current play target. Used for orchestration patterns such as removing a node from a load balancer before updating it, or running a health check from a monitoring server. -- *First introduced: Mission 2.3*

## F

**Facts** -- System information automatically gathered by Ansible at the start of each play (unless disabled). Includes OS family, IP addresses, memory, disk layout, and more. Accessed as variables (e.g., `ansible_distribution`). See also: **Gather facts**. -- *First introduced: Mission 1.1*

## G

**Gather facts** -- The implicit first task of every Ansible play (unless `gather_facts: false`). Runs the `setup` module on each target host and populates the `ansible_*` variable namespace. Can be expensive on large inventories; disable when facts are not needed.

**Gateway Simulation / Operation: First Contact** -- The Module 1 capstone exercise. A timed 75-minute simulation that combines inventory management, SSH hardening, service control, multi-OS configuration, and role-based organisation into a single scenario. Passing earns the rank of Lieutenant.

**General Snowflake** -- Lore villain of the Master Simulation. A flag officer who insists every ship in the fleet be configured by hand, rejecting automation and standardisation. Represents the ultimate anti-pattern: unique, unreproducible infrastructure.

**group_vars** -- A directory containing YAML files that define variables applying to all hosts in a named inventory group. File names match group names (e.g., `group_vars/webservers.yml`). Higher specificity than `all` group vars, lower than `host_vars`. -- *First introduced: Mission 1.4*

## H

**Handler** -- A special Ansible task that runs only when notified by another task. Typically used for service restarts or reloads after configuration changes. Handlers run once at the end of the play, regardless of how many tasks notify them. -- *First introduced: Mission 1.2*

**Hardening index** -- A numerical score (0--100) produced by Lynis representing the overall security posture of a system. Used in SDC missions as the measurable target: students must raise the index above a specified threshold. -- *First introduced: Mission 2.2*

**host_vars** -- A directory containing YAML files that define variables for individual hosts. File names match inventory hostnames. Highest specificity in the directory-based variable hierarchy, overriding `group_vars`.

## I

**Idempotency** -- The property that running the same operation multiple times produces the same result as running it once. In Ansible, a well-written playbook should report `changed=0` on subsequent runs. This is the core design principle tested in every SDC mission.

**Inventory** -- A file or directory that defines the hosts and groups Ansible manages. Can be INI or YAML format, static or dynamic. The inventory is the map of your fleet. -- *First introduced: Mission 1.1*

**Iron Curtain** -- The badge awarded upon completing the Master Simulation (Operation: Iron Curtain). Represents mastery of automated compliance enforcement across a heterogeneous fleet under adversarial conditions.

## J

**Jinja2 template** -- A text file containing Jinja2 expressions (`{{ variable }}`, `{% if %}`, `{% for %}`) that Ansible renders into host-specific output. Used with the `ansible.builtin.template` module to generate configuration files dynamically. -- *First introduced: Mission 1.4*

## L

**Lint / linting** -- The process of statically analysing code for style issues, anti-patterns, and errors without executing it. In the SDC context, `ansible-lint` and `yamllint` are the primary tools, enforced in CI pipelines. -- *First introduced: Mission 2.4*

**Lynis** -- An open-source security auditing tool for Unix-based systems. Performs hundreds of individual tests covering authentication, file permissions, kernel parameters, networking, and more. Produces a hardening index score. -- *First introduced: Mission 2.2*

## M

**Master Simulation / Operation: Iron Curtain** -- The Module 2 capstone. A comprehensive scenario requiring students to enforce CIS-aligned hardening across a mixed-OS fleet, implement CI/CD pipelines, handle rolling updates, and defeat General Snowflake's manual-configuration doctrine.

**max_fail_percentage** -- An Ansible play-level setting that stops execution when the percentage of failed hosts exceeds the threshold. Prevents a bad update from propagating across the entire fleet. Works in conjunction with `serial`. -- *First introduced: Mission 2.3*

**Module** -- A unit of code that Ansible executes on managed hosts. Modules handle specific tasks (installing packages, managing files, controlling services). Always reference by fully qualified collection name (FQCN) in playbooks (e.g., `ansible.builtin.copy`, not `copy`).

**Molecule** -- A testing framework for Ansible roles and playbooks. Provides a workflow of create, converge, verify, and destroy phases using ephemeral Docker containers. The standard testing tool across all SDC missions. -- *First introduced: Mission 2.1*

## N

**Notify** -- A task-level directive that triggers a named handler when the task reports a change. Multiple notifications to the same handler result in a single handler execution. -- *First introduced: Mission 1.2*

## P

**Play / Playbook** -- A play is a mapping of hosts to tasks. A playbook is a YAML file containing one or more plays. Playbooks are the primary unit of Ansible automation -- repeatable, version-controlled, and auditable. -- *First introduced: Mission 1.2*

**Private YOLO-Deploy** -- Lore villain. A reckless operator who pushes untested changes directly to production with no CI checks, no Molecule tests, and no rollback plan. Represents the absence of testing discipline and deployment safeguards. -- *First introduced: Mission 2.1*

**Privilege escalation / become** -- The mechanism by which Ansible elevates permissions on a managed host. The `become` directive uses `sudo` by default. Can be scoped to an entire play or individual tasks. See: **Become**.

## R

**Role** -- An Ansible content unit that bundles tasks, handlers, templates, files, variables, and defaults into a reusable, shareable directory structure. Roles enforce separation of concerns and are the standard way to organise non-trivial automation. -- *First introduced: Mission 1.5*

**Rolling update / serial** -- A deployment strategy where only a subset of hosts are updated at a time (controlled by the `serial` keyword). Maintains fleet availability during updates by ensuring some hosts remain operational. -- *First introduced: Mission 2.3*

**run_once** -- An Ansible task directive that ensures a task executes on only one host in the play, regardless of how many hosts are targeted. Useful for one-time actions like database migrations or control-plane commands. -- *First introduced: Mission 2.3*

## S

**SDC (Starfall Defence Corps)** -- The interstellar military organisation responsible for defending human-colonised systems against the Voidborn. Students are cadets enrolled in the SDC Academy, training to become fleet automation engineers.

**SSH Root Fairy** -- Lore villain. A mythical being who leaves root login enabled and password authentication wide open on every ship. Represents the most basic SSH misconfiguration: allowing direct root access over the network. -- *First introduced: Mission 1.2*

**STIG (Security Technical Implementation Guide)** -- A set of security hardening standards published by DISA (Defense Information Systems Agency). More prescriptive and US-DoD-specific than CIS Benchmarks. Referenced in SDC lore as the military-grade counterpart to CIS controls.

**sysctl** -- A Linux kernel interface for modifying runtime parameters (e.g., IP forwarding, SYN cookies, ICMP redirects). Managed in Ansible via `ansible.posix.sysctl`. Persistent changes require entries in `/etc/sysctl.d/`. -- *First introduced: Mission 1.3*

## T

**Tag** -- An Ansible metadata label applied to tasks, roles, or plays. Allows selective execution with `--tags` or `--skip-tags`. In compliance work, tags map to specific CIS control numbers for targeted remediation. -- *First introduced: Mission 2.2*

**Task** -- The smallest unit of work in Ansible. A task calls a single module with specific parameters. Tasks execute sequentially within a play and report one of: ok, changed, failed, or skipped.

**Template** -- See: **Jinja2 template**. The `ansible.builtin.template` module renders a Jinja2 source file and deploys the result to the managed host. -- *First introduced: Mission 1.4*

**Testinfra** -- A Python testing framework (pytest plugin) for validating the state of servers. Used in Molecule's verify phase to assert that packages are installed, services are running, ports are listening, and files have correct permissions. -- *First introduced: Mission 2.1*

## V

**Variable precedence** -- The order in which Ansible resolves conflicting variable definitions. There are 22 levels of precedence (from lowest: command-line defaults, to highest: extra vars via `-e`). Understanding precedence prevents unexpected overrides. Key levels for SDC missions: role defaults < `group_vars` < `host_vars` < play vars < extra vars.

**Vault password file** -- A plaintext file containing the password used to decrypt Ansible Vault-encrypted files. Referenced in `ansible.cfg` via `vault_password_file`. Must never be committed to version control. Note: if the file does not exist, all Ansible commands will fail. -- *First introduced: Mission 1.5*

**Verify** -- A Molecule test phase that runs Testinfra (or another verifier) to assert the managed host is in the desired state after converge. This is where pass/fail is determined. -- *First introduced: Mission 2.1*

**Voidborn** -- The existential threat to human-colonised space. An alien intelligence that exploits misconfigured systems, unpatched vulnerabilities, and poor operational hygiene to infiltrate fleet networks. Every SDC mission is framed as defending against Voidborn incursion.

---

*"Know the words, know the war. Terminology is the first line of defence against ambiguity -- and ambiguity is how the Voidborn get in."*
-- ARIA, orientation address to incoming cadets
