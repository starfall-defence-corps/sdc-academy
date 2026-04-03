# FM-4: CIS & Compliance Reference
> Starfall Defence Corps — Field Manual

---

A fleet that ignores its own hardening standards is already compromised. CIS Benchmarks are the baseline — the minimum acceptable posture for any vessel in active service. This manual maps those benchmarks to Ansible automation so you can enforce them at scale, verify them with Lynis, and tag them for selective deployment.

---

## What CIS Benchmarks Are

The Centre for Internet Security (CIS) publishes consensus-driven configuration baselines for operating systems, cloud platforms, and applications. These are not opinions — they are vetted by hundreds of security professionals and updated with each OS release.

Every CIS Benchmark is split into two levels:

| Level | Meaning | Impact |
|-------|---------|--------|
| **Level 1** | Essential hardening. Low risk of breaking functionality. Every system should meet this bar. | Minimal operational disruption |
| **Level 2** | Defence-in-depth. May restrict functionality. Applied where the threat model demands it. | May require application-level changes |

### How Controls Are Numbered

Controls follow a hierarchical numbering scheme: `Section.Subsection.Control`.

```
5.2.4 — Section 5 (Access/Auth), Subsection 2 (SSH Server), Control 4 (Root Login)
```

Each control includes a description, rationale, audit procedure (how to check), and remediation (how to fix). Your Ansible tasks are the automated remediation. Your Molecule tests are the automated audit.

---

## CIS to Ansible Mapping — Ubuntu 22.04

The following tables map specific CIS controls to Ansible implementations. Each example is a working task you can drop into a playbook. These cover the controls most relevant to the SDC curriculum.

### Section 1 — Initial Setup

| Control | Description | Ansible Module | Level |
|---------|-------------|----------------|-------|
| 1.5.1 | Core dumps restricted | `ansible.builtin.copy` | 1 |
| 1.7.1 | Warning banner configured | `ansible.builtin.copy` | 1 |

**1.5.1 — Restrict Core Dumps**

Core dumps can leak credentials and encryption keys from memory. Disable them fleet-wide.

```yaml
- name: "CIS 1.5.1 — Restrict core dumps"
  ansible.builtin.copy:
    dest: /etc/security/limits.d/99-core-dump.conf
    content: "* hard core 0\n"
    mode: "0644"
  tags: [cis, cis_1, cis_1_5_1]
```

**1.7.1 — Warning Banner**

Legal warning banners deter unauthorised access and satisfy audit requirements.

```yaml
- name: "CIS 1.7.1 — Deploy login warning banner"
  ansible.builtin.copy:
    dest: /etc/issue.net
    content: |
      *** STARFALL DEFENCE CORPS — AUTHORISED PERSONNEL ONLY ***
      Unauthorised access is a violation of SDC Regulation 7-14.
    mode: "0644"
  tags: [cis, cis_1, cis_1_7_1]
```

### Section 3 — Network Configuration

| Control | Description | Ansible Module | Level |
|---------|-------------|----------------|-------|
| 3.1.1 | IP forwarding disabled | `ansible.posix.sysctl` | 1 |
| 3.3.2 | ICMP redirects not accepted | `ansible.posix.sysctl` | 1 |
| 3.3.3 | Secure ICMP redirects not accepted | `ansible.posix.sysctl` | 1 |
| 3.3.4 | Suspicious packets logged | `ansible.posix.sysctl` | 1 |

**3.1.1 — Disable IP Forwarding**

Unless a node is explicitly a router, IP forwarding is a lateral-movement vector.

```yaml
- name: "CIS 3.1.1 — Disable IP forwarding"
  ansible.posix.sysctl:
    name: net.ipv4.ip_forward
    value: "0"
    sysctl_set: true
    reload: true
  tags: [cis, cis_3, cis_3_1_1]
```

**3.3.2 — Reject ICMP Redirects**

ICMP redirects can be used to manipulate routing tables. Reject them on all interfaces.

```yaml
- name: "CIS 3.3.2 — Do not accept ICMP redirects"
  ansible.posix.sysctl:
    name: "{{ item }}"
    value: "0"
    sysctl_set: true
    reload: true
  loop:
    - net.ipv4.conf.all.accept_redirects
    - net.ipv4.conf.default.accept_redirects
  tags: [cis, cis_3, cis_3_3_2]
```

**3.3.3 — Reject Secure ICMP Redirects**

Even "secure" ICMP redirects are a risk on hosts that are not routers.

```yaml
- name: "CIS 3.3.3 — Do not accept secure ICMP redirects"
  ansible.posix.sysctl:
    name: "{{ item }}"
    value: "0"
    sysctl_set: true
    reload: true
  loop:
    - net.ipv4.conf.all.secure_redirects
    - net.ipv4.conf.default.secure_redirects
  tags: [cis, cis_3, cis_3_3_3]
```

**3.3.4 — Log Suspicious Packets**

Martian packets (impossible source addresses) indicate reconnaissance or spoofing. Log them.

```yaml
- name: "CIS 3.3.4 — Log suspicious packets"
  ansible.posix.sysctl:
    name: "{{ item }}"
    value: "1"
    sysctl_set: true
    reload: true
  loop:
    - net.ipv4.conf.all.log_martians
    - net.ipv4.conf.default.log_martians
  tags: [cis, cis_3, cis_3_3_4]
```

### Section 5 — Access, Authentication, and Authorisation

| Control | Description | Ansible Module | Level |
|---------|-------------|----------------|-------|
| 5.1.8 | Cron restricted to authorised users | `ansible.builtin.copy` | 1 |
| 5.2.4 | SSH root login disabled | `ansible.builtin.lineinfile` | 1 |
| 5.2.5 | SSH password auth disabled | `ansible.builtin.lineinfile` | 1 |
| 5.2.7 | SSH MaxAuthTries set to 4 or fewer | `ansible.builtin.lineinfile` | 1 |
| 5.2.13 | SSH ClientAliveInterval configured | `ansible.builtin.lineinfile` | 1 |
| 5.2.16 | SSH LoginGraceTime set to 60 or less | `ansible.builtin.lineinfile` | 1 |

**5.1.8 — Restrict Cron to Authorised Users**

Only explicitly authorised accounts should schedule tasks.

```yaml
- name: "CIS 5.1.8 — Restrict cron to authorised users"
  ansible.builtin.copy:
    dest: /etc/cron.allow
    content: "root\n"
    owner: root
    group: root
    mode: "0600"
  tags: [cis, cis_5, cis_5_1_8]
```

**5.2.4 — Disable SSH Root Login**

Root login over SSH bypasses audit trails. Force operators to authenticate as themselves, then escalate.

```yaml
- name: "CIS 5.2.4 — Disable SSH root login"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?PermitRootLogin'
    line: "PermitRootLogin no"
  notify: Restart sshd
  tags: [cis, cis_5, cis_5_2_4]
```

**5.2.5 — Disable SSH Password Authentication**

Key-based authentication eliminates brute-force risk entirely.

```yaml
- name: "CIS 5.2.5 — Disable SSH password authentication"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?PasswordAuthentication'
    line: "PasswordAuthentication no"
  notify: Restart sshd
  tags: [cis, cis_5, cis_5_2_5]
```

**5.2.7 — Limit SSH MaxAuthTries**

Restricting authentication attempts slows brute-force attacks and triggers fail2ban sooner.

```yaml
- name: "CIS 5.2.7 — Set SSH MaxAuthTries"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?MaxAuthTries'
    line: "MaxAuthTries 4"
  notify: Restart sshd
  tags: [cis, cis_5, cis_5_2_7]
```

**5.2.13 — SSH ClientAliveInterval**

Idle sessions are hijacking targets. Terminate them automatically.

```yaml
- name: "CIS 5.2.13 — Set SSH ClientAliveInterval"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?ClientAliveInterval'
    line: "ClientAliveInterval 300"
  notify: Restart sshd
  tags: [cis, cis_5, cis_5_2_13]
```

**5.2.16 — SSH LoginGraceTime**

Reduce the window for unauthenticated connections to consume resources.

```yaml
- name: "CIS 5.2.16 — Set SSH LoginGraceTime"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?LoginGraceTime'
    line: "LoginGraceTime 60"
  notify: Restart sshd
  tags: [cis, cis_5, cis_5_2_16]
```

### Section 6 — System Maintenance

| Control | Description | Ansible Module | Level |
|---------|-------------|----------------|-------|
| 6.1.3 | /etc/shadow permissions | `ansible.builtin.file` | 1 |
| 6.1.4 | /etc/gshadow permissions | `ansible.builtin.file` | 1 |

**6.1.3 — /etc/shadow Permissions**

Password hashes are high-value targets. Lock down the shadow file.

```yaml
- name: "CIS 6.1.3 — Set /etc/shadow permissions"
  ansible.builtin.file:
    path: /etc/shadow
    owner: root
    group: shadow
    mode: "0640"
  tags: [cis, cis_6, cis_6_1_3]
```

**6.1.4 — /etc/gshadow Permissions**

Same principle as shadow — group password hashes must be protected.

```yaml
- name: "CIS 6.1.4 — Set /etc/gshadow permissions"
  ansible.builtin.file:
    path: /etc/gshadow
    owner: root
    group: shadow
    mode: "0640"
  tags: [cis, cis_6, cis_6_1_4]
```

---

## CIS to Ansible Mapping — Rocky Linux 9 Differences

Rocky Linux 9 shares most CIS controls with Ubuntu 22.04, but the underlying tools and paths diverge. Know the differences before you write cross-platform playbooks.

| Area | Ubuntu 22.04 | Rocky Linux 9 |
|------|--------------|---------------|
| **Package manager** | `ansible.builtin.apt` | `ansible.builtin.dnf` |
| **Firewall tool** | `ufw` / `community.general.ufw` | `firewalld` / `ansible.posix.firewalld` |
| **SSH service name** | `ssh` (or `sshd`) | `sshd` |
| **Shadow group** | `shadow` | `root` |
| **Cron package** | `cron` | `cronie` |
| **Sysctl path** | `/etc/sysctl.d/` | `/etc/sysctl.d/` (same) |
| **SELinux** | Not present (AppArmor) | Enforcing by default — do not disable |
| **CIS section numbering** | Ubuntu-specific benchmark | RHEL 9 benchmark (different control IDs in places) |

### Key Implementation Differences

**Shadow file group ownership** differs. Handle it with a conditional:

```yaml
- name: "CIS 6.1.3 — Set /etc/shadow permissions"
  ansible.builtin.file:
    path: /etc/shadow
    owner: root
    group: "{{ 'shadow' if ansible_os_family == 'Debian' else 'root' }}"
    mode: "0640"
  tags: [cis, cis_6, cis_6_1_3]
```

**Firewall rules** require entirely different modules. Use `when` conditionals or separate task files:

```yaml
- name: "Allow SSH — Ubuntu"
  community.general.ufw:
    rule: allow
    port: "22"
    proto: tcp
  when: ansible_os_family == "Debian"

- name: "Allow SSH — Rocky"
  ansible.posix.firewalld:
    service: ssh
    permanent: true
    state: enabled
    immediate: true
  when: ansible_os_family == "RedHat"
```

**SELinux** on Rocky must remain enforcing. CIS for RHEL 9 explicitly requires it. If your playbook disables SELinux, you are failing the benchmark, not passing it.

---

## Lynis Quick Reference

Lynis is an open-source security auditing tool. It performs a system scan and produces a hardening index — a numerical score reflecting how many controls the host satisfies. Use it as a before/after metric when applying CIS remediations.

### Installation

```yaml
# Ubuntu
- name: Install Lynis
  ansible.builtin.apt:
    name: lynis
    state: present

# Rocky
- name: Install Lynis
  ansible.builtin.dnf:
    name: lynis
    state: present
```

### Running a Scan

From the command line:

```bash
sudo lynis audit system --quick --no-colors
```

The `--quick` flag skips interactive prompts. `--no-colors` produces clean output for parsing and logging.

### Running via Ansible Ad-Hoc

Scan all hosts in your inventory without writing a playbook:

```bash
ansible all -m ansible.builtin.command \
  -a "lynis audit system --quick --no-colors" \
  -b --become
```

Or target a specific group:

```bash
ansible webservers -m ansible.builtin.command \
  -a "lynis audit system --quick --no-colors" \
  -b --become
```

### Reading the Hardening Index

Lynis outputs a score between 0 and 100 at the end of its report:

```
Hardening index : 67 [#############       ]
```

| Score Range | Assessment |
|-------------|------------|
| 0–49 | Critical — immediate remediation required |
| 50–69 | Below standard — significant gaps remain |
| 70–84 | Acceptable — meets minimum fleet requirements |
| 85–100 | Hardened — exceeds baseline, ready for hostile sectors |

### Interpreting Results

Lynis categorises findings into three types:

| Symbol | Meaning | Action |
|--------|---------|--------|
| `[WARNING]` | Active vulnerability or misconfiguration | Fix immediately |
| `[SUGGESTION]` | Hardening opportunity, not critical | Evaluate and apply where appropriate |
| `[OK]` | Control satisfied | No action needed |

The full report is written to `/var/log/lynis-report.dat`. Parse it programmatically:

```bash
grep "suggestion\[\]" /var/log/lynis-report.dat | sort -u
```

---

## CIS vs STIG

Two compliance frameworks dominate the security hardening space. Know which one applies to your mission.

| Dimension | CIS Benchmark | DISA STIG |
|-----------|---------------|-----------|
| **Publisher** | Centre for Internet Security | Defence Information Systems Agency (DoD) |
| **Audience** | Industry-wide, any organisation | US Department of Defence and contractors |
| **Strictness** | Moderate — Level 1 is broadly safe | High — assumes hostile threat environment |
| **Licensing** | Free PDF download | Free, publicly available |
| **Levels** | Level 1 (baseline), Level 2 (defence-in-depth) | CAT I (critical), CAT II (medium), CAT III (low) |
| **Update cycle** | Per OS release | Per OS release + quarterly updates |
| **Audit tool** | Lynis, OpenSCAP | SCAP Compliance Checker (SCC), OpenSCAP |
| **Ansible tooling** | Manual or community roles | `ansible-lockdown` project provides STIG roles |
| **SDC usage** | Missions 2.2, Master Simulation | Reference only — CIS is the SDC standard |

**When to use CIS**: Default choice. Covers the vast majority of hardening requirements without restricting operational functionality. The SDC curriculum is built on CIS.

**When to use STIG**: When operating in environments that mandate DoD compliance, or when the threat model calls for maximum restriction at the cost of operational flexibility.

---

## Ansible Tags for CIS

Tagging every CIS task allows selective enforcement. When a scan reveals a single failing control, you should not need to re-run the entire hardening playbook — target the specific section.

### Tagging Convention

Apply three levels of tags to every CIS task:

```yaml
tags: [cis, cis_5, cis_5_2_4]
#      ^     ^       ^
#      |     |       +-- Specific control
#      |     +---------- Section
#      +---------------- All CIS tasks
```

### Running Selective Controls

Apply only SSH hardening (Section 5.2):

```bash
ansible-playbook harden.yml --tags cis_5
```

Apply a single control:

```bash
ansible-playbook harden.yml --tags cis_5_2_4
```

Apply everything except network configuration:

```bash
ansible-playbook harden.yml --tags cis --skip-tags cis_3
```

### Combining Tags with Check Mode

Audit without changing anything — report what would be remediated:

```bash
ansible-playbook harden.yml --tags cis --check --diff
```

This is the Ansible equivalent of a Lynis scan: it tells you what is out of compliance without modifying the system. Use it before applying changes to production vessels.

---

## Mission References

| Mission | Relevance |
|---------|-----------|
| **2.2 — Compliance as Code** | CIS benchmarks deep dive. Lynis scanning. Tagged controls. First mission where you enforce CIS at scale. |
| **Master Simulation** | Full CIS compliance across 6 nodes. Multi-OS. Everything in this manual applied under pressure. |

---

*Filed under SDC Academy Field Manuals. Consult FM-1 for module details, FM-3 for Molecule testing of compliance tasks.*
