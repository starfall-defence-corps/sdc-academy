# FM-2: YAML & Jinja2 Quick Reference
> Starfall Defence Corps — Field Manual

YAML is the language you write Ansible in. Jinja2 is the template engine that makes it dynamic. Master both or your playbooks will break in ways the error messages will not explain.

This manual is a reference, not a tutorial. If you need the walkthrough, start with [Mission 1.2: Lock the Door](https://github.com/starfall-defence-corps/mission-1-2-lock-the-door) for YAML structure and [Mission 1.4: Many Ships](https://github.com/starfall-defence-corps/mission-1-4-many-ships) for Jinja2 templating.

---

## Part 1: YAML

YAML (YAML Ain't Markup Language) is a data serialisation format. Every Ansible playbook, inventory file, and variable file is YAML. Get the fundamentals right here and you will avoid hours of debugging indentation errors in the field.

### Data Types

```yaml
# Strings
ship_name: Artemis
ship_class: "Vanguard-II"       # Quotes optional unless value contains special chars

# Integers
hull_strength: 9500
deck_count: 12

# Floats
shield_efficiency: 0.97

# Booleans
weapons_online: true
stealth_mode: false

# Null
last_breach_date: null           # Also accepted: ~
cargo_manifest:                  # Empty value is also null
```

> **First encountered**: Mission 1.1 — your fleet inventory file uses all of these.

### Lists

Block style (one item per line):

```yaml
active_vessels:
  - Artemis
  - Constellation
  - Vigilant
```

Flow style (inline):

```yaml
active_vessels: [Artemis, Constellation, Vigilant]
```

List of dictionaries (the most common pattern in playbooks):

```yaml
tasks:
  - name: Harden SSH configuration
    ansible.builtin.template:
      src: sshd_config.j2
      dest: /etc/ssh/sshd_config

  - name: Restart SSH daemon
    ansible.builtin.service:
      name: sshd
      state: restarted
```

> **First encountered**: Mission 1.2 — your first playbook is a list of tasks.

### Dictionaries

Block style:

```yaml
ship_specs:
  name: Artemis
  class: Vanguard-II
  crew: 340
  weapons_online: true
```

Flow style:

```yaml
ship_specs: {name: Artemis, class: Vanguard-II, crew: 340}
```

Nested dictionaries:

```yaml
fleet:
  sector_7:
    flagship: Artemis
    escorts:
      - Vigilant
      - Iron Resolve
  sector_12:
    flagship: Constellation
    escorts:
      - Stalwart
```

### Multiline Strings

Literal block (`|`) — preserves newlines exactly:

```yaml
motd_banner: |
  ==========================================
  STARFALL DEFENCE CORPS — CLASSIFIED SYSTEM
  Unauthorised access will be prosecuted.
  ==========================================
```

Folded block (`>`) — folds newlines into spaces (becomes one paragraph):

```yaml
ship_description: >
  The Artemis is a Vanguard-class cruiser
  assigned to Sector 7 patrol. She carries
  a crew of 340 and full weapons complement.
```

Strip modifier (`-`) — removes the trailing newline:

```yaml
# With trailing newline (default)
banner: |
  Line one
  Line two

# Without trailing newline
banner: |-
  Line one
  Line two
```

| Indicator | Newlines      | Trailing newline |
|-----------|---------------|------------------|
| `\|`      | Preserved     | Yes              |
| `\|-`     | Preserved     | No               |
| `\|+`     | Preserved     | All kept         |
| `>`       | Folded        | Yes              |
| `>-`      | Folded        | No               |
| `>+`      | Folded        | All kept         |

> **First encountered**: Mission 1.2 — SSH banners use literal blocks. Mission 1.4 — templates use both styles.

### Boolean Coercion Gotcha

YAML 1.1 (used by PyYAML, which Ansible uses) treats all of these as booleans:

```yaml
# All of these become True
enabled: yes
enabled: Yes
enabled: YES
enabled: true
enabled: True
enabled: on

# All of these become False
enabled: no
enabled: No
enabled: false
enabled: off
```

If you mean the **string** `"yes"`, you must quote it:

```yaml
survey_response: "yes"          # String "yes"
survey_response: yes            # Boolean true — not what you wanted
```

YAML 1.2 removed `yes`/`no`/`on`/`off` as booleans, but Ansible still uses YAML 1.1 under the hood. Know this. It will bite you.

### Anchors and Aliases

Define a value once with `&`, reuse it with `*`:

```yaml
defaults: &security_defaults
  ssh_port: 2222
  permit_root: false
  max_auth_tries: 3

web_server:
  <<: *security_defaults
  role: web

db_server:
  <<: *security_defaults
  role: database
  ssh_port: 2223              # Override one value
```

The `<<` merge key inserts all key-value pairs from the anchor. Individual keys can be overridden after the merge. Useful for DRY variable files, but use sparingly — deeply nested merges become hard to debug.

### Comments

```yaml
# Full line comment
ship_name: Artemis   # Inline comment
```

YAML comments are not preserved by most parsers. Ansible will never see them. Use them for human readers.

### Quoting Rules — When You MUST Quote

| Situation | Example | Why |
|-----------|---------|-----|
| Value starts with `{` or `[` | `"{{ variable }}"` | YAML sees a flow mapping/sequence |
| Value contains `: ` (colon-space) | `"http://fleet:8080"` | YAML sees a key-value pair |
| Value is a YAML boolean word | `"yes"`, `"no"`, `"true"` | Parsed as boolean otherwise |
| Value is a number but you want a string | `"0700"`, `"3.0"` | Parsed as int/float otherwise |
| Value contains `#` | `"C# hardening"` | YAML sees a comment |
| Value starts with `*`, `&`, `!`, `%`, `@`, `` ` `` | `"*priority"` | Reserved YAML indicators |
| Empty string | `""` | Bare empty is null, not empty string |

When in doubt, quote it. Double quotes allow escape sequences (`\n`, `\t`). Single quotes are literal (no escapes). Either works for the cases above.

---

## Part 2: Jinja2 in Ansible

Jinja2 is the template engine Ansible uses for dynamic content. It appears in two places: template files (`.j2`) and inline within playbook YAML. The syntax is the same in both, but the rules around quoting differ.

### Variable Substitution

```yaml
# In a playbook
- name: Set hostname
  ansible.builtin.hostname:
    name: "{{ inventory_hostname }}"

# In a template (.j2 file)
Hostname: {{ inventory_hostname }}
Sector: {{ sector_assignment }}
```

Variables come from: inventory, `group_vars/`, `host_vars/`, `vars:` sections, registered results, and Ansible facts.

```yaml
# Accessing nested values
network_ip: "{{ ansible_default_ipv4.address }}"

# Accessing list items
first_dns: "{{ dns_servers[0] }}"
```

> **First encountered**: Mission 1.4 — variables and templates are the core objective.

### Filters

Filters transform data. Syntax: `{{ value | filter }}`. You can chain them: `{{ value | filter1 | filter2 }}`.

**String filters**:

```yaml
# Case manipulation
hostname_lower: "{{ ship_name | lower }}"          # artemis
hostname_upper: "{{ ship_name | upper }}"          # ARTEMIS

# Search and replace
clean_name: "{{ raw_input | replace(' ', '_') }}"
sanitised: "{{ log_line | regex_replace('[0-9]+', 'X') }}"
```

**Default values** — critical for defensive coding:

```yaml
# Use default if variable is undefined
ssh_port: "{{ custom_ssh_port | default(22) }}"

# Use default if variable is undefined OR empty
ssh_port: "{{ custom_ssh_port | default(22, true) }}"
```

**Data structure filters**:

```yaml
# Convert dict to list of {key, value} pairs
- name: Apply sysctl settings
  ansible.posix.sysctl:
    name: "{{ item.key }}"
    value: "{{ item.value }}"
  loop: "{{ sysctl_settings | dict2items }}"

# Merge two dictionaries
combined: "{{ defaults | combine(overrides) }}"

# Deep merge (recursive)
combined: "{{ defaults | combine(overrides, recursive=True) }}"
```

> **`dict2items` first encountered**: Mission 2.2 — iterating over CIS benchmark settings.

**Serialisation filters**:

```yaml
# Convert to YAML or JSON (useful in templates)
config_block: "{{ settings | to_yaml }}"
api_payload: "{{ data | to_json }}"
human_readable: "{{ data | to_nice_yaml }}"
```

**Other commonly used filters**:

```yaml
# List operations
unique_hosts: "{{ all_hosts | unique }}"
sorted_ships: "{{ fleet | sort(attribute='name') }}"
web_only: "{{ fleet | selectattr('role', 'eq', 'web') | list }}"

# Math
max_value: "{{ scores | max }}"
total: "{{ values | sum }}"

# Type casting
port_number: "{{ ssh_port | int }}"
enabled: "{{ flag | bool }}"

# Path manipulation
filename: "{{ filepath | basename }}"
directory: "{{ filepath | dirname }}"
```

### Conditionals

**In templates** (`.j2` files):

```jinja
{% if weapons_online %}
ALERT: Weapons systems are HOT.
{% elif maintenance_mode %}
NOTE: Ship is in maintenance cycle.
{% else %}
STATUS: Standard patrol configuration.
{% endif %}
```

**In playbooks** (`when:` clause):

```yaml
- name: Install firewalld on RHEL-based systems
  ansible.builtin.dnf:
    name: firewalld
    state: present
  when: ansible_os_family == "RedHat"

# Multiple conditions
- name: Apply hardening only on production web servers
  ansible.builtin.include_role:
    name: cis_hardening
  when:
    - environment == "production"
    - "'web' in group_names"

# Boolean checks
- name: Restart if config changed
  ansible.builtin.service:
    name: sshd
    state: restarted
  when: sshd_config.changed
```

> **First encountered**: Mission 1.4 — conditionals handle multi-OS playbooks.

### Loops

**In templates**:

```jinja
{% for ship in fleet_roster %}
{{ ship.name }} — {{ ship.class }} — Sector {{ ship.sector }}
{% endfor %}
```

**In playbooks** (modern `loop:`):

```yaml
- name: Create fleet user accounts
  ansible.builtin.user:
    name: "{{ item }}"
    groups: defence_corps
    state: present
  loop:
    - cadet_reeves
    - cadet_okafor
    - lt_vasquez
```

**Looping over dictionaries**:

```yaml
- name: Apply sysctl hardening
  ansible.posix.sysctl:
    name: "{{ item.key }}"
    value: "{{ item.value }}"
    sysctl_set: true
    reload: true
  loop: "{{ sysctl_params | dict2items }}"
```

**Loop with index**:

```jinja
{% for port in open_ports %}
Rule {{ loop.index }}: ALLOW {{ port }}/tcp
{% endfor %}
```

The legacy `with_items:` still works but `loop:` is the current standard.

### Whitespace Control

In templates, Jinja2 tags can introduce unwanted blank lines. Use the strip modifier (`-`) to consume whitespace:

```jinja
{# Without whitespace control — leaves blank lines #}
{% if enable_banner %}
{{ banner_text }}
{% endif %}

{# With whitespace control — clean output #}
{%- if enable_banner %}
{{ banner_text }}
{%- endif %}
```

| Syntax | Effect |
|--------|--------|
| `{% ... %}` | Normal — whitespace preserved |
| `{%- ... %}` | Strip whitespace before the tag |
| `{% ... -%}` | Strip whitespace after the tag |
| `{%- ... -%}` | Strip both sides |
| `{{- var -}}` | Same rules apply to variable tags |

Use whitespace control in templates that generate config files. Extra blank lines in `/etc/ssh/sshd_config` are harmless but sloppy. Extra blank lines in a cron file can break things.

### Undefined Variables and `default()`

Ansible fails hard on undefined variables by default. The `default()` filter is your safety net:

```yaml
# Provide a fallback
max_retries: "{{ custom_retries | default(3) }}"

# Provide fallback even for empty strings and false
always_set: "{{ maybe_empty | default('none', true) }}"

# Omit the parameter entirely if undefined
- name: Create user
  ansible.builtin.user:
    name: "{{ username }}"
    shell: "{{ custom_shell | default(omit) }}"
```

The `omit` special value tells Ansible to skip the parameter entirely, as if you never wrote it. This is different from passing an empty string or null.

### The Quoting Gotcha

When a YAML value **starts with** `{{`, the YAML parser sees `{` and thinks it is a flow mapping. You must quote:

```yaml
# BROKEN — YAML parser error
hostname: {{ inventory_hostname }}

# CORRECT
hostname: "{{ inventory_hostname }}"
```

In Jinja2 template files (`.j2`), you do NOT need quotes — the file is processed as a template, not as YAML:

```jinja
Hostname: {{ inventory_hostname }}
```

Rule of thumb: **In YAML files, always quote Jinja2 expressions.** In `.j2` files, never quote them (unless you want literal quote characters in the output).

---

## Part 3: Common Gotchas

These are the errors that waste the most cadet time across all missions. Learn them here or learn them the hard way in a timed simulation.

### Indentation: 2 Spaces, Never Tabs

YAML uses spaces for indentation. Tabs are illegal. The Ansible standard is 2 spaces per level.

```yaml
# CORRECT — 2 spaces
- hosts: all
  tasks:
    - name: Install package
      ansible.builtin.apt:
        name: nginx
        state: present

# BROKEN — mixed or wrong indentation
- hosts: all
  tasks:
  - name: Install package     # Wrong: task should be indented under tasks
      ansible.builtin.apt:
          name: nginx          # Wrong: 4 extra spaces
```

Configure your editor to insert spaces when you press Tab, and set the indent width to 2. Do this before Mission 1.2 or you will fight your editor the entire time.

### The "Colon Space" Trap

YAML interprets `key: value` as a dictionary entry. If your string contains `: ` (colon followed by a space), YAML will split it:

```yaml
# BROKEN — YAML sees "http" as a key
url: http://fleet-command:8080/api

# CORRECT — quoted
url: "http://fleet-command:8080/api"

# Also safe — no space after colon (but don't rely on this)
url: http://fleet-command:8080/api   # Only works because port has no space after
```

The trigger is specifically colon followed by a space. `key:value` without a space is a string. But do not rely on this — quote anything with a colon to be safe.

### Boolean Values: `yes` Is Not a String

```yaml
# This becomes boolean true, not the string "yes"
answer: yes

# These are strings
answer: "yes"
answer: 'yes'
```

This bites hardest in inventory files and variable files. If a downstream template or conditional expects a string, passing a boolean will produce unexpected behaviour. When you want a string, quote it.

### Empty Values: `null` vs `""` vs Omitted

```yaml
key:              # null (key exists, value is null)
key: null         # null (explicit)
key: ~            # null (tilde shorthand)
key: ""           # Empty string (key exists, value is "")
# key omitted     # Key does not exist at all
```

These are four different states. They behave differently in conditionals:

```yaml
# Given: my_var is null
when: my_var                    # False (null is falsy)
when: my_var is defined         # True (it exists)
when: my_var is not none        # False (it is none)

# Given: my_var is ""
when: my_var                    # False (empty string is falsy)
when: my_var is defined         # True
when: my_var | length > 0       # False
```

### Jinja2 in `when:` — No Double Braces

The `when:` clause is already evaluated as a Jinja2 expression. Adding `{{ }}` wraps it in a string, which is almost never what you want:

```yaml
# WRONG — this evaluates the variable, converts to string, then tests truthiness
- name: Check OS
  ansible.builtin.debug:
    msg: "RedHat detected"
  when: "{{ ansible_os_family == 'RedHat' }}"

# CORRECT — bare expression
- name: Check OS
  ansible.builtin.debug:
    msg: "RedHat detected"
  when: ansible_os_family == "RedHat"
```

Ansible will warn you about this with a deprecation notice, but the warning is easy to miss in dense output. Get the habit right now: **`when:` clauses never use `{{ }}`**.

### Template Whitespace: Extra Newlines in Rendered Files

When your template has Jinja2 blocks, each `{% %}` tag produces a blank line in the output:

```jinja
# Input template
{% for rule in firewall_rules %}
{{ rule }}
{% endfor %}

# Output (3 rules) — note the extra blank lines
rule1

rule2

rule3
```

Fix with whitespace stripping:

```jinja
{% for rule in firewall_rules -%}
{{ rule }}
{% endfor -%}
```

Or use `#jinja2: trim_blocks:True, lstrip_blocks:True` at the top of your template to enable automatic stripping.

> **First encountered**: Mission 1.4 — templates that generate config files must have clean output.

---

## Quick Reference Card

| What | Syntax | Example |
|------|--------|---------|
| Variable | `{{ }}` | `{{ ssh_port }}` |
| Statement | `{% %}` | `{% if enabled %}` |
| Comment | `{# #}` | `{# TODO: remove #}` |
| Filter | `\|` | `{{ name \| upper }}` |
| Default | `default()` | `{{ x \| default(22) }}` |
| Loop var | `loop.index` | First iteration = 1 |
| Loop var | `loop.index0` | First iteration = 0 |
| Loop var | `loop.first` | `true` on first iteration |
| Loop var | `loop.last` | `true` on last iteration |
| Omit param | `omit` | `{{ x \| default(omit) }}` |
| Ternary | `ternary()` | `{{ test \| ternary('yes','no') }}` |

---

*FM-2 — Starfall Defence Corps Academy. Know your data formats. The Voidborn won't wait while you debug a missing quote.*
