# Integration Guide for uv

Comprehensive guide for integrating uv with Docker, CI/CD systems, development tools, and third-party services.

## Table of Contents

- [Docker Integration](#docker-integration)
- [GitHub Actions](#github-actions)
- [GitLab CI/CD](#gitlab-cicd)
- [Pre-commit](#pre-commit)
- [Ruff Integration](#ruff-integration)
- [IDEs and Editors](#ides-and-editors)
- [Dependency Bots](#dependency-bots)
- [Alternative Indexes](#alternative-indexes)
- [Cloud Platforms](#cloud-platforms)

## Docker Integration

### Official Docker Images

uv provides official images on GitHub Container Registry:

**Base images:**
```dockerfile
# Distroless (minimal)
FROM ghcr.io/astral-sh/uv:latest
FROM ghcr.io/astral-sh/uv:0.10.2

# Alpine-based
FROM ghcr.io/astral-sh/uv:alpine
FROM ghcr.io/astral-sh/uv:alpine3.23

# Debian-based
FROM ghcr.io/astral-sh/uv:debian-slim
FROM ghcr.io/astral-sh/uv:trixie-slim

# With Python pre-installed
FROM ghcr.io/astral-sh/uv:python3.12-slim
FROM ghcr.io/astral-sh/uv:python3.13-alpine
```

### Basic Dockerfile

```dockerfile
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Install dependencies
RUN uv sync --frozen --no-dev

# Copy application code
COPY . .

# Run application
CMD ["uv", "run", "main.py"]
```

### Multi-Stage Build

Optimize image size with multi-stage builds:

```dockerfile
# Build stage
FROM python:3.12-slim AS builder
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /bin/

WORKDIR /app

# Install dependencies (cached layer)
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-project --no-dev

# Install project
COPY . .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Runtime stage
FROM python:3.12-slim
WORKDIR /app

# Copy only the virtual environment
COPY --from=builder /app/.venv /app/.venv

# Copy application code
COPY --from=builder /app .

# Activate virtual environment
ENV PATH="/app/.venv/bin:$PATH"

CMD ["python", "main.py"]
```

### Optimizations

#### Cache Mounts

Use BuildKit cache mounts for faster builds:

```dockerfile
ENV UV_LINK_MODE=copy

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen
```

#### Bytecode Compilation

Compile Python bytecode for faster startup:

```dockerfile
ENV UV_COMPILE_BYTECODE=1

RUN uv sync --frozen --no-dev
```

#### Intermediate Layers

Separate dependency installation from project code:

```dockerfile
# Install dependencies first (cached)
FROM python:3.12-slim AS deps
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project

# Install project
FROM deps AS builder
COPY . .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked

# Final image
FROM python:3.12-slim
COPY --from=builder /app/.venv /app/.venv
ENV PATH="/app/.venv/bin:$PATH"
WORKDIR /app
COPY --from=builder /app .
CMD ["python", "main.py"]
```

#### Workspace Handling

For workspaces, use `--no-install-workspace`:

```dockerfile
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-workspace

COPY . .

RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked
```

### Development Containers

#### Docker Compose for Development

```yaml
# compose.yaml
services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/.venv  # Preserve virtual environment
    environment:
      - UV_PROJECT_ENVIRONMENT=/app/.venv
    develop:
      watch:
        - action: sync
          path: .
          target: /app
          ignore:
            - .venv/
        - action: rebuild
          path: ./pyproject.toml
```

```bash
# Run with watch mode
docker compose watch
```

#### Dev Container

For VS Code Remote Containers:

```json
// .devcontainer/devcontainer.json
{
  "name": "Python uv Dev Container",
  "image": "ghcr.io/astral-sh/uv:python3.12-slim",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "charliermarsh.ruff"
      ]
    }
  },
  "postCreateCommand": "uv sync",
  "remoteUser": "vscode"
}
```

### Docker Best Practices

1. **.dockerignore**: Exclude `.venv`, `__pycache__`, `.git`
2. **Cache mounts**: Use for `/root/.cache/uv`
3. **Multi-stage builds**: Separate build and runtime
4. **Pin versions**: Use specific uv image tags
5. **Compile bytecode**: Set `UV_COMPILE_BYTECODE=1`
6. **Layer caching**: Install deps before copying code
7. **Non-root user**: Run as non-root in production
8. **Health checks**: Add `HEALTHCHECK` instruction

## GitHub Actions

### Setup Action

Use official setup-uv action:

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

### Matrix Testing

Test across multiple Python versions and OSes:

```yaml
jobs:
  test:
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]
        os: [ubuntu-latest, macos-latest, windows-latest]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v6
      
      - name: Install uv
        uses: astral-sh/setup-uv@v7
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Run tests
        run: |
          uv sync --locked --dev
          uv run pytest
```

### Caching

Built-in caching with setup-uv:

```yaml
- name: Install uv with cache
  uses: astral-sh/setup-uv@v7
  with:
    enable-cache: true
```

Manual caching:

```yaml
jobs:
  test:
    env:
      UV_CACHE_DIR: /tmp/.uv-cache
    
    steps:
      - name: Restore cache
        uses: actions/cache@v5
        with:
          path: /tmp/.uv-cache
          key: uv-${{ runner.os }}-${{ hashFiles('uv.lock') }}
          restore-keys: |
            uv-${{ runner.os }}-
      
      - name: Install dependencies
        run: uv sync --locked
      
      - name: Minimize cache
        run: uv cache prune --ci
```

### Publishing to PyPI

Using trusted publishing (no tokens needed):

```yaml
name: Publish

on:
  push:
    tags:
      - v*

jobs:
  publish:
    runs-on: ubuntu-latest
    environment:
      name: pypi
    permissions:
      id-token: write
      contents: read
    
    steps:
      - uses: actions/checkout@v6
      
      - name: Install uv
        uses: astral-sh/setup-uv@v7
      
      - name: Install Python
        run: uv python install 3.12
      
      - name: Build
        run: uv build
      
      - name: Smoke test
        run: |
          uv run --isolated --with dist/*.whl python -c "import my_package"
      
      - name: Publish
        run: uv publish
```

### Private Repositories

Access private repos with PAT:

```yaml
steps:
  - name: Configure Git credentials
    run: |
      echo "${{ secrets.GITHUB_TOKEN }}" | gh auth login --with-token
      gh auth setup-git
  
  - name: Install dependencies
    run: uv sync --locked
```

## GitLab CI/CD

### Basic Pipeline

```yaml
# .gitlab-ci.yml
variables:
  UV_CACHE_DIR: "$CI_PROJECT_DIR/.cache/uv"
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  paths:
    - .cache/uv
    - .cache/pip

before_script:
  - curl -LsSf https://astral.sh/uv/install.sh | sh
  - export PATH="$HOME/.local/bin:$PATH"
  - uv python install

test:
  stage: test
  script:
    - uv sync --locked --dev
    - uv run pytest
```

### Multi-Version Testing

```yaml
test:
  stage: test
  parallel:
    matrix:
      - PYTHON_VERSION: ["3.11", "3.12", "3.13"]
  script:
    - uv python install $PYTHON_VERSION
    - uv sync --locked --python $PYTHON_VERSION
    - uv run pytest
```

### Docker-based Pipeline

```yaml
test:
  image: ghcr.io/astral-sh/uv:python3.12-slim
  script:
    - uv sync --locked --dev
    - uv run pytest

build:
  image: ghcr.io/astral-sh/uv:python3.12-slim
  script:
    - uv build
  artifacts:
    paths:
      - dist/
```

## Pre-commit

Integrate uv with pre-commit hooks:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: uv-lock
        name: uv lock
        entry: uv lock
        language: system
        files: ^(pyproject\.toml|uv\.lock)$
        pass_filenames: false
      
      - id: uv-sync
        name: uv sync
        entry: uv sync
        language: system
        files: ^(pyproject\.toml|uv\.lock)$
        pass_filenames: false
        stages: [post-checkout, post-merge]
  
  # Run tools via uvx
  - repo: local
    hooks:
      - id: ruff-format
        name: ruff format
        entry: uvx ruff format
        language: system
        types: [python]
      
      - id: ruff
        name: ruff check
        entry: uvx ruff check --fix
        language: system
        types: [python]
      
      - id: mypy
        name: mypy
        entry: uvx mypy
        language: system
        types: [python]
```

**Install pre-commit:**
```bash
uvx pre-commit install
uvx pre-commit run --all-files
```

## Ruff Integration

uv and ruff work seamlessly together (both from Astral):

### Project Setup

```bash
# Add ruff as dev dependency
uv add ruff --dev

# Run via uv
uv run ruff check .
uv run ruff format .

# Or install as tool
uv tool install ruff
ruff check .
```

### Configuration

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W"]
ignore = ["E501"]

[tool.ruff.format]
quote-style = "double"
```

### VS Code Integration

```json
// .vscode/settings.json
{
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll": true,
      "source.organizeImports": true
    }
  },
  "ruff.path": ["uvx", "ruff"]
}
```

### CI Integration

```yaml
- name: Lint
  run: uvx ruff check .

- name: Format check
  run: uvx ruff format --check .
```

## IDEs and Editors

### VS Code

**Python extension configuration:**

```json
// .vscode/settings.json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true,
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": [
    "tests"
  ]
}
```

**Tasks:**

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "uv sync",
      "type": "shell",
      "command": "uv sync",
      "problemMatcher": []
    },
    {
      "label": "uv run tests",
      "type": "shell",
      "command": "uv run pytest",
      "group": {
        "kind": "test",
        "isDefault": true
      }
    }
  ]
}
```

### PyCharm/IntelliJ

1. **Set Python interpreter**: Settings → Project → Python Interpreter
2. **Select virtual environment**: Choose `.venv/bin/python`
3. **Run configurations**: Use `uv run` prefix
4. **External tools**: Add uv commands

**External tool example:**
- Name: `uv sync`
- Program: `uv`
- Arguments: `sync`
- Working directory: `$ProjectFileDir$`

### Vim/Neovim

**With CoC:**

```json
// coc-settings.json
{
  "python.pythonPath": ".venv/bin/python",
  "python.venvPath": ".",
  "python.formatting.provider": "ruff",
  "python.linting.enabled": true,
  "python.linting.ruffEnabled": true
}
```

**With vim-test:**

```vim
let test#python#runner = 'pytest'
let test#python#pytest#executable = 'uv run pytest'
```

### Emacs

```elisp
;; .dir-locals.el
((python-mode . ((python-shell-virtualenv-root . ".venv")
                 (python-pytest-executable . "uv run pytest"))))
```

## Dependency Bots

### Dependabot

Configure dependabot to update uv.lock:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**Note**: Dependabot doesn't natively support uv.lock yet. Export requirements.txt:

```bash
uv export --no-hashes > requirements.txt
```

### Renovate

Full uv support:

```json
// renovate.json
{
  "extends": ["config:base"],
  "lockFileMaintenance": {
    "enabled": true,
    "schedule": ["before 5am on monday"]
  },
  "packageRules": [
    {
      "matchManagers": ["uv"],
      "enabled": true
    }
  ]
}
```

## Alternative Indexes

### PyPI Alternatives

**JFrog Artifactory:**

```toml
[[tool.uv.index]]
name = "jfrog"
url = "https://company.jfrog.io/artifactory/api/pypi/pypi/simple"
default = true
```

**Azure Artifacts:**

```bash
export UV_INDEX_URL="https://pkgs.dev.azure.com/org/_packaging/feed/pypi/simple/"
uv pip install package
```

**AWS CodeArtifact:**

```bash
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

uv pip install package
```

### Private PyPI

```toml
[[tool.uv.index]]
name = "private"
url = "https://pypi.company.com/simple"

[tool.uv.sources]
internal-package = { index = "private" }
```

**With authentication:**

```bash
# .netrc
machine pypi.company.com
login username
password token

# Or environment variable
export UV_INDEX_URL="https://user:token@pypi.company.com/simple"
```

## Cloud Platforms

### AWS Lambda

**Dockerfile for Lambda:**

```dockerfile
FROM public.ecr.aws/lambda/python:3.12

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR ${LAMBDA_TASK_ROOT}

COPY pyproject.toml uv.lock ./

RUN uv sync --frozen --no-dev --no-install-project

COPY . .

RUN uv sync --frozen --no-dev

ENV PATH="${LAMBDA_TASK_ROOT}/.venv/bin:$PATH"

CMD ["app.handler"]
```

### Google Cloud Run

```dockerfile
FROM python:3.12-slim

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

COPY . .

ENV PATH="/app/.venv/bin:$PATH"
ENV PORT=8080

CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 app:app
```

### Azure Container Apps

```dockerfile
FROM python:3.12-slim

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --compile-bytecode

COPY . .

EXPOSE 80
ENV PATH="/app/.venv/bin:$PATH"

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "80"]
```

### Heroku

```bash
# Use heroku-python-uv buildpack
heroku buildpacks:set https://github.com/heroku/heroku-buildpack-python
```

**Procfile:**
```
web: uv run gunicorn app:app
worker: uv run celery worker
```

### Railway

```toml
# railway.toml
[build]
command = "curl -LsSf https://astral.sh/uv/install.sh | sh && source $HOME/.local/bin && uv sync --frozen"

[deploy]
startCommand = "uv run python app.py"
```

## Testing Frameworks

### pytest

```bash
# Add pytest
uv add pytest --dev

# Run tests
uv run pytest

# With coverage
uv add pytest-cov --dev
uv run pytest --cov=src tests/
```

### tox

While uv replaces much of tox's functionality, you can still integrate:

```ini
# tox.ini
[tox]
env_list = py311,py312,py313

[testenv]
commands =
    uv sync --frozen
    uv run pytest
```

### nox

```python
# noxfile.py
import nox

@nox.session(python=["3.11", "3.12", "3.13"])
def tests(session):
    session.run("uv", "sync", "--frozen", external=True)
    session.run("uv", "run", "pytest", external=True)
```

## Documentation Tools

### MkDocs

```bash
# Install as tool
uv tool install 'mkdocs[material]'

# Serve docs
mkdocs serve

# Or run temporarily
uvx --with mkdocs-material mkdocs serve
```

### Sphinx

```bash
# Add to project
uv add sphinx --optional docs

# Build docs
uv run --extra docs sphinx-build docs build
```

## Best Practices

1. **Pin uv version** in CI/CD for reproducibility
2. **Use caching** in all CI systems
3. **Cache prune** at end of CI jobs: `uv cache prune --ci`
4. **Lock before commit**: Ensure `uv.lock` is up-to-date
5. **Test matrix**: Cover all supported Python versions
6. **Docker layer optimization**: Install deps before code
7. **Use official images**: Prefer `ghcr.io/astral-sh/uv` images
8. **Environment variables**: Use for sensitive credentials
9. **Validate in CI**: Run `uv sync --locked` to check lock freshness
10. **Document setup**: Include uv installation in README

## Common Integration Patterns

### Makefile

```makefile
.PHONY: install dev test lint format clean

install:
	uv sync --frozen --no-dev

dev:
	uv sync --frozen

test:
	uv run pytest

lint:
	uvx ruff check .
	uvx mypy src/

format:
	uvx ruff format .

clean:
	rm -rf .venv dist/ *.egg-info
	find . -type d -name __pycache__ -exec rm -rf {} +
```

### Package.json (for mixed projects)

```json
{
  "scripts": {
    "install": "uv sync",
    "test": "uv run pytest",
    "lint": "uvx ruff check .",
    "format": "uvx ruff format ."
  }
}
```

### Justfile

```just
# Install dependencies
install:
    uv sync --frozen

# Run tests
test:
    uv run pytest

# Lint code
lint:
    uvx ruff check .
    uvx mypy src/

# Format code
format:
    uvx ruff format .

# Run development server
dev:
    uv run python -m app
```

## Troubleshooting

### Docker Issues

**Problem**: Cache not working
**Solution**: Ensure BuildKit enabled: `export DOCKER_BUILDKIT=1`

**Problem**: Slow Docker builds
**Solution**: Use multi-stage builds and cache mounts

### CI/CD Issues

**Problem**: CI runs out of disk space
**Solution**: Run `uv cache prune --ci` at end of jobs

**Problem**: Inconsistent CI results
**Solution**: Use `--locked` or `--frozen` flags

### Authentication Issues

**Problem**: Can't access private packages
**Solution**: Use credentials via environment variables or netrc

**Problem**: Token expires in CI
**Solution**: Use trusted publishing or refresh tokens

### Performance Issues

**Problem**: Slow dependency resolution
**Solution**: Use lockfile, enable caching

**Problem**: Large Docker images
**Solution**: Use multi-stage builds, slim base images

## Additional Resources

- **Docker examples**: https://github.com/astral-sh/uv-docker-example
- **GitHub Actions setup**: https://github.com/astral-sh/setup-uv
- **Integration guides**: https://docs.astral.sh/uv/guides/integration/
- **Trusted publishing**: https://github.com/astral-sh/trusted-publishing-examples
