# Dev Proxy GitHub Actions - AI Coding Instructions

## Architecture Overview

This is a collection of GitHub Actions for integrating [Dev Proxy](https://aka.ms/devproxy) into CI/CD workflows. The project follows GitHub's composite action pattern with each action in its own directory containing an `action.yml` file.

## Project Structure

- **Root `action.yml`**: Stub file that directs users to documentation
- **Action directories**: `install/`, `start/`, `stop/`, `record-start/`, `record-stop/`
- **Tests**: `.github/workflows/test-*.yml` for validation and `tests/node-js/` for integration testing

## Key Patterns

### Composite Actions Structure
Each action follows this pattern:
```yaml
name: Action Name
description: Brief description
inputs: # optional
  param-name:
    description: Parameter description
    required: true|false
    default: "value"
runs:
  using: composite
  steps: # bash commands using curl to interact with Dev Proxy API
```

### Dev Proxy Integration Points
- **API Base**: `http://127.0.0.1:8897/proxy` for control operations
- **Proxy Port**: `127.0.0.1:8000` for HTTP/HTTPS traffic interception
- **Environment Variables**: Always manage `http_proxy` and `https_proxy` in `$GITHUB_ENV`

### Critical Environment Management
When starting Dev Proxy, set proxy environment variables:
```bash
echo "http_proxy=http://127.0.0.1:8000" >> $GITHUB_ENV
echo "https_proxy=http://127.0.0.1:8000" >> $GITHUB_ENV
```

When stopping, **always reset** them:
```bash
echo "http_proxy=" >> $GITHUB_ENV
echo "https_proxy=" >> $GITHUB_ENV
```

### Start Action Complexity
The `start/action.yml` uses `pyTooling/Actions/with-post-step@v5.1.0` for lifecycle management:
- **Main**: Installs certificates, starts proxy, sets environment variables
- **Post**: Auto-stops proxy when `auto-stop: true` (default) and cleans environment

### Certificate Handling
Dev Proxy requires certificate trust for HTTPS interception:
- Certificate path: `~/.config/dev-proxy/rootCert/rootCert.pfx`
- Export to PEM, install to system CA store, update certificates
- Handle gracefully if certificate missing (proxy still works for HTTP)

## Testing Strategy

### Test Workflows
- `test-install-latest.yml` / `test-install-specific.yml`: Installation validation
- `test-all-defaults.yml`: Basic proxy functionality with curl
- `test-all-node.yml`: Real application integration test

### Local Testing
Use `.vscode/extensions.json` recommended extensions:
- `SanjulaGanepola.github-local-actions`: Test actions locally
- `redhat.vscode-yaml`: YAML validation

## Development Conventions

### API Calls
Use curl with explicit error handling. Recording control uses JSON payload:
```bash
curl -X POST http://127.0.0.1:8897/proxy \
  -H "Content-Type: application/json" \
  -d '{"recording": true|false}'
```

### Input Validation
For optional config files, use parameter expansion:
```bash
config_arg=""
if [ -n "${{ inputs.config-file }}" ]; then
  config_arg="-c ${{ inputs.config-file }}"
fi
```

### Wait Patterns
Dev Proxy startup requires polling the log file:
```bash
while true; do
  if grep -q "Configure your operating system to use this proxy's port and address 127.0.0.1:8000" $log_file; then
    break
  fi
  sleep 1
done
```

## External Dependencies

- **Dev Proxy**: Installed via `https://aka.ms/devproxy/setup.sh`
- **pyTooling/Actions/with-post-step**: Enables cleanup actions
- **System tools**: `openssl`, `curl`, `sudo` for certificate management

---
> Source: [dev-proxy-tools/actions](https://github.com/dev-proxy-tools/actions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-14 -->
