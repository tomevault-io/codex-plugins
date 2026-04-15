# GitHub Profile Repository
This is a personal GitHub profile repository for Behrooz Azarkhalili showcasing AI/ML expertise, technical stack, and professional experience. It serves as a dynamic profile page with automated WakaTime coding statistics integration.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively
This repository does NOT contain traditional software applications, build processes, or test suites. Instead, focus on profile content management and GitHub Actions maintenance:

- Navigate to repository root: `/home/runner/work/behroozazarkhalili/behroozazarkhalili`
- Primary content file: `README.md` (184 lines, showcases technical expertise)
- GitHub Actions: `.github/workflows/Waka-Readme.yml` (WakaTime integration)
- Documentation: `.github/ACTIONS_STATUS.md` (Actions troubleshooting guide)
- Profile image: `Images/Banner.jpeg` (1080x834 JPEG)
- Dependency management: `renovate.json` (Renovate configuration)

## Repository Structure Validation
Always validate repository structure before making changes:
- `ls -la` -- View all files including hidden ones (takes <1 second)
- `file Images/Banner.jpeg` -- Verify profile image integrity (takes <1 second) 
- `wc -l README.md` -- Check README.md length (should be ~184 lines, takes <1 second)
- `yamllint .github/workflows/Waka-Readme.yml` -- Validate workflow syntax if yamllint available (takes <2 seconds)

## GitHub Actions Workflow Management

### WakaTime Workflow
- **File**: `.github/workflows/Waka-Readme.yml`
- **Purpose**: Updates README.md with coding statistics from WakaTime
- **Schedule**: Daily at 7 AM UTC (midnight PDT)
- **Timeout**: 15 minutes with graceful error handling
- **Dependencies**: Requires WAKATIME_API_KEY and GH_TOKEN secrets

### Manual Workflow Trigger
You cannot directly trigger workflows from this environment, but document the process:
1. Go to GitHub Actions tab in web interface
2. Select "Waka Readme" workflow  
3. Click "Run workflow" button
4. Select main branch and execute

### Workflow Troubleshooting
- **Common Issue**: GitHub GraphQL API timeouts (504 errors)
- **Resolution**: Workflow includes `continue-on-error: true` and retry logic
- **Monitoring**: Check `.github/ACTIONS_STATUS.md` for documented issues
- **Expected Behavior**: Occasional failures are normal due to API timeouts

## Content Management

### README.md Editing
- **Structure**: Profile introduction → Core Expertise → Technical Stack → Contact/Publications
- **Key Sections**: 
  - Lines 1-6: Personal introduction and tagline
  - Lines 7-20: Core expertise in LLM fine-tuning, multimodal ML/DL
  - Lines 21-68: Technical stack with technology badges
  - Lines 69+: Additional sections (publications, tools, statistics)
- **WakaTime Integration**: Automated sections marked with `<!--START_SECTION:waka-->` and `<!--END_SECTION:waka-->`
- **Badge Format**: Uses img.shields.io badges with height="20" consistently

### Profile Content Validation
Always validate content changes:
- `grep -n "START_SECTION:waka" README.md` -- Locate WakaTime sections (takes <1 second)
- `grep -n "img.shields.io" README.md | wc -l` -- Count technology badges (takes <1 second)
- `head -10 README.md` -- Review profile introduction (takes <1 second)

## Common Tasks

### Adding New Technology Badges
1. Follow existing pattern: `<img src="https://img.shields.io/badge/[Technology]-282C34?logo=[logo]" alt="[Technology]" height="20" />`
2. Maintain 282C34 background color for consistency
3. Set height="20" for all badges
4. Place in appropriate technical stack category

### Updating Profile Content
1. Edit README.md sections outside WakaTime automation zones
2. Maintain markdown table structure for technical stack
3. Keep professional tone consistent with existing content
4. Validate changes do not break WakaTime insertion points

### Managing GitHub Actions Issues
1. Review `.github/ACTIONS_STATUS.md` for known issues
2. Check workflow logs in GitHub Actions tab
3. Update timeout-minutes if needed (current: 15 minutes)
4. Document new issues in ACTIONS_STATUS.md following existing format

## Validation Scenarios
After making changes, always perform these validation steps:

### Content Integrity Check
1. `wc -l README.md` -- Verify line count hasn't drastically changed
2. `grep "START_SECTION:waka" README.md` -- Ensure WakaTime markers intact
3. `grep "img.shields.io" README.md | head -5` -- Verify badge format consistency

### Workflow Validation  
1. Check `.github/workflows/Waka-Readme.yml` syntax
2. Verify timeout-minutes value is appropriate (15+ minutes)
3. Confirm continue-on-error: true is present for resilience

### File System Validation
1. `ls -la .github/` -- Confirm GitHub directory structure
2. `file Images/Banner.jpeg` -- Verify profile image integrity
3. `cat renovate.json` -- Check dependency management configuration

## Repository Characteristics
- **No build processes** - This is a content repository, not a software project
- **No test suites** - Content validation is manual/visual
- **No application runtime** - Profile serves as static showcase
- **No package dependencies** - Only GitHub Actions and external services
- **Primary validation** - Content accuracy, workflow functionality, profile presentation

## Critical Warnings
- **NEVER** modify content between `<!--START_SECTION:waka-->` and `<!--END_SECTION:waka-->` markers
- **NEVER** remove or modify WakaTime workflow secrets configuration
- **NEVER** delete ACTIONS_STATUS.md without preserving troubleshooting information
- **ALWAYS** preserve existing badge formatting and color scheme (282C34)
- **ALWAYS** maintain professional tone in all profile content

## Troubleshooting Common Issues

### WakaTime Workflow Failures
1. Check if isolated incident (single failure acceptable)
2. Look for 504 timeout errors in GitHub Actions logs
3. Verify secrets (WAKATIME_API_KEY, GH_TOKEN) are configured
4. Wait for next scheduled run (daily at 7 AM UTC)
5. Consider manual workflow dispatch if >3 consecutive failures

### Content Display Issues
1. Verify markdown table syntax in technical stack section
2. Check badge URLs are accessible
3. Ensure no broken image links in Images/ directory
4. Validate HTML tags are properly closed in badge sections

### Repository Structure Problems
1. Confirm `.github/` directory permissions and structure
2. Verify `Images/` directory contains expected files
3. Check file encoding of README.md (should be UTF-8)
4. Ensure no unexpected binary files committed

## Quick Reference Commands
Essential commands for immediate use:

```bash
# Repository navigation
cd /home/runner/work/behroozazarkhalili/behroozazarkhalili

# Content validation
wc -l README.md                    # Check README length
grep -n "waka" README.md           # Find WakaTime sections  
head -20 README.md                 # Review introduction

# Structure verification
ls -la .github/                    # Check GitHub directory
file Images/Banner.jpeg            # Verify profile image
cat renovate.json                  # Review configuration

# Git operations
git status                         # Check repository state
git log --oneline -5               # Recent commit history
git diff                           # View unstaged changes
```

## Expected Timing
All operations in this repository are fast due to content-only nature:
- File validation: <1 second each
- Content editing: <5 seconds for typical changes
- Git operations: <2 seconds each
- Structure verification: <1 second each

No long-running builds, tests, or deployments exist in this repository.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/behroozazarkhalili) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
