<div align="center">

  <h1>devops-pipeline-templates</h1>
  <p><strong>Reusable DevSecOps GitHub Actions Workflows for Enterprise CI/CD Standardization</strong></p>

  <p>
    <img src="https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white" alt="GitHub Actions" />
    <img src="https://img.shields.io/badge/Security_Scan-Aquasec_Trivy-00AEEF?style=flat-square&logo=trivy&logoColor=white" alt="Trivy" />
    <img src="https://img.shields.io/badge/Containers-Docker_Buildx-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker" />
    <img src="https://img.shields.io/badge/IaC_Quality-Terraform_Lint-7B42BC?style=flat-square&logo=terraform&logoColor=white" alt="Terraform" />
    <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" alt="License" />
  </p>

</div>

---

### Overview

`devops-pipeline-templates` is a collection of production-hardened, reusable GitHub Actions workflows designed to eliminate CI/CD pipeline duplication, enforce container security scanning, and automate GitOps releases across organizations.

By referencing these central workflows via `workflow_call`, engineering teams can enforce security gates (such as failing builds on `CRITICAL` CVEs) without maintaining ad-hoc CI scripts.

---

### Workflow Catalog

#### 1. Docker Buildx & Security Scan (`docker-build-scan.yml`)
- **Engine:** Docker Buildx with GitHub Actions caching (`cache-from/to: type=gha`).
- **DevSecOps Gate:** Integrated Aquasec Trivy vulnerability scanner.
- **Reporting:** Exports SARIF vulnerability reports directly into the GitHub Security tab.
- **Publishing:** Publishes container images only if vulnerability criteria pass.

```yaml
jobs:
  build:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/docker-build-scan.yml@main
    with:
      image_name: ghcr.io/myorg/microservice
      image_tag: ${{ github.sha }}
      fail_on_vulnerabilities: true
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

#### 2. Terraform Quality & Lint (`terraform-quality.yml`)
- **Checks:** `terraform fmt -check -recursive` and `terraform validate`.
- **Backend Bypass:** Runs `terraform init -backend=false` for fast validation without requiring cloud state access.

```yaml
jobs:
  validate-iac:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/terraform-quality.yml@main
    with:
      working_directory: environments/prod
      terraform_version: 1.7.0
```

#### 3. GitOps Synchronizer (`gitops-sync.yml`)
- **Mechanism:** Automatically clones the target GitOps repository (e.g. `k8s-gitops-catalog`), replaces the container image tag with the freshly built SHA, and creates a signed commit back to trigger ArgoCD reconciliation.

```yaml
jobs:
  sync:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/gitops-sync.yml@main
    with:
      manifest_repo: myorg/k8s-gitops-catalog
      deployment_file: apps/api/deployment.yaml
      new_tag: ${{ github.sha }}
    secrets:
      GITOPS_PUSH_TOKEN: ${{ secrets.PAT_GITOPS_SYNC }}
```

---

### Repository Layout

```text
devops-pipeline-templates/
├── .github/
│   └── workflows/
│       ├── docker-build-scan.yml     # Reusable container build + Trivy scanner
│       ├── gitops-sync.yml           # Reusable manifest image tag updater
│       └── terraform-quality.yml     # Reusable IaC format & syntax validator
├── examples/
│   └── caller-workflow.yml          # End-to-end integration sample
└── README.md
```

---

### Security Policy & Vulnerability Gates

| Severity | Default Action | Overridable |
| :--- | :--- | :--- |
| `CRITICAL` | Fails pipeline with exit code 1 | Yes (`fail_on_vulnerabilities: false`) |
| `HIGH` | Fails pipeline with exit code 1 | Yes (`fail_on_vulnerabilities: false`) |
| `MEDIUM` | Logged and reported in SARIF | Informational |
| `LOW` | Logged and reported in SARIF | Informational |

---

### License

Distributed under the MIT License. Developed and maintained by [Brandon Mendieta](https://github.com/NeoScraids).
