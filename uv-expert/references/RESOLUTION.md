# Resolution Strategies and Dependency Management

Comprehensive guide to uv dependency resolution strategies, overrides, and constraints.

## Table of Contents

- [Resolution Strategies](#resolution-strategies)
- [Dependency Overrides](#dependency-overrides)
- [Dependency Constraints](#dependency-constraints)

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
