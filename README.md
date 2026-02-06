<p align="center">
  <img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" alt="Kubernetes Logo" width="400"/>
</p>

<h1 align="center">Deploy Node App - Kubernetes</h1>

<p align="center">
  <strong>Guia pratico de deploy de uma aplicacao Node.js com Docker e Kubernetes</strong>
</p>

<p align="center">
  <a href="#sobre-o-projeto">Sobre</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#arquitetura">Arquitetura</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#pre-requisitos">Pre-requisitos</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#instalacao">Instalacao</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#deploy-kubernetes">Deploy</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#comandos-uteis">Comandos</a>&nbsp;&nbsp;|&nbsp;&nbsp;
  <a href="#autores">Autores</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-20-339933?logo=node.js&logoColor=white" alt="Node.js"/>
  <img src="https://img.shields.io/badge/Express-4.21-000000?logo=express&logoColor=white" alt="Express"/>
  <img src="https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?logo=kubernetes&logoColor=white" alt="Kubernetes"/>
  <img src="https://img.shields.io/badge/Kind-Local%20Cluster-FF6B6B" alt="Kind"/>
</p>

---

## Sobre o Projeto

Este repositorio contem o material pratico do artigo **"Kubernetes - Orquestracao de Containers"**, que aborda os conceitos de virtualizacao, containers e a plataforma Kubernetes como orquestrador de containers.

O projeto consiste em uma **aplicacao Node.js com Express** que demonstra na pratica como:

- Criar uma imagem Docker de uma aplicacao
- Configurar um cluster Kubernetes local com Kind (Kubernetes in Docker)
- Realizar o deploy da aplicacao no cluster com **3 replicas**
- Utilizar **Liveness Probes** para autocorrecao (self-healing)
- Simular falhas para observar o Kubernetes reiniciando pods automaticamente
- Aplicar **balanceamento de carga** entre os pods via Service

---

## Arquitetura

O cluster Kubernetes criado neste projeto segue a seguinte topologia:

```
                    +---------------------+
                    |   Control Plane     |
                    |   (Master Node)     |
                    |                     |
                    |  - API Server       |
                    |  - Scheduler        |
                    |  - Controller Mgr   |
                    |  - etcd             |
                    +--------+------------+
                             |
              +--------------+--------------+
              |                             |
    +---------v----------+       +----------v---------+
    |    Worker Node 1   |       |    Worker Node 2   |
    |                    |       |                    |
    |  +------+ +------+ |       |  +------+          |
    |  |Pod 1 | |Pod 2 | |       |  |Pod 3 |          |
    |  |:3000 | |:3000 | |       |  |:3000 |          |
    |  +------+ +------+ |       |  +------+          |
    +--------------------+       +--------------------+
```

**Componentes:**

| Arquivo | Descricao |
|---|---|
| `server.js` | Aplicacao Express com rota principal (`/`) e rota de falha simulada (`/unstable`) |
| `Dockerfile` | Imagem Docker baseada em Node.js 20 |
| `kind-config.yaml` | Configuracao do cluster Kind com 1 control-plane e 2 workers |
| `deployment.yaml` | Deployment Kubernetes com 3 replicas e liveness probe |
| `service.yaml` | Service do tipo NodePort para expor a aplicacao |

---

## Funcionalidades da Aplicacao

### `GET /`
Retorna uma pagina HTML exibindo o **hostname do pod** que atendeu a requisicao. Inclui um delay simulado de 1.5s para representar chamadas a APIs externas ou banco de dados.

### `GET /unstable`
Simula falhas aleatorias - **30% de chance** de o processo encerrar com `process.exit(1)`. Isso aciona a **Liveness Probe** do Kubernetes, que detecta a queda e reinicia o pod automaticamente, demonstrando o recurso de **autocorrecao (self-healing)**.

---

## Pre-requisitos

| Requisito | Versao Minima |
|---|---|
| Sistema Operacional | Windows / Linux / macOS |
| RAM | 8 GB |
| Docker | Instalado e em execucao |
| Node.js | 18+ |
| Kind | Mais recente |
| Kubectl | Mais recente |
| Git | Mais recente |

### Instalacao dos pacotes (Windows via Chocolatey)

Abra o **CMD como administrador** e instale o Chocolatey seguindo as instrucoes em [chocolatey.org/install](https://chocolatey.org/install#individual).

Em seguida, instale as dependencias:

```bash
choco install kind
choco install kubernetes-cli
choco install nodejs.install
choco install git.install
```

> Reinicie o computador apos a instalacao do Docker/Kind.

---

## Instalacao

### 1. Clonar o repositorio

```bash
git clone https://github.com/Michelangelo-Costa/deploy-node-app.git
cd deploy-node-app
```

### 2. Instalar dependencias

```bash
npm install
```

### 3. Executar localmente (sem Kubernetes)

```bash
node server.js
```

Acesse no navegador:

| Rota | URL |
|---|---|
| Pagina principal | [http://localhost:3000](http://localhost:3000) |
| Simulacao de falha | [http://localhost:3000/unstable](http://localhost:3000/unstable) |

---

## Deploy Kubernetes

### Passo 1 - Criar a imagem Docker

```bash
docker build -t node-app:v1 .
```

### Passo 2 - Criar o cluster Kind

```bash
kind create cluster --config kind-config.yaml --name my-kind-1
```

Isso cria um cluster com **1 control-plane** e **2 worker nodes**.

### Passo 3 - Carregar a imagem no cluster

```bash
kind load docker-image node-app:v1 --name my-kind-1
```

### Passo 4 - Aplicar os manifests Kubernetes

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Passo 5 - Encaminhar a porta para acesso local

```bash
kubectl port-forward svc/node-app-service 8080:3000
```

Acesse a aplicacao em: **[http://localhost:8080](http://localhost:8080)**

---

## Comandos Uteis

```bash
# Ver todos os pods e seus nos
kubectl get pods -o wide

# Ver logs de um pod especifico
kubectl logs <nome-do-pod>

# Ver logs de um pod que foi reiniciado (container anterior)
kubectl logs -p <nome-do-pod>

# Ver os nos do cluster
kubectl get nodes

# Ver status do deployment
kubectl get deployments

# Ver servicos ativos
kubectl get services

# Descrever um pod em detalhes (util para debug)
kubectl describe pod <nome-do-pod>

# Escalar o numero de replicas
kubectl scale deployment node-app --replicas=5

# Deletar o cluster
kind delete cluster --name my-kind-1
```

---

## Estrutura do Projeto

```
deploy-node-app/
|-- server.js              # Aplicacao Node.js + Express
|-- package.json           # Dependencias do projeto
|-- Dockerfile             # Build da imagem Docker
|-- kind-config.yaml       # Config do cluster Kind (1 master + 2 workers)
|-- deployment.yaml        # Deployment K8s (3 replicas + liveness probe)
|-- service.yaml           # Service K8s (NodePort)
`-- README.md              # Este arquivo
```

---

## Conceitos Abordados

| Conceito | Descricao |
|---|---|
| **Virtualizacao** | Tecnica que permite executar multiplos ambientes isolados em um unico hardware |
| **Containers** | Virtualizacao a nivel de SO, compartilhando o kernel e isolando processos |
| **Kubernetes** | Plataforma open-source para orquestracao de containers (criada pelo Google) |
| **Pod** | Menor unidade deployavel no Kubernetes, encapsula um ou mais containers |
| **Deployment** | Gerencia o ciclo de vida dos pods, incluindo replicas e atualizacoes |
| **Service** | Abstrai o acesso aos pods via IP/DNS estavel e balanceamento de carga |
| **Liveness Probe** | Verifica se o container esta saudavel; reinicia automaticamente em caso de falha |
| **Kind** | Ferramenta para rodar clusters Kubernetes locais usando Docker |

---

## Vantagens do Kubernetes

| Vantagem | Descricao |
|---|---|
| Escalabilidade automatica | Escala aplicacoes conforme a demanda de trafego |
| Alta disponibilidade | Garante funcionamento mesmo com falhas de componentes |
| Gerenciamento simplificado | Controle de multiplos containers por um unico ponto |
| Portabilidade | Agnostico a provedor, executa em qualquer ambiente |
| Autocorrecao | Reinicia containers com falha automaticamente |
| Balanceamento de carga | Distribui trafego entre pods de forma inteligente |

---

## Referencias

- [Documentacao Oficial do Kubernetes](https://kubernetes.io/pt-br/docs/home/)
- [Kind - Kubernetes in Docker](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Chocolatey - Gerenciador de Pacotes](https://chocolatey.org/install#individual)
- [Containerizacao - IBM](https://www.ibm.com/br-pt/think/topics/containerization)
- TANENBAUM, A. S. (2016). *Sistemas Operacionais Modernos*. Sao Paulo: Pearson.
- FREIRE, J. E. L. (2021). *Orquestracao de containers usando Kubernetes e Docker Swarm*. Universidade da Beira Interior, Portugal.

---

## Autores

| Nome | Papel |
|---|---|
| **Marcus Vinicius Rangel Coelho** | Organizacao do roteiro e abrangencia teorica |
| **Vitoria Leda Nogueira** | Organizacao do roteiro e abrangencia teorica |
| **Martiniano Gomes Barros Cirqueira Neto** | Qualidade da apresentacao teorica |
| **Michelangelo Costa** | Execucao da atividade pratica |
| **Igor Santos** | Execucao da atividade pratica |
| **Caius Souza Ribeiro** | Monitor - suporte a duvidas |
| **Luis Otavio de Siqueira Pimentel** | Monitor - suporte a duvidas |
| **Lucas Barros Mota** | Monitor - suporte a duvidas |
| **Angelo Resplandes Rodrigues** | Conclusao e interacao com o publico |

---

<p align="center">
  Feito com dedicacao para a disciplina de Sistemas Operacionais
</p>
