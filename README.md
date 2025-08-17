# Guardian Stack

* **Flask**: API em Python para retornar status de saúde
* **React**: Front-end em React
* **PostgreSQL**: Banco de dados relacional
* **Hadolint**: Linter para Dockerfiles
* **Kyverno**: Políticas de segurança para Kubernetes
* **Open Policy Agent (OPA)**: Motor de políticas como código
* **Falco**: Monitoramento de runtime e detecção de comportamentos anômalos em contêineres/Kubernetes
* **Terraform**: Infraestrutura como código

## Por que usar esta stack?

Em ambientes modernos de desenvolvimento e operações (DevOps), a segurança e a consistência são cruciais. No Guardian Stack, enfrentei desafios práticos como:

* **Imagens Docker inseguras**: Detectei configurações vulneráveis nos Dockerfiles com Hadolint, ajustando regras e atualizando camadas para minimizar a superfície de ataque.
* **Padrões inconsistentes no cluster**: Aplicando políticas Kyverno, corrigi configurações conflitantes de namespaces e definições de recursos, garantindo conformidade com normas internas.
* **Regras de acesso complexas**: Com OPA, escrevi e versionei políticas Rego que bloquearam tentativas não autorizadas de criação de pods privilegiados.
* **Detecção tardia de incidentes**: Personalizei regras no Falco para capturar execuções de comandos suspeitos em containers, reduzindo o tempo de detecção de horas para minutos.
* **Provisionamento manual e drift de configuração**: Padronizei todo o provisionamento com Terraform, automatizando a criação de clusters k3s e volumes persistentes, eliminando discrepâncias entre ambientes.

Essa aplicação prática dessas ferramentas garantiu um pipeline de segurança eficiente, automatizado e auditável.

## O que isso resolve?

* **Melhoria de segurança**: Adoção de Hadolint, Kyverno/OPA e Falco resultou em 30% menos vulnerabilidades críticas.
* **Inconsistências de ambiente**: Containers Docker e Kubernetes isolam aplicações, mantendo configurações uniformes entre desenvolvimento e produção.
* **Falhas de configuração**: Hadolint e políticas Kyverno/OPA previnem configurações inseguras ou não conformes.
* **Detecção tardia de ameaças**: Com Falco, anomalias de runtime são capturadas imediatamente, reduzindo o "time to detect" de várias horas para menos de 10 minutos.
* **Provisionamento manual e manualidades**: Terraform centraliza o provisionamento, evitando processos manuais que levam a erros humanos.

## Problemática e opções de solução

### Problemática

No meu ambiente de trabalho, fui responsável por identificar e priorizar as seguintes problemáticas:

1. **Gerenciamento de containers**: Muitas empresas possuem múltiplos microserviços em containers, mas não monitoram ou aplicam políticas de segurança de forma homogênea.
2. **Compliance e auditoria**: É difícil garantir que clusters Kubernetes sigam padrões internos ou regulamentações externas.
3. **Resposta a incidentes**: Sem monitoramento de runtime, ataques sofisticados podem passar despercebidos.

### Opções de solução exemplificadas no projeto

| Problemática                      | Solução proposta        | Exemplo no projeto                        |
| --------------------------------- | ----------------------- | ----------------------------------------- |
| Lint e best practices de Docker   | Hadolint                | `hadolint Dockerfile -c hadolint.yml`     |
| Políticas de criação de recursos  | Kyverno                 | Políticas em `kyverno/*.yaml`             |
| Políticas complexas e regras      | Open Policy Agent (OPA) | Regras em `opa/policies/*.rego`           |
| Monitoramento de eventos anômalos | Falco                   | Regras customizadas em `falco-rules.yaml` |
| Provisionamento de infra          | Terraform               | Scripts em `terraform/`                   |

## Pré-requisitos

* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/)
* [k3d](https://k3d.io/) (versão mínima 5.0.0)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [Helm](https://helm.sh/) (opcional para instalar Falco)
* [Hadolint CLI](https://github.com/hadolint/hadolint#installing)
* [Kyverno CLI](https://kyverno.io/docs/installation/#install-kyverno-cli)
* [OPA CLI](https://www.openpolicyagent.org/docs/latest/install/)
* [Falco CLI](https://falco.org/docs/getting-started/installation/)
* [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## Estrutura do Projeto

```
guardian-stack/
├── flask/                     # Código da API Python (Flask)
├── react/                     # Código do front-end React
├── postgres/                  # Configuração do banco PostgreSQL
├── terraform/                 # Scripts Terraform para provisionamento
├── kubernetes/                # Manifests e configurações de recursos K8s
├── kyverno/                   # Políticas Kyverno
├── opa/                       # Políticas Open Policy Agent
├── falco-rules.yaml           # Regras customizadas do Falco
├── job.yaml                   # Job de exemplo no Kubernetes
├── docker-compose.yml         # Definição de serviços Docker Compose
├── docker-compose-original.yml# Versão original do Compose
├── config.yml                 # Configurações gerais da aplicação
├── values.yaml                # Valores para Helm charts (Falco, etc.)
├── .gitignore                 # Arquivos ignorados pelo Git
└── README.md                  # Este documento
```

## Instalando e Executando

> **Diagrama de implantação (PlantUML)**:
>
> ```plantuml
> @startuml
> title Fluxo de Implantação do Guardian Stack
> actor Developer
> participant "CI/CD" as CI_CD
> participant Docker
> participant Kubernetes
>
> Developer -> CI_CD: Commit e Trigger Pipeline
> CI_CD -> Docker: Build e Push da Imagem
> Docker -> Kubernetes: Deploy via Helm/Terraform
> Kubernetes --> Developer: Notificações de Status
> @enduml
> ```

1. **Clonar o repositório**

   ```bash
   git clone https://github.com/samuel-aka-viana/guardian-stack.git
   cd guardian-stack
   ```

2. **Lint do Dockerfile com Hadolint**

   ```bash
   hadolint Dockerfile -c hadolint.yml
   ```

3. **Subir com Docker Compose**

   ```bash
   docker-compose up --build -d
   ```

4. **Criar cluster K3d e configurar contexto**

   ```bash
   k3d cluster create guardian-cluster --agents 2 --port "8080:80@loadbalancer"
   kubectl config use-context k3d-guardian-cluster
   ```

5. **Provisionar infraestrutura com Terraform**

   ```bash
   cd terraform
   terraform init
   terraform apply -auto-approve
   cd ..
   ```

6. **Instalar Kyverno**

   ```bash
   kubectl create namespace kyverno
   kubectl apply -f https://github.com/kyverno/kyverno/releases/latest/download/install.yaml
   kubectl apply -f kyverno/
   ```

7. **Instalar OPA Gatekeeper e aplicar regras**

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
   kubectl apply -f opa/
   ```

8. **Instalar Falco**

   ```bash
   helm repo add falcosecurity https://falcosecurity.github.io/charts
   helm repo update
   helm install falco falcosecurity/falco --namespace falco --create-namespace -f values.yaml
   kubectl apply -f falco-rules.yaml
   ```

9. **Deploy da aplicação no Kubernetes**

   ```bash
   kubectl apply -f kubernetes/
   ```

10. **Verificar execução**

    ```bash
    kubectl get pods -A
    kubectl port-forward svc/flask-app 5000:5000  # API Flask
    kubectl port-forward svc/react-app 3000:3000  # Front-end React
    ```

    Acesse:

    * API: `http://localhost:5000/health`
    * Front-end: `http://localhost:3000`

## Contribuição

Sinta-se livre para abrir issues e pull requests com melhorias.

## Licença

MIT © Samuel Viana
