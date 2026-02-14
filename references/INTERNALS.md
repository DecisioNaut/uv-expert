# uv Internals and Best Practices

Comprehensive guide to uv internals, best practices, and common patterns.

## Table of Contents

- [Internals](#internals)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)
- [Debugging Checklist](#debugging-checklist)
- [Additional Resources](#additional-resources)

## Internals

### Resolver Behavior

uv uses a **PubGrub resolver** (also used by Cargo, pub):

1. **Start**: List all direct dependencies
2. **Pick**: Select highest compatible version
3. **Propagate**: Gather dependencies of selected package
4. **Check**: Verify no conflicts with existing selections
5. **Backtrack**: If conflict, try alternative version
6. **Repeat**: Until all dependencies resolved or proven impossible

**Optimization:**
- Parallel metadata fetching
- Aggressive pre-fetching
- Caching at every step
- Smart version ordering (prefer recent & compatible)

### Lock File Format

```toml
version = 1
requires-python = ">=3.11"

[[package]]
name = "requests"
version = "2.31.0"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "certifi", specifier = ">=2017.4.17" },
    { name = "charset-normalizer", specifier = ">=2,<4" },
    { name = "idna", specifier = ">=2.5,<4" },
    { name = "urllib3", specifier = ">=1.21.1,<3" },
]

[[package.distributions]]
name = "requests-2.31.0-py3-none-any.whl"
url = "https://files.pythonhosted.org/..."
hash = "sha256:58cd2187c01e70e6e26505bca751777aa9f2ee0b7f4300988b709f44e013003f"
```

**Fields:**
- `version`: Lock file format version
- `requires-python`: Python version constraint
- `package`: Individual package entry
- `dependencies`: Runtime dependencies
- `distributions`: Available wheels/sdists with hashes

### Virtual Environment Structure

```
.venv/
├── bin/              # Executables (python, pip, installed scripts)
├── lib/python3.12/   # Installed packages
│   └── site-packages/
├── pyvenv.cfg        # Virtual environment config
└── uv.lock           # Not stored here, in project root
```

**pyvenv.cfg:**
```ini
home = /usr/local/bin
include-system-site-packages = false
version = 3.12.2
executable = /usr/local/bin/python3.12
command = /Users/user/.cargo/bin/uv venv
```

### Package Installation Steps

1. **Resolution**: Determine package versions
2. **Download**: Fetch wheels or sdists
3. **Build** (if needed): Compile sdist to wheel
4. **Unpack**: Extract wheel to temporary location
5. **Install**: Copy/link files to site-packages
6. **Scripts**: Generate entry point scripts in bin/
7. **Metadata**: Write package metadata to site-packages/

### Wheel vs Source Distribution

**Wheel (.whl):**
- Pre-built binary
- Fast installation (just unzip)
- Platform-specific
- Preferred by uv

**Source distribution (.tar.gz, .zip):**
- Source code
- Requires build step
- Platform-independent
- Fallback when no wheel available

**Installation preference:**
1. Compatible wheel from cache
2. Compatible wheel from index
3. Build wheel from sdist (cached)
4. Build wheel from sdist (fresh)

### Performance Characteristics

**Typical speeds (vs pip):**
- Cold cache: 10-25x faster
- Warm cache: 80-100x faster
- Resolution only: 15-50x faster

**Factors:**
- Parallel downloads/builds
- Rust implementation
- Efficient caching
- Smart resolution
- Optimized I/O

## Best Practices

### Development

1. **Use lockfiles**: Commit `uv.lock` to version control
2. **Pin Python**: Use `.python-version` file
3. **Sync regularly**: `uv sync` after pulling changes
4. **Check diffs**: Review `uv.lock` changes in PRs
5. **Dev dependencies**: Use dependency groups, not optional dependencies

### Production

1. **Frozen installs**: Always use `--frozen` or `--locked`
2. **No dev deps**: Use `--no-dev`
3. **Compile bytecode**: Set `UV_COMPILE_BYTECODE=1`
4. **Read-only cache**: Consider `UV_NO_CACHE=1`
5. **Verify hashes**: Lockfile includes hashes automatically

### CI/CD

1. **Pin uv version**: Ensure reproducible builds
2. **Cache aggressively**: Cache `UV_CACHE_DIR`
3. **Prune cache**: Run `uv cache prune --ci` at end
4. **Use `--frozen`**: Never `--locked` or upgrade in CI
5. **Matrix test**: Test all supported Python versions

### Security

1. **Review lock changes**: Check for unexpected upgrades
2. **Use hashes**: Lockfile verifies package integrity
3. **Private repos**: Use environment variables for credentials
4. **Audit dependencies**: Regularly check for vulnerabilities
5. **Minimal credentials**: Read-only tokens where possible

### Performance

1. **Link mode**: Use `hardlink` for speed
2. **Enable cache**: Never disable in development
3. **Parallel jobs**: Default is optimal, don't reduce
4. **Exclude newer**: Pin to date for reproducibility
5. **Bytecode compilation**: Enable for production

## Common Patterns

### Monorepo Setup

```toml
# Root pyproject.toml
[tool.uv.workspace]
members = ["lib-a", "lib-b", "app"]

# lib-a/pyproject.toml
[project]
name = "lib-a"
dependencies = ["requests"]

# app/pyproject.toml
[project]
name = "app"
dependencies = [
    "lib-a",
    "lib-b",
]
```

### Microservices

Each service has its own environment:

```bash
services/
├── api/
│   ├── pyproject.toml
│   ├── uv.lock
│   └── .venv/
├── worker/
│   ├── pyproject.toml
│   ├── uv.lock
│   └── .venv/
└── shared/
    └── pyproject.toml  # Shared library
```

### Plugin Architecture

```toml
# Core application
[project]
name = "myapp"
dependencies = ["plugin-interface"]

[project.optional-dependencies]
plugins = [
    "myapp-plugin-a",
    "myapp-plugin-b",
]

# Install with plugins
# uv sync --extra plugins
```

### Development Tooling

```toml
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "ruff>=0.3",
    "mypy>=1.8",
]

[tool.uv]
dev-dependencies = [
    "pytest-cov>=4.0",
    "pre-commit>=3.0",
]

# Install all dev tools
# uv sync --all-extras --dev
```

## Debugging Checklist

When encountering issues:

- [ ] Check uv version: `uv --version`
- [ ] Try verbose mode: `uv -v` or `uv -vv`
- [ ] Inspect lock file: Check for unexpected versions
- [ ] Clear cache: `uv cache clean`
- [ ] Check Python version: `uv python list`
- [ ] Verify network: `curl https://pypi.org`
- [ ] Check authentication: Test with curl/wget
- [ ] Review environment: `env | grep UV_`
- [ ] Test minimal example: Fresh project with issue
- [ ] Check GitHub issues: Search for similar problems

## Additional Resources

- **Resolver algorithm**: https://github.com/pubgrub-rs/pubgrub
- **Lock file spec**: https://github.com/astral-sh/uv/blob/main/LOCKFILE.md
- **Performance benchmarks**: https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md
- **Contributing guide**: https://github.com/astral-sh/uv/blob/main/CONTRIBUTING.md
- **Issue tracker**: https://github.com/astral-sh/uv/issues
