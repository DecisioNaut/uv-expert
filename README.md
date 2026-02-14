# uv-expert: Agent Skill for Python Package Management

Comprehensive AI agent skill for mastering **uv**, the extremely fast Python package and project manager written in Rust.

## Overview

This skill provides deep expertise in uv, covering everything from basic project management to advanced resolution strategies, Docker integration, CI/CD workflows, and complex authentication scenarios. Designed for AI agents to assist with sophisticated Python development workflows.

## What is uv?

**uv** is a blazingly fast Python package and project manager that replaces:
- **pip** and **pip-tools** (10-100x faster)
- **pipx** (via `uvx`)
- **poetry** and **pdm** (project management)
- **pyenv** (Python version management)
- **virtualenv** (virtual environment creation)

Key features:
- ⚡ 10-100x faster than pip
- 🔒 Universal lockfiles with cross-platform support
- 🐍 Python version management built-in
- 📦 Workspace support for monorepos
- 🐳 Docker and CI/CD optimized
- 🔄 Fully pip-compatible interface

## Installation

This skill can be used by AI agents that support the Agent Skills specification.

### For GitHub Copilot

1. Copy this directory to your skills folder:
   ```bash
   cp -r uv-expert ~/.copilot/skills/
   ```

2. The skill will be automatically detected when discussing Python packaging, uv, or related topics.

### For Other AI Agents

Ensure your agent can read:
- `SKILL.md` - Main skill file with quick reference
- `references/` - Detailed documentation on specific topics

## Skill Structure

```
uv-expert/
├── SKILL.md                           # Main entry point (<500 lines)
├── README.md                          # This file
├── LICENSE                            # MIT License
├── CHANGELOG.md                       # Version history
└── references/                        # Detailed documentation
    ├── PROJECTS.md                    # Project management & workspaces
    ├── SCRIPTS_TOOLS.md              # Scripts with inline deps & CLI tools
    ├── PYTHON_MANAGEMENT.md          # Installing & managing Python versions
    ├── PIP_INTERFACE.md              # pip compatibility & migration
    ├── INTEGRATIONS.md               # Docker, CI/CD, pre-commit, IDEs
    ├── RESOLUTION.md                 # Resolution strategies, overrides, constraints
    ├── AUTHENTICATION.md             # HTTP, Git, third-party auth
    ├── CACHING_PERFORMANCE.md        # Caching & performance tuning
    ├── PLATFORM_INDEXES.md           # Platform deps & custom indexes
    ├── DEBUG_TROUBLESHOOTING.md      # Debug tools & issue resolution
    └── INTERNALS.md                  # Internals, best practices, patterns
```

## When to Use This Skill

The agent should invoke this skill when discussions involve:

- **Python package management**: installing packages, managing dependencies
- **Project setup**: creating Python projects, configuring `pyproject.toml`
- **Virtual environments**: creating isolated Python environments
- **Dependency resolution**: solving version conflicts, understanding lockfiles
- **Python version management**: installing or switching Python versions
- **Scripts**: running Python scripts with inline dependencies (PEP 723)
- **CLI tools**: installing and running command-line tools like `ruff`, `black`
- **Monorepos**: managing multiple Python packages in one repository
- **Docker**: containerizing Python applications
- **CI/CD**: setting up GitHub Actions, GitLab CI, etc.
- **Migration**: moving from pip, poetry, pipenv, or conda to uv
- **Performance optimization**: speeding up Python package operations
- **Authentication**: accessing private PyPI servers or Git repositories

## Coverage

### Core Topics (SKILL.md)

- Installation and setup
- Quick start guide
- Common workflows (projects, scripts, tools)
- Command reference
- Best practices
- Troubleshooting basics

### Detailed Topics (references/)

#### PROJECTS.md
- Creating and structuring projects
- Managing dependencies and dependency groups
- Understanding and working with lockfiles (`uv.lock`)
- Workspace management for monorepos
- Building and publishing packages
- Resolution strategies and overrides
- Platform-specific dependencies

#### SCRIPTS_TOOLS.md
- Running standalone scripts with `uv run`
- PEP 723 inline dependency metadata
- Installing CLI tools with `uv tool` / `uvx`
- Tool isolation and management
- Script locking for reproducibility
- Development tooling patterns

#### PYTHON_MANAGEMENT.md
- Installing Python versions with uv
- Managing multiple Python installations
- Python discovery and preference
- Pinning Python versions (`.python-version`)
- CI/CD Python setup
- Alternative implementations (PyPy, GraalPy)

#### PIP_INTERFACE.md
- Drop-in pip replacement commands
- Migrating from pip and pip-tools
- Virtual environment management
- Requirements file compilation
- Environment syncing
- pip-tools compatibility

#### INTEGRATIONS.md
- Docker: images, multi-stage builds, optimizations
- GitHub Actions: setup-uv, caching, matrix testing
- GitLab CI/CD pipelines
- Pre-commit hooks integration
- Ruff and code quality tools
- IDE configuration (VS Code, PyCharm)
- Dependency bots (Dependabot, Renovate)
- Cloud platforms (AWS Lambda, GCP, Azure)

#### RESOLUTION.md
- Resolution strategies (highest, lowest, lowest-direct)
- Dependency overrides and constraints
- Per-dependency resolution control

#### AUTHENTICATION.md
- HTTP authentication (netrc, environment variables)
- Git authentication (SSH, HTTPS tokens)
- Third-party services (JFrog, Azure, Google)
- TLS/SSL certificates and security

#### CACHING_PERFORMANCE.md
- Cache location and structure
- Cache management and CI/CD optimization
- Link modes and bytecode compilation
- Network tuning and concurrent downloads

#### PLATFORM_INDEXES.md
- Platform-specific dependencies
- Environment markers
- Custom package indexes
- Multi-platform lockfiles

#### DEBUG_TROUBLESHOOTING.md
- Verbose output and dry runs
- Resolution tree and dependency inspection
- Common problems and solutions
- Virtual environment troubleshooting

#### INTERNALS.md
- PubGrub resolver algorithm
- Lockfile format and structure
- Best practices (development, production, CI/CD)
- Common patterns (monorepos, microservices)

## Quick Examples

### Creating a New Project

```bash
uv init my-project
cd my-project
uv add requests pandas
uv run python main.py
```

### Running a Script with Dependencies

```python
# script.py
# /// script
# dependencies = ["httpx", "rich"]
# ///

import httpx
from rich import print

response = httpx.get("https://api.github.com")
print(response.json())
```

```bash
uv run script.py
```

### Installing a Tool

```bash
# Temporary run
uvx ruff check .

# Permanent installation
uv tool install ruff
ruff check .
```

### Docker Deployment

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

COPY . .
CMD ["uv", "run", "main.py"]
```

## Skill Metadata

- **Name**: uv-expert
- **Version**: 1.0.0
- **License**: MIT
- **Created**: 2026
- **uv Version Coverage**: 0.5.x - 0.10.x
- **Skill Level**: Level 2 (Main file + references)

## Contributing

This skill is generated following the Agent Skills specification. To improve:

1. **Update documentation**: Modify `SKILL.md` or reference files
2. **Add examples**: Include real-world use cases
3. **Cover new features**: Update when uv releases new versions
4. **Improve clarity**: Enhance explanations based on common questions

## Resources

- **uv Documentation**: https://docs.astral.sh/uv/
- **uv GitHub Repository**: https://github.com/astral-sh/uv
- **Astral (creators)**: https://astral.sh/
- **Agent Skills Spec**: https://github.com/astral-sh/agent-skills

## License

MIT License - See LICENSE file for details.

## Acknowledgments

- **Astral** for creating uv and advancing Python tooling
- **Agent Skills** community for the specification framework
- Documentation sourced from official uv docs and GitHub repository

---

**Note for AI Agents**: When using this skill, start with `SKILL.md` for overview and quick reference. Dive into specific reference files only when users need detailed information on particular topics. Use progressive disclosure to avoid overwhelming users with too much information at once.
