# Scripts and Tools Management with uv

Comprehensive guide to running Python scripts with inline dependencies and managing command-line tools with uv.

## Table of Contents

- [Running Scripts](#running-scripts)
- [Script Dependencies](#script-dependencies)
- [Tools Management](#tools-management)
- [Advanced Script Features](#advanced-script-features)
- [Tool Usage Patterns](#tool-usage-patterns)
- [Best Practices](#best-practices)

## Running Scripts

### Basic Script Execution

```bash
# Run simple script
uv run script.py

# Run with arguments
uv run script.py arg1 arg2

# Run from stdin
echo 'print("hello")' | uv run -

# Run with here-document
uv run - <<EOF
print("hello world!")
EOF
```

### Script Without Dependencies

Simple scripts that only use standard library:

```python
# hello.py
import os
import sys

print(f"Hello from Python {sys.version}")
print(f"Home directory: {os.path.expanduser('~')}")
```

```bash
uv run hello.py
```

### Scripts in Projects

When in a project directory with `pyproject.toml`:

```bash
# Run script with project dependencies available
uv run script.py

# Run script without installing project
uv run --no-project script.py

# Run with additional dependencies
uv run --with rich script.py
```

### Specifying Python Version

```bash
# Use specific Python version
uv run --python 3.11 script.py
uv run --python pypy@3.10 script.py

# Use latest Python
uv run --python 3 script.py
```

## Script Dependencies

### Inline Dependency Metadata (PEP 723)

Declare dependencies directly in script using inline TOML metadata:

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests<3",
#   "rich",
#   "click>=8.0",
# ]
# ///

import requests
from rich.console import Console
import click

@click.command()
def main():
    console = Console()
    resp = requests.get("https://api.github.com/repos/astral-sh/uv")
    console.print(resp.json(), highlight=True)

if __name__ == "__main__":
    main()
```

### Creating Scripts with Metadata

```bash
# Initialize script with inline metadata
uv init --script my-script.py

# Initialize with specific Python version
uv init --script my-script.py --python 3.12
```

**Generated template:**

```python
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///

def main():
    print("Hello from my-script!")

if __name__ == "__main__":
    main()
```

### Adding Dependencies to Scripts

```bash
# Add dependencies
uv add --script my-script.py requests rich

# Add with version constraints
uv add --script my-script.py 'requests>=2.28,<3'

# Add multiple dependencies
uv add --script my-script.py requests rich click
```

**Result in script:**

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests>=2.28,<3",
#   "rich",
#   "click",
# ]
# ///
```

### Removing Dependencies from Scripts

```bash
# Remove dependency
uv remove --script my-script.py requests
```

### Running Scripts with Dependencies

```bash
# uv automatically creates isolated environment
uv run my-script.py

# With shebang, make script executable
chmod +x my-script.py
./my-script.py
```

**Output:**
```
Reading inline script metadata from: my-script.py
Installed 5 packages in 12ms
<Response [200]>
```

### Requesting Dependencies at Runtime

Without inline metadata, use `--with`:

```bash
# Run with temporary dependencies
uv run --with requests --with rich script.py

# With version constraints
uv run --with 'requests>=2.28,<3' script.py

# Multiple dependencies
uv run --with requests --with rich --with click script.py
```

## Tools Management

Tools are command-line applications distributed as Python packages (similar to pipx).

### Running Tools Temporarily (uvx)

```bash
# Run tool without installing (uvx is alias for uv tool run)
uvx ruff check .
uvx black --check .
uvx mypy src/

# With arguments
uvx cowsay "Hello, uv!"

# From specific version
uvx ruff@0.3.0 check .
uvx ruff@latest check .

# From specific package (command ≠ package name)
uvx --from httpie http https://api.github.com
```

### Installing Tools Permanently

```bash
# Install tool
uv tool install ruff
uv tool install black
uv tool install mypy

# Install with extras
uv tool install 'mypy[faster-cache,reports]'

# Install specific version
uv tool install 'ruff>=0.3.0,<0.4.0'
uv tool install ruff==0.3.5

# Install from git
uv tool install git+https://github.com/user/repo
uv tool install git+https://github.com/user/repo@main
uv tool install git+https://github.com/user/repo@v1.0.0

# Install with Git LFS
uv tool install --lfs git+https://github.com/astral-sh/lfs-cowsay
```

### Managing Installed Tools

```bash
# List installed tools
uv tool list

# Show tool details
uv tool dir ruff

# Update tool
uv tool upgrade ruff

# Update all tools
uv tool upgrade --all

# Uninstall tool
uv tool uninstall ruff

# Uninstall all tools
uv tool uninstall --all
```

### Tool Isolation

**Key concept**: Tools are isolated from projects and each other.

```bash
# This will FAIL - tool packages not importable in Python
uv tool install httpie
python -c "import httpie"  # ModuleNotFoundError

# Correct usage
http https://example.com  # Use as command
```

**Tool directories:**
```bash
# Find tool installation directory
uv tool dir

# Find tool binary directory
uv tool dir --bin
```

### Tools with Additional Dependencies

```bash
# Run tool with plugins/extensions
uvx --with mkdocs-material mkdocs serve

# Install tool with additional packages
uv tool install mkdocs --with mkdocs-material

# Multiple additional packages
uvx --with mkdocs-material --with mkdocs-mermaid mkdocs build
```

### Tools with Multiple Executables

Some packages provide multiple commands:

```bash
# Install package (all executables available)
uv tool install httpie

# Now available: http, https, httpie
http https://api.github.com
https --verify=no https://internal-api.company.com
```

**Install related tools together:**

```bash
# Install with executables from additional packages
uv tool install ansible --with-executables-from ansible-core,ansible-lint

# Result: ansible, ansible-playbook, ansible-lint, etc.
```

### Tool Python Version

```bash
# Use specific Python version for tool
uv tool install --python 3.11 black
uvx --python 3.12 ruff check .

# Update tool's Python version
uv tool upgrade --python 3.12 black
```

## Advanced Script Features

### Script Locking

Lock script dependencies for reproducibility:

```bash
# Create lockfile for script
uv lock --script my-script.py

# Creates: my-script.py.lock
```

**Benefits:**
- Reproducible script execution
- Faster subsequent runs
- Explicit dependency versions

**Usage after locking:**

```bash
# uv automatically uses lockfile if present
uv run my-script.py

# Force lock update
uv lock --script my-script.py

# Export locked dependencies
uv export --script my-script.py > requirements.txt
```

### Reproducibility with exclude-newer

Limit packages to versions released before specific date:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests",
#   "rich",
# ]
# [tool.uv]
# exclude-newer = "2024-01-01T00:00:00Z"
# ///
```

**Effect**: Only uses package versions released before 2024-01-01.

### Alternative Package Indexes

Use custom package indexes in scripts:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "my-private-package",
# ]
# [[tool.uv.index]]
# url = "https://custom-pypi.org/simple"
# ///
```

**Or add via command:**

```bash
uv add --script my-script.py --index https://custom-pypi.org/simple my-private-package
```

### Script Shebangs

Make scripts executable without `uv run`:

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = ["requests"]
# ///

import requests
print(requests.get("https://example.com"))
```

```bash
# Make executable
chmod +x script.py

# Run directly
./script.py
```

**Shebang variations:**

```python
# Standard form
#!/usr/bin/env -S uv run --script

# With specific Python
#!/usr/bin/env -S uv run --script --python 3.12

# With additional flags
#!/usr/bin/env -S uv run --script --quiet
```

### GUI Scripts (Windows)

Scripts with `.pyw` extension run with `pythonw` on Windows:

```python
# hello.pyw
from tkinter import Tk, ttk

root = Tk()
root.title("uv GUI Example")
frm = ttk.Frame(root, padding=10)
frm.grid()
ttk.Label(frm, text="Hello from uv!").grid(column=0, row=0)
root.mainloop()
```

```powershell
# Windows: runs without console window
uv run hello.pyw

# With dependencies
uv run --with PyQt5 app.pyw
```

### Scripts with Environment Variables

```python
# /// script
# requires-python = ">=3.12"
# dependencies = ["python-dotenv"]
# ///

import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("API_KEY")
print(f"API Key: {api_key}")
```

```bash
# Run with environment variables
API_KEY=secret uv run script.py

# Or use .env file
echo "API_KEY=secret" > .env
uv run script.py
```

## Tool Usage Patterns

### Development Tools Workflow

```bash
# Formatting
uvx black .
uvx ruff format .

# Linting
uvx ruff check .
uvx ruff check --fix .

# Type checking
uvx mypy src/

# Testing
uvx pytest

# All at once
uvx ruff check . && uvx mypy src/ && uvx pytest
```

### Documentation Generation

```bash
# Serve docs locally
uvx --with mkdocs-material mkdocs serve

# Build docs
uvx --with mkdocs-material mkdocs build

# Sphinx
uvx --with sphinx sphinx-build docs/ build/
```

### Package Management Utilities

```bash
# Check package versions
uvx pipdeptree
uvx pip-audit

# Package statistics
uvx pipinfo requests

# Security scanning
uvx safety scan
```

### Code Quality Tools

```bash
# Multiple formatters
uvx black .
uvx isort .
uvx autopep8 --in-place --recursive .

# Multiple linters
uvx ruff check .
uvx pylint src/
uvx flake8

# Type checkers
uvx mypy src/
uvx pyright
```

### Project Scaffolding

```bash
# Create project structure
uvx cookiecutter gh:cookiecutter/cookiecutter-pypackage

# Create FastAPI project
uvx create-fastapi-app my-api

# Django project
uvx django startproject mysite
```

### Data Science Tools

```bash
# Jupyter notebooks
uvx --with notebook jupyter notebook

# Jupyter Lab
uvx --with jupyterlab jupyter lab

# Data exploration
uvx --with pandas pandas-profiling

# Visualization
uvx --with plotly plotly-dash
```

### Web Servers and REPL Tools

```bash
# HTTP servers
uvx --from httpie http-prompt
python -m http.server  # Standard library

# Python REPL alternatives
uvx ptpython
uvx ipython
uvx bpython
```

## Best Practices

### Scripts

1. **Use inline metadata** for scripts shared with others
2. **Lock scripts** that need reproducibility (`uv lock --script`)
3. **Add shebangs** for scripts in PATH
4. **Set `requires-python`** to specify minimum Python version
5. **Document dependencies** with comments in metadata
6. **Use `--no-project`** when running scripts in project directories to avoid unwanted imports
7. **Version constraints** should be minimal - prefer `>=` over exact pins
8. **Test scripts** with different Python versions using `--python`
9. **Use environment variables** for sensitive data, not hardcoded
10. **Make scripts standalone** - avoid relying on external files

### Tools

1. **Install frequently-used tools** permanently with `uv tool install`
2. **Use `uvx` for one-off commands** to avoid cluttering tool list
3. **Keep tools updated** with `uv tool upgrade --all`
4. **Check tool directory** if experiencing PATH issues
5. **Use `--with` for plugins** instead of installing separately
6. **Specify versions** for critical tools to ensure consistency
7. **Document tool requirements** in project README
8. **Avoid using tools from project dependencies** - install separately
9. **Use `--python` flag** if tool requires specific Python version
10. **Regularly audit tools** with `uv tool list` and remove unused ones

### Performance

1. **Cache is automatic** - uv reuses downloaded packages
2. **Locked scripts start faster** - dependencies resolved once
3. **Tools are isolated** - no dependency conflicts
4. **Use `--quiet` flag** to suppress output for scripts in pipelines
5. **Pre-install tools** in CI/CD for faster builds

### Security

1. **Review inline metadata** before running untrusted scripts
2. **Use `exclude-newer`** for reproducible security posture
3. **Audit tool dependencies** periodically
4. **Prefer official package indexes** - verify custom indexes
5. **Lock scripts from external sources** before committing

## Common Patterns

### Task Runner Script

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "click>=8.0",
#   "rich>=13.0",
# ]
# ///

import click
from rich.console import Console

console = Console()

@click.group()
def cli():
    """Project task runner"""
    pass

@cli.command()
def test():
    """Run tests"""
    console.print("[bold green]Running tests...[/bold green]")
    # ...

@cli.command()
def lint():
    """Run linting"""
    console.print("[bold blue]Linting code...[/bold blue]")
    # ...

if __name__ == "__main__":
    cli()
```

### Data Processing Script

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "pandas>=2.0",
#   "numpy>=1.24",
# ]
# ///

import pandas as pd
import numpy as np
import sys

def process_data(input_file: str, output_file: str):
    df = pd.read_csv(input_file)
    # Processing...
    df.to_csv(output_file, index=False)
    print(f"Processed {len(df)} rows")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: process.py INPUT OUTPUT")
        sys.exit(1)
    process_data(sys.argv[1], sys.argv[2])
```

### API Client Script

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "requests>=2.31",
#   "click>=8.0",
# ]
# ///

import requests
import click
import os

API_KEY = os.getenv("API_KEY")
BASE_URL = "https://api.example.com"

@click.command()
@click.argument("endpoint")
def api_call(endpoint: str):
    """Make API call to ENDPOINT"""
    headers = {"Authorization": f"Bearer {API_KEY}"}
    response = requests.get(f"{BASE_URL}/{endpoint}", headers=headers)
    click.echo(response.json())

if __name__ == "__main__":
    api_call()
```

### Development Tool Wrapper

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///

import subprocess
import sys

def run_tools():
    """Run all development tools"""
    tools = [
        ["uvx", "ruff", "format", "."],
        ["uvx", "ruff", "check", "."],
        ["uvx", "mypy", "src/"],
        ["uvx", "pytest"],
    ]
    
    for tool in tools:
        print(f"Running: {' '.join(tool)}")
        result = subprocess.run(tool)
        if result.returncode != 0:
            print(f"Error running {tool[1]}")
            sys.exit(1)
    
    print("All tools passed!")

if __name__ == "__main__":
    run_tools()
```

## Troubleshooting

### Script Issues

**Error**: `ModuleNotFoundError` when running script
**Solution**: Add dependency to inline metadata with `uv add --script`

**Error**: Script runs slow on first execution
**Solution**: Normal - uv creates environment. Use `uv lock --script` for faster subsequent runs

**Error**: Can't execute script with shebang
**Solution**: Make script executable: `chmod +x script.py`

**Error**: Script uses wrong Python version
**Solution**: Specify in inline metadata or use `uv run --python X.Y`

**Error**: Conflicting dependencies in script
**Solution**: Use constraints or specific versions in dependencies array

### Tool Issues

**Error**: Tool command not found after install
**Solution**: Check `uv tool dir --bin` is in PATH. Run `uv tool update-shell`

**Error**: Tool fails with import error
**Solution**: Tool may require additional dependencies. Use `--with` or `--with-executables-from`

**Error**: Multiple tool versions needed
**Solution**: Tools are global - use `uvx TOOL@VERSION` for specific versions

**Error**: Tool installed but old version
**Solution**: Run `uv tool upgrade TOOL` or `uv tool upgrade --all`

**Error**: Tool installation fails
**Solution**: Check Python version compatibility with `--python` flag

### Performance Issues

**Error**: Slow script startup
**Solution**: Lock script dependencies: `uv lock --script script.py`

**Error**: Tool takes long to start
**Solution**: First run downloads dependencies. Subsequent runs use cache

**Error**: Large cache size
**Solution**: Clean cache: `uv cache clean` (will re-download on next use)

## Integration Examples

### Makefile Integration

```makefile
.PHONY: format lint test

format:
	uvx ruff format .
	uvx black .

lint:
	uvx ruff check .
	uvx mypy src/

test:
	uvx pytest -v

all: format lint test
```

### Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: ruff
        name: ruff
        entry: uvx ruff check --fix
        language: system
        types: [python]
```

### Shell Aliases

```bash
# ~/.bashrc or ~/.zshrc
alias fmt='uvx ruff format . && uvx black .'
alias lint='uvx ruff check .'
alias test='uvx pytest'
alias typecheck='uvx mypy src/'
```

### VS Code Tasks

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Format Code",
      "type": "shell",
      "command": "uvx ruff format ."
    },
    {
      "label": "Lint Code",
      "type": "shell",
      "command": "uvx ruff check ."
    }
  ]
}
```
