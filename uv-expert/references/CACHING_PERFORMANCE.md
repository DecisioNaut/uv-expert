# Caching and Performance Optimization

Comprehensive guide to uv caching mechanisms and performance tuning.

## Table of Contents

- [Caching](#caching)
- [Performance Optimization](#performance-optimization)

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
