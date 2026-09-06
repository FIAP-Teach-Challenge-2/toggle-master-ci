# toggle-master-ci

Workflows reutilizáveis do GitHub Actions para CI/DevSecOps, compartilhados
pelos 5 microsserviços do ToggleMaster (`auth-service`, `flag-service`,
`targeting-service`, `evaluation-service`, `analytics-service`).

## Workflows

### `reusable-ci-go.yml`

Para serviços em Go. Jobs: lint (golangci-lint) → test (go build + go
test) → security-scan (gosec + Trivy fs) → docker-build-push (scan de
imagem com Trivy, push para o ECR apenas na `main`).

Uso a partir do `.github/workflows/ci.yml` de um repositório de serviço:

```yaml
name: CI Pipeline - <service-name>

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: FIAP-Teach-Challenge-2/toggle-master-ci/.github/workflows/reusable-ci-go.yml@main
    with:
      service-name: <service-name>
      ecr-repository: togglemaster/<service-name>
    secrets: inherit
```

### `reusable-ci-python.yml`

Para serviços Python/Flask. Os mesmos 4 jobs, com ferramentas Python
(flake8, bandit, pytest). Input extra `needs-postgres` (boolean, padrão
`false`) carrega o `db/init.sql` em um container de serviço Postgres antes
de rodar os testes — defina como `true` para serviços cuja aplicação se
conecta ao Postgres na importação do módulo (`flag-service`,
`targeting-service`).

```yaml
jobs:
  ci:
    uses: FIAP-Teach-Challenge-2/toggle-master-ci/.github/workflows/reusable-ci-python.yml@main
    with:
      service-name: <service-name>
      ecr-repository: togglemaster/<service-name>
      needs-postgres: true
    secrets: inherit
```

## Secrets necessários (configurados em cada repositório que chama o workflow)

`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` —
credenciais temporárias da AWS Academy. Precisam ser atualizadas sempre
que a sessão do Lab Academy reiniciar. Veja `COORDENACAO.md` (adicionado
em uma tarefa posterior) para saber quem precisa configurar esses secrets.
