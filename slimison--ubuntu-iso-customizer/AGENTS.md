# GitHub Copilot Instructions for Ubuntu ISO Customizer

## Project Overview
This project creates customized Ubuntu ISO images with automated post-installation scripts for development environments.

## Code Style & Standards

### Shell Scripting
- Use bash for all shell scripts
- Include proper shebang: `#!/bin/bash`
- Use `set -euo pipefail` for error handling
- Add comprehensive logging with timestamps
- Use functions for reusable code blocks
- Include parameter validation and help text
- Use meaningful variable names with proper quoting
- Add error handling with cleanup functions

### File Organization
- Scripts go in `scripts/` directory
- Configuration files in `configs/` directory
- Documentation in `docs/` directory
- Package lists in `configs/package-lists/`
- Tool installers in `scripts/tools/`

### Documentation
- Use Markdown for all documentation
- Include code examples with proper syntax highlighting
- Add troubleshooting sections
- Keep README.md updated with latest features
- Document all configuration options

## Development Patterns

### Script Structure
```bash
#!/bin/bash
set -euo pipefail

# Global variables
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/ubuntu-customizer.log"

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Error handling
cleanup() {
    # Cleanup code here
    log "Cleaning up..."
}
trap cleanup EXIT

# Main function
main() {
    log "Starting script: $0"
    # Script logic here
}

main "$@"
```

### Package Installation
- Always update package lists before installation
- Use non-interactive mode for apt
- Add error checking for each installation
- Log installation progress
- Verify installation success

### ISO Customization
- Extract ISO to temporary directory
- Modify filesystem.squashfs for persistent changes
- Update preseed files for automation
- Use casper hooks for auto-execution
- Generate new ISO with proper labels

## Security Considerations
- Enable UFW firewall by default
- Configure SSH with key-based authentication
- Set secure file permissions (755 for scripts, 644 for configs)
- Use sudo appropriately, avoid running entire scripts as root
- Validate user inputs and file paths

## Testing Guidelines
- Test scripts on clean Ubuntu installations
- Verify ISO boots and auto-executes properly
- Test with different Ubuntu versions
- Include rollback mechanisms for failed installations
- Log all operations for debugging

## Common Libraries & Tools
- `xorriso` for ISO manipulation
- `squashfs-tools` for filesystem modification
- `genisoimage` for ISO creation
- `rsync` for file synchronization
- `jq` for JSON processing (if needed)

## Error Handling Best Practices
- Use meaningful exit codes
- Provide clear error messages
- Include context in error logs
- Implement retry mechanisms where appropriate
- Clean up temporary files on exit

## Performance Considerations
- Use parallel processing where safe
- Cache downloaded packages
- Minimize ISO size by removing unnecessary files
- Use efficient compression for squashfs

When generating code for this project, prioritize reliability, security, and user experience. Always include proper error handling and logging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Slimison) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
