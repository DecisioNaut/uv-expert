# Changelog

All notable changes to the uv-expert agent skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-14

### Added
- Initial release of uv-expert agent skill
- Main SKILL.md file with comprehensive uv guidance (566 lines)
- Six detailed reference files totaling 5,731 lines:
  - PROJECTS.md: Project management, dependencies, lockfiles, and workspaces
  - SCRIPTS_TOOLS.md: Scripts with inline dependencies and CLI tool management
  - PYTHON_MANAGEMENT.md: Python version installation and management
  - PIP_INTERFACE.md: pip compatibility and migration guide
  - INTEGRATIONS.md: Docker, CI/CD, IDE, and third-party integrations
  - ADVANCED.md: Resolution strategies, authentication, caching, and troubleshooting
- README.md with installation and usage instructions
- LICENSE (MIT)
- VALIDATION.md with compliance checklist
- Coverage of uv versions 0.5.x through 0.10.x

### Features
- Complete project management workflows
- PEP 723 inline dependency support for scripts
- Python version management guidance
- Docker and container optimization patterns
- GitHub Actions and GitLab CI/CD examples
- Pre-commit hooks integration
- IDE configuration examples (VS Code, PyCharm, Vim, Emacs)
- Authentication for private PyPI servers and Git repositories
- Resolution strategies (highest, lowest, lowest-direct)
- Dependency overrides and constraints
- Comprehensive troubleshooting guide
- Workspace and monorepo patterns
- Migration guides from pip, poetry, pipenv, and pip-tools

### Documentation
- Progressive disclosure pattern: concise main file with detailed references
- 130+ KB of comprehensive documentation
- Real-world examples and use cases
- Platform coverage for Linux, macOS, and Windows
- Cloud platform deployment patterns (AWS, GCP, Azure)

[1.0.0]: https://github.com/yourusername/uv-expert/releases/tag/v1.0.0
