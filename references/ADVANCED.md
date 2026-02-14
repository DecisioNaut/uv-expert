# Advanced Topics for uv

Comprehensive guide to advanced uv features, resolution strategies, authentication, caching, troubleshooting, and internals.

## Table of Contents

- [Resolution Strategies](#resolution-strategies)
- [Dependency Overrides](#dependency-overrides)
- [Dependency Constraints](#dependency-constraints)
- [Authentication](#authentication)
- [Caching](#caching)
- [Performance Optimization](#performance-optimization)
- [Platform-Specific Dependencies](#platform-specific-dependencies)
- [Custom Package Indexes](#custom-package-indexes)
- [Debug and Introspection](#debug-and-introspection)
- [Troubleshooting](#troubleshooting)
- [Internals](#internals)

## Resolution Strategies

uv supports three resolution strategies to control dependency version selection.

### Highest Resolution (Default)

Installs the latest compatible versions:

```bash
# Default behavior
uv add requests

# Explicit
uv add --resolution highest requests
```

```toml
# pyproject.toml
[tool.uv]
resolution = "highest"
```

**Use when:**
- Starting new projects
- Want latest features and bug fixes
- Following semantic versioning

### Lowest Resolution

Installs the minimum compatible versions:

```bash
uv add --resolution lowest requests
```

```toml
[tool.uv]
resolution = "lowest"
```

**Use when:**
- Testing minimum supported versions
- Ensuring backward compatibility
- Conservative dependency management

### Lowest-Direct Resolution

Latest versions for direct dependencies, minimum for transitive:

```bash
uv add --resolution lowest-direct requests
```

```toml
[tool.uv]
resolution = "lowest-direct"
```

**Use when:**
- Balance between latest and conservative
- Reduce dependency conflicts
- Library development

### Strategy Comparison

```bash
# Example: Adding httpx (depends on certifi, httpcore, idna, sniffio)

# Highest: httpx==0.27.0, certifi==2024.2.2, httpcore==1.0.5
uv add --resolution highest httpx

# Lowest: httpx==0.23.0, certifi==2021.5.30, httpcore==0.15.0
uv add --resolution lowest httpx

# Lowest-direct: httpx==0.27.0, certifi==2021.5.30, httpcore==0.15.0
uv add --resolution lowest-direct httpx
```

### Per-Dependency Resolution

Override strategy for specific packages:

```toml
[tool.uv.sources]
# Always latest for this package
security-lib = { index = "pypi", resolution = "highest" }

# Minimum version for compatibility testing
legacy-lib = { index = "pypi", resolution = "lowest" }
```

## Dependency Overrides

Force specific versions regardless of requirements.

### Basic Overrides

```toml
[tool.uv]
override-dependencies = [
    "numpy==1.26.0",
    "pandas<2.0",
]
```

**Effect:** All packages requesting numpy or pandas will use these versions.

### Override Sources

```toml
[tool.uv]
override-dependencies = [
    "security-fix @ git+https://github.com/org/security-fix@main",
]
```

### Use Cases

**Security patches:**
```toml
# Force patched version even if dependencies request vulnerable version
[tool.uv]
override-dependencies = ["requests==2.31.0"]  # CVE fix
```

**Platform-specific fixes:**
```toml
[tool.uv]
override-dependencies = [
    "torch==2.0.0+cpu ; platform_machine != 'x86_64'"
]
```

**Dependency conflicts:**
```toml
# When package-a wants pkg==1.0 and package-b wants pkg==2.0
[tool.uv]
override-dependencies = ["pkg==1.5"]  # Compatible middle ground
```

### Override vs Constraint

**Override**: Ignores all other requirements
**Constraint**: Additional restriction on existing requirements

```toml
# Override: Forces version even if dependencies want different version
[tool.uv]
override-dependencies = ["requests==2.31.0"]

# Constraint: Only applied if dependency is already requested
[tool.uv]
constraint-dependencies = ["requests<3.0"]
```

## Dependency Constraints

Add restrictions without directly depending on packages.

### Basic Constraints

```toml
[tool.uv]
constraint-dependencies = [
    "numpy<2.0",         # Upper bound
    "pandas>=1.5",       # Lower bound
    "scipy>=1.10,<2.0",  # Range
]
```

**Use when:**
- Need compatibility restrictions
- Don't want package in final environment
- Testing dependency ranges

### Constraint Sources

```bash
# From file
uv pip install --constraint constraints.txt package

# From URL
uv pip install --constraint https://example.com/constraints.txt package
```

**constraints.txt:**
```
numpy<2.0
pandas>=1.5,<2.0
scikit-learn==1.3.*
```

### Use Cases

**CI matrix testing:**
```yaml
# Test against different constraint sets
- name: Test with minimum versions
  run: uv pip install --constraint constraints-min.txt -e .

- name: Test with maximum versions
  run: uv pip install --constraint constraints-max.txt -e .
```

**Platform compatibility:**
```toml
[tool.uv]
constraint-dependencies = [
    "numpy<2.0 ; python_version < '3.12'",
    "tensorflow<2.16 ; platform_system == 'Windows'",
]
```

## Authentication

### HTTP Authentication

#### netrc File

Best practice for persistent credentials:

```bash
# ~/.netrc
machine pypi.org
login username
password token

machine github.com
login token
password x-oauth-basic

machine gitlab.example.com
login oauth2
password <token>
```

**Permissions:**
```bash
chmod 600 ~/.netrc
```

#### Environment Variables

```bash
# Basic auth
export UV_INDEX_URL="https://username:password@pypi.example.com/simple"

# Token auth
export UV_INDEX_URL="https://token@pypi.example.com/simple"

# AWS CodeArtifact
export UV_INDEX_URL=$(aws codeartifact get-repository-endpoint \
  --domain my-domain \
  --repository my-repo \
  --format pypi \
  --query repositoryEndpoint \
  --output text)

export UV_INDEX_TOKEN=$(aws codeartifact get-authorization-token \
  --domain my-domain \
  --query authorizationToken \
  --output text)
```

#### Configuration File

```toml
# pyproject.toml
[[tool.uv.index]]
name = "private"
url = "https://${PRIVATE_PYPI_USER}:${PRIVATE_PYPI_TOKEN}@pypi.example.com/simple"
```

**Note:** Environment variables substitution supported in .env files.

### Git Authentication

#### SSH Keys

**Recommended approach:**

```bash
# Ensure SSH agent is running
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Use SSH URLs in dependencies
```

```toml
[project]
dependencies = [
    "package @ git+ssh://git@github.com/org/repo.git",
]
```

#### HTTPS with Tokens

**Personal Access Token (PAT):**

```toml
[project]
dependencies = [
    "package @ git+https://${GITHUB_TOKEN}@github.com/org/private-repo.git",
]
```

**GitHub Actions:**
```yaml
- name: Install dependencies
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: uv sync
```

**git-credential Helper:**

```bash
# Store token in git credential manager
git config --global credential.helper store
echo "https://token@github.com" > ~/.git-credentials
```

#### GitLab CI Token

```yaml
# .gitlab-ci.yml
variables:
  GIT_STRATEGY: clone
  
before_script:
  - git config --global url."https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.com/".insteadOf "https://gitlab.com/"
  - uv sync
```

### Third-Party Services

#### JFrog Artifactory

```bash
# API key
export UV_INDEX_URL="https://username:${ARTIFACTORY_API_KEY}@company.jfrog.io/artifactory/api/pypi/pypi/simple"

# Auth via config
curl -u username:${ARTIFACTORY_API_KEY} \
  https://company.jfrog.io/artifactory/api/pypi/pypi/simple
```

#### Azure Artifacts

```bash
# Get token
az artifacts universal download \
  --organization https://dev.azure.com/org \
  --feed feed-name \
  --name package \
  --version 1.0.0

# Configure
export UV_INDEX_URL="https://pkgs.dev.azure.com/org/_packaging/feed/pypi/simple/"
export UV_INDEX_USERNAME="azure"
export UV_INDEX_PASSWORD=$(az account get-access-token --query accessToken -o tsv)
```

#### Google Artifact Registry

```bash
# gcloud auth
gcloud auth print-access-token | uv pip install \
  --index-url https://oauth2accesstoken:$(gcloud auth print-access-token)@python.pkg.dev/project/repository/simple \
  package
```

### TLS/SSL Certificates

#### Custom CA Certificates

```bash
# System CA bundle
export UV_CA_BUNDLE="/etc/ssl/certs/ca-certificates.crt"

# Custom certificate
export UV_CA_BUNDLE="/path/to/custom-ca-bundle.crt"

# Combine with system certificates
cat /etc/ssl/certs/ca-certificates.crt custom-ca.crt > combined-ca-bundle.crt
export UV_CA_BUNDLE="combined-ca-bundle.crt"
```

#### Client Certificates

```bash
export UV_CLIENT_CERT="/path/to/client.pem"
export UV_CLIENT_KEY="/path/to/client-key.pem"
```

#### Disable Verification (Not Recommended)

```bash
# Only for testing/debugging
export UV_NO_VERIFY_SSL=1
```

### Credential Security

**Best practices:**

1. **Never commit credentials** - Use environment variables or .netrc
2. **Use .env files** - Keep .env in .gitignore
3. **Rotate tokens regularly** - Especially for CI/CD
4. **Use minimal scopes** - Read-only access when possible
5. **Prefer trusted publishing** - For PyPI publishing (no tokens)
6. **CI secrets** - Use GitHub Secrets, GitLab CI/CD Variables
7. **Local development** - Use keychain/credential managers

**Example .env:**
```bash
# .env (in .gitignore)
PRIVATE_PYPI_TOKEN=pypi-AgEIcH...
GITHUB_TOKEN=ghp_abc...
ARTIFACTORY_API_KEY=AKC...
```

## Caching

### Cache Location

**Default locations:**
```bash
# Linux
~/.cache/uv

# macOS
~/Library/Caches/uv

# Windows
%LOCALAPPDATA%\uv\cache
```

**Custom location:**
```bash
export UV_CACHE_DIR="/custom/cache/path"
```

### Cache Structure

```
uv/
├── archive-v0/          # Downloaded package archives
├── built-wheels-v3/     # Built wheels from source
├── git-v0/              # Git repository clones
├── interpreter-v0/      # Python interpreter installations
└── simple-v12/          # Package metadata
```

### Cache Management

**View cache info:**
```bash
uv cache dir          # Show cache directory
uv cache size         # Show cache size
```

**Clean cache:**
```bash
# Remove all cached data
uv cache clean

# Remove specific package
uv cache clean requests

# Prune old/unused data (CI optimized)
uv cache prune --ci
```

### Cache in CI/CD

**GitHub Actions:**
```yaml
env:
  UV_CACHE_DIR: /tmp/.uv-cache

steps:
  - name: Restore cache
    uses: actions/cache@v5
    with:
      path: /tmp/.uv-cache
      key: uv-${{ runner.os }}-${{ hashFiles('uv.lock') }}
      restore-keys: uv-${{ runner.os }}-
  
  - name: Install
    run: uv sync
  
  - name: Minimize cache
    run: uv cache prune --ci
```

**GitLab CI:**
```yaml
variables:
  UV_CACHE_DIR: "$CI_PROJECT_DIR/.cache/uv"

cache:
  paths:
    - .cache/uv

after_script:
  - uv cache prune --ci
```

**Docker BuildKit:**
```dockerfile
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen
```

### Global Cache Benefits

- **Shared across projects**: One download serves all projects
- **Faster installations**: Reuse built wheels
- **Bandwidth savings**: Download once, use many times
- **Disk space**: ~100MB-1GB typically

### Cache Tuning

**Aggressive caching (default):**
```bash
# uv will reuse cached data whenever possible
uv sync
```

**Refresh cache:**
```bash
# Force re-download of metadata
uv sync --refresh

# Force rebuild from source
uv sync --no-binary :all:
```

**Read-only cache:**
```bash
# Don't write to cache (CI environments)
export UV_NO_CACHE=1
```

## Performance Optimization

### Parallel Resolution

uv resolves dependencies in parallel by default (no configuration needed).

### Link Mode

Control how packages are installed:

```bash
# Copy files (default, safest)
export UV_LINK_MODE=copy

# Hardlinks (faster, saves space)
export UV_LINK_MODE=hardlink

# Symlinks (fastest, but may break some packages)
export UV_LINK_MODE=symlink

# Clone (copy-on-write on supported filesystems)
export UV_LINK_MODE=clone
```

**Recommendation**: Use `copy` for reliability, `hardlink` for performance.

### Bytecode Compilation

Pre-compile Python files:

```bash
export UV_COMPILE_BYTECODE=1
uv sync
```

Benefits:
- Faster Python startup
- Smaller Docker images (with multi-stage builds)
- Better for production deployments

### Concurrent Downloads

```bash
# Increase concurrent downloads (default: 50)
export UV_CONCURRENT_DOWNLOADS=100

# Reduce for unstable networks
export UV_CONCURRENT_DOWNLOADS=10
```

### Exclude Newer

Limit to packages released before date:

```bash
# Reproducible builds
uv sync --exclude-newer 2024-01-01

# In lockfile
uv lock --exclude-newer 2024-01-01
```

**Use for:**
- Reproducible CI builds
- Investigating when issues were introduced
- Pinning to known-good state

### Network Timeout

```bash
# Increase timeout for slow networks (default: 30s)
export UV_HTTP_TIMEOUT=120

# Retries
export UV_HTTP_RETRIES=5
```

### Native vs Pure Python

```bash
# Prefer binary wheels
uv add package

# Force source build
uv add --no-binary package package

# No binary for all packages
uv sync --no-binary :all:

# Only for specific packages
uv sync --no-binary numpy,pandas
```

## Platform-Specific Dependencies

### Environment Markers

```toml
[project]
dependencies = [
    "pywin32 ; platform_system == 'Windows'",
    "pyobjc ; platform_system == 'Darwin'",
    "uvloop ; platform_system == 'Linux'",
    "typing-extensions ; python_version < '3.11'",
]
```

**Available markers:**
- `platform_system`: 'Linux', 'Windows', 'Darwin'
- `platform_machine`: 'x86_64', 'aarch64', 'arm64'
- `python_version`: '3.11', '3.12', etc.
- `python_full_version`: '3.12.0', '3.12.1'
- `sys_platform`: 'linux', 'win32', 'darwin'
- `os_name`: 'posix', 'nt'

### Conditional Dependencies

```toml
[project.optional-dependencies]
linux = [
    "uvloop>=0.17",
    "python-prctl>=1.8",
]
macos = [
    "pyobjc-framework-Cocoa>=9.0",
]
windows = [
    "pywin32>=305",
    "wmi>=1.5",
]

# Install platform-specific extras
# Linux: uv sync --extra linux
# macOS: uv sync --extra macos
# Windows: uv sync --extra windows
```

### Multi-Platform Lockfiles

uv lockfiles support multiple platforms:

```bash
# Generate lock for multiple platforms
uv lock --python-platform linux --python-platform darwin --python-platform windows

# Sync for current platform only
uv sync --locked
```

**Cross-platform example:**
```toml
[[package]]
name = "torch"
version = "2.0.0"

[[package.metadata]]
requires-python = ">=3.8"

[[package.metadata.distributions]]
name = "torch-2.0.0-cp312-cp312-manylinux_2_17_x86_64.whl"
url = "https://..."
platform = "linux"

[[package.metadata.distributions]]
name = "torch-2.0.0-cp312-cp312-macosx_11_0_arm64.whl"
url = "https://..."
platform = "darwin"
```

## Custom Package Indexes

### Multiple Indexes

```toml
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "private"
url = "https://pypi.company.com/simple"
default = true

[[tool.uv.index]]
name = "pypi"
url = "https://pypi.org/simple"
```

### Index Priority

```toml
# Default index used for unlisted packages
[[tool.uv.index]]
name = "private"
url = "https://pypi.company.com/simple"
default = true

# Explicit index for specific packages
[tool.uv.sources]
torch = { index = "pytorch" }
internal-lib = { index = "private" }
requests = { index = "pypi" }
```

### Index Assignments

**By name:**
```toml
[tool.uv.sources]
package-name = { index = "private" }
```

**By namespace:**
```toml
[tool.uv.sources]
"company-*" = { index = "private" }
```

### Flat vs Simple Index

**Simple index (default):**
```
https://pypi.org/simple/
  └── package-name/
      ├── package-name-1.0.tar.gz
      └── package-name-1.1-py3-none-any.whl
```

**Flat index:**
```
https://example.com/packages/
  ├── package-name-1.0.tar.gz
  └── package-name-1.1-py3-none-any.whl
```

Configure flat index:
```bash
export UV_INDEX_URL="https://example.com/packages/"
```

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
