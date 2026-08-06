# ADR 002: Exposição do serviço via LoadBalancer

*   **Data:** Agosto de 2026
*   **Status:** Aceito

## Contexto
A API precisa ser acessível através da rede para receber requisições de clientes e dos testes de carga automatizados. Precisávamos escolher como expor essa porta para a internet de forma simples e segura.

## Decisão
A exposição do serviço será feita através do objeto `Service` do Kubernetes do tipo **`LoadBalancer`**. Ele mapeia a porta `80` externamente para a porta `8000` (onde a API Python está rodando internamente) dos containers.

## Consequências
*   **Positivas:** Configuração nativa e extremamente simples; o provedor de nuvem provisiona automaticamente o IP público e gerencia o balanceamento de tráfego entre as cópias da API.
*   **Negativas:** Incide um custo adicional sobre a instância do LoadBalancer do provedor; no futuro, se a aplicação crescer muito, migrar para um *Ingress Controller* será necessário para otimizar custos.
