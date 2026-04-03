# FM-3: Molecule & Testing Reference
> Starfall Defence Corps — Field Manual

---

You do not deploy what you have not tested. Molecule and Testinfra are the weapons that prove your infrastructure code works before it touches production. This manual covers the full testing lifecycle.

---

## Molecule Lifecycle

Molecule manages the test cycle for Ansible roles. Each command maps to a phase.

| Command | Purpose |
|---------|---------|
| `molecule create` | Spin up test instances (Docker containers, typically) |
| `molecule converge` | Run the playbook against those instances |
| `molecule verify` | Execute Testinfra assertions against the converged state |
| `molecule destroy` | Tear down all test instances |
| `molecule test` | Full cycle: create, converge, verify, destroy |
| `molecule login` | SSH into a running instance for manual inspection |

### Full Cycle Breakdown

`molecule test` runs this sequence:

```
dependency → cleanup → destroy → syntax → create → prepare → converge → idempotence → verify → cleanup → destroy
```

During development, run individual phases to iterate faster:

```bash
# Stand up instances once
molecule create

# Iterate on your role
molecule converge

# Iterate on your tests
molecule verify

# Tear down when finished
molecule destroy
```

### Targeting a Specific Scenario

```bash
molecule test -s <scenario-name>
molecule converge -s hardening
```

The default scenario lives in `molecule/default/`. Additional scenarios each get their own subdirectory.

---

## molecule.yml Configuration

The scenario configuration file. Every section controls a phase.

### dependency

Install collections or roles before converge.

```yaml
dependency:
  name: galaxy
  options:
    requirements-file: requirements.yml
```

### driver

Which backend creates instances. Docker is standard for SDC missions.

```yaml
driver:
  name: docker
```

Other drivers: `default` (pre-existing hosts), `delegated` (you manage creation yourself).

### platforms

Define the test instances. Each entry becomes a container.

```yaml
platforms:
  - name: web-01
    image: geerlingguy/docker-ubuntu2204-ansible
    pre_build_image: true
    command: ""
    published_ports:
      - "0.0.0.0:2221:22/tcp"
    groups:
      - webservers
      - fleet

  - name: db-01
    image: geerlingguy/docker-rockylinux9-ansible
    pre_build_image: true
    command: ""
    published_ports:
      - "0.0.0.0:2222:22/tcp"
    groups:
      - databases
      - fleet
```

Key fields:
- **name** — hostname inside the scenario
- **image** — Docker image (use `geerlingguy` images for systemd support)
- **pre_build_image** — `true` skips Dockerfile build, uses image as-is
- **command** — set to `""` for systemd-enabled images
- **published_ports** — SSH port mapping for Testinfra connections
- **groups** — Ansible inventory groups this host belongs to

### provisioner

Controls how Ansible runs during converge.

```yaml
provisioner:
  name: ansible
  inventory:
    links:
      hosts: ../../workspace/inventory/hosts.ini
      group_vars: ../../workspace/inventory/group_vars/
      host_vars: ../../workspace/inventory/host_vars/
  config_options:
    defaults:
      remote_user: root
```

Use `inventory.links` to point Molecule at the student's real inventory files.

### verifier

Which tool runs assertions.

```yaml
verifier:
  name: testinfra
```

Testinfra is the SDC standard. The alternative is `ansible` (uses assert tasks), but Testinfra gives you Python's full assertion power.

---

## Testinfra Assertions

Testinfra uses a `host` fixture injected into every test function. Each property returns an object with attributes you assert against.

### host.file(path)

Inspect files and directories.

```python
def test_sshd_config(host):
    f = host.file("/etc/ssh/sshd_config")
    assert f.exists
    assert f.is_file
    assert f.user == "root"
    assert f.group == "root"
    assert f.mode == 0o600
    assert f.contains("PermitRootLogin no")
```

| Attribute | Returns |
|-----------|---------|
| `.exists` | `True` if path exists |
| `.is_file` | `True` if regular file |
| `.is_directory` | `True` if directory |
| `.mode` | Octal permissions (e.g. `0o755`) |
| `.user` | Owner username |
| `.group` | Group name |
| `.contains(string)` | `True` if file contains the string |
| `.content_string` | Full file contents as string |

### host.service(name)

Check systemd service state.

```python
def test_sshd_running(host):
    svc = host.service("sshd")
    assert svc.is_running
    assert svc.is_enabled
```

| Attribute | Returns |
|-----------|---------|
| `.is_running` | `True` if active |
| `.is_enabled` | `True` if enabled at boot |

### host.package(name)

Verify package installation.

```python
def test_fail2ban_installed(host):
    pkg = host.package("fail2ban")
    assert pkg.is_installed
```

| Attribute | Returns |
|-----------|---------|
| `.is_installed` | `True` if package present |
| `.version` | Installed version string |

### host.socket(spec)

Check listening ports. The spec format is `tcp://address:port`.

```python
def test_ssh_listening(host):
    assert host.socket("tcp://0.0.0.0:22").is_listening

def test_http_listening(host):
    assert host.socket("tcp://0.0.0.0:80").is_listening
```

| Attribute | Returns |
|-----------|---------|
| `.is_listening` | `True` if socket is bound |

### host.user(name)

Inspect system users.

```python
def test_deploy_user(host):
    u = host.user("deploy")
    assert u.exists
    assert u.shell == "/bin/bash"
    assert u.home == "/home/deploy"
    assert "sudo" in u.groups
```

| Attribute | Returns |
|-----------|---------|
| `.exists` | `True` if user exists |
| `.uid` | Numeric user ID |
| `.gid` | Numeric primary group ID |
| `.groups` | List of group names |
| `.home` | Home directory path |
| `.shell` | Login shell path |

### host.group(name)

Inspect system groups.

```python
def test_ops_group(host):
    g = host.group("ops")
    assert g.exists
```

| Attribute | Returns |
|-----------|---------|
| `.exists` | `True` if group exists |
| `.gid` | Numeric group ID |

### host.process

Query running processes.

```python
def test_nginx_process(host):
    procs = host.process.filter(comm="nginx")
    assert len(procs) > 0
```

| Method | Returns |
|--------|---------|
| `.filter(comm=..., user=...)` | List of matching processes |
| `.get(comm=...)` | Single process (raises if not exactly one) |

### host.command(cmd)

Run arbitrary commands and inspect output.

```python
def test_ansible_version(host):
    cmd = host.command("ansible --version")
    assert cmd.rc == 0
    assert "core 2" in cmd.stdout

def test_no_world_writable(host):
    cmd = host.command("find /etc -perm -002 -type f")
    assert cmd.stdout.strip() == ""
```

| Attribute | Returns |
|-----------|---------|
| `.rc` | Exit code (0 = success) |
| `.stdout` | Standard output string |
| `.stderr` | Standard error string |

### host.system_info

Identify the operating system.

```python
def test_os_family(host):
    assert host.system_info.type == "linux"
    assert host.system_info.distribution in ["ubuntu", "rocky"]
```

| Attribute | Returns |
|-----------|---------|
| `.type` | OS type (`linux`, `freebsd`, etc.) |
| `.distribution` | Distro name (lowercase) |
| `.release` | Version string |
| `.codename` | Release codename (Debian/Ubuntu) |

### host.sysctl(param)

Read kernel parameters directly.

```python
def test_ip_forwarding_disabled(host):
    assert host.sysctl("net.ipv4.ip_forward") == 0

def test_syn_cookies(host):
    assert host.sysctl("net.ipv4.tcp_syncookies") == 1
```

Returns the parameter value directly (int or string depending on the kernel parameter).

### host.interface(name)

Inspect network interfaces.

```python
def test_loopback(host):
    lo = host.interface("lo")
    assert lo.exists
    assert "127.0.0.1" in lo.addresses
```

| Attribute | Returns |
|-----------|---------|
| `.exists` | `True` if interface exists |
| `.addresses` | List of IP addresses |

---

## Multi-Platform Testing

Fleet infrastructure runs multiple OS families. Your tests must handle both.

### SSH Connection with Multiple Hosts

Testinfra connects to Molecule instances via SSH using published ports:

```bash
pytest --hosts='ssh://root@localhost:2221,ssh://root@localhost:2222' \
       --ssh-identity-file=../../.ssh/id_rsa \
       tests/
```

### Conditional Assertions by OS

Use `host.system_info.distribution` to branch logic when behaviour differs across platforms.

```python
def test_firewall_service(host):
    distro = host.system_info.distribution
    if distro == "ubuntu":
        assert host.service("ufw").is_enabled
    elif distro == "rocky":
        assert host.service("firewalld").is_enabled

def test_ssh_package(host):
    distro = host.system_info.distribution
    if distro == "ubuntu":
        assert host.package("openssh-server").is_installed
    elif distro == "rocky":
        assert host.package("openssh-server").is_installed
```

### Parametrised Tests

Use pytest parametrize for values that differ by OS:

```python
import pytest

SSH_PACKAGES = {
    "ubuntu": "openssh-server",
    "rocky": "openssh-server",
}

def test_ssh_installed(host):
    distro = host.system_info.distribution
    pkg_name = SSH_PACKAGES.get(distro, "openssh-server")
    assert host.package(pkg_name).is_installed
```

### conftest.py for Host Selection

Standard pattern used across SDC missions:

```python
import pytest
import testinfra

def pytest_addoption(parser):
    parser.addoption("--hosts", action="store", default=None)
    parser.addoption("--ssh-identity-file", action="store", default=None)

@pytest.fixture(scope="module", params=hosts)
def host(request):
    yield testinfra.get_host(request.param)
```

---

## Common Patterns

### Testing SSH Configuration

```python
def test_sshd_hardened(host):
    cfg = host.file("/etc/ssh/sshd_config")
    assert cfg.exists
    assert cfg.contains("PermitRootLogin no")
    assert cfg.contains("PasswordAuthentication no")
    assert cfg.contains("MaxAuthTries 3")
    assert host.service("sshd").is_running
    assert host.socket("tcp://0.0.0.0:22").is_listening
```

### Testing Firewall Rules

```python
def test_ufw_active(host):
    """Ubuntu firewall."""
    cmd = host.command("ufw status")
    assert "Status: active" in cmd.stdout
    assert "22/tcp" in cmd.stdout

def test_firewalld_active(host):
    """Rocky/RHEL firewall."""
    assert host.service("firewalld").is_running
    cmd = host.command("firewall-cmd --list-services")
    assert "ssh" in cmd.stdout
```

### Testing File Permissions

```python
def test_sensitive_files(host):
    for path, expected_mode in [
        ("/etc/shadow", 0o640),
        ("/etc/gshadow", 0o640),
        ("/etc/passwd", 0o644),
    ]:
        f = host.file(path)
        assert f.exists
        assert f.mode == expected_mode, f"{path} mode {oct(f.mode)} != {oct(expected_mode)}"
```

### Testing Service State

```python
REQUIRED_SERVICES = ["sshd", "rsyslog", "cron"]
BANNED_SERVICES = ["telnet", "rsh"]

def test_required_running(host):
    for name in REQUIRED_SERVICES:
        svc = host.service(name)
        assert svc.is_running, f"{name} not running"
        assert svc.is_enabled, f"{name} not enabled"

def test_banned_stopped(host):
    for name in BANNED_SERVICES:
        svc = host.service(name)
        assert not svc.is_running, f"{name} should not be running"
```

### Testing Package Presence / Absence

```python
REQUIRED_PACKAGES = ["fail2ban", "auditd", "unattended-upgrades"]
BANNED_PACKAGES = ["telnetd", "rsh-server"]

def test_required_installed(host):
    for name in REQUIRED_PACKAGES:
        assert host.package(name).is_installed, f"{name} not installed"

def test_banned_removed(host):
    for name in BANNED_PACKAGES:
        assert not host.package(name).is_installed, f"{name} should be removed"
```

### Testing Sysctl Values

```python
HARDENED_SYSCTLS = {
    "net.ipv4.ip_forward": 0,
    "net.ipv4.tcp_syncookies": 1,
    "net.ipv4.conf.all.accept_redirects": 0,
    "net.ipv4.conf.all.send_redirects": 0,
    "net.ipv6.conf.all.accept_redirects": 0,
}

def test_sysctl_hardened(host):
    for param, expected in HARDENED_SYSCTLS.items():
        actual = host.sysctl(param)
        assert actual == expected, f"{param} = {actual}, expected {expected}"
```

---

## Mission Cross-Reference

| Mission | Testing Focus |
|---------|---------------|
| 2.1 — Weapon Handling Test | Molecule deep dive, obstacle course (write tests, write roles to pass tests) |
| 2.2 — Compliance as Code | CIS benchmark tests, Lynis integration, tagged compliance controls |
| 2.3 — Fleet Sync | Testing rolling updates, delegation patterns, fleet-wide assertions |
| Master Simulation | Full synthesis — build and test a complete hardening pipeline |

---

*FM-3 — Starfall Defence Corps Academy, 2187*
