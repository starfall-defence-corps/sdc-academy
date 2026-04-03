# FM-1: Ansible Module Reference
> Starfall Defence Corps -- Field Manual

Classification: UNCLASSIFIED // Training Use Only
Revision: 1.0 | Date: 2187.04.03

---

This reference covers every Ansible module used across the SDC Academy curriculum. Each card provides the fully qualified collection name, a working example, the security context, and the mission where it first appears.

**How to read a card**: FQCN is the canonical module name. Always use it in playbooks -- short names are ambiguous and `ansible-lint` will flag them.

---

## Quick Reference Table

| Module | Category | First Used | Primary Security Purpose |
|--------|----------|------------|--------------------------|
| `ansible.builtin.file` | File Mgmt | 1.3 | Enforce permissions, ownership |
| `ansible.builtin.copy` | File Mgmt | 1.3 | Deploy hardened config files |
| `ansible.builtin.template` | File Mgmt | 1.4 | Generate host-specific configs |
| `ansible.builtin.lineinfile` | File Mgmt | 1.2 | Patch single config directives |
| `ansible.builtin.stat` | File Mgmt | 1.3 | Pre-flight file checks |
| `ansible.builtin.service` | Services | 1.2 | Enable/disable daemons |
| `ansible.builtin.systemd` | Services | 1.3 | Systemd-specific control |
| `ansible.builtin.apt` | Packages | 1.3 | Debian/Ubuntu package state |
| `ansible.builtin.dnf` | Packages | 1.3 | RHEL/Rocky package state |
| `ansible.builtin.package` | Packages | 1.4 | OS-agnostic package control |
| `ansible.builtin.user` | Users | 1.3 | Account provisioning/removal |
| `ansible.builtin.group` | Users | 1.3 | Group management |
| `community.general.ufw` | Firewall | 1.3 | Ubuntu firewall rules |
| `ansible.posix.firewalld` | Firewall | 1.3 | RHEL firewall rules |
| `ansible.posix.sysctl` | System | 2.2 | Kernel parameter hardening |
| `ansible.builtin.cron` | System | 1.3 | Scheduled tasks |
| `ansible.builtin.authorized_key` | SSH | 1.2 | SSH public key deployment |
| `ansible.builtin.include_vars` | Vault | 1.5 | Load encrypted variable files |
| `ansible.builtin.wait_for` | Orchestration | 2.3 | Port/service readiness checks |
| `ansible.builtin.fail` | Orchestration | 2.3 | Controlled abort on bad state |
| `ansible.builtin.assert` | Orchestration | 2.3 | Pre/post-condition validation |
| `ansible.builtin.debug` | Orchestration | 1.1 | Variable inspection |
| `ansible.builtin.shell` | Orchestration | 1.1 | Raw shell execution |
| `ansible.builtin.command` | Orchestration | 1.1 | Process execution (no shell) |

---

## File Management

---

### `ansible.builtin.file`

Set permissions, ownership, symlinks, and directory state. The backbone of filesystem hardening.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Lock down shadow file permissions
  ansible.builtin.file:
    path: /etc/shadow
    owner: root
    group: shadow
    mode: "0640"
```

**Security use case**: Enforce least-privilege file permissions across the fleet. CIS benchmarks mandate specific modes on `/etc/passwd`, `/etc/shadow`, `/etc/gshadow`, and log directories.

> **Common Mistakes**
> - Omitting the leading zero or quotes on `mode`. Write `mode: "0640"`, not `mode: 640`. The bare integer `640` is interpreted as decimal, producing octal `01200` -- not what you want.
> - Using `state: directory` when you meant `state: file`. They do different things. `state: file` modifies an existing file; `state: directory` creates a directory.
> - Forgetting `recurse: true` when setting permissions on a directory tree.

---

### `ansible.builtin.copy`

Deploy static files from the control node to managed hosts. For templated content, use `template` instead.

**First used**: Mission 1.3 -- Clean Sweep (sysctl config files)

```yaml
- name: Deploy hardened sysctl configuration
  ansible.builtin.copy:
    src: files/99-sdc-hardening.conf
    dest: /etc/sysctl.d/99-sdc-hardening.conf
    owner: root
    group: root
    mode: "0644"
  notify: Reload sysctl
```

**Security use case**: Push known-good configuration files (sysctl, limits.d, audit rules) to all ships. The source file is version-controlled, so drift is detectable.

> **Common Mistakes**
> - Using `copy` when the file needs host-specific values. If you see `{{ anything }}` in the file, you need `template`, not `copy`.
> - Forgetting `notify` for services that need to reload after config changes.
> - Setting `content:` and `src:` simultaneously -- they are mutually exclusive.

---

### `ansible.builtin.template`

Render Jinja2 templates and deploy them to managed hosts. Variables are substituted at deploy time.

**First used**: Mission 1.4 -- Many Ships

```yaml
- name: Deploy fleet-specific MOTD
  ansible.builtin.template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: "0644"
```

**Security use case**: Generate host-specific SSH banners, firewall rules, or config files that vary by OS, role, or environment. One template covers the entire fleet.

> **Common Mistakes**
> - Placing templates in `files/` instead of `templates/`. Ansible looks for `src:` relative to the role's `templates/` directory.
> - Using `{{ variable }}` without quoting in YAML. If the value starts with `{`, YAML parses it as a dict. Always quote: `"{{ variable }}"`.
> - Leaving debug `{{ vars }}` dumps in production templates.

---

### `ansible.builtin.lineinfile`

Ensure a single line exists (or is absent) in a file. Surgical edits -- one line at a time.

**First used**: Mission 1.2 -- Lock the Door (sshd_config)

```yaml
- name: Disable root SSH login
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: "^#?PermitRootLogin"
    line: "PermitRootLogin no"
    validate: "sshd -t -f %s"
  notify: Restart sshd
```

**Security use case**: Patch individual directives in SSH, PAM, or login.defs without replacing the entire file. The `validate` parameter prevents deploying broken configs.

> **Common Mistakes**
> - Writing a `regexp` that matches multiple lines. `lineinfile` operates on the *last* match. If your regex is too broad, you get unpredictable edits.
> - Forgetting `regexp` entirely -- this causes duplicate lines on every run (not idempotent).
> - Not using `validate` on critical configs like `sshd_config`. A syntax error there locks you out.

---

### `ansible.builtin.stat`

Check whether a file exists, its type, size, permissions, or checksum. Does not modify anything.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Check if legacy config exists
  ansible.builtin.stat:
    path: /etc/old-firewall.conf
  register: legacy_conf

- name: Remove legacy config if present
  ansible.builtin.file:
    path: /etc/old-firewall.conf
    state: absent
  when: legacy_conf.stat.exists
```

**Security use case**: Pre-flight checks before remediation. Verify a file exists before modifying it, or confirm a backup was created before overwriting.

> **Common Mistakes**
> - Accessing `stat.exists` without registering the result first.
> - Assuming `stat.mode` returns a string. It returns an octal string like `"0644"` -- compare accordingly.

---

## Service Management

---

### `ansible.builtin.service`

Manage services across init systems. Works with systemd, SysVinit, and others.

**First used**: Mission 1.2 -- Lock the Door

```yaml
- name: Ensure SSH daemon is running and enabled
  ansible.builtin.service:
    name: sshd
    state: started
    enabled: true
```

**Security use case**: Ensure critical security services (sshd, auditd, fail2ban) are running. Disable unnecessary services to reduce attack surface.

> **Common Mistakes**
> - Using `state: restarted` unconditionally. This disrupts connections on every run. Use handlers with `notify` for conditional restarts.
> - Confusing `enabled` (start on boot) with `state` (running now). You usually want both.

---

### `ansible.builtin.systemd`

Systemd-specific service control. Supports daemon-reload, masked units, and scope.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Disable and mask unused service
  ansible.builtin.systemd:
    name: cups
    state: stopped
    enabled: false
    masked: true
```

**Security use case**: Mask services that should never run (cups, avahi-daemon, rpcbind). Masking prevents manual start -- stronger than just disabling.

> **Common Mistakes**
> - Forgetting `daemon_reload: true` after deploying a new unit file. Without it, systemd doesn't see the change.
> - Using `ansible.builtin.systemd` on non-systemd hosts. Use `ansible.builtin.service` for cross-platform compatibility.

---

## Package Management

---

### `ansible.builtin.apt`

Manage packages on Debian/Ubuntu systems.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Install security packages
  ansible.builtin.apt:
    name:
      - ufw
      - fail2ban
      - unattended-upgrades
    state: present
    update_cache: true
    cache_valid_time: 3600
```

**Security use case**: Ensure security tooling is installed across all Debian-family ships. Remove known-vulnerable or unnecessary packages with `state: absent`.

> **Common Mistakes**
> - Omitting `update_cache: true` on a fresh system. The package index may be empty, causing failures.
> - Running `update_cache` on every task without `cache_valid_time`. This hammers mirrors and slows runs. Set `cache_valid_time: 3600` (seconds) to cache for one hour.
> - Using `apt` on a RHEL host. Use `ansible.builtin.package` or conditionals for multi-OS playbooks.

---

### `ansible.builtin.dnf`

Manage packages on RHEL, Rocky, Fedora, and other dnf-based systems.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Install firewalld on Rocky
  ansible.builtin.dnf:
    name:
      - firewalld
      - audit
    state: present
```

**Security use case**: Same as `apt` -- enforce package state on Red Hat-family systems.

> **Common Mistakes**
> - Using `dnf` on a Debian host. Guard with `when: ansible_os_family == "RedHat"` or use `ansible.builtin.package`.
> - Forgetting that `dnf` does not have `update_cache` as a standalone param -- it refreshes metadata automatically.

---

### `ansible.builtin.package`

OS-agnostic package management. Delegates to the host's native package manager.

**First used**: Mission 1.4 -- Many Ships

```yaml
- name: Ensure audit daemon is installed (any OS)
  ansible.builtin.package:
    name: audit
    state: present
```

**Security use case**: Write a single task that works on both Debian and RHEL fleets. Useful in roles that target heterogeneous environments.

> **Common Mistakes**
> - Assuming package names are identical across distros. `audit` on RHEL is `auditd` on Debian. Use variables or `ansible_os_family` conditionals for differing names.
> - Trying to use `apt`-specific params like `update_cache` -- the generic module does not support them.

---

## User Management

---

### `ansible.builtin.user`

Create, modify, or remove user accounts.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Create service account with no login shell
  ansible.builtin.user:
    name: svc_monitor
    shell: /usr/sbin/nologin
    system: true
    create_home: false
```

**Security use case**: Provision service accounts with restricted shells. Remove unauthorized accounts. Enforce password expiration with `password_expire_max`.

> **Common Mistakes**
> - Setting `password` to a plaintext string. The module expects a hashed password. Use `password: "{{ 'secret' | password_hash('sha512') }}"`.
> - Forgetting `remove: true` with `state: absent`. Without it, the home directory is orphaned.

---

### `ansible.builtin.group`

Create or remove groups.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Ensure ops group exists
  ansible.builtin.group:
    name: sdc_ops
    state: present
```

**Security use case**: Enforce group-based access control. Create groups before assigning users to them via the `user` module's `groups` parameter.

> **Common Mistakes**
> - Creating a user with `groups: sdc_ops` before the group exists. The `group` task must run first -- use task ordering or role dependencies.

---

## Firewall

---

### `community.general.ufw`

Manage UFW (Uncomplicated Firewall) on Ubuntu/Debian.

**Requires**: `community.general` collection (install via `ansible-galaxy collection install community.general`)

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Allow SSH and enable UFW
  community.general.ufw:
    rule: allow
    port: "22"
    proto: tcp

- name: Enable UFW with default deny
  community.general.ufw:
    state: enabled
    default: deny
    direction: incoming
```

**Security use case**: Default-deny ingress firewall. Whitelist only required ports (SSH, HTTPS). This is a foundational CIS benchmark control.

> **Common Mistakes**
> - Enabling UFW *before* allowing SSH. This locks you out instantly. Always allow SSH first, then enable.
> - Forgetting `proto: tcp`. Without it, both TCP and UDP are affected -- usually not intended.
> - Not setting `default: deny` for incoming. UFW defaults to allow, which provides no protection.

---

### `ansible.posix.firewalld`

Manage firewalld zones and rules on RHEL/Rocky.

**Requires**: `ansible.posix` collection (install via `ansible-galaxy collection install ansible.posix`)

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Allow SSH in public zone (permanent)
  ansible.posix.firewalld:
    service: ssh
    zone: public
    permanent: true
    immediate: true
    state: enabled
```

**Security use case**: Zone-based firewall management. Assign interfaces to zones, control services and ports per zone.

> **Common Mistakes**
> - Setting `permanent: true` without `immediate: true`. The rule is saved but not active until next reload/reboot.
> - Confusing `service:` with `port:`. Use `service: ssh` for well-known services, `port: "8443/tcp"` for custom ports.

---

## System Configuration

---

### `ansible.posix.sysctl`

Set kernel parameters via sysctl. This is the correct module to use.

**Requires**: `ansible.posix` collection

**First used**: Mission 2.2 -- Compliance as Code

```yaml
- name: Disable IP forwarding
  ansible.posix.sysctl:
    name: net.ipv4.ip_forward
    value: "0"
    sysctl_set: true
    state: present
    reload: true
```

**Security use case**: Kernel hardening -- disable IP forwarding, enable SYN cookies, restrict ICMP redirects, enable ASLR. These are CIS Level 1 benchmark controls.

> **Common Mistakes**
> - Using `ansible.builtin.sysctl` instead. See deprecation note below.
> - Forgetting `sysctl_set: true`. Without it, the value is written to the file but not applied to the running kernel.
> - Not quoting numeric values. Write `value: "0"`, not `value: 0`.

---

### `ansible.builtin.sysctl` (DEPRECATED)

> **WARNING**: This module is deprecated in favor of `ansible.posix.sysctl`. It will be removed in a future Ansible release. Migrate all usage to `ansible.posix.sysctl`. The syntax is identical -- only the FQCN changes.

---

### `ansible.builtin.cron`

Manage cron jobs.

**First used**: Mission 1.3 -- Clean Sweep

```yaml
- name: Schedule daily security audit
  ansible.builtin.cron:
    name: "daily-lynis-scan"
    hour: "2"
    minute: "30"
    job: "/usr/sbin/lynis audit system --quick --quiet >> /var/log/lynis-daily.log 2>&1"
    user: root
```

**Security use case**: Schedule recurring security scans, log rotation, or compliance checks. The `name` field makes the job idempotent.

> **Common Mistakes**
> - Omitting `name`. Without it, Ansible cannot identify the job for updates or removal -- you get duplicates on every run.
> - Forgetting to redirect output. Cron sends stdout/stderr as mail by default, which fills mailboxes or bounces.

---

## SSH

---

### `ansible.builtin.authorized_key`

Manage SSH authorized keys for user accounts.

**First used**: Mission 1.2 -- Lock the Door

```yaml
- name: Deploy operator SSH key
  ansible.builtin.authorized_key:
    user: sdc_operator
    key: "{{ lookup('file', 'files/operator_ed25519.pub') }}"
    state: present
    exclusive: false
```

**Security use case**: Centrally manage who can SSH into each ship. Use `exclusive: true` to remove all keys not managed by Ansible -- nuclear option for key hygiene.

> **Common Mistakes**
> - Setting `exclusive: true` without including ALL authorized keys in your playbook. This removes every key not listed, including your own.
> - Deploying private keys instead of public keys. The `key:` parameter takes the `.pub` file content.
> - Forgetting that the user's `~/.ssh/` directory must exist with correct permissions (700). The module creates it, but only if the user exists.

---

## Vault & Secrets

---

### `ansible.builtin.include_vars`

Load variables from a YAML file (including Vault-encrypted files) into the play.

**First used**: Mission 1.5 -- Clean House

```yaml
- name: Load encrypted credentials
  ansible.builtin.include_vars:
    file: vault/credentials.yml
```

**Security use case**: Separate secrets from playbook logic. The encrypted file stays in version control; the vault password stays out of it.

> **Common Mistakes**
> - Committing the vault password file to Git. Add it to `.gitignore` immediately.
> - Referencing `vault_password_file` in `ansible.cfg` before the file exists. This breaks all Ansible commands. Ship the line commented out; have operators uncomment after setup.

---

### `ansible-vault` CLI Reference

Not a module -- a command-line tool for encrypting and decrypting files.

| Command | Purpose |
|---------|---------|
| `ansible-vault create secrets.yml` | Create a new encrypted file |
| `ansible-vault edit secrets.yml` | Decrypt, edit in `$EDITOR`, re-encrypt |
| `ansible-vault encrypt existing.yml` | Encrypt an existing plaintext file |
| `ansible-vault decrypt secrets.yml` | Decrypt to plaintext (use sparingly) |
| `ansible-vault view secrets.yml` | View contents without decrypting to disk |
| `ansible-vault encrypt_string 'value' --name 'var_name'` | Encrypt a single variable inline |
| `ansible-vault rekey secrets.yml` | Change the encryption password |

**First used**: Mission 1.5 -- Clean House

**Security use case**: Protect passwords, API keys, and certificates at rest. Vault-encrypted files are safe to commit. The vault password itself must be managed separately (password manager, CI/CD secret, or secure file).

> **Common Mistakes**
> - Encrypting entire playbooks instead of just the variable files. Encrypt the smallest scope possible.
> - Using `ansible-vault decrypt` and forgetting to re-encrypt. The plaintext file ends up committed.
> - Using a weak vault password. Treat it like any production credential.

---

## Orchestration (Module 2)

---

### `ansible.builtin.wait_for`

Wait for a condition before proceeding -- port open, file exists, or line appears in a log.

**First used**: Mission 2.3 -- Fleet Sync

```yaml
- name: Wait for application port to accept connections
  ansible.builtin.wait_for:
    port: 8443
    host: "{{ inventory_hostname }}"
    delay: 5
    timeout: 60
```

**Security use case**: Rolling updates -- confirm each ship's service is healthy before proceeding to the next. Prevents fleet-wide outages from bad deployments.

> **Common Mistakes**
> - Setting `timeout` too low. Network latency or slow services cause false failures.
> - Forgetting `delay` for services that need startup time. Without it, the check starts immediately and may hit a false positive on a lingering old process.

---

### `ansible.builtin.fail`

Abort the play with a custom message. Use for guardrails.

**First used**: Mission 2.3 -- Fleet Sync

```yaml
- name: Abort if free disk space is critically low
  ansible.builtin.fail:
    msg: "ABORT: {{ inventory_hostname }} has only {{ free_space_mb }}MB free"
  when: free_space_mb | int < 500
```

**Security use case**: Pre-flight safety checks. Halt deployment if conditions are unsafe -- disk full, wrong OS version, missing prerequisites.

> **Common Mistakes**
> - Using `fail` without `when`. This unconditionally aborts the play every time.

---

### `ansible.builtin.assert`

Validate conditions and fail with a descriptive message if they are not met.

**First used**: Mission 2.3 -- Fleet Sync

```yaml
- name: Validate minimum Ansible version
  ansible.builtin.assert:
    that:
      - ansible_version.full is version('2.14', '>=')
      - ansible_os_family in ['Debian', 'RedHat']
    fail_msg: "Unsupported platform or Ansible version"
    success_msg: "Pre-flight checks passed"
```

**Security use case**: Enforce preconditions before running hardening playbooks. Verify OS family, kernel version, or required variables are defined.

> **Common Mistakes**
> - Using Python comparison operators (`==`, `!=`) instead of Jinja2 tests (`is version`, `is defined`).
> - Putting complex logic in `that:` instead of computing it in a prior task and asserting on the registered result.

---

### `ansible.builtin.debug`

Print variables or messages during execution. Essential for troubleshooting.

**First used**: Mission 1.1 -- Fleet Census (ad-hoc)

```yaml
- name: Show gathered network facts
  ansible.builtin.debug:
    var: ansible_default_ipv4.address
    verbosity: 1
```

**Security use case**: Verify variable values during development. Use `verbosity: 1` or higher to hide output in normal runs -- prevents leaking sensitive data in CI logs.

> **Common Mistakes**
> - Leaving `debug` tasks with `verbosity: 0` (default) in production playbooks. They clutter output and may expose secrets.
> - Using `msg:` and `var:` simultaneously. They are mutually exclusive.

---

### `ansible.builtin.shell`

Execute commands through the host's shell (`/bin/sh`). Supports pipes, redirects, and environment variables.

**First used**: Mission 1.1 -- Fleet Census (ad-hoc)

```yaml
- name: Check for unauthorized SUID binaries
  ansible.builtin.shell: >
    find / -perm -4000 -type f 2>/dev/null
  register: suid_binaries
  changed_when: false
```

> **SECURITY WARNING**: The `shell` module is powerful and dangerous. It is not idempotent by default. Prefer purpose-built modules (`file`, `copy`, `user`, etc.) whenever possible. If you must use `shell`:
> - Always set `changed_when` to control idempotency.
> - Never pass untrusted variables directly into the command string (injection risk).
> - Use `creates:` or `removes:` to make it idempotent where applicable.
> - `ansible-lint` will flag shell usage -- acknowledge it only when no module alternative exists.

> **Common Mistakes**
> - Using `shell` to install packages instead of `apt`/`dnf`. Modules handle idempotency; shell does not.
> - Forgetting `changed_when: false` for read-only commands. Without it, Ansible reports "changed" every run.

---

### `ansible.builtin.command`

Execute a command directly (no shell). Safer than `shell` -- no pipes, redirects, or variable expansion.

**First used**: Mission 1.1 -- Fleet Census (ad-hoc)

```yaml
- name: Check SSH daemon version
  ansible.builtin.command:
    cmd: sshd -V
  register: sshd_version
  changed_when: false
```

**Security use case**: Preferred over `shell` when you do not need shell features. Eliminates shell injection risk. Use for running binaries with fixed arguments.

> **Common Mistakes**
> - Using `command` when you need pipes or redirects. It will fail silently or error. Use `shell` for those cases.
> - Same idempotency issue as `shell` -- always set `changed_when` or use `creates:`/`removes:`.

---

## Cross-Reference: Module by Mission

| Mission | Key Modules Introduced |
|---------|----------------------|
| **1.1** Fleet Census | `command`, `shell`, `debug`, `ping` (ad-hoc) |
| **1.2** Lock the Door | `lineinfile`, `service`, `authorized_key`, handlers |
| **1.3** Clean Sweep | `apt`, `dnf`, `ufw`, `firewalld`, `copy`, `file`, `user`, `group`, `systemd`, `cron`, `stat` |
| **1.4** Many Ships | `template`, `package`, `group_vars` (not a module -- a pattern) |
| **1.5** Clean House | `include_vars`, `ansible-vault` CLI |
| **2.1** Weapon Handling Test | Molecule/Testinfra (no new Ansible modules) |
| **2.2** Compliance as Code | `ansible.posix.sysctl`, `copy` (limits.d), `lineinfile` (sshd_config hardening) |
| **2.3** Fleet Sync | `wait_for`, `fail`, `assert`, `delegate_to`, `serial` |
| **2.4** Defence in Depth | `ansible-lint` (not a module -- a CI tool) |

---

## Collection Installation Reference

Several modules require collections that are not bundled with `ansible-core`. Install them before use.

```bash
# Install required collections
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix

# Or use a requirements.yml
cat > collections/requirements.yml << 'EOF'
collections:
  - name: community.general
  - name: ansible.posix
EOF
ansible-galaxy collection install -r collections/requirements.yml
```

> All SDC Academy missions handle collection installation automatically via `make setup`. This reference is for operators working outside the training environment.

---

*Filed under: SDC Academy Training Command // FM-1 Rev 1.0*
*This field manual is maintained in the sdc-academy repository. Report errors via issue tracker.*
