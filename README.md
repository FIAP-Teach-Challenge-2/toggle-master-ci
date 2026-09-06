# toggle-master-ci

Reusable GitHub Actions CI/DevSecOps workflows shared by the 5 ToggleMaster
microservices (`auth-service`, `flag-service`, `targeting-service`,
`evaluation-service`, `analytics-service`).

## Workflows

### `reusable-ci-go.yml`

For Go services. Jobs: lint (golangci-lint) → test (go build + go test) →
security-scan (gosec + Trivy fs) → docker-build-push (Trivy image scan,
push to ECR on `main` only).

Usage from a service repo's `.github/workflows/ci.yml`:

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

For Python/Flask services. Same 4 jobs, Python tooling (flake8, bandit,
pytest). Extra input `needs-postgres` (boolean, default `false`) loads
`db/init.sql` into a Postgres service container before running tests —
set it to `true` for services whose app connects to Postgres at import
time (`flag-service`, `targeting-service`).

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

## Required secrets (set on each calling repo)

`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` — AWS
Academy temporary credentials. Must be refreshed whenever the Academy lab
session restarts. See `COORDENACAO.md` for who needs to set these.
