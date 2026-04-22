# Release Management

This document outlines the process for managing releases and keeping this fork synchronized with the upstream repository.

## Overview

This is a fork of [mletenay/home-assistant-goodwe-inverter](https://github.com/mletenay/home-assistant-goodwe-inverter) with custom enhancements. The goal is to:
- Maintain compatibility with Home Assistant
- Apply custom patches and features
- Stay synchronized with upstream improvements
- Provide clear release management

## Version Numbering

This project follows semantic versioning in the format: `MAJOR.MINOR.PATCH.BUILD`

Example: `0.9.9.31`
- `0.9.9` = upstream version
- `31` = custom build number

**When to bump the build number:**
- For any custom patches or features applied to the upstream version
- Each release (even if upstream hasn't changed)
- Do not bump when only syncing with upstream unless making modifications

## Release Process

### 1. Make Your Changes

Edit the necessary files (e.g., config validation, features, bug fixes).

```bash
# Example: editing poll frequency validation
nano custom_components/goodwe/config_flow.py
```

### 2. Update Version

Bump the build number in `custom_components/goodwe/manifest.json`:

```json
{
  "version": "0.9.9.31"  // Increment the last number
}
```

### 3. Commit

Create a clear, descriptive commit message:

```bash
git add -A
git commit -m "feat: allow decimal values for poll frequency (scan_interval)

- Changed CONF_SCAN_INTERVAL validation from int to float
- Added minimum value of 0.1 seconds to prevent unreasonably high polling rates
- Maintains backward compatibility with existing integer values
- Enables finer control over polling intervals (e.g., 1.5, 2.5 seconds)"
```

### 4. Push

```bash
git push origin master
```

### 5. Tag

Create an annotated tag for the release:

```bash
git tag -a v0.9.9.31 -m "v0.9.9.31: Allow decimal values for poll frequency"
git push origin v0.9.9.31
```

### 6. Create GitHub Release

```bash
gh release create v0.9.9.31 \
  --title "v0.9.9.31" \
  --notes "## Changes

**Allow decimal values for poll frequency (scan_interval)**

- Changed CONF_SCAN_INTERVAL validation from int to float
- Added minimum value of 0.1 seconds
- Maintains backward compatibility
- Enables finer control over polling intervals (e.g., 1.5, 2.5 seconds)

## Installation

Download from release assets or update through HACS."
```

## Keeping in Sync with Upstream

### Option 1: Manual Sync (Recommended for Occasional Updates)

Use GitHub's built-in sync button:

1. Go to your fork: `github.com/ongas/home-assistant-goodwe-inverter`
2. Click **"Sync fork"** button (top right)
3. Click **"Update branch"**
4. Pull locally: `git pull origin master`

### Option 2: GitHub Actions (Recommended for Regular Updates)

Automate syncing with upstream using a scheduled GitHub Action.

Create `.github/workflows/sync-upstream.yml`:

```yaml
name: Sync with Upstream

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday
  workflow_dispatch:  # Manual trigger option

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      
      - name: Sync upstream
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git remote add upstream https://github.com/mletenay/home-assistant-goodwe-inverter.git
          git fetch upstream master
          git merge upstream/master -m "Sync with upstream"
          git push origin master
```

Then trigger manually from Actions tab when needed.

### Option 3: Local Git (Most Control)

```bash
# Add upstream remote (one-time setup)
git remote add upstream https://github.com/mletenay/home-assistant-goodwe-inverter.git

# Fetch latest upstream
git fetch upstream

# Merge or rebase (merge is safer for public repos)
git merge upstream/master

# Resolve any conflicts if needed
# Then push to your fork
git push origin master
```

## Handling Conflicts During Sync

If upstream and your changes conflict:

1. **During merge**: Git will mark conflict areas
2. **Resolve conflicts**: Edit files, keep your changes where needed
3. **Verify no regressions**: Test the merged code
4. **Complete merge**: `git add . && git commit -m "Merge upstream changes"`
5. **Push**: `git push origin master`

**Best practices:**
- Keep your custom changes minimal and isolated (e.g., in `config_flow.py`)
- Document why custom changes exist (in comments and commit messages)
- Test after syncing before releasing

## Configuration Change Checklist

When making configuration changes (like the poll frequency update):

- [ ] Update validation schema in `config_flow.py`
- [ ] Ensure backward compatibility (old configs should still work)
- [ ] Update `manifest.json` version
- [ ] Test with various input types (int, float, string)
- [ ] Check `coordinator.py` and related files use the config correctly
- [ ] Document in release notes and commit message
- [ ] Create release with clear notes explaining the change

## HACS Integration

This fork is added to HACS as a custom repository:

**For Users:**
1. In HACS, add custom repository: `https://github.com/ongas/home-assistant-goodwe-inverter`
2. Category: Integration
3. Install and track updates through HACS

**For Maintainers:**
- Releases are automatically detected by HACS
- Version number in `manifest.json` determines available versions
- Users can install any tagged release

## Testing Before Release

Before creating a release:

```bash
# Run any available tests
python -m pytest custom_components/goodwe/

# Validate Home Assistant compatibility
# (Manual testing in HA instance)

# Check manifest.json is valid
# (Home Assistant will validate this automatically)
```

## Release Announcement

When creating releases, include:
- **What changed**: Feature, bugfix, or sync
- **Why it matters**: How it improves the integration
- **Compatibility**: Works with which HA versions
- **Installation**: How to update (HACS or manual)
- **Known issues**: Any limitations or breaking changes (if any)

Example:

```markdown
## v0.9.9.31: Decimal Poll Frequency Support

### ✨ Features
- Allow decimal values for scan_interval (e.g., 1.5, 2.5 seconds)
- Better granularity for polling intervals

### 🔄 Compatibility
- Backward compatible with existing integer values
- Works with HA 2023.12+

### 📦 Installation
- Update through HACS or download from releases

### ⚠️ Notes
- Minimum interval: 0.1 seconds
- No configuration changes required (existing configs work as-is)
```

## Troubleshooting

### Release not showing in HACS
- Verify tag matches `v*` format (e.g., `v0.9.9.31`)
- Check `manifest.json` version matches tag
- Wait a few minutes for HACS cache to update

### Merge conflicts with upstream
- Review each conflict carefully
- Keep custom changes minimal
- Test merged code thoroughly
- Document conflict resolution in commit

### Version numbering confusion
- Upstream: `0.9.9`
- Build number: auto-increment (31, 32, 33...)
- Example progression: `0.9.9.30` → `0.9.9.31` → `0.9.9.32`

## References

- [Upstream Repository](https://github.com/mletenay/home-assistant-goodwe-inverter)
- [Semantic Versioning](https://semver.org/)
- [HACS Documentation](https://hacs.xyz/)
- [Home Assistant Integration Development](https://developers.home-assistant.io/docs/creating_integration_manifest)
