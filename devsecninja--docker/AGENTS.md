# Copilot Instructions for Docker Infrastructure Repository

## Project Overview

This is an Ansible-based infrastructure automation repository that manages Docker servers using the **Ansible Pull** approach. Servers automatically pull their configuration from this repository and apply changes when Git commits are detected.

## Technology Stack

- **Ansible**: 2.20+ (Infrastructure as Code)
- **Python**: 3.8+ (Ansible runtime)
- **Docker**: Container platform managed by Ansible
- **Task**: Task runner / build tool (taskfile.dev)
- **YAML**: Configuration and playbook syntax
- **Bash**: Helper scripts
- **Bats**: Testing framework
- **GitHub Actions**: CI/CD pipeline

## Key Commands

### Important: Always Use Task Runner
When suggesting commands to the user, **always prefer `task` commands** from `Taskfile.yml` over raw commands. Only fall back to raw commands if no equivalent task exists. Run `task --list` to discover available tasks.

### Important: Always Use mise for Tool Management
When installing or managing tool versions (Python, Ansible, ansible-lint, yamllint, Bats, Task, etc.), **always use `mise`** instead of `pip install`, `apt install`, or other package managers. Tool versions are defined in `.mise.toml` and must stay in sync.
```bash
# Install all tools at versions defined in .mise.toml
mise install

# Check installed tool versions
mise ls

# Update a tool version (edits .mise.toml)
mise use pipx:ansible-lint@26.5.0
```

### Task Runner (Recommended)
```bash
# Show all available tasks
task --list

# Install all dependencies
task install

# Run all tests
task test

# Run linting and syntax checks
task ci:quick

# Run full CI pipeline locally
task ci:local

# Check what would change without applying
task ansible:check

# Show detailed help
task help
```

### Testing
```bash
# Run all Bats tests (with Task)
task test

# Run specific test suite
task test:lint
task test:syntax
task test:docker

# Run all Bats tests (without Task)
./tests/bash/run-tests.sh

# Run specific test
./tests/bash/run-tests.sh --test lint-test.bats

# Run in CI mode (for automated pipelines)
./tests/bash/run-tests.sh --ci
```

### Running Ansible
```bash
# Dry-run with Task
task ansible:check

# Apply configuration with Task
task ansible:pull

# Manual ansible-pull execution
sudo ansible-pull \
    --url https://github.com/DevSecNinja/docker.git \
    --checkout main \
    --directory /var/lib/ansible/local \
    --inventory ansible/inventory/hosts.yml \
    --extra-vars "target_host=$(hostname)" \
    --only-if-changed \
    ansible/playbooks/main.yml
```

### Local Development
```bash
# Install all dependencies (with Task)
task install

# Install required Ansible collections
ansible-galaxy collection install -r ansible/requirements.yml

# Check playbook syntax locally
ansible-playbook ansible/playbooks/main.yml --syntax-check
```

## Repository Structure

```
ansible/
├── requirements.yml         # External roles and collections
├── playbooks/
│   └── main.yml            # Main playbook for ansible-pull
├── inventory/
│   ├── hosts.yml           # Server inventory
│   └── host_vars/          # Host-specific variables
├── roles/                   # Custom Ansible roles
│   ├── ansible_pull_setup/ # Ansible-pull automation
│   ├── chezmoi/            # Dotfiles management
│   ├── docker_compose_modules/  # Modular Docker Compose
│   └── ufw/                # Firewall configuration
└── scripts/
    └── ansible-pull.sh     # Installation wrapper

tests/
└── bash/                    # Bats test suite
    ├── run-tests.sh        # Test runner
    ├── lint-test.bats      # Linting tests
    ├── syntax-test.bats    # Syntax tests
    ├── docker-test.bats    # Docker provisioning tests
    ├── ansible-pull-test.bats  # ansible-pull tests
    └── github-ssh-keys-test.bats  # GitHub SSH keys tests
```

## Code Conventions

### Ansible Playbooks and Roles
- Use YAML with 2-space indentation
- Follow Ansible best practices for role structure
- All playbooks must start with `---`
- Use descriptive task names with proper capitalization
- Use `ansible.builtin.*` module names explicitly
- Always include `meta/main.yml` with role dependencies
- Use `defaults/main.yml` for default variables
- Use `handlers/main.yml` for service restarts

### YAML Style
- Use 2-space indentation (never tabs)
- No trailing whitespace
- End files with a single newline
- Use `---` document start marker
- Quote strings when they contain special characters
- Use lowercase for booleans: `true`, `false`

### Naming Conventions
- Roles: lowercase with underscores (e.g., `ansible_pull_setup`)
- Variables: lowercase with underscores (e.g., `server_features`)
- Host names: lowercase (e.g., `svlazext`)
- Tags: lowercase (e.g., `docker`, `traefik`)

### File Organization
- Group related tasks in role subdirectories
- Use `tasks/main.yml` as the entry point for roles
- Split complex roles into separate task files
- Store templates in `templates/` directory
- Keep defaults in `defaults/main.yml`

## Testing Requirements

### Before Making Changes
1. **ALWAYS run tests first**: `./tests/bash/run-tests.sh`
   - This is a required step before committing any Ansible or YAML changes
   - Fix all test failures before proceeding
2. For specific test categories:
   - Linting: `./tests/bash/run-tests.sh --test lint-test.bats`
   - Syntax: `./tests/bash/run-tests.sh --test syntax-test.bats`
   - Docker: `./tests/bash/run-tests.sh --test docker-test.bats`
   - GitHub SSH Keys: `./tests/bash/run-tests.sh --test github-ssh-keys-test.bats`

### After Making Changes
- **All YAML files must pass yamllint** (no errors, warnings acceptable)
- **All Ansible files must pass ansible-lint** (no errors, warnings acceptable)
- Playbooks must pass syntax-check
- GitHub Actions workflow must succeed
- **Run linting again before final commit** to ensure all issues are resolved

### CI/CD Pipeline
The repository uses the [Bats testing framework](https://github.com/bats-core/bats-core) for comprehensive testing:
- Lint checks (yamllint, ansible-lint)
- Syntax validation (Ansible playbooks and shell scripts)
- Docker provisioning tests
- Ansible-pull functionality tests
- GitHub SSH keys role tests

All tests are located in `tests/bash/` and use the Bats test format.

Workflows trigger on:
- Push to `main` or `copilot/**` branches
- Pull requests
- Manual workflow dispatch

See [tests/bash/README.md](../tests/bash/README.md) for detailed testing documentation.

## Important Boundaries

### DO NOT Modify
- `.git/` directory or git history
- LICENSE file (MIT License)
- Production server configurations without explicit approval
- Secrets or credentials (use Ansible Vault if needed)

### DO Modify Carefully
- `ansible/inventory/hosts.yml` - Server inventory (only with clear understanding)
- `ansible/playbooks/main.yml` - Main playbook (ensure backward compatibility)
- GitHub Actions workflows - Test thoroughly before merging

### ALWAYS
- Follow existing code structure and patterns
- Test changes using provided test scripts
- Maintain backward compatibility with existing servers
- Document significant changes in commit messages
- Use feature branches for development
- Keep changes minimal and focused

## Example Workflows

### Adding a New Ansible Role
1. Create role structure:
   ```bash
   cd ansible/roles
   mkdir -p newrole/{tasks,defaults,meta,templates,handlers}
   ```

2. Create `tasks/main.yml`:
   ```yaml
   ---
   - name: Example task
     ansible.builtin.debug:
       msg: "This is a new role"
   ```

3. Add `meta/main.yml` with dependencies:
   ```yaml
   ---
   dependencies: []
   ```

4. Add to `ansible/playbooks/main.yml`
5. Test with CI pipeline

### Adding a New Server
1. Add to `ansible/inventory/hosts.yml`:
   ```yaml
   NEWSERVER:
     ansible_host: newserver.local
     ansible_user: ansible
     server_features:
       - docker
   ```

2. (Optional) Add host-specific vars in `ansible/inventory/host_vars/NEWSERVER.yml`
3. Test the configuration

### Modifying Docker Compose Modules
1. Edit module definition in `ansible/roles/docker_compose_modules/vars/modules/`
2. Update role tasks if needed in `ansible/roles/docker_compose_modules/tasks/`
3. Test with: `bash ansible/scripts/tests/docker-test.sh`

## Security Considerations

- Never commit secrets, API keys, or passwords
- Use Ansible Vault for sensitive data (when implemented)
- Follow principle of least privilege for user permissions
- Keep UFW firewall rules restrictive
- Review Docker container security best practices
- Validate all external inputs

## Documentation

- Update README.md for significant feature additions
- Update INSTALL.md for installation procedure changes
- Keep inline comments minimal unless explaining complex logic
- Use descriptive task names that explain the purpose
- Document role variables in `defaults/main.yml` comments

## Common Pitfalls to Avoid

1. Don't use tabs in YAML files (use 2 spaces)
2. **Don't skip linting and syntax checks** - Always run before committing
3. Don't break ansible-pull functionality (it's core to this repo)
4. Don't hardcode values that should be variables
5. Don't remove or modify unrelated tests
6. Don't add unnecessary dependencies
7. Always use `--no-pager` with git commands in scripts to avoid interactive prompts
8. **Always verify linting passes** before marking work as complete

## Additional Resources

- Ansible Documentation: https://docs.ansible.com/
- Ansible Best Practices: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
- Docker Documentation: https://docs.docker.com/
- Repository README: [README.md](../README.md)
- Installation Guide: [INSTALL.md](../INSTALL.md)

---
> Source: [DevSecNinja/docker](https://github.com/DevSecNinja/docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
