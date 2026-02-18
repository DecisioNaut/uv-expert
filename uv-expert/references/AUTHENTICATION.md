# Authentication for uv

Comprehensive guide to authentication with package indexes, Git repositories, and third-party services.

## Table of Contents

- [HTTP Authentication](#http-authentication)
- [Git Authentication](#git-authentication)
- [Third-Party Services](#third-party-services)
- [TLS/SSL Certificates](#tlsssl-certificates)
- [Credential Security](#credential-security)

## HTTP Authentication

### netrc File

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

### Environment Variables

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

### Configuration File

```toml
# pyproject.toml
[[tool.uv.index]]
name = "private"
url = "https://${PRIVATE_PYPI_USER}:${PRIVATE_PYPI_TOKEN}@pypi.example.com/simple"
```

**Note:** Environment variables substitution supported in .env files.

## Git Authentication

### SSH Keys

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

### HTTPS with Tokens

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

### GitLab CI Token

```yaml
# .gitlab-ci.yml
variables:
  GIT_STRATEGY: clone
  
before_script:
  - git config --global url."https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.com/".insteadOf "https://gitlab.com/"
  - uv sync
```

## Third-Party Services

### JFrog Artifactory

```bash
# API key
export UV_INDEX_URL="https://username:${ARTIFACTORY_API_KEY}@company.jfrog.io/artifactory/api/pypi/pypi/simple"

# Auth via config
curl -u username:${ARTIFACTORY_API_KEY} \
  https://company.jfrog.io/artifactory/api/pypi/pypi/simple
```

### Azure Artifacts

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

### Google Artifact Registry

```bash
# gcloud auth
gcloud auth print-access-token | uv pip install \
  --index-url https://oauth2accesstoken:$(gcloud auth print-access-token)@python.pkg.dev/project/repository/simple \
  package
```

## TLS/SSL Certificates

### Custom CA Certificates

```bash
# System CA bundle
export UV_CA_BUNDLE="/etc/ssl/certs/ca-certificates.crt"

# Custom certificate
export UV_CA_BUNDLE="/path/to/custom-ca-bundle.crt"

# Combine with system certificates
cat /etc/ssl/certs/ca-certificates.crt custom-ca.crt > combined-ca-bundle.crt
export UV_CA_BUNDLE="combined-ca-bundle.crt"
```

### Client Certificates

```bash
export UV_CLIENT_CERT="/path/to/client.pem"
export UV_CLIENT_KEY="/path/to/client-key.pem"
```

### Disable Verification (Not Recommended)

```bash
# Only for testing/debugging
export UV_NO_VERIFY_SSL=1
```

## Credential Security

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
