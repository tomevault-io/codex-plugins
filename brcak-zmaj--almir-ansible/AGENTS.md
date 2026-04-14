# Copilot Instructions for almir.ansible Collection

> This collection is published to Ansible Galaxy as `brcak_zmaj.almir_ansible`.

## Project Structure

| Location | Purpose |
|----------|---------|
| `roles/` | Public Ansible roles for Galaxy distribution |
| `plugins/` | Collection plugins (if any) |
| `galaxy.yml` | Collection metadata and versioning |

## Ansible Authoring Rules

### Module Usage - ALWAYS Use FQCN

```yaml
# CORRECT
- ansible.builtin.copy:
- ansible.builtin.template:
- ansible.builtin.service:
- ansible.posix.mount:
- community.general.archive:

# WRONG - Never use short names
- copy:
- template:
```

### Variables & Defaults - Critical Rules

**`defaults/main.yml` is the ONLY place for default values.**

```yaml
# WRONG - Hardcoding values that exist in defaults
- ansible.builtin.file:
    path: /etc/netdata           # BAD: Use {{ netdata_config_dir }}
    mode: '0755'                 # BAD: Use {{ netdata_config_dir_mode }}

# WRONG - Redundant default() filter when variable exists in defaults
when: my_var | default(false) | bool
loop: "{{ my_list | default([]) }}"

# CORRECT - Direct reference to variables from defaults/main.yml
when: my_var | bool
loop: "{{ my_list }}"
path: "{{ netdata_config_dir }}"
mode: "{{ netdata_config_dir_mode }}"
```

### Conditionals - Be Explicit

```yaml
# WRONG
when: my_var

# CORRECT
when: my_var is defined and my_var | bool
```

### OS-Specific Tasks - Always Guard

```yaml
# Pattern from roles/proxmox/tasks/main.yml, roles/docker_tuning/tasks/main.yml
- name: Include OS-specific tasks
  ansible.builtin.include_tasks: "{{ ansible_os_family }}.yml"
  when: ansible_os_family in ['RedHat', 'Debian']
```

### Handlers - All Service Restarts

```yaml
# In tasks - notify handler
- ansible.builtin.template:
    src: config.j2
    dest: "{{ config_path }}"
  notify: Restart myservice

# In handlers/main.yml - pattern from roles/netdata/handlers/main.yml
- name: Restart netdata
  become: true
  ansible.builtin.service:
    name: netdata
    state: restarted
```

### File Management - Always Set Permissions

```yaml
# Pattern from roles/netdata/tasks/main.yml
- ansible.builtin.get_url:
    url: "{{ netdata_kickstart_url }}"
    dest: "{{ netdata_kickstart_script_path }}"
    mode: "{{ netdata_kickstart_script_mode }}"
```

### Task Naming & Structure

- **ALL** tasks, handlers, blocks **MUST** have `name:`
- Use tags consistently (see `tags: netdata` pattern)
- Use `block/rescue/always` for multi-step operations
- Line length under 160 characters

## Role Structure Requirements

Every role must follow this structure:

```
role_name/
├── tasks/main.yml       # Entry point, includes OS-specific files
├── handlers/main.yml    # Service restart/reload handlers
├── defaults/main.yml    # ALL default values with documentation
├── vars/                # OS-specific overrides if needed
├── templates/           # Jinja2 templates
├── files/               # Static files
└── README.md            # Required documentation
```

## Documentation Pattern

Follow README examples in existing roles. Must include:

- Role purpose and description
- Supported platforms (`ansible_os_family`)
- Required variables (no defaults)
- Optional variables with defaults
- Example playbook usage
- Dependencies and collections

## Common Commands

```bash
# Lint roles
ansible-lint roles/

# Dry run playbook
ansible-playbook -i inventory playbook.yml --check

# Show changes
ansible-playbook -i inventory playbook.yml --diff

# Build collection for Galaxy
ansible-galaxy collection build

# Install collection locally
ansible-galaxy collection install brcak_zmaj.almir_ansible
```

## Safety Rules

- **NEVER** invent variables, paths, hosts, users, or environments
- **NEVER** use `ansible.builtin.shell` or `ansible.builtin.command` when a native module exists
- **ASK** before running validation or tests on remote systems
- **ASK** if requirements are ambiguous or unclear
- **NEVER** suggest generating, modifying, copying, or relocating SSH keys unless explicitly requested

## Private vs. Public Roles

This collection contains **public, reusable roles** published to Ansible Galaxy. For private infrastructure automation:
- See `/mnt/software/Ansible/` for the main private workspace
- Private roles live in `/mnt/software/Ansible/roles/`
- Production playbooks are in `/mnt/software/Ansible/Gitea/`

Public collection guidelines:
- Every role MUST include a comprehensive README.md
- Test across multiple OS families (RedHat, Debian, Ubuntu)
- Keep defaults configurable; minimize hardcoding
- Document all variables with descriptions
- Use semantic versioning in `galaxy.yml`

## Galaxy Publishing

When updating a public role:

1. Make changes in `roles/role_name/`
2. Update `README.md` with new variables/behavior
3. Increment `version:` in `galaxy.yml` (e.g., 1.5.1 → 1.5.2)
4. Commit and push to GitHub
5. Collection is automatically published via GitHub Actions

## References

- **Ansible Instructions**: See `/mnt/software/Ansible/.cursor/rules/ansible.mdc`
- **Extended Guidance**: `/mnt/software/Ansible/CLAUDE.md` covers Ansible, Python, Docker standards
- **Example Patterns**: Study roles in `roles/{netdata,proxmox,docker_tuning,NUT,raspberry_pi}` for best practices
- **Private Workspace**: `/mnt/software/Ansible/.github/copilot-instructions.md` for workspace-specific guidance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brcak-zmaj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
