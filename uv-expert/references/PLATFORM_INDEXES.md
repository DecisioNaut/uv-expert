# Platform-Specific Dependencies and Custom Indexes

Comprehensive guide to platform-specific dependencies and custom package indexes.

## Table of Contents

- [Platform-Specific Dependencies](#platform-specific-dependencies)
- [Custom Package Indexes](#custom-package-indexes)

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
