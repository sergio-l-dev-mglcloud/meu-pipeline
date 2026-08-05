# Architecture Documentation

Este documento descreve a arquitetura da API de Pedidos, seus requisitos não-funcionais e as decisões que a sustentam.

## 1. Recursos da Arquitetura

Antes de mapear os containers, é fundamental entender o que compõe o ambiente de execução (Cluster Kubernetes) e o que reside fora dele:

### Dentro do Cluster Kubernetes (MGC)
*   **Deployment (`cloud-application`):** Gerencia os pods da API FastAPI. Configurado com *requests* baixos (100m CPU / 128Mi RAM) e *limits* (500m CPU / 256Mi RAM) para otimização de custos.
*   **HPA (Horizontal Pod Autoscaler):** Mantém a alta disponibilidade e performance escalando de **2 a 6 réplicas** baseado no uso de CPU (alvo de 70%).
*   **Service (`LoadBalancer`):** Ponto de entrada do tráfego, expondo a porta 80 e roteando internamente para a porta 8000 dos containers.
*   **ServiceMonitor:** Objeto customizado (Prometheus Operator) que varre o endpoint `/metrics` da API a cada 15 segundos.
*   **Secrets (`db-secret`, `registry-secret`):** Armazenam credenciais sensíveis para acesso ao banco e puxada de imagens.

### Fora do Cluster (Recursos Externos)
*   **Banco de Dados (PostgreSQL 16):** Instância de banco relacional isolada do cluster (Serviço Gerenciado).
*   **Container Registry (MGC):** Repositório onde as imagens Docker da API são versionadas e armazenadas.
*   **Prometheus / Grafana:** Stack de observabilidade que consome as métricas exportadas pela API.
*   **GitHub Actions:** Pipeline de CI/CD que constrói a imagem e orquestra o deploy no cluster.

---

## 2. Diagrama C2 (Container)

*Nota: O termo "C2" refere-se ao Nível 2 do Modelo C4 (Diagrama de Containers), que foca nas tecnologias, responsabilidades e protocolos de comunicação.*

```mermaid
flowchart TD
    %% C4 Model - Level 2: Container Diagram
    
    subgraph SystemBoundary ["Sistema de Pedidos (Move Tech)"]
        direction TB
        
        subgraph ClusterK8s ["Cluster Kubernetes (Magalu Cloud)"]
            direction TB
            subgraph API_Pods ["API de Pedidos (HPA: 2 a 6 réplicas)"]
                API_Container["Container: API FastAPI\n(Python / Uvicorn)\nGerencia Pedidos e Itens"]
            end
            LB["Service: LoadBalancer\nRoteia tráfego externo"]
            SM["ServiceMonitor\nPrometheus Operator"]
        end

        subgraph ExternalSystems ["Sistemas Externos"]
            DB[("Container: Banco PostgreSQL 16\nDados de Pedidos e Itens")]
            Registry["Container Registry\n(MGC)"]
            Prometheus["Prometheus\nMonitoramento"]
        end
    end

    Client["Cliente / k6 LoadTest"] -- "Requisições REST\nHTTPS/HTTP (TCP 80)" --> LB
    LB -- "HTTP (TCP 8000)" --> API_Container
    
    API_Container -- "TCP / Protocolo Postgres\n(Porta 5432)" --> DB
    API_Container -. "Métricas /metrics\nHTTP" .-> SM
    SM -. "Scrape (15s)" .-> Prometheus
    
    Registry -. "Pull de Imagem\nHTTPS" .-> API_Pods
