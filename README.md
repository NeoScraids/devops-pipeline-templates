<div align="center">

  <h1>devops-pipeline-templates</h1>
  <p><strong>Plantillas Reutilizables de CI/CD para GitHub Actions con Enfoque DevSecOps</strong></p>

  <p>
    <img src="https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white" alt="GitHub Actions" />
    <img src="https://img.shields.io/badge/Escaneo_de_Seguridad-Aquasec_Trivy-00AEEF?style=flat-square&logo=trivy&logoColor=white" alt="Trivy" />
    <img src="https://img.shields.io/badge/Contenedores-Docker_Buildx-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker" />
    <img src="https://img.shields.io/badge/Calidad_IaC-Terraform_Lint-7B42BC?style=flat-square&logo=terraform&logoColor=white" alt="Terraform" />
    <img src="https://img.shields.io/badge/Licencia-MIT-blue?style=flat-square" alt="Licencia" />
  </p>

</div>

---

### Descripción General

`devops-pipeline-templates` es un catálogo de flujos de trabajo reutilizables de **GitHub Actions** diseñados para estandarizar el ciclo de entrega continua (CI/CD), eliminar código duplicado y aplicar políticas estrictas de seguridad de contenedores (**DevSecOps**) en proyectos empresariales.

Al consumir estos flujos mediante `workflow_call`, los equipos de desarrollo e infraestructura garantizan barreras automáticas de calidad (como el bloqueo de compilaciones ante vulnerabilidades críticas de seguridad) sin necesidad de reescribir pipelines en cada repositorio.

---

### Catálogo de Flujos de Trabajo

#### 1. Construcción Docker y Escaneo de Vulnerabilidades (`docker-build-scan.yml`)
- **Motor:** Docker Buildx con almacenamiento de caché multicapa en GitHub Actions (`cache-from/to: type=gha`).
- **Control DevSecOps:** Integración nativa con el escáner de vulnerabilidades **Aquasec Trivy**.
- **Reportes:** Exportación de reportes SARIF directos a la pestaña *Security* del repositorio en GitHub.
- **Publicación:** Publica la imagen en el registro de contenedores únicamente si supera el umbral de seguridad.

```yaml
jobs:
  build:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/docker-build-scan.yml@main
    with:
      image_name: ghcr.io/mi-organizacion/microservicio
      image_tag: ${{ github.sha }}
      fail_on_vulnerabilities: true
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

#### 2. Calidad y Validación de Terraform (`terraform-quality.yml`)
- **Verificaciones:** Formateo canónico con `terraform fmt -check -recursive` y validación sintáctica con `terraform validate`.
- **Modo Aislado:** Ejecuta `terraform init -backend=false` para realizar validaciones rápidas sin requerir acceso al backend remoto.

```yaml
jobs:
  validate-iac:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/terraform-quality.yml@main
    with:
      working_directory: environments/prod
      terraform_version: 1.7.0
```

#### 3. Sincronización GitOps (`gitops-sync.yml`)
- **Mecanismo:** Clona automáticamente el repositorio de manifiestos GitOps (ej. `k8s-gitops-catalog`), actualiza la etiqueta de la imagen con el nuevo SHA generado y realiza el commit para activar la reconciliación de ArgoCD.

```yaml
jobs:
  sync:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/gitops-sync.yml@main
    with:
      manifest_repo: mi-organizacion/k8s-gitops-catalog
      deployment_file: apps/api/deployment.yaml
      new_tag: ${{ github.sha }}
    secrets:
      GITOPS_PUSH_TOKEN: ${{ secrets.PAT_GITOPS_SYNC }}
```

---

### Estructura del Repositorio

```text
devops-pipeline-templates/
├── .github/
│   └── workflows/
│       ├── docker-build-scan.yml     # Construcción de contenedor + escáner Trivy
│       ├── gitops-sync.yml           # Actualizador de etiquetas en catálogo GitOps
│       └── terraform-quality.yml     # Validador de formato y sintaxis para Terraform
├── examples/
│   └── caller-workflow.yml          # Ejemplo de integración de extremo a extremo
└── README.md
```

---

### Políticas de Severidad y Barreras de Seguridad

| Severidad | Acción por Defecto | Modificable |
| :--- | :--- | :--- |
| `CRITICAL` | Falla el pipeline con código de salida 1 | Sí (`fail_on_vulnerabilities: false`) |
| `HIGH` | Falla el pipeline con código de salida 1 | Sí (`fail_on_vulnerabilities: false`) |
| `MEDIUM` | Registrado y reportado en SARIF | Informativo |
| `LOW` | Registrado y reportado en SARIF | Informativo |

---

### Licencia

Distribuido bajo la Licencia MIT. Desarrollado y mantenido por [Brandon Mendieta](https://github.com/NeoScraids).
