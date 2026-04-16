# NixOS Configuration Repository - AI Agent Instructions

## Architecture Overview

This is a **flake-based NixOS configuration** with a modular structure separating system-level and user-level configurations:

- **`flake.nix`**: Entry point defining two outputs:
  - `nixosConfigurations`: System configurations (currently `Bromma-Laptop`)
  - `homeConfigurations`: User configurations via home-manager (currently `aaron`)
- **`Systems/`**: Host-specific system configurations (e.g., `Bromma-Laptop/configuration.nix`)
- **`Users/`**: Home-manager configurations for user environments
- **`Modules/`**: Reusable configuration modules organized by category
- **`Themes/`**: Wallpapers and visual assets

### Module System Pattern

All custom modules follow a **NixOS module pattern** with options:

```nix
# Example: Modules/Applications/Tailscale/default.nix
{
  options = {
    module.apps.tailscale.enable = mkOption {
      description = "Enable Tailscale Module.";
      default = true;
      type = types.bool;
    };
  };
  
  config = mkIf cfg.enable {
    # Actual configuration here
  };
}
```

- Modules are **imported in system configurations**, not enabled via flake inputs
- Use the namespace `module.<category>.<name>.enable` for toggleable features
- `Modules/Common/` is imported by all systems and provides base settings

## Key Configuration Patterns

### System-Level vs User-Level Split

- **System configs** (`Systems/*/configuration.nix`): Hardware, drivers, system packages, services
- **User configs** (`Users/*/home.nix`): Dotfiles, shell configuration, user packages, theming

Example imports in system config:
```nix
imports = [
  ./hardware-configuration.nix
  ../../Modules/Common
  ../../Modules/Users/aaron
  ../../Modules/Applications/3DPrinting
];
```

### Dual GPU Setup (NVIDIA + AMD)

This laptop uses **NVIDIA Prime Offload** with AMD iGPU as primary:
- AMD iGPU (`amdgpu`) is the default for Wayland/desktop
- NVIDIA dGPU available via `nvidia-offload` command for demanding applications
- Bus IDs: NVIDIA=`PCI:100:0:0`, AMD=`PCI:101:0:0`
- Fine-grained power management enabled to save battery

### Shell Configuration (Zsh)

- **Oh-My-Zsh** with plugins: `git`, `docker`, `docker-compose`, `z`
- **Oh-My-Posh** with Catppuccin theme
- **Catppuccin Mocha** theme downloaded at runtime via `builtins.fetchurl` (with SHA256)
- Custom aliases defined in `home.shellAliases`

## Essential Commands

### System Rebuild
```bash
# Full system rebuild (uses flake)
sudo nixos-rebuild switch --flake /home/aaron/.dotfiles
# Aliased as: nix-rebuild
```

### Home Manager Rebuild
```bash
# User environment rebuild
home-manager switch --flake /home/aaron/.dotfiles
# Aliased as: home-rebuild
```

### Garbage Collection
```bash
# Remove old generations and packages
sudo nix-collect-garbage -d
```

### Flake Updates
```bash
# Update all flake inputs (nixpkgs, home-manager, etc.)
nix flake update
```

## Module Organization

```
Modules/
├── Common/           # Base config applied to all systems (timezone, locale, flakes enabled)
│   └── Printing/     # CUPS printing with comprehensive driver support
├── Applications/     # Optional application bundles
│   ├── 3DPrinting/   # Orca Slicer, Snapmaker Luban (uses permittedInsecurePackages)
│   └── Tailscale/    # VPN service with enable option
├── Hardware/         # Hardware-specific configurations
│   └── GPU/Nvidia/   # (Currently unused, inline in system config)
└── Users/            # User account definitions
    └── aaron/        # Creates user with groups, shell, and udev rules
```

## Critical Conventions

### When Adding New Modules

1. **Create with options pattern**: Always use `mkOption` with `mkIf` guards
2. **Set sensible defaults**: `default = false` for optional apps, `true` for common services
3. **Import in appropriate location**: 
   - System modules → `Systems/*/configuration.nix`
   - User modules → automatically imported if needed, or in `Users/*/home.nix`

### When Modifying System Packages

- **System packages**: Add to `environment.systemPackages` in `configuration.nix`
- **User packages**: Add to `home.packages` in `home.nix`
- Never install packages imperatively with `nix-env`

### Unfree/Insecure Packages

- Unfree allowed globally in `Modules/Common/default.nix`
- For insecure packages, add to `nixpkgs.config.permittedInsecurePackages` in the specific module

### Special Dependencies

- **1Password SSH agent**: Configured in `home.nix` with socket path `~/.1password/agent.sock`
- **PlatformIO udev rules**: Added via `services.udev.packages` in user module
- **Docker**: User needs `docker` group membership (set in user module)

## Development Environment

- **Primary formatter**: `nixfmt-rfc-style` (installed system-wide)
- **LSP**: `nixd` for Nix language server
- **Editor**: VSCode (system package) with expected Nix extensions
- **Shell**: Zsh with completions, syntax highlighting, autosuggestions enabled

## Hardware-Specific Notes

- **ASUS laptop**: Uses `asusctl` and `asusd` service for hardware control
- **Bluetooth**: Experimental features enabled in settings
- **Boot**: systemd-boot with latest kernel (`linuxPackages_latest`)
- **Desktop**: KDE Plasma 6 with Wayland on SDDM
- **Auto-login**: Enabled for user `aaron`

## External Integrations

- **Avahi**: Enabled for network printer discovery
- **Tailscale**: VPN with system service when module enabled
- **Docker**: Virtualization enabled at system level
- **Steam**: Gaming with gamescope session support
- **CIFS mounts**: Commented examples show pattern for network shares with credentials file

## Testing Changes

After modifying configurations:
1. Run `sudo nixos-rebuild switch --flake /home/aaron/.dotfiles` for system changes
2. Run `home-manager switch --flake /home/aaron/.dotfiles` for user changes
3. No separate test command - rebuild IS the test (atomic rollback available via boot menu)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bioresestance) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
