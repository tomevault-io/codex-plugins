# Homebrew Formulae Repository

## Testing Commands

```bash
# Test syntax for all formulae
brew test-bot --only-tap-syntax

# Build and install a single formula from source
brew install --build-from-source Formula/<formula>.rb

# Test a formula after installation
brew test <formula>

# Run full test suite (used in CI for PRs)
brew test-bot --only-formulae
```

When working from a Copilot worktree outside Homebrew's tap directory, Homebrew
may reject `Formula/<formula>.rb` paths. For local install/test validation,
temporarily symlink the worktree under `$(brew --repository)/Library/Taps/...`,
run Homebrew with `HOMEBREW_NO_REQUIRE_TAP_TRUST=1`, and remove the symlink
afterward.

## Architecture

This is a Homebrew tap with automated version updates:

- **Formula/*.rb** - Homebrew formula definitions (Ruby)
- **scripts/update-*.py** - Python scripts that auto-update each formula by fetching latest releases/commits from GitHub

The update workflow runs daily and creates PRs when new versions are detected. Each update script:
1. Fetches latest version info via `gh` CLI
2. Downloads the tarball and calculates SHA256
3. Updates the formula file with new URL, version, and checksum

## Code Style

- Ruby formulae: 2-space indentation, Homebrew Ruby style
- Python scripts: PEP 8, snake_case naming
- Formula class names: TitleCase matching filename (e.g., `codex.rb` → `class Codex`)
- Always include: HTTPS URLs, SHA256 checksums, `license` field, `test do...end` block

## Formula Structure

Formulae follow this pattern:
```ruby
class FormulaName < Formula
  desc "Short description"
  homepage "https://example.com"
  url "https://example.com/archive/v1.2.3.tar.gz"
  version "1.2.3"
  sha256 "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  license "MIT"

  depends_on "..." => :build  # build-time deps
  depends_on "..."            # runtime deps

  def install
    # installation steps
  end

  test do
    system "true"
  end
end
```

## Formula-Specific Notes

- `Formula/omlx.rb` is a head-only formula for the unstable custom `scaryrawr/omlx` fork. Upstream `jundot/omlx` formula changes are useful references, but keep the local formula pointed at the fork unless instructed otherwise.
- `omlx` installs optional `mlx-audio` from a pinned source resource. When updating that pin, recalculate the tarball SHA256 and inspect `mlx-audio`'s dependency metadata before keeping or adding `inreplace` patches.
- `omlx` forces selected native Python packages, including `watchfiles`, to build from source with headerpad linker flags so Homebrew can rewrite Mach-O install names during a HEAD reinstall. Current Homebrew does not accept `--HEAD` with `brew reinstall`; reinstall the head-only formula normally and preserve its options. Do not remove packages from `PIP_NO_BINARY` without retesting that linkage fix.
- When `scaryrawr/omlx` changes its exact `mlx` pin, inspect optional engine metadata—especially the pinned `scaryrawr/mflux` image dependency—for incompatible bounds. Before relaxing a bound, require a clean resolver/`pip check` result and a real image-generation smoke test on the new MLX version.
- For Python formulae that intentionally let pip resolve upstream dependencies instead of vendoring `resource` blocks, use an explicit virtualenv plus `pip install` rather than `Language::Python::Virtualenv#pip_install_and_link`, which installs with `--no-deps`. If native Python extensions fail Homebrew linkage rewriting, force those packages to build from source with headerpad linker flags and add the needed build dependencies.

## Update Script Pattern

Update scripts use `gh` CLI for GitHub API access and `curl` for downloads:
```python
subprocess.run(["gh", "api", f"repos/{repo}/releases/latest", "--jq", ".tag_name"], ...)
subprocess.run(["curl", "-sL", url, "-o", temp_file.name], check=True)
```

Always use `check=True` with subprocess and handle `CalledProcessError`.

---
> Source: [scaryrawr/homebrew-formulae](https://github.com/scaryrawr/homebrew-formulae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-29 -->
