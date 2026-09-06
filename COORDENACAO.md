# Coordenação — Pipeline de CI (Juliette)

Este documento explica como o pipeline de CI (responsabilidade da
Juliette, Seção 2 do PDF) se conecta com o trabalho de CD/GitOps (Daniel)
e de infraestrutura (Emerson).

## O que o pipeline de CI faz

Para cada um dos 5 microsserviços (`auth-service`, `flag-service`,
`targeting-service`, `evaluation-service`, `analytics-service`), a cada
push/PR na `main`:

1. Lint (golangci-lint ou flake8)
2. Build + testes unitários
3. Security Scan: SAST (gosec/bandit) + SCA (Trivy). **Se uma
   vulnerabilidade CRÍTICA for encontrada, o pipeline falha e para aqui.**
4. Build da imagem Docker, scan de vulnerabilidades na imagem (mesma regra
   de bloqueio), login no ECR e push da imagem com a tag `${{ github.sha }}`
   (o SHA completo do commit) — **apenas em push na `main`**.

O pipeline **para aqui**. Ele não mexe no repositório de GitOps nem no
ArgoCD — isso é responsabilidade do Daniel.

Os workflows reutilizáveis estão centralizados em
`FIAP-Teach-Challenge-2/toggle-master-ci` — cada um dos 5 repositórios
de serviço só tem um arquivo `.github/workflows/ci.yml` de ~15 linhas que
chama esse workflow compartilhado.

## Para o Daniel (CD/GitOps)

- **Formato da tag da imagem**: `${{ github.sha }}` — ou seja, o SHA
  completo do commit (ex: `a1b2c3d4e5f6...`), não uma versão semântica.
  Seu step de atualização do `deployment.yaml`/`kustomization.yaml` no
  repositório de GitOps deve usar exatamente esse valor.
- O registro é o ECR da conta AWS Academy do grupo. O nome de cada
  repositório ECR está definido em `ecr-repository` no `ci.yml` de cada
  serviço — ver seção abaixo sobre nomenclatura.
- Antes de eu (Juliette) fazer o primeiro push real de uma imagem, avise
  se você quiser um valor de teste específico para validar seu step de
  GitOps sem esperar o pipeline completo.

## Para o Emerson (IaC/Terraform)

- **Nomenclatura dos repositórios ECR** (proposta, a confirmar):
  - `togglemaster/auth-service`
  - `togglemaster/flag-service`
  - `togglemaster/targeting-service`
  - `togglemaster/evaluation-service`
  - `togglemaster/analytics-service`
- A criação dos repositórios ECR está no seu escopo (Seção 1 do PDF,
  "Repositórios: 5 repositórios no ECR"). Se seu Terraform ainda não
  estiver pronto e eu precisar testar meu pipeline antes, posso criar os
  5 repositórios manualmente via AWS CLI com esses nomes exatos — nesse
  caso, **não rode `terraform apply`** para criá-los de novo; rode
  `terraform import` para adotar os repositórios existentes no seu state.
  Me avise qual dos dois cenários você prefere.

## Pendência de acesso — importante para quem tem admin nos 5 repos

Como só tenho acesso de leitura (pull) aos 5 repositórios de serviço, vou
entregar o CI via fork + Pull Request. Isso tem uma consequência técnica:
**uma PR aberta a partir de um fork não recebe os secrets do repositório
de origem**, por segurança do GitHub. Ou seja, mesmo depois de cada PR
ser mergeada, o job de push para o ECR só vai funcionar se alguém com
acesso **admin** em cada um dos 5 repositórios configurar estes 3 secrets
(por repositório, são as credenciais temporárias da AWS Academy):

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`

Essas credenciais expiram quando a sessão do AWS Academy Lab é reiniciada
— vão precisar ser atualizadas de tempos em tempos enquanto o desafio
estiver em andamento. Quem tem admin nos 5 repositórios, por favor avisar
para combinarmos isso.

## Aprovação de execução dos workflows (importante)

Cada uma das 5 Pull Requests que abri introduz um arquivo de workflow
(`.github/workflows/ci.yml`) totalmente novo no repositório. Por
segurança, o GitHub não deixa esse workflow rodar automaticamente quando
vem de uma PR de fork — é preciso que alguém com acesso de **maintainer**
no repositório entre na aba "Checks" da PR e clique em "Approve and run
workflow" (esse botão só aparece depois que um maintainer olha a PR; até
lá, nem existe uma execução pendente para aprovar — o pipeline fica sem
nenhum check, silenciosamente).

Sem esse clique, a pipeline nunca roda — não é um bug do código, é uma
trava de segurança do GitHub para PRs vindas de forks. Isso é necessário
nas 5 PRs (`auth-service`, `flag-service`, `targeting-service`,
`evaluation-service`, `analytics-service`), além da configuração dos
secrets AWS mencionada acima.
