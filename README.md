# NoyanSecOps - Comprehensive DevSecOps Pipeline

NoyanSecOps is a GitHub Composite Action designed to seamlessly integrate automated security scanning into your CI/CD pipelines. By calling this single action, you deploy a robust suite of industry-standard security tools that scan your repository for secrets, application vulnerabilities, infrastructure misconfigurations, and container compliance.

NoyanSecOps provides a configurable security pipeline with built-in validation, caching, configurable policies, automatic security configuration discovery, and standardized SARIF reporting.

## Integrated Security Tools

This action orchestrates the following security scanners, running them sequentially and generating standardized reports:

* **Gitleaks**: Performs deep secret scanning to detect hardcoded passwords, API keys, and tokens in your repository history.
* **Semgrep**: A lightweight Static Application Security Testing (SAST) tool that identifies bugs and vulnerabilities in your application code.
* **Trivy (Aqua Security)**: Conducts Software Composition Analysis (SCA) and filesystem scanning to detect CVEs in OS packages and language-specific dependencies.
* **Checkov (Bridgecrew)**: Scans Infrastructure as Code (IaC) files (Terraform, CloudFormation, Kubernetes, etc.) for security and compliance misconfigurations.
* **Hadolint**: Parses and lints Dockerfiles to ensure they adhere to best practices and security guidelines.

## Key Features

### Configuration Discovery

NoyanSecOps automatically detects supported security configuration files from your repository, while also allowing users to provide custom configuration paths through workflow inputs.

Supported configurations include:

* Semgrep configuration (`.semgrep.yml`, `.semgrep.yaml`)
* Trivy configuration (`trivy.yaml`, `.trivy.yaml`)
* Trivy ignore file (`.trivyignore`)
* Checkov configuration (`.checkov.yml`, `.checkov.yaml`)
* Hadolint configuration (`.hadolint.yml`, `.hadolint.yaml`)

Manual configuration always takes priority over automatic discovery.

### Validation Layer

Before executing security scanners, NoyanSecOps performs a pre-flight validation step to verify:

* Security configuration files exist and are valid paths
* Input values are correctly configured
* Severity levels are supported
* Fail policy configuration is valid
* Required workspace directories are available

### Caching

NoyanSecOps supports caching for:

* Trivy vulnerability database
* Semgrep cache
* Python package dependencies used by security tools

## Security Policy & Fail Behavior

NoyanSecOps provides a configurable security gate through the `fail_policy` input.

Example:

```yaml
fail_policy: true
```

When enabled, the pipeline fails if security findings matching the configured severity policy are detected.

When disabled, findings are reported but pipeline execution continues.

## Repository Compatibility & SARIF Output

NoyanSecOps generates SARIF reports for Semgrep, Trivy, Checkov, and Hadolint.

All reports are stored in:

```text
noyansecops-results/

├── semgrep.sarif
├── trivy.sarif
├── checkov.sarif
└── hadolint.sarif
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `github_token` | GitHub Token for API access, SARIF upload and reporting. | Yes | N/A |
| `fail_policy` | Enable security gate failure. | No | false |
| `severity` | Security severity levels to evaluate. | No | CRITICAL,HIGH |
| `run_gitleaks` | Enable secret scanning. | No | true |
| `run_semgrep` | Enable SAST scanning. | No | true |
| `run_trivy` | Enable SCA scanning. | No | true |
| `run_checkov` | Enable IaC scanning. | No | true |
| `run_hadolint` | Enable Dockerfile scanning. | No | true |

## Usage

```yaml
name: NoyanSecOps

on: [workflow_dispatch, push]

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  test:
    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run NoyanSecOps
        uses: TaghikhaniAlireza/NoyanSecOps@v0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          fail_policy: false
          severity: CRITICAL,HIGH
```

## Versioning

NoyanSecOps follows semantic versioning.

Recommended:

```yaml
uses: TaghikhaniAlireza/NoyanSecOps@v0
```

For strict version control:

```yaml
uses: TaghikhaniAlireza/NoyanSecOps@v0.2.1
```

## Artifact Retention

By default, generated security reports are preserved for 14 days.

Artifact:

```text
noyansecops-security-reports
```

contains:

* Semgrep SARIF report
* Trivy SARIF report
* Checkov SARIF report
* Hadolint SARIF report

Retention can be customized through `retention_days`.

## License

This project is distributed under the MIT License.
