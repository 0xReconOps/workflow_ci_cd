# DevSecOps Methodology - CI/CD Pipeline

This document presents the **DevSecOps** practices implemented in our CI/CD pipelines with GitHub Actions.

---

## 1. Infrastructure and Execution Security

### Least Privilege (GitHub Actions Permissions)
*   **Action**: We see in the pipeline `permissions: contents: read`.
*   **Purpose**: Limits the temporary authentication token (`GITHUB_TOKEN`) to read-only access on the repository. This prevents malicious code or release tampering if a third-party tool used in the jobs is compromised by an attacker.

### Dependency Pinning by Commit Hash (SHA)
*   **Action**: Usage of unique identifiers (e.g., `actions/checkout@3d3c42e...`) instead of mutable version tags like `@7.0.1`.
*   **Purpose**: Prevents *Supply Chain* attacks. If an official action version is compromised and altered remotely on GitHub, the pipeline remains secure because it only executes code matching that exact cryptographic fingerprint.

---

## 2. Static Application Security Testing (SAST)

The `test` job integrates two levels of static validation to catch issues as early as possible (*Shift Left Security*).

### Syntax Validation (Linting)
*   **Action**: Execution of `htmlhint` and `node --check`.
*   **Purpose**: Detects structural errors, bad coding practices, and first-level potential vulnerabilities before compiling the application.

### Code Analysis with SonarQube
*   **Action**: Integration of `sonarqube-scan-action` couplée à un `Quality Gate`.
*   **Purpose**: Actively scans for complex security vulnerabilities (OWASP Top 10), code smells (technical debt), and bugs. The pipeline is configured to immediately block the process via the `Quality Gate` if the source code fails to meet the safety and quality criteria set by the organization.

---

## 3. Software Supply Chain Security (SCA)

### Trivy Scan (Aqua Security)
*   **Action**: Execution of the `check_security` job using the `Trivy` tool.
*   **Purpose**: Scans the file system (`scan-type: 'fs'`) to look for vulnerable dependencies or accidentally exposed secrets.
*   **Strict Control**: Setting `exit-code: '1'` combined with `severity: 'CRITICAL,HIGH'` guarantees that the pipeline breaks if a high or critical vulnerability is found. The application cannot be deployed until the flaw is fixed.

---

## 4. Dynamic Application Security Testing (DAST)

### OWASP ZAP Baseline Scan
*   **Action**: Temporary deployment of the application in a Docker container, followed by the `owasp_zap_scan` job.
*   **Purpose**: Tests the running application (*Dynamic Application Security Testing*) from an outside perspective, simulating an attacker's behavior. It identifies network configuration flaws, missing security headers, and deployment-related runtime vulnerabilities.
*   **Evidence Collection**: The generated HTML security report is automatically archived via GitHub artifacts for audit by security teams.
