---
name: uv-expert
description: Master complex Python project management with uv - an extremely fast Python package and project manager. Handle projects, scripts, tools, Python versions, workspaces, Docker integration, CI/CD, pip interface, dependency resolution, and advanced workflows. Use when working with uv, migrating from pip/poetry/pipenv, managing Python projects, or when users mention uv, uvx, pyproject.toml, uv.lock, virtual environments, or complex dependency management.
license: MIT
compatibility: Requires uv (curl -LsSf https://astral.sh/uv/install.sh | sh), supports macOS, Linux, and Windows
metadata:
  author: uv-expert contributors
  version: "1.0.0"
  tags: python, uv, package-manager, dependency-management, project-management, docker, ci-cd
---

# uv Expert

Comprehensive guidance for mastering uv, the extremely fast Python package and project manager written in Rust. This skill covers complex use cases including project management, scripts, tools, workspaces, Docker integration, CI/CD pipelines, and advanced dependency resolution strategies.

## When to Use This Skill

Activate this skill when you need to:

- **Manage Python projects** with uv (create, configure, dependencies, lockfiles, workspaces)
- **Run scripts** with inline dependency metadata (PEP 723)
- **Install and use tools** (uvx, tool management)
- **Manage Python versions** (install, pin, upgrade multiple versions)
- **Migrate from pip, poetry, or pipenv** to uv
- **Set up CI/CD pipelines** with GitHub Actions, GitLab CI, or other platforms
- **Containerize applications** using uv in Docker
- **Resolve complex dependency conflicts** with advanced resolution strategies
- **Work with monorepos** using uv workspaces
- **Use the pip interface** for drop-in compatibility

## Core Concepts

### What is uv?

uv is an extremely fast Python package and project manager that:

- Replaces pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv, and more
- Is 10-100x faster than pip
- Provides comprehensive project management with universal lockfiles
- Manages Python versions automatically
- Supports workspaces for monorepo management
- Uses global caching for disk efficiency
- Written in Rust for maximum performance

### Key Components

**Projects**: Full-fledged Python projects with `pyproject.toml`, `uv.lock`, and virtual environments
**Scripts**: Single-file Python scripts with inline dependency metadata
**Tools**: Command-line tools installed and run via `uvx` (pipx replacement)
**Workspaces**: Multiple related packages in a single repository with shared lockfile
**pip Interface**: Drop-in replacement for pip, pip-tools, and virtualenv commands

## Installation and Setup

### Install uv

```bash
# macOS and Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# With pip (not recommended for production)
pip install uv

# Update uv
uv self update
```

### Verify Installation

```bash
uv --version
uv help
```

## Quick Start Guide

### 1. Creating a Project

```bash
# Create new project
uv init my-project
cd my-project

# Initialize in existing directory
mkdir existing-project && cd existing-project
uv init

# Create with specific Python version
uv init --python 3.12 my-project

# Create application vs library
uv init --app my-app      # No build-system
uv init --lib my-lib      # With build-system for packaging
```

**Project Structure Created:**
```
my-project/
├── .gitignore
├── .python-version
├── README.md
├── main.py (or src/my_project/__init__.py for libraries)
└── pyproject.toml
```

### 2. Managing Dependencies

```bash
# Add dependencies
uv add requests
uv add 'flask>=3.0'
uv add pytest --dev

# Add from git
uv add git+https://github.com/user/repo

# Add with extras
uv add 'fastapi[all]'

# Remove dependencies
uv remove requests

# Upgrade specific package
uv lock --upgrade-package requests

# Upgrade all packages
uv lock --upgrade
```

### 3. Running Code

```bash
# Run a script in project environment
uv run main.py

# Run a command
uv run python -c "import requests; print(requests.__version__)"

# Run with specific Python version
uv run --python 3.11 main.py

# Sync environment then run without uv
uv sync
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
python main.py
```

### 4. Working with Scripts

```bash
# Create script with inline metadata
uv init --script my-script.py --python 3.12

# Add dependencies to script
uv add --script my-script.py requests rich

# Run script (uv manages dependencies automatically)
uv run my-script.py

# Lock script dependencies
uv lock --script my-script.py
```

**Example Script with Inline Metadata:**

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://api.github.com/repos/astral-sh/uv")
pprint(resp.json())
```

### 5. Running and Installing Tools

```bash
# Run tool temporarily (uvx is alias for uv tool run)
uvx ruff check .
uvx --from httpie http https://example.com

# Install tool permanently
uv tool install ruff
uv tool install 'mypy[faster-cache]'

# Run with specific version
uvx ruff@0.3.0 check .

# Upgrade tools
uv tool upgrade ruff
uv tool upgrade --all

# List installed tools
uv tool list

# Uninstall tool
uv tool uninstall ruff
```

### 6. Managing Python Versions

```bash
# Install Python versions
uv python install 3.12
uv python install 3.11 3.12 3.13

# List available versions
uv python list

# Pin Python version for project
uv python pin 3.12

# Use specific Python for command
uv run --python 3.11 main.py
uv venv --python 3.12.0

# Upgrade Python patch versions
uv python upgrade 3.12
uv python upgrade  # All versions
```

## Common Workflows

### Migrating from pip

```bash
# Convert requirements.txt to uv project
uv init
uv add -r requirements.txt

# Or use pip interface
uv venv
uv pip install -r requirements.txt
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
```

### Migrating from Poetry

```bash
# Convert poetry project
uv init --no-readme  # In existing directory
# Dependencies from pyproject.toml are automatically recognized
uv sync
```

### Creating a Lockfile

```bash
# Create initial lockfile
uv lock

# Update lockfile
uv lock --upgrade

# Update specific package
uv lock --upgrade-package requests

# Validate lockfile is up-to-date
uv lock --locked
```

### Exporting Dependencies

```bash
# Export to requirements.txt
uv export --format requirements-txt > requirements.txt

# Export without hashes
uv export --format requirements-txt --no-hashes > requirements.txt

# Export with dev dependencies
uv export --format requirements-txt --all-extras --dev > requirements-dev.txt
```

### Building and Publishing

```bash
# Build distributions
uv build

# Publish to PyPI
uv publish

# Publish to TestPyPI
uv publish --publish-url https://test.pypi.org/legacy/
```

## Advanced Features

### Workspaces (Monorepo Management)

For projects with multiple related packages:

**Root pyproject.toml:**
```toml
[project]
name = "my-workspace"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["shared-lib"]

[tool.uv.sources]
shared-lib = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/experimental"]
```

```bash
# Run command in workspace root
uv run main.py

# Run in specific package
uv run --package shared-lib pytest

# Sync entire workspace
uv sync
```

See [references/PROJECTS.md](references/PROJECTS.md#workspaces) for detailed workspace configuration.

### Dependency Overrides and Constraints

```toml
[tool.uv]
# Override dependency versions
override-dependencies = [
    "numpy==1.24.0",  # Force specific version
]

# Constrain resolution
constraint-dependencies = [
    "pandas<2.0",  # Limit version range
]
```

### Alternative Package Indexes

```bash
# Use custom index
uv add --index https://custom-pypi.org/simple package-name

# Configure in pyproject.toml
```

```toml
[[tool.uv.index]]
name = "custom"
url = "https://custom-pypi.org/simple"
default = true
```

### Platform-Specific Dependencies

```toml
[project]
dependencies = [
    "universal-package",
]

[project.optional-dependencies]
windows = ["pywin32; sys_platform == 'win32'"]
linux = ["systemd-python; sys_platform == 'linux'"]
```

## Docker Integration

### Basic Dockerfile

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --locked --no-dev

COPY . .
CMD ["uv", "run", "main.py"]
```

### Optimized Multi-Stage Build

```dockerfile
# Build stage
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project

COPY . .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked

# Runtime stage
FROM python:3.12-slim
COPY --from=builder /app/.venv /app/.venv
ENV PATH="/app/.venv/bin:$PATH"
WORKDIR /app
COPY --from=builder /app .
CMD ["python", "main.py"]
```

See [references/INTEGRATIONS.md](references/INTEGRATIONS.md#docker) for more Docker patterns.

## GitHub Actions Integration

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      
      - name: Install uv
        uses: astral-sh/setup-uv@v7
        with:
          version: "0.10.2"
          enable-cache: true
      
      - name: Set up Python
        run: uv python install
      
      - name: Install dependencies
        run: uv sync --locked --all-extras --dev
      
      - name: Run tests
        run: uv run pytest
```

See [references/INTEGRATIONS.md](references/INTEGRATIONS.md#github-actions) for complete CI/CD examples.

## Troubleshooting

### Common Issues

**Problem:** `No solution found when resolving dependencies`
**Solution:** Check for conflicting version constraints. Use `--resolution lowest` or override-dependencies.

**Problem:** Package not found in environment after `uv add`
**Solution:** Ensure you run code with `uv run` or manually sync with `uv sync`.

**Problem:** Lockfile out of sync
**Solution:** Run `uv lock` to regenerate, or `uv sync --locked` to enforce existing lockfile.

**Problem:** Python version not found
**Solution:** uv will auto-download. To disable: `export UV_NO_PYTHON_DOWNLOADS=1`

**Problem:** Build failures for compiled packages
**Solution:** Install system dependencies (e.g., `apt-get install python3-dev build-essential`)

### Performance Tips

1. **Use caching** in CI/CD with `--cache-dir`
2. **Enable bytecode compilation** for production: `UV_COMPILE_BYTECODE=1`
3. **Use universal lockfiles** for platform-independent resolution
4. **Leverage global cache** - uv deduplicates dependencies automatically
5. **Use `--no-install-project` for intermediate Docker layers**

## Reference Files

For comprehensive documentation on specific topics:

- **[references/PROJECTS.md](references/PROJECTS.md)** - Project structure, dependencies, workspaces, configuration
- **[references/SCRIPTS_TOOLS.md](references/SCRIPTS_TOOLS.md)** - Scripts, tools, inline metadata, reproducibility
- **[references/PYTHON_MANAGEMENT.md](references/PYTHON_MANAGEMENT.md)** - Python installation, versions, discovery
- **[references/PIP_INTERFACE.md](references/PIP_INTERFACE.md)** - pip compatibility, migration, uv pip commands
- **[references/INTEGRATIONS.md](references/INTEGRATIONS.md)** - Docker, CI/CD, pre-commit, dependency bots
- **[references/RESOLUTION.md](references/RESOLUTION.md)** - Resolution strategies, dependency overrides and constraints
- **[references/AUTHENTICATION.md](references/AUTHENTICATION.md)** - HTTP, Git, and third-party service authentication
- **[references/CACHING_PERFORMANCE.md](references/CACHING_PERFORMANCE.md)** - Caching mechanisms and performance tuning
- **[references/PLATFORM_INDEXES.md](references/PLATFORM_INDEXES.md)** - Platform-specific dependencies and custom indexes
- **[references/DEBUG_TROUBLESHOOTING.md](references/DEBUG_TROUBLESHOOTING.md)** - Debug tools and issue resolution
- **[references/INTERNALS.md](references/INTERNALS.md)** - Internals, best practices, and common patterns

## Command Quick Reference

```bash
# Project Management
uv init [--app|--lib] [--python VERSION] [PATH]
uv add [--dev] [--optional GROUP] PACKAGE
uv remove PACKAGE
uv sync [--locked] [--frozen] [--no-dev]
uv lock [--upgrade] [--upgrade-package PKG]
uv run [--python VERSION] [--no-project] COMMAND

# Scripts
uv init --script FILE.py
uv add --script FILE.py PACKAGE
uv run [--script] FILE.py
uv lock --script FILE.py

# Tools
uvx [--from PACKAGE] COMMAND
uv tool install [--from URL] PACKAGE
uv tool uninstall PACKAGE
uv tool upgrade [--all] PACKAGE
uv tool list

# Python Versions
uv python install [VERSION...]
uv python list [--only-installed]
uv python pin VERSION
uv python upgrade [VERSION]

# pip Interface
uv pip install PACKAGE
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
uv venv [--python VERSION]

# Building & Publishing
uv build [--sdist|--wheel]
uv publish [--token TOKEN]
```

## Best Practices

1. **Always use lockfiles** (`uv.lock`) for reproducible builds
2. **Pin Python versions** with `.python-version` files
3. **Use workspaces** for monorepo management instead of path dependencies
4. **Separate dev and production dependencies** with `--dev` flag
5. **Cache in CI/CD** for faster builds
6. **Use scripts with inline metadata** for shared utilities
7. **Install tools globally** with `uv tool install` instead of in projects
8. **Test with `--frozen`** in CI to ensure lockfile is up-to-date
9. **Use `--no-install-project`** in Docker for better layer caching
10. **Enable bytecode compilation** in production: `UV_COMPILE_BYTECODE=1`

## Learning Resources

- **Official Documentation**: https://docs.astral.sh/uv/
- **GitHub Repository**: https://github.com/astral-sh/uv
- **Docker Examples**: https://github.com/astral-sh/uv-docker-example
- **Changelog**: https://github.com/astral-sh/uv/blob/main/CHANGELOG.md
- **Discord Community**: https://discord.com/invite/astral-sh

## Next Steps

1. **Get familiar** with basic project commands: `uv init`, `uv add`, `uv run`
2. **Practice scripts** with inline dependencies for quick prototyping
3. **Explore workspaces** if managing multiple related packages
4. **Set up CI/CD** with caching for optimal performance
5. **Containerize** your application using uv Docker best practices
6. **Deep dive** into reference files for advanced features

---

For detailed information on any topic, consult the appropriate reference file or run `uv help <command>` for command-specific help.
