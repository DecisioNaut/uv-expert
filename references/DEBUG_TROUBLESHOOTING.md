# Debug and Troubleshooting

Comprehensive guide to debugging uv issues and resolving common problems.

## Table of Contents

- [Debug and Introspection](#debug-and-introspection)
- [Troubleshooting](#troubleshooting)

## Debug and Introspection

### Verbose Output

```bash
# Basic verbosity
uv sync -v

# More verbose
uv sync -vv

# Maximum verbosity (debug)
uv sync -vvv
```

Shows:
- Dependency resolution steps
- Download progress
- Build output
- Cache hits/misses

### Dry Run

```bash
# See what would be installed without installing
uv sync --dry-run

# With output
uv add --dry-run requests
```

### Resolution Tree

```bash
# Show why package is included
uv tree

# Show reverse dependencies
uv tree --invert

# Show specific package
uv tree --package requests

# Depth limit
uv tree --depth 2
```

**Example output:**
```
myproject v0.1.0
├── requests v2.31.0
│   ├── certifi v2024.2.2
│   ├── charset-normalizer v3.3.2
│   ├── idna v3.6
│   └── urllib3 v2.2.0
└── pandas v2.2.0
    ├── numpy v1.26.4
    ├── python-dateutil v2.8.2
    │   └── six v1.16.0
    └── pytz v2024.1
```

### Dependency Graph Export

```bash
# Export as JSON
uv tree --format json > deps.json

# Export as requirements
uv export > requirements.txt

# Export without hashes
uv export --no-hashes > requirements.txt

# Export specific group
uv export --only-group dev > requirements-dev.txt
```

### Lock File Inspection

```bash
# View lock file
cat uv.lock

# Check for updates
uv lock --dry-run

# Diff lock file
uv lock --upgrade-package requests
git diff uv.lock
```

### Environment Inspection

```bash
# Show Python path
uv run python -c "import sys; print(sys.executable)"

# Show site packages
uv run python -m site

# Show installed packages
uv pip list
uv pip freeze

# Show package details
uv pip show requests
```

## Troubleshooting

### Resolution Failures

**Problem:** Cannot resolve dependencies

```bash
# Try with highest resolution
uv add --resolution highest package

# Try with lowest
uv add --resolution lowest package

# Check dependency tree
uv add package -v
```

**Solution patterns:**
1. Add constraints: `package<2.0`
2. Use overrides: `override-dependencies = ["conflicting-pkg==1.5"]`
3. Update other dependencies: `uv sync --upgrade`
4. Check package metadata issues

### Build Failures

**Problem:** Package fails to build from source

```bash
# Install with binary wheel
uv add --no-build package

# Check build requirements
uv add package -vv

# Install build dependencies
sudo apt-get install python3-dev build-essential
```

**Common causes:**
- Missing system libraries
- Compiler not available
- Outdated pip/setuptools (uv handles this)

### Network Issues

**Problem:** Timeouts or connection errors

```bash
# Increase timeout
UV_HTTP_TIMEOUT=300 uv sync

# More retries
UV_HTTP_RETRIES=10 uv sync

# Use alternative index
uv add --index-url https://mirrors.aliyun.com/pypi/simple/ package

# Check connectivity
curl -I https://pypi.org/simple/
```

### Cache Corruption

**Problem:** Strange errors, partially downloaded packages

```bash
# Clean cache
uv cache clean

# Rebuild everything
uv sync --reinstall

# Or specific package
uv cache clean package
uv add package
```

### Lock File Conflicts

**Problem:** Merge conflicts in uv.lock

```bash
# Regenerate lock file
uv lock --upgrade

# Or resolve manually, then
uv lock
```

### Permission Errors

**Problem:** Cannot write to cache or virtual environment

```bash
# Fix cache permissions
chmod -R u+w ~/.cache/uv

# Fix virtual environment
rm -rf .venv
uv sync

# Use different cache location
export UV_CACHE_DIR="$HOME/custom-cache"
```

### Python Version Conflicts

**Problem:** Wrong Python version used

```bash
# Check discovered Python
uv python list

# Pin project Python
echo "3.12" > .python-version

# Install specific Python
uv python install 3.12

# Use specific Python
uv sync --python 3.12
```

### Virtual Environment Issues

**Problem:** Virtual environment corrupted or broken

```bash
# Remove and recreate
rm -rf .venv
uv sync

# Or
uv venv --force
uv sync
```

### Git Dependency Issues

**Problem:** Cannot clone git dependencies

```bash
# Check SSH keys
ssh -T git@github.com

# Use HTTPS instead
# Change: git+ssh://git@github.com/org/repo
# To: git+https://github.com/org/repo

# Or with token
git+https://token@github.com/org/repo
```

### Import Errors After Install

**Problem:** Package installed but import fails

```bash
# Verify installation
uv pip list | grep package
uv pip show package

# Check if editable install needed
uv pip install -e .

# Verify Python path
uv run python -c "import sys; print(sys.path)"

# Reinstall
uv pip uninstall package
uv add package
```
