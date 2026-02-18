# pip Interface Compatibility

Comprehensive guide to using uv's drop-in replacement for pip, pip-tools, and virtualenv commands.

## Table of Contents

- [Overview](#overview)
- [Virtual Environments](#virtual-environments)
- [Package Installation](#package-installation)
- [Dependency Compilation](#dependency-compilation)
- [Package Inspection](#package-inspection)
- [Migration Guide](#migration-guide)
- [Compatibility Notes](#compatibility-notes)

## Overview

uv provides a pip-compatible interface for teams not ready to adopt projects but want speed improvements.

### Key Commands

```bash
uv venv        # Create virtual environment (virtualenv replacement)
uv pip install # Install packages (pip install replacement)
uv pip compile # Compile requirements (pip-compile replacement)
uv pip sync    # Install exact requirements (pip-sync replacement)
uv pip list    # List installed packages
uv pip show    # Show package information
uv pip freeze  # Export installed packages
uv pip uninstall # Remove packages
uv pip check   # Verify dependencies
```

### When to Use pip Interface

**Use pip interface when:**
- Migrating gradually from pip/pip-tools
- Have existing requirements.txt workflows
- Need pip-compatible CLI
- Working with legacy projects
- Team not ready for full uv adoption

**Use project interface when:**
- Starting new Python projects
- Want integrated dependency management
- Need lockfiles and workspaces
- Prefer modern workflows

## Virtual Environments

### Creating Environments

```bash
# Create virtual environment
uv venv

# Create with specific name
uv venv my-env

# Create with specific Python
uv venv --python 3.12
uv venv --python pypy@3.10

# Create at specific path
uv venv /path/to/venv

# Create with system packages
uv venv --system-site-packages
```

**Default location**: `.venv/` in current directory

### Environment Activation

```bash
# Unix/macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate

# Or use uv pip with --system (see below)
```

### System Python Flag

Install directly to active environment without `--system`:

```bash
# Set globally
export UV_SYSTEM_PYTHON=1

# Or use flag
uv pip install --system requests
```

**Use in Docker** (where isolation already provided):
```dockerfile
ENV UV_SYSTEM_PYTHON=1
RUN uv pip install -r requirements.txt
```

## Package Installation

### Basic Installation

```bash
# Install package
uv pip install requests

# Install multiple packages
uv pip install requests flask numpy

# Install with version constraints
uv pip install 'requests>=2.28,<3'
uv pip install 'flask==3.0.0'

# Install with extras
uv pip install 'fastapi[all]'
```

### Installing from Files

```bash
# Install from requirements.txt
uv pip install -r requirements.txt

# With constraints
uv pip install -r requirements.txt -c constraints.txt

# From pyproject.toml
uv pip install -r pyproject.toml

# Install package editably
uv pip install -e .
uv pip install -e path/to/package
```

### Installing from URLs

```bash
# From git
uv pip install git+https://github.com/user/repo
uv pip install git+https://github.com/user/repo@main
uv pip install git+https://github.com/user/repo@v1.0.0

# From tarball
uv pip install https://example.com/package-1.0.tar.gz

# From wheel
uv pip install https://example.com/package-1.0-py3-none-any.whl

# Local path
uv pip install /path/to/package
uv pip install file:///absolute/path
```

### Uninstalling Packages

```bash
# Uninstall package
uv pip uninstall requests

# Uninstall multiple
uv pip uninstall requests flask numpy

# Uninstall from requirements file
uv pip uninstall -r requirements.txt

# Uninstall all (except pip, setuptools, wheel)
uv pip freeze | uv pip uninstall -r /dev/stdin
```

## Dependency Compilation

### Basic Compilation

```bash
# Compile requirements.in to requirements.txt
uv pip compile requirements.in -o requirements.txt

# Compile pyproject.toml
uv pip compile pyproject.toml -o requirements.txt

# Print to stdout
uv pip compile requirements.in
```

### Compilation Options

```bash
# Include hashes
uv pip compile requirements.in --generate-hashes

# Universal resolution (platform-independent)
uv pip compile requirements.in --universal

# Specific Python version
uv pip compile requirements.in --python-version 3.12

# Specific platform
uv pip compile requirements.in --python-platform linux

# With extras
uv pip compile pyproject.toml --extra dev --extra test

# All extras
uv pip compile pyproject.toml --all-extras

# Upgrade packages
uv pip compile requirements.in --upgrade
uv pip compile requirements.in --upgrade-package requests
```

### Multi-File Compilation

```bash
# Multiple input files
uv pip compile requirements.in dev-requirements.in

# Output to separate files
uv pip compile requirements.in -o requirements.txt
uv pip compile dev-requirements.in -o dev-requirements.txt

# With constraints
uv pip compile requirements.in -c constraints.txt
```

### Lockfile Generation

```bash
# Generate lockfile (like pip-tools)
uv pip compile requirements.in -o requirements.txt --generate-hashes

# Update specific package
uv pip compile requirements.in -P requests --upgrade-package requests

# Upgrade all
uv pip compile requirements.in --upgrade -o requirements.txt
```

## Syncing Environments

### Basic Sync

```bash
# Sync to exact requirements
uv pip sync requirements.txt

# Sync multiple files
uv pip sync requirements.txt dev-requirements.txt

# Dry run
uv pip sync requirements.txt --dry-run
```

**Key difference from `pip install -r`:**
- `pip install -r`: Installs packages (additive)
- `uv pip sync`: Installs exactly listed packages (removes extras)

### Sync Workflow

```bash
# 1. Compile dependencies
uv pip compile requirements.in -o requirements.txt

# 2. Sync environment to match
uv pip sync requirements.txt

# Result: Environment matches requirements.txt exactly
```

### Sync with Extras

```bash
# Sync including extras
uv pip sync requirements.txt --extra dev

# Multiple extras
uv pip sync requirements.txt --extra dev --extra test

# All extras
uv pip sync requirements.txt --all-extras
```

## Package Inspection

### Listing Packages

```bash
# List installed packages
uv pip list

# List with versions
uv pip list --format columns

# List as JSON
uv pip list --format json

# Only local packages (exclude standard library)
uv pip list --local

# Include editable packages
uv pip list --editable
```

### Package Information

```bash
# Show package details
uv pip show requests

# Show multiple packages
uv pip show requests flask

# Verbose output
uv pip show -v requests
```

### Freezing Requirements

```bash
# Export installed packages
uv pip freeze

# Export to file
uv pip freeze > requirements.txt

# Exclude certain packages
uv pip freeze --exclude-editable > requirements.txt
```

### Dependency Check

```bash
# Verify dependency consistency
uv pip check

# Reports conflicts and missing dependencies
```

### Tree View

```bash
# Not built into uv, use pipdeptree
uvx pipdeptree
uvx pipdeptree --packages requests

# Or use uv tree in project mode
cd my-project
uv tree
```

## Migration Guide

### From pip

**Before (pip):**
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**After (uv):**
```bash
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt

# Or with system flag
uv venv
export UV_SYSTEM_PYTHON=1
uv pip install -r requirements.txt
```

### From pip-tools

**Before (pip-tools):**
```bash
pip-compile requirements.in
pip-sync requirements.txt
```

**After (uv):**
```bash
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
```

### From virtualenv

**Before (virtualenv):**
```bash
virtualenv .venv
source .venv/bin/activate
```

**After (uv):**
```bash
uv venv
source .venv/bin/activate
```

### Converting to uv Projects

**Step 1**: Initialize project
```bash
uv init --no-readme  # In existing directory
```

**Step 2**: Import dependencies
```bash
# From requirements.txt
uv add -r requirements.txt

# Or manually add to pyproject.toml then sync
uv sync
```

**Step 3**: Generate lockfile
```bash
uv lock
```

**Step 4**: Use project commands
```bash
uv run python script.py
uv sync
```

## Compatibility Notes

### Differences from pip

**Behavior differences:**

1. **Installation speed**: 10-100x faster than pip
2. **Dependency resolution**: More strict by default
3. **Cache location**: Different from pip (uv-specific)
4. **Virtual environment**: Created differently
5. **Platform support**: May have different platform tags

**Command differences:**

```bash
# pip
pip install --upgrade package

# uv (must recompile requirements)
uv pip compile requirements.in --upgrade-package package
uv pip sync requirements.txt
```

### pip-tools Compatibility

**Compatible commands:**
- `pip-compile` → `uv pip compile`
- `pip-sync` → `uv pip sync`

**Differences:**
- uv uses different resolver (PubGrub)
- Hash generation may differ
- Annotation comments different
- Faster resolution

### Feature Parity

**Supported pip features:**
✅ Basic install/uninstall
✅ Requirements files
✅ Constraints files
✅ Editable installs
✅ Extras
✅ Git/URL installs
✅ Local paths
✅ Python version constraints
✅ Platform markers
✅ Hashes

**Unsupported/different:**
❌ `pip download` (use `uv pip compile` instead)
❌ `pip wheel` (use `uv build`)
❌ `pip search` (PyPI disabled this)
⚠️ Some edge cases in dependency resolution

### Requirements File Format

uv supports standard requirements.txt format:

```txt
# Standard format
requests==2.31.0
flask>=3.0,<4.0

# With hashes
requests==2.31.0 \
    --hash=sha256:...

# With markers
pywin32==306; sys_platform == "win32"

# Git URLs
git+https://github.com/user/repo@v1.0.0

# Local paths
./packages/my-package
-e ./src/my-lib

# Constraints
-c constraints.txt

# Includes
-r base.txt
```

## Advanced Features

### Custom Indexes

```bash
# Use alternative index
uv pip install --index-url https://custom.pypi.org/simple package

# Extra indexes
uv pip install --extra-index-url https://custom.pypi.org/simple package

# In requirements.txt
--index-url https://custom.pypi.org/simple
package-name
```

### Resolution Strategies

```bash
# Default: highest
uv pip compile requirements.in

# Lowest compatible
uv pip compile requirements.in --resolution lowest

# Lowest direct dependencies
uv pip compile requirements.in --resolution lowest-direct
```

### Platform-Specific Resolution

```bash
# Compile for specific platform
uv pip compile requirements.in --python-platform linux

# Universal (all platforms)
uv pip compile requirements.in --universal

# Specific Python version
uv pip compile requirements.in --python-version 3.12
```

### Build Configuration

```bash
# Install from source (no binary)
uv pip install --no-binary :all: package

# Install only binary (no build from source)
uv pip install --only-binary :all: package

# Specific packages
uv pip install --no-binary pyyaml --only-binary :all: requests
```

### Authentication

```bash
# Using environment variables
export UV_INDEX_URL=https://user:password@custom.pypi.org/simple
uv pip install package

# Using netrc
# ~/.netrc
machine custom.pypi.org
login user
password secret

# Using keyring
uv pip install --keyring-provider auto package
```

## Common Workflows

### Development Workflow

```bash
# Create environment
uv venv

# Compile development requirements
uv pip compile requirements.in dev-requirements.in

# Sync environment
uv pip sync requirements.txt dev-requirements.txt

# Install project editably
uv pip install -e .
```

### CI/CD Workflow

```bash
# Compile in CI (fail on changes)
uv pip compile requirements.in --dry-run
if [ $? -ne 0 ]; then
    echo "Requirements out of date!"
    exit 1
fi

# Install exact requirements
uv pip sync requirements.txt --no-deps
```

### Production Deployment

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app
COPY requirements.txt .

# Install exact versions
RUN uv pip sync requirements.txt --system

COPY . .
CMD ["python", "app.py"]
```

### Hybrid Workflow

Support both uv and pip:

```bash
# Generate both formats
uv lock
uv export -o requirements.txt

# Users can use either
# With uv:
uv sync

# With pip:
pip install -r requirements.txt
```

## Best Practices

1. **Use `uv pip sync`** for exact environments, not `uv pip install -r`
2. **Commit lockfiles** (requirements.txt with hashes)
3. **Use `--system`** in Docker containers
4. **Compile regularly** to catch dependency updates
5. **Use `--universal`** for cross-platform projects
6. **Separate dev requirements** for faster prod installs
7. **Pin Python version** with `--python-version`
8. **Use hashes** for security: `--generate-hashes`
9. **Test with `--resolution lowest`** for broad compatibility
10. **Consider migrating** to uv projects for new work

## Troubleshooting

### Installation Fails

**Error**: `No solution found`
**Solution**: Check for conflicts with `uv pip check`, use `--resolution lowest`

**Error**: `Failed to build package`
**Solution**: Install system dependencies, use `--no-binary` or `--only-binary`

### Environment Issues

**Error**: `Not in virtual environment`
**Solution**: Use `--system` flag or activate venv

**Error**: `Permission denied`
**Solution**: Don't use `sudo`, check venv permissions

### Migration Issues

**Error**: `Can't find requirements.txt`
**Solution**: Specify path: `uv pip install -r path/to/requirements.txt`

**Error**: `Dependency version mismatch`
**Solution**: Recompile: `uv pip compile --upgrade requirements.in`

### "Syncing Issues

**Error**: `Packages remain after sync`
**Solution**: `uv pip sync` removes unlisted packages - check requirements.txt

**Error**: `Missing packages after sync`
**Solution**: Check for missing dependencies in requirements.txt

## Command Reference

```bash
# Virtual environments
uv venv [PATH]
uv venv --python X.Y

# Installation
uv pip install PACKAGE
uv pip install -r requirements.txt
uv pip install -e .
uv pip uninstall PACKAGE

# Compilation
uv pip compile requirements.in
uv pip compile -o requirements.txt
uv pip compile --upgrade
uv pip compile --upgrade-package PKG

# Syncing
uv pip sync requirements.txt
uv pip sync --dry-run

# Inspection
uv pip list
uv pip show PACKAGE
uv pip freeze
uv pip check

# Options
--system              # Use system Python
--python VERSION      # Specify Python version
--index-url URL       # Custom package index
--extra-index-url URL # Additional index
--no-binary PACKAGE   # Don't use wheels
--only-binary PACKAGE # Only use wheels
--universal           # Platform-independent
--generate-hashes     # Add hashes
```
