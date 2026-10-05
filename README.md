[README.md](https://github.com/user-attachments/files/33061132/README.md)
# 🚀 Laboratório Kubernetes: WordPress + MySQL com Auto Scaling (HPA)

Este repositório contém a infraestrutura e os manifestos necessários para implantar um ambiente altamente disponível e escalável do **WordPress** com banco de dados **MySQL** em um cluster local utilizando o **Minikube**.

O objetivo principal deste laboratório foi configurar o escalonamento automático horizontal de Pods (**HPA - Horizontal Pod Autoscaler**) com base no consumo real de CPU coletado pelo **Metrics Server**.

---

## 🏗️ Arquitetura do Projeto

A infraestrutura foi dividida de maneira declarativa utilizando os seguintes componentes do Kubernetes:
* **Secrets (`configuracoes.yaml`):** Armazenamento seguro de credenciais confidenciais (senha de root do banco de dados).
* **PersistentVolumeClaims (`configuracoes.yaml`):** Volumes de armazenamento persistente dedicados para o MySQL e WordPress, garantindo a retenção dos dados em caso de reinicialização dos nós ou Pods.
* **Deployments (`mysql-deployment.yaml` e `wordpress-deployment.yaml`):** Gerenciamento e declaração dos ciclos de vida dos contêineres, controle de réplicas e injeção de variáveis de ambiente.
* **Services (`services-wordpress.yaml` e `mysql-service.yaml`):** Abstrações de rede. O MySQL está exposto internamente via `ClusterIP` e o WordPress externamente via `NodePort` para mapeamento dinâmico com o Minikube.
* **Horizontal Pod Autoscaler (HPA):** Mecanismo automático para escalonamento de réplicas do WordPress variando entre **1 e 5 réplicas**, acionado quando a média de consumo de CPU ultrapassar **50%**.

---

## 🛠️ Como Executar este Laboratório

### 1. Pré-requisitos
* [Minikube](https://minikube.sigs.k8s.io/docs/start/) instalado e configurado no PATH do sistema.
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) rodando como driver/motor local.
* [Kubectl](https://kubernetes.io/docs/tasks/tools/) configurado para gerenciar o cluster.

### 2. Inicializar o Ambiente
Inicie o cluster Minikube utilizando o driver do Docker:
```powershell
minikube start --driver=docker
```

Ative o **Metrics Server** (indispensável para o funcionamento do HPA):
```powershell
minikube addons enable metrics-server
```

### 3. Aplicar os Manifestos no Cluster
Navegue até a pasta onde os arquivos estão localizados e execute o deploy em massa:
```powershell
kubectl apply -f .
```

### 4. Configurar o Escalonamento Automático (HPA)
Com os Deployments e recursos ativos, aplique a regra de HPA para o WordPress:
```powershell
kubectl autoscale deployment wordpress-deployment --cpu-percent=50 --min=1 --max=5
```

Verifique se o Autoscaler está coletando as métricas com sucesso:
```powershell
kubectl get hpa
```

### 5. Acessar a Aplicação
Como estamos rodando localmente via Docker, execute o utilitário do Minikube para criar o túnel de rede e abrir o site diretamente no seu navegador padrão:
```powershell
minikube service wordpress-service
```

---

## 🧪 Validando o Auto Scaling (Stress Test)

Para assistir o Kubernetes escalando os contêineres automaticamente diante de uma alta carga de acessos simultâneos, siga os passos abaixo:

1. Em um terminal, acompanhe o comportamento do HPA em tempo real (modo *watch*):
   ```powershell
   kubectl get hpa -w
   ```
2. Abra uma segunda aba do PowerShell e execute o script abaixo em loop contínuo para gerar carga de acessos HTTP (substitua pela porta ativa fornecida pelo túnel do Minikube):
   ```powershell
   while($true) { Invoke-WebRequest -Uri "http://127.0.0.1:<PORTA_DO_TUNEL>" -UseBasicParsing | Out-Null }
   ```
3. Veja o consumo saltar na primeira janela. O número na coluna `REPLICAS` irá saltar de `1` para o número necessário para estabilizar o consumo. Ao interromper o script com `Ctrl + C`, o Kubernetes executará o *Scale Down* após o período de estabilização, limpando os Pods sobressalentes automaticamente.

---
Elaborado durante as práticas do curso de Docker e Kubernetes. 🚀
