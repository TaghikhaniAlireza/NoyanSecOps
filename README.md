# NoyanSecOps - Comprehensive DevSecOps Pipeline

NoyanSecOps is a GitHub Composite Action designed to seamlessly integrate automated security scanning into your CI/CD pipelines. By calling this single action, you deploy a robust suite of industry-standard security tools that scan your repository for secrets, application vulnerabilities, infrastructure misconfigurations, and container compliance.

## Integrated Security Tools

This action orchestrates the following security scanners, running them sequentially and generating standardized reports:

*   **Gitleaks**: Performs deep secret scanning to detect hardcoded passwords, API keys, and tokens in your repository history.
*   **Semgrep**: A lightweight Static Application Security Testing (SAST) tool that identifies bugs and vulnerabilities in your application code.
*   **Trivy (Aqua Security)**: Conducts Software Composition Analysis (SCA) and filesystem scanning to detect CVEs in OS packages and language-specific dependencies.
*   **Checkov (Bridgecrew)**: Scans Infrastructure as Code (IaC) files (Terraform, CloudFormation, Kubernetes, etc.) for security and compliance misconfigurations.
*   **Hadolint**: Parses and lints Dockerfiles to ensure they adhere to best practices and security guidelines.

## Repository Compatibility & SARIF Output

NoyanSecOps generates `.sarif` (Static Analysis Results Interchange Format) files for Semgrep, Trivy, Checkov, and Hadolint. The behavior of these reports depends on your repository's visibility:

### Public Repositories
For public repositories, GitHub provides the Advanced Security features for free. This action automatically uploads the generated SARIF files directly to the GitHub Security tab using the `github/codeql-action/upload-sarif` action. You can view all vulnerabilities directly in the "Security" -> "Code scanning alerts" section of your repository.

### Private Repositories
For private repositories, displaying alerts in the GitHub Security tab requires a GitHub Advanced Security (GHAS) license. 
However, **NoyanSecOps is fully functional on private repositories without a GHAS license**. If GHAS is not enabled, the `upload-sarif` step will be skipped or fail safely, but the action will still upload the SARIF files as standard pipeline **Artifacts**. You can download these artifacts directly from the Actions run summary to review the vulnerabilities locally or ingest them into an external CSPM/SIEM platform.

## Inputs

| Name           | Description                                                                 | Required | Default |
|----------------|-----------------------------------------------------------------------------|----------|---------|
| `github_token` | GitHub token for API access (required for Gitleaks and SARIF upload steps). | Yes      | N/A     |

## Usage

To use NoyanSecOps in your repository, create a workflow file (e.g., `.github/workflows/devsecops.yml`) and include the following configuration. 

**Important:** The calling workflow must perform the code checkout (`actions/checkout`) with `fetch-depth: 0` before invoking NoyanSecOps. It also requires specific permissions to write security events.
```yaml
name: DevSecOps Pipeline

on:
  push:
branches: [ "main", "develop" ]
  pull_request:
branches: [ "main" ]

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  security-scan:
name: Run NoyanSecOps
runs-on: ubuntu-latest
steps:
- name: Checkout Repository
uses: actions/checkout@v4
with:
fetch-depth: 0

- name: Execute NoyanSecOps Pipeline
uses: YOUR_GITHUB_USERNAME/NoyanSecOps@v1
with:
github_token: ${{ secrets.GITHUB_TOKEN }}
```
*(Note: Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username or organization name where this action is published).*

## Artifact Retention

By default, all generated SARIF files are preserved as workflow artifacts for 14 days. You can download `semgrep-report`, `trivy-report`, `checkov-report`, and `hadolint-report` from the GitHub Actions run summary page.

## License

This project is distributed under the MIT License.

