# ADR 003: Granularidade dos serviços (Monolito de Domínio Único)

*   **Data:** Agosto de 2026
*   **Status:** Aceito

## Contexto
O escopo da nossa aplicação lida com o domínio de "Pedidos" (criação, listagem, cancelamento) e "Itens do Pedido". Havia a dúvida se deveríamos separar essas funções em sistemas menores e independentes (chamados de microsserviços).

## Decisão
A arquitetura adotará o estilo **Monolítico Modular (Single-Service API)**. Toda a lógica de negócio e as regras do domínio de Pedidos ficarão centralizadas em uma única aplicação (a API) que se comunica diretamente com o banco de dados PostgreSQL.

## Consequências
*   **Positivas:** As transações são mais seguras e rápidas, pois o banco de dados garante que um Pedido e seus Itens sejam salvos juntos sem erros; o sistema é muito mais simples de testar e colocar no ar (apenas uma imagem Docker).
*   **Negativas:** Se apenas uma parte do sistema der muito trabalho (por exemplo, gerar relatórios de itens), a aplicação inteira precisará ser duplicada (escalada), gastando um pouco mais de recursos do que o necessário.
