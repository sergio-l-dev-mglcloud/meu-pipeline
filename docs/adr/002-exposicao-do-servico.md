# ADR 0002: Estratégia de Exposição e Roteamento de Tráfego

- **Status:** Aprovado
- **Data:** 2026-08-05

## Contexto
A aplicação precisa receber tráfego externo (requisições dos usuários e testes de carga). Precisamos decidir qual estratégia de exposição de rede adotar no Kubernetes: usar um **Ingress Controller** (como Nginx) ou um **Load Balancer** nativo do provedor de nuvem.

## Decisão
Decidimos utilizar o objeto `Service` do Kubernetes do tipo **`LoadBalancer`** para expor a aplicação na porta 80, roteando o tráfego diretamente para a porta 8000 dos containers da API. 

## Consequências
- **Positivas:** Configuração nativa, extremamente simples e rápida para o escopo atual. O provedor de nuvem provisiona automaticamente o IP público e gerencia o balanceamento de tráfego entre as réplicas da API, sem exigir a instalação e configuração adicional de um Ingress Controller.
- **Negativas:** Cada serviço exposto consome um Load Balancer dedicado, o que pode aumentar custos em cenários com muitos microsserviços. No futuro, se a aplicação crescer, migrar para um Ingress permitirá roteamento por caminho (path-based routing) e centralização de IP com menor custo.
