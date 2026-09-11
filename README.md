# devops-pipeline-templates

Workflows reutilizables de GitHub Actions que uso como base en distintos proyectos. La idea es no reescribir el pipeline en cada repo: lo llamas con `workflow_call` y listo.

## Workflows disponibles

### `docker-build-scan.yml`
Construye la imagen con Docker Buildx (con cache de GitHub Actions), corre Trivy para escanear vulnerabilidades y si encuentra criticas o altas falla el pipeline. Si pasa, publica en el registry.

```yaml
jobs:
  build:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/docker-build-scan.yml@main
    with:
      image_name: ghcr.io/mi-org/microservicio
      image_tag: ${{ github.sha }}
      fail_on_vulnerabilities: true
    secrets:
      REGISTRY_USERNAME: ${{ github.actor }}
      REGISTRY_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
```

### `terraform-quality.yml`
Corre `terraform fmt -check` y `terraform validate` sin necesitar backend remoto (usa `-backend=false`).

```yaml
jobs:
  validate:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/terraform-quality.yml@main
    with:
      working_directory: environments/prod
      terraform_version: 1.7.0
```

### `gitops-sync.yml`
Despues de buildear una imagen nueva, clona el repo de manifiestos K8s, actualiza el tag de la imagen y hace commit. ArgoCD detecta el cambio y reconcilia.

```yaml
jobs:
  sync:
    uses: NeoScraids/devops-pipeline-templates/.github/workflows/gitops-sync.yml@main
    with:
      manifest_repo: mi-org/k8s-gitops-catalog
      deployment_file: apps/api/deployment.yaml
      new_tag: ${{ github.sha }}
    secrets:
      GITOPS_PUSH_TOKEN: ${{ secrets.PAT_GITOPS_SYNC }}
```

## Estructura

```
.github/workflows/
  docker-build-scan.yml
  terraform-quality.yml
  gitops-sync.yml
examples/
  caller-workflow.yml    # Ejemplo de como llamar los 3 workflows
```

## Licencia

MIT
