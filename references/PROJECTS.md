# Project Management with uv

Comprehensive guide to managing Python projects with uv, including structure, dependencies, configuration, workspaces, and advanced project management strategies.

## Table of Contents

- [Project Basics](#project-basics)
- [Project Structure](#project-structure)
- [Managing Dependencies](#managing-dependencies)
- [Lockfiles](#lockfiles)
- [Project Configuration](#project-configuration)
- [Workspaces](#workspaces)
- [Building and Publishing](#building-and-publishing)
- [Advanced Topics](#advanced-topics)

## Project Basics

### Creating Projects

```bash
# Create new project in new directory
uv init my-project

# Initialize in existing directory
uv init

# Create with specific Python version
uv init --python 3.12 my-project

# Create specific project types
uv init --app my-app        # Application (no build-system)
uv init --lib my-library    # Library (with build-system)
uv init --package my-pkg    # Package (deprecated, use --lib)
```

### Project Types

**Application (`--app`)**:
- Entry point for execution
- No `build-system` in `pyproject.toml`
- Not meant to be installed as dependency
- Example: web servers, CLI tools, scripts

**Library (`--lib`)**:
- Meant to be imported by other packages
- Includes `build-system` configuration
- Source layout: `src/package_name/`
- Example: utility libraries, frameworks

## Project Structure

### Standard Layout

```
my-project/
├── .venv/                  # Virtual environment (auto-created)
├── .python-version        # Pinned Python version
├── .gitignore             # Git ignore rules
├── README.md              # Project documentation
├── pyproject.toml         # Project metadata and dependencies
├── uv.lock                # Universal lockfile (auto-generated)
└── main.py                # Entry point (apps)
    OR
└── src/
    └── my_project/        # Source code (libraries)
        └── __init__.py
```

### Key Files

#### pyproject.toml

Project metadata and dependency configuration:

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.31.0",
    "rich>=13.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "ruff>=0.3.0",
]
test = [
    "pytest>=8.0.0",
    "pytest-cov>=4.0.0",
]

[build-system]
requires = ["uv_build>=0.10.2,<0.11.0"]
build-backend = "uv_build"

[tool.uv]
# uv-specific configuration
dev-dependencies = [
    "mypy>=1.8.0",
]
```

#### .python-version

Specifies the default Python version for the project:

```
3.12
```

#### uv.lock

Universal lockfile with exact resolved versions (auto-managed by uv):

```toml
version = 1
requires-python = ">=3.12"

[[package]]
name = "certifi"
version = "2024.2.2"
source = { registry = "https://pypi.org/simple" }
# ...
```

**Important**: Commit `uv.lock` to version control for reproducible builds.

### Virtual Environment

**Location**: `.venv/` in project root

**Auto-created**: First time you run `uv sync`, `uv run`, or `uv lock`

**Manual creation**:
```bash
uv venv
uv venv --python 3.11  # Specific Python version
```

**Activation** (optional when using `uv run`):
```bash
# macOS/Linux
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

## Managing Dependencies

### Adding Dependencies

```bash
# Add package
uv add requests

# Add with version constraint
uv add 'requests>=2.28,<3.0'
uv add 'requests==2.31.0'

# Add to specific group
uv add pytest --dev
uv add pytest --optional test

# Add with extras
uv add 'fastapi[all]'
uv add 'scikit-learn[tests,docs]'

# Add from git
uv add git+https://github.com/user/repo
uv add git+https://github.com/user/repo@v1.0.0
uv add git+https://github.com/user/repo@main
uv add git+https://github.com/user/repo@abc123

# Add from local path
uv add --editable ./path/to/local/package
uv add file:///absolute/path/to/package

# Add from requirements.txt
uv add -r requirements.txt
```

### Removing Dependencies

```bash
# Remove package
uv remove requests

# Remove dev dependency
uv remove pytest --dev

# Remove optional dependency
uv remove pytest --optional test
```

### Upgrading Dependencies

```bash
# Upgrade specific package
uv lock --upgrade-package requests

# Upgrade package and its dependencies
uv lock --upgrade-package requests --upgrade

# Upgrade all packages
uv lock --upgrade

# Upgrade within constraints
uv lock --upgrade-package 'requests>=2.28,<3.0'
```

### Development Dependencies

Two approaches for development dependencies:

**1. Using `--dev` flag:**
```bash
uv add pytest --dev
uv add ruff --dev
```

```toml
[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "ruff>=0.3.0",
]
```

**2. Using `tool.uv.dev-dependencies`:**
```toml
[tool.uv]
dev-dependencies = [
    "mypy>=1.8.0",
    "black>=24.0.0",
]
```

**Difference**: `tool.uv.dev-dependencies` are uv-specific and not included in distributions.

### Dependency Groups

Organize dependencies by purpose:

```toml
[project.optional-dependencies]
# Testing dependencies
test = [
    "pytest>=8.0.0",
    "pytest-cov>=4.0.0",
    "pytest-asyncio>=0.23.0",
]

# Documentation dependencies
docs = [
    "mkdocs>=1.5.0",
    "mkdocs-material>=9.5.0",
]

# Development tools
dev = [
    "ruff>=0.3.0",
    "mypy>=1.8.0",
]
```

**Installing groups:**
```bash
# Install specific group
uv sync --extra test

# Install multiple groups
uv sync --extra test --extra docs

# Install all extras
uv sync --all-extras

# Include dev dependencies
uv sync --extra test --dev
```

## Lockfiles

### Understanding uv.lock

**Universal lockfile**: Platform-independent, includes all platforms by default

**Contents**:
- Exact versions of all dependencies
- Source URLs and hashes
- Resolution metadata
- Python version constraints

**Auto-managed**: uv updates the lockfile when:
- Adding/removing dependencies
- Running `uv lock`
- Running `uv sync` (if lockfile is stale)

### Working with Lockfiles

```bash
# Generate/update lockfile
uv lock

# Update lockfile for specific package
uv lock --upgrade-package requests

# Validate lockfile is up-to-date
uv lock --locked  # Error if lockfile is stale

# Sync without updating lockfile
uv sync --frozen

# Assert lockfile is current and sync
uv sync --locked
```

### Lockfile Strategies

**Development Workflow:**
```bash
# Allow lockfile updates during development
uv sync
```

**CI/CD Workflow:**
```bash
# Enforce exact lockfile in CI
uv sync --locked --no-dev

# Or with frozen (skip lockfile validation)
uv sync --frozen --no-dev
```

**Production Deployment:**
```bash
# Reproducible installs
uv sync --frozen --no-dev

# With bytecode compilation
UV_COMPILE_BYTECODE=1 uv sync --frozen --no-dev
```

### Exporting Lockfile

```bash
# Export to requirements.txt
uv export --format requirements-txt > requirements.txt

# Without hashes
uv export --no-hashes > requirements.txt

# With dev dependencies
uv export --dev > requirements-dev.txt

# With specific extras
uv export --extra test --extra docs > requirements.txt

# For pip-tools compatibility
uv pip compile pyproject.toml -o requirements.txt
```

## Project Configuration

### uv Configuration in pyproject.toml

```toml
[tool.uv]
# Development dependencies (uv-specific)
dev-dependencies = [
    "pytest>=8.0.0",
]

# Override dependency versions globally
override-dependencies = [
    "numpy==1.24.0",
]

# Constrain dependency versions
constraint-dependencies = [
    "pandas<2.0",
]

# Custom package indexes
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cu118"
explicit = true

# Source overrides
[tool.uv.sources]
my-package = { git = "https://github.com/user/repo" }
local-package = { path = "../local-package", editable = true }

# Environment settings
[tool.uv]
no-binary = ["pyyaml"]  # Don't use wheels for these packages
no-build-isolation = ["local-package"]  # Skip build isolation
```

### Environment Variables

Configure uv behavior via environment variables:

```bash
# Cache directory
export UV_CACHE_DIR=/custom/cache

# Disable managed Python downloads
export UV_NO_PYTHON_DOWNLOADS=1

# Use system Python
export UV_SYSTEM_PYTHON=1

# Link mode (copy, hardlink, symlink)
export UV_LINK_MODE=copy

# Compile bytecode
export UV_COMPILE_BYTECODE=1

# Project environment path
export UV_PROJECT_ENVIRONMENT=/custom/venv

# Exclude newer packages (reproducibility)
export UV_EXCLUDE_NEWER=2024-01-01T00:00:00Z
```

### Configuration Files

uv reads configuration from multiple locations (in order):

1. **`pyproject.toml`** - Project-specific configuration
2. **`uv.toml`** - uv-specific configuration file
3. **`~/.config/uv/uv.toml`** - User-level configuration
4. **Environment variables** - Highest priority

Example `uv.toml`:

```toml
[pip]
index-url = "https://custom-pypi.org/simple"
extra-index-url = ["https://pypi.org/simple"]

[cache]
dir = "/custom/cache"

[python]
preference = "only-managed"  # or "managed" or "system" or "only-system"
```

## Workspaces

Workspaces enable managing multiple related packages in a single repository with a shared lockfile.

### When to Use Workspaces

**Use workspaces when:**
- Managing monorepo with multiple Python packages
- Packages share common dependencies
- Want unified versioning and testing
- Need atomic updates across packages

**Don't use workspaces when:**
- Packages have conflicting requirements
- Need separate virtual environments per package
- Packages are independently versioned

### Creating a Workspace

**Root `pyproject.toml`:**
```toml
[project]
name = "my-workspace"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["shared-lib", "utils-lib"]

[tool.uv.sources]
shared-lib = { workspace = true }
utils-lib = { workspace = true }

[tool.uv.workspace]
members = [
    "packages/*",      # All packages in packages/
    "libs/core",       # Specific path
]
exclude = [
    "packages/experimental",
]
```

**Workspace Structure:**
```
my-workspace/
├── pyproject.toml          # Root/workspace config
├── uv.lock                 # Shared lockfile
├── src/
│   └── my_workspace/
│       └── __init__.py
└── packages/
    ├── shared-lib/
    │   ├── pyproject.toml
    │   └── src/
    │       └── shared_lib/
    │           └── __init__.py
    └── utils-lib/
        ├── pyproject.toml
        └── src/
            └── utils_lib/
                └── __init__.py
```

### Workspace Members

**Member `pyproject.toml`:**
```toml
[project]
name = "shared-lib"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.31.0",
]

[build-system]
requires = ["uv_build>=0.10.2,<0.11.0"]
build-backend = "uv_build"
```

**Dependencies between members:**
```toml
# In another member's pyproject.toml
[project]
name = "api-service"
dependencies = [
    "shared-lib",  # Workspace member
    "flask>=3.0.0",
]

[tool.uv.sources]
shared-lib = { workspace = true }
```

### Working with Workspaces

```bash
# Commands default to workspace root
uv sync           # Sync entire workspace
uv lock           # Update shared lockfile
uv run main.py    # Run in root package

# Target specific workspace member
uv run --package shared-lib pytest
uv run --package api-service python -m app

# Build specific member
uv build --package shared-lib

# Add dependency to specific member
cd packages/shared-lib
uv add requests
# Or from root
uv add --package shared-lib requests
```

### Workspace Sources

**Shared sources in root** apply to all members:

```toml
[tool.uv.sources]
# All members use this version
numpy = { git = "https://github.com/numpy/numpy", tag = "v1.24.0" }
```

**Member-specific sources** override root:

```toml
# In packages/experimental/pyproject.toml
[tool.uv.sources]
numpy = { git = "https://github.com/numpy/numpy", tag = "v2.0.0" }
```

### Workspace vs Path Dependencies

**Workspace dependencies** (recommended for monorepos):
```toml
[tool.uv.sources]
my-lib = { workspace = true }
```

**Path dependencies** (independent packages):
```toml
[tool.uv.sources]
my-lib = { path = "../my-lib", editable = true }
```

**Key differences:**
- Workspaces share single lockfile
- Path dependencies have separate environments
- Workspaces require uniform `requires-python`

## Building and Publishing

### Building Distributions

```bash
# Build both sdist and wheel
uv build

# Build only source distribution
uv build --sdist

# Build only wheel
uv build --wheel

# Build from different directory
uv build --package my-lib

# Output to custom directory
uv build --out-dir custom-dist/
```

**Build outputs** (in `dist/`):
- `package-name-0.1.0.tar.gz` - Source distribution
- `package_name-0.1.0-py3-none-any.whl` - Wheel distribution

### Publishing to PyPI

**Using trusted publishing (recommended):**

```bash
# Publish with trusted publishing
uv publish
```

**Using API token:**

```bash
# Publish with token
uv publish --token $PYPI_TOKEN

# Or set in environment
export UV_PUBLISH_TOKEN=$PYPI_TOKEN
uv publish

# Publish to TestPyPI
uv publish --publish-url https://test.pypi.org/legacy/
```

**Complete publish workflow:**

```bash
# 1. Update version in pyproject.toml
# 2. Test build
uv build
uv run --isolated --with dist/*.whl python -c "import my_package"

# 3. Publish
uv publish

# 4. Verify
pip install my-package
python -c "import my_package; print(my_package.__version__)"
```

### Version Management

```bash
# View current version
uv version

# View short version
uv version --short

# View as JSON
uv version --output-format json
```

**Update version** in `pyproject.toml`:
```toml
[project]
version = "0.2.0"
```

**Dynamic versioning** (using build backend):
```toml
[project]
dynamic = ["version"]

[tool.uv_build]
version = { from = "src/my_package/__init__.py" }
```

## Advanced Topics

### Resolution Strategies

Control how uv resolves dependencies:

```bash
# Default: highest compatible versions
uv lock

# Lowest compatible versions (for testing)
uv lock --resolution lowest

# Lowest direct dependencies, highest transitive
uv lock --resolution lowest-direct
```

**Configure in pyproject.toml:**
```toml
[tool.uv]
resolution = "highest"  # or "lowest", "lowest-direct"
```

### Dependency Overrides

Force specific versions globally:

```toml
[tool.uv]
override-dependencies = [
    "numpy==1.24.0",           # Pin exact version
    "protobuf<4",              # Set upper bound
]
```

Use cases:
- Work around dependency conflicts
- Security patches for transitive dependencies
- Test compatibility with specific versions

### Platform-Specific Dependencies

```toml
[project.dependencies]
dependencies = [
    "common-package",
]

[project.optional-dependencies]
# Platform markers
windows = [
    "pywin32; sys_platform == 'win32'",
]
unix = [
    "systemd-python; sys_platform == 'linux'",
    "uvloop; sys_platform != 'win32'",
]
```

**Environment markers:**
- `sys_platform` - Platform identifier
- `platform_system` - OS name
- `python_version` - Python version
- `implementation_name` - Python implementation

### Reproducible Builds

Ensure identical environments:

```toml
[tool.uv]
# Only consider packages released before date
exclude-newer = "2024-01-01T00:00:00Z"
```

```bash
# Lock with exclusion
UV_EXCLUDE_NEWER=2024-01-01T00:00:00Z uv lock

# Sync reproducibly
uv sync --frozen --no-dev
```

### Custom Package Indexes

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cu118"
explicit = true  # Only use for explicitly listed packages

[[tool.uv.index]]
name = "private"
url = "https://pypi.company.com/simple"
default = true   # Use as default index
```

```bash
# Install from specific index
uv add --index pytorch torch torchvision

# Set default index
uv pip install --index-url https://custom.pypi.org/simple package
```

### Git Dependencies

```bash
# Latest from branch
uv add git+https://github.com/user/repo

# Specific branch
uv add git+https://github.com/user/repo@main

# Specific tag
uv add git+https://github.com/user/repo@v1.0.0

# Specific commit
uv add git+https://github.com/user/repo@abc123

# With Git LFS support
uv add --lfs git+https://github.com/user/repo
```

**In pyproject.toml:**
```toml
[tool.uv.sources]
my-package = { git = "https://github.com/user/repo", rev = "main" }
```

### Path Dependencies

```bash
# Editable install (development)
uv add --editable ./path/to/package

# Non-editable install
uv add file:///absolute/path/to/package
```

```toml
[tool.uv.sources]
my-local-package = { path = "../my-local-package", editable = true }
```

### No Binary/Build Isolation

```toml
[tool.uv]
# Don't use binary distributions (force build from source)
no-binary = ["pyyaml", "lxml"]

# Skip build isolation for specific packages
no-build-isolation = ["my-local-package"]
```

## Best Practices

1. **Always commit `uv.lock`** for reproducible builds
2. **Use `uv sync --locked`** in CI/CD to enforce lockfile
3. **Pin Python version** with `.python-version`
4. **Separate concerns** with optional dependency groups
5. **Use workspaces** for monorepos instead of path dependencies
6. **Test with `--resolution lowest`** to ensure broad compatibility
7. **Export lockfile** as `requirements.txt` for pip compatibility
8. **Regular updates**: Run `uv lock --upgrade` periodically
9. **Security**: Use `override-dependencies` for patching vulnerabilities
10. **Document dependencies**: Add comments in `pyproject.toml` for non-obvious choices

## Common Patterns

### Hybrid pip/uv Project

```bash
# During transition, support both
uv export --no-hashes > requirements.txt
git add requirements.txt uv.lock
```

### Monorepo with Shared Dependencies

```toml
[tool.uv.workspace]
members = ["services/*", "libs/*"]

# Shared dependencies in root
[project]
dependencies = ["pydantic>=2.0"]
```

### Private Package Registry

```toml
[[tool.uv.index]]
name = "company"
url = "https://${PYPI_USER}:${PYPI_PASSWORD}@pypi.company.com/simple"

# Or use netrc/keyring for authentication
```

### Multi-Environment Testing

```bash
# Test with different Python versions
uv run --python 3.11 pytest
uv run --python 3.12 pytest
uv run --python 3.13 pytest

# Test with different dependency versions
uv lock --upgrade-package 'requests>=2.28,<2.29'
uv sync --locked && uv run pytest
```

## Troubleshooting

### Project Not Found

**Error**: `No project found`
**Solution**: Ensure `pyproject.toml` exists with `[project]` section

### Lock File Out of Sync

**Error**: `The lockfile is out of date`
**Solution**: Run `uv lock` to regenerate, or `uv sync --frozen` to skip check

### Dependency Conflicts

**Error**: `No solution found when resolving dependencies`
**Solutions**:
1. Check conflicting version constraints
2. Use `override-dependencies` to force versions
3. Use `constraint-dependencies` to limit ranges
4. Try `--resolution lowest` to find minimum compatible versions

### Build Failures

**Error**: Package build fails
**Solutions**:
1. Install system dependencies (e.g., `build-essential`, `python3-dev`)
2. Use `no-binary` to build from source
3. Check package requirements documentation
4. Use different index if pre-built wheels available

### Workspace Issues

**Error**: `Member not found in workspace`
**Solution**: Ensure path matches `members` glob and `pyproject.toml` exists

**Error**: `Conflicting requires-python`
**Solution**: Workspaces require uniform Python version across all members
