# Python Version Management with uv

Comprehensive guide to installing, managing, and working with multiple Python versions using uv.

## Table of Contents

- [Overview](#overview)
- [Installing Python](#installing-python)
- [Managing Versions](#managing-versions)
- [Python Discovery](#python-discovery)
- [Configuration](#configuration)
- [Advanced Usage](#advanced-usage)
- [Troubleshooting](#troubleshooting)

## Overview

uv can install and manage Python versions automatically, replacing tools like pyenv, asdf, or manual Python installations.

### Key Features

- **Automatic downloads**: Python installed on-demand when needed
- **Multiple versions**: Install and switch between any Python version
- **Multiple implementations**: CPython, PyPy, GraalPy
- **Fast**: Downloads pre-built binaries from python-build-standalone
- **Cross-platform**: Works on macOS, Linux, and Windows
- **No compilation**: Pre-built binaries, no need to compile from source

### Python Distributions

uv uses **python-build-standalone** distributions from Astral:

- Standalone, relocatable Python distributions
- Pre-built for all major platforms
- Optimized for size and performance
- Regular updates with latest Python releases
- Includes recent stable and pre-release versions

**Supported implementations:**
- **CPython**: Official Python implementation (3.8-3.14+)
- **PyPy**: Fast JIT-compiled Python (3.8-3.10)
- **GraalPy**: Experimental (use with caution)

## Installing Python

### Basic Installation

```bash
# Install latest Python version
uv python install

# Install specific version
uv python install 3.12
uv python install 3.11.7

# Install multiple versions
uv python install 3.11 3.12 3.13

# Install PyPy
uv python install pypy@3.10
uv python install pypy@3.11

# Install GraalPy (experimental)
uv python install graalpy@23.1
```

### Version Specifiers

```bash
# Major version (latest 3.x)
uv python install 3

# Major.minor (latest patch)
uv python install 3.12

# Exact version
uv python install 3.12.1

# Implementation-specific
uv python install pypy@3.10
uv python install cpython@3.12.1
```

### Installation Options

```bash
# Install with default executables (python, python3)
uv python install --default

# Reinstall (useful for updates to distributions)
uv python install --reinstall 3.12

# Install multiple at once
uv python install 3.11 3.12 3.13 pypy@3.10
```

**Note**: By default, uv only installs versioned executables (e.g., `python3.12`). Use `--default` to also create `python` and `python3` symlinks.

### Where Python is Installed

**Default location**:
- **Unix**: `~/.local/share/uv/python`
- **Windows**: `%LOCALAPPDATA%\uv\python`
- **macOS**: `~/Library/Application Support/uv/python`

**Installation structure:**
```
~/.local/share/uv/python/
├── cpython-3.12.1-linux-x86_64-gnu/
├── cpython-3.11.7-linux-x86_64-gnu/
└── pypy3.10-7.3.13-linux-x86_64/
```

**Executables location**:
Symlinked to `~/.local/bin/` (added to PATH during installation)

## Managing Versions

### Listing Python Versions

```bash
# List all available Python versions
uv python list

# List only installed versions
uv python list --only-installed

# List with details
uv python list --all-versions

# List specific implementation
uv python list --implementation cpython
uv python list --implementation pypy
```

**Example output:**
```
cpython-3.12.1-linux-x86_64-gnu      <installed>
cpython-3.11.7-linux-x86_64-gnu      <installed>
cpython-3.13.0-linux-x86_64-gnu      <download available>
pypy3.10-7.3.13-linux-x86_64         <installed>
```

### Upgrading Python

```bash
# Upgrade to latest patch version
uv python upgrade 3.12

# Upgrade all installed versions
uv python upgrade

# Check for upgrades without installing
uv python upgrade --dry-run
```

**Example:**
```bash
# Current: Python 3.12.0
uv python upgrade 3.12
# After: Python 3.12.1 (latest patch)
```

### Pinning Python Version

Pin Python version for project:

```bash
# Pin to specific version
uv python pin 3.12
uv python pin 3.12.1

# Pin to latest compatible
uv python pin 3.12.x

# Pin to PyPy
uv python pin pypy@3.10
```

**Creates `.python-version` file:**
```
3.12
```

**Pin behavior:**
- Project commands use this version automatically
- `uv run`, `uv sync`, `uv venv` respect the pin
- Can be overridden with `--python` flag

### Removing Python Versions

```bash
# Uninstall specific version
uv python uninstall 3.11

# Uninstall multiple
uv python uninstall 3.11 3.12

# Uninstall PyPy
uv python uninstall pypy@3.10
```

**Note**: Only removes uv-managed Python installations.

## Python Discovery

uv searches for Python in this order:

1. **`--python` flag**: Explicitly specified version
2. **Virtual environment**: Active or project `.venv`
3. **`.python-version`**: Project-pinned version
4. **`UV_PYTHON`**: Environment variable
5. **Managed Python**: uv-installed versions
6. **System Python**: PATH search

### Discovery Rules

**Exact version requested:**
```bash
uv run --python 3.12.1 script.py
# Must find Python 3.12.1 exactly
```

**Version range:**
```bash
uv run --python 3.12 script.py
# Finds latest 3.12.x available
```

**Implementation-specific:**
```bash
uv run --python pypy@3.10 script.py
# Finds PyPy 3.10.x
```

### Python Preference

Control which Python sources to use:

```bash
# Only use managed Python
uv run --python-preference only-managed script.py

# Prefer managed, fallback to system
uv run --python-preference managed script.py

# Prefer system, fallback to managed
uv run --python-preference system script.py

# Only use system Python
uv run --python-preference only-system script.py
```

**Configure in `pyproject.toml`:**
```toml
[tool.uv]
python-preference = "only-managed"  # or "managed", "system", "only-system"
```

### Automatic Downloads

**By default**, uv automatically downloads Python when needed:

```bash
# No Python 3.13 installed
uv run --python 3.13 script.py
# Downloads Python 3.13 automatically
```

**Disable automatic downloads:**

```bash
# Environment variable
export UV_NO_PYTHON_DOWNLOADS=1
uv run --python 3.13 script.py  # Error if not installed

# Flag
uv run --no-python-downloads --python 3.13 script.py
```

**Configure in `pyproject.toml`:**
```toml
[tool.uv]
python-downloads = false
```

## Configuration

### Python Cache Directory

Python installations are cached:

```bash
# View Python cache directory
uv python dir

# Clear Python cache
uv cache clean --python
```

**Set custom cache location:**
```bash
export UV_PYTHON_CACHE_DIR=/custom/path
```

### Installation Variants

Python distributions come in platform-specific variants:

**Linux variants:**
- `gnu`: Standard GNU/Linux (default)
- `musl`: Alpine Linux/musl-based systems

**Architecture:**
- `x86_64`: 64-bit Intel/AMD
- `aarch64`: 64-bit ARM
- `armv7`: 32-bit ARM
- `i686`: 32-bit Intel

**Examples:**
- `cpython-3.12.1-linux-x86_64-gnu`
- `cpython-3.12.1-linux-aarch64-musl`
- `cpython-3.12.1-macos-aarch64-none`
- `cpython-3.12.1-windows-x86_64-shared`

### Executable Paths

After installation, Python executables are available:

```bash
# Versioned executables (always created)
python3.12
python3.11
pypy3.10

# Unversioned executables (with --default)
python
python3

# Find executable location
which python3.12
```

Add to shell profile for persistence:

```bash
# ~/.bashrc or ~/.zshrc
export PATH="$HOME/.local/bin:$PATH"
```

## Advanced Usage

### Using Specific Python in Commands

```bash
# Virtual environment
uv venv --python 3.12
uv venv --python pypy@3.10

# Run command
uv run --python 3.11 script.py

# Install dependencies
uv sync --python 3.12

# Tool installation
uv tool install --python 3.11 black
uvx --python 3.12 ruff check
```

### Multiple Python Testing

Test code across multiple Python versions:

```bash
# Test with each version
for version in 3.11 3.12 3.13; do
    echo "Testing with Python $version"
    uv run --python $version pytest
done

# Or use tox/nox for structured testing
```

**Example workflow:**
```bash
# Install test versions
uv python install 3.11 3.12 3.13

# Test each
uv run --python 3.11 pytest
uv run --python 3.12 pytest
uv run --python 3.13 pytest
```

### Python Version in CI/CD

**GitHub Actions:**
```yaml
strategy:
  matrix:
    python-version: ["3.11", "3.12", "3.13"]

steps:
  - uses: astral-sh/setup-uv@v7
    with:
      python-version: ${{ matrix.python-version }}
  
  - run: uv sync
  - run: uv run pytest
```

**GitLab CI:**
```yaml
variables:
  UV_PYTHON: "3.12"

before_script:
  - curl -LsSf https://astral.sh/uv/install.sh | sh
  - uv python install $UV_PYTHON
```

### Python in Docker

```dockerfile
# Install specific Python version
FROM ubuntu:22.04
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

RUN uv python install 3.12
ENV PATH="/root/.local/bin:$PATH"
```

Use pre-built images:
```dockerfile
FROM ghcr.io/astral-sh/uv:python3.12-slim
# Python 3.12 already installed
```

### Pre-release Versions

Install pre-release Python versions:

```bash
# Install latest 3.14 alpha/beta
uv python install 3.14

# Install specific pre-release
uv python install 3.14.0a1
```

**Use cases:**
- Testing new Python features
- Verifying compatibility early
- Contributing to Python development

### Alternative Implementations

**PyPy** (fast Python with JIT):
```bash
# Install PyPy
uv python install pypy@3.10

# Use in project
uv python pin pypy@3.10
uv venv

# Benchmark performance
uv run --python pypy@3.10 python benchmark.py
```

**GraalPy** (experimental):
```bash
# Install GraalPy
uv python install graalpy@23.1

# Use with caution
uv run --python graalpy@23.1 script.py
```

## Troubleshooting

### Python Not Found

**Error**: `No Python found for version X.Y`

**Solutions:**
1. Install the version: `uv python install X.Y`
2. Check available versions: `uv python list`
3. Use different version specifier (e.g., `3.12` instead of `3.12.1`)

### Installation Fails

**Error**: Python installation fails

**Solutions:**
1. Check internet connectivity
2. Retry installation: `uv python install --reinstall X.Y`
3. Check disk space
4. Verify platform support: `uv python list --all-platforms`

### Executable Not in PATH

**Error**: `python3.12: command not found`

**Solutions:**
1. Ensure `~/.local/bin` in PATH:
   ```bash
   export PATH="$HOME/.local/bin:$PATH"
   ```

2. Check installation: `uv python list --only-installed`

3. Install with default executables: `uv python install --default`

### Version Conflicts

**Error**: Wrong Python version used

**Solutions:**
1. Check active version: `which python`, `python --version`
2. Pin version explicitly: `uv python pin X.Y`
3. Use `--python` flag: `uv run --python X.Y`
4. Deactivate conflicting virtualenv

### Platform Issues

**Alpine Linux:**
```bash
# Use musl variant
uv python install 3.12-musl
```

**ARM systems:**
```bash
# Should work automatically
uv python install 3.12
# Downloads aarch64 or armv7 variant
```

**Windows:**
```powershell
# Standard installation
uv python install 3.12
# Installs shared variant with DLLs
```

### Managed vs System Python

**Problem**: Want to use system Python only

**Solution**:
```bash
# Disable managed Python
export UV_NO_MANAGED_PYTHON=1

# Or use preference
uv run --python-preference only-system script.py
```

**Problem**: Want to ignore system Python

**Solution**:
```bash
# Only use managed Python
export UV_PYTHON_PREFERENCE=only-managed

# Or in config
[tool.uv]
python-preference = "only-managed"
```

### Distribution Updates

**Problem**: Installed Python has bugs

**Solution**:
```bash
# Reinstall to get updated distribution
uv python install --reinstall 3.12

# Check for upgrades
uv python upgrade --dry-run
```

### Multiple Installations

**Problem**: Multiple Python installations conflicting

**Check what's installed:**
```bash
# System installations
which -a python
which -a python3

# uv installations
uv python list --only-installed

# Paths being searched
echo $PATH
```

**Clean approach:**
```bash
# Remove old installations
uv python uninstall 3.11

# Reinstall fresh
uv python install 3.12

# Set preference
uv python pin 3.12
```

## Best Practices

1. **Pin Python versions** in projects with `.python-version`
2. **Use managed Python** for consistency across environments
3. **Install multiple versions** for cross-version testing
4. **Regular upgrades**: Run `uv python upgrade` periodically
5. **Specify in CI/CD**: Explicitly set Python version in pipelines
6. **Document requirements**: Note Python version in README
7. **Test pre-releases**: Try alpha/beta versions to catch issues early
8. **Clean unused versions**: Remove old Python installations
9. **Use implementation-specific** where needed (PyPy for performance)
10. **Disable auto-download** in production for explicit control

## Common Patterns

### Development Setup

```bash
# Install development Python versions
uv python install 3.11 3.12 3.13

# Pin for project
cd my-project
uv python pin 3.12

# Verify
python --version  # Should be 3.12
```

### Cross-Version Testing

```bash
#!/bin/bash
# test_all_versions.sh

VERSIONS="3.11 3.12 3.13"

for VERSION in $VERSIONS; do
    echo "Testing with Python $VERSION"
    uv python install $VERSION
    uv run --python $VERSION pytest || exit 1
done

echo "All versions passed!"
```

### CI/CD Matrix

```yaml
# .github/workflows/test.yml
jobs:
  test:
    strategy:
      matrix:
        python: ["3.11", "3.12", "3.13"]
        os: [ubuntu-latest, macos-latest, windows-latest]
    
    steps:
      - uses: astral-sh/setup-uv@v7
        with:
          python-version: ${{ matrix.python }}
```

### Docker Multi-Stage

```dockerfile
# Build with specific Python
FROM ghcr.io/astral-sh/uv:python3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app
COPY . .
RUN uv sync --frozen

# Runtime with same Python
FROM python:3.12-slim
COPY --from=builder /app/.venv /app/.venv
ENV PATH="/app/.venv/bin:$PATH"
```

### Version Migration

```bash
# Migrating from 3.11 to 3.12

# Install new version
uv python install 3.12

# Update project
cd my-project
uv python pin 3.12

# Recreate environment
rm -rf .venv
uv sync

# Test
uv run pytest

# Update CI/CD configs to use 3.12
```

### PyPy Performance Testing

```bash
# Install both CPython and PyPy
uv python install 3.12
uv python install pypy@3.10

# Benchmark script
echo "Testing CPython 3.12"
time uv run --python 3.12 python benchmark.py

echo "Testing PyPy 3.10"  
time uv run --python pypy@3.10 python benchmark.py
```

## Reference

### Command Summary

```bash
# Installation
uv python install [VERSION...]
uv python install --default [VERSION]
uv python install --reinstall [VERSION]

# Management
uv python list [--only-installed]
uv python pin <VERSION>
uv python upgrade [VERSION]
uv python uninstall <VERSION>

# Information
uv python dir
uv python find <VERSION>

# Usage
uv run --python <VERSION> COMMAND
uv venv --python <VERSION>
uv sync --python <VERSION>
```

### Environment Variables

```bash
UV_PYTHON                    # Python version to use
UV_PYTHON_PREFERENCE         # Python source preference
UV_NO_PYTHON_DOWNLOADS       # Disable automatic downloads
UV_PYTHON_CACHE_DIR          # Python installation directory
UV_NO_MANAGED_PYTHON         # Disable managed Python
```

### Configuration Options

```toml
[tool.uv]
python-preference = "managed"  # Python source preference
python-downloads = true        # Enable/disable auto-downloads
requires-python = ">=3.12"     # Minimum Python version
```

### Version Specifier Formats

```
3              # Latest 3.x
3.12           # Latest 3.12.x
3.12.1         # Exact version
3.12+          # 3.12 or higher
<3.13          # Below 3.13
pypy@3.10      # PyPy 3.10
cpython@3.12   # CPython 3.12
```
