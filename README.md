# Deploy Volunteer Service

Repositório GitOps com os manifestos Kubernetes do **Volunteer Service** da plataforma SolidaryTech.

## Visão Geral

Monitorado pelo **ArgoCD**. A tag da imagem em `manifests/deployment.yaml` é atualizada **automaticamente** pelo CI do repositório [`volunteer-service`](https://github.com/brianmonteiro54/volunteer-service) a cada push na `main`. Não edite a tag manualmente.

Stack: **Python/Flask + DynamoDB** · porta `8083` · namespace `solidarytech-volunteer`.

> Diferente do ngo/donation, este serviço **não usa PostgreSQL** — então não há initContainer de `DATABASE_URL`, nem `init-db`, nem `externalsecret`/`secretstore`. O acesso ao DynamoDB usa as credenciais do secret `aws-credentials` (criado pelo Terraform no namespace).

## Manifestos

| Arquivo | Descrição |
|---|---|
| `deployment.yaml` | Deployment (2 réplicas, rolling update, probes); usa o CMD gunicorn do Dockerfile |
| `service.yaml` | Service ClusterIP na porta 8083 |
| `ingress.yaml` | Ingress nginx, rota `/volunteers` |

## Pré-requisitos (uma vez)

1. **Infra aplicada** (`terraform apply`): cria o ECR `solidarytech/volunteer-service`, a tabela DynamoDB `solidarytech-<env>-volunteers`, os namespaces e o secret `aws-credentials`.
2. **Nome da tabela**: em `deployment.yaml`, ajuste `AWS_DYNAMODB_TABLE` para o nome real (ex.: `solidarytech-prod-volunteers` ou `solidarytech-dev-volunteers`, conforme o ambiente).
3. **App no ArgoCD** apontando para este repositório (`path: manifests`, namespace `solidarytech-volunteer`).

> **Acesso público:** `http://solidarytech.pt/volunteer` (o ingress reescreve o path singular para a rota real `/volunteers` da aplicação). Aponte o DNS de `solidarytech.pt` para o IP/Hostname do Load Balancer do ingress-nginx. O subpath também funciona: `http://solidarytech.pt/volunteer/1` → `/volunteers/1`.
