# ADR 001: Banco de dados fora do cluster Kubernetes

*   **Data:** 2026-08-05
*   **Status:** Aceito

## Contexto
A API de Pedidos necessita de um banco de dados relacional (PostgreSQL) com forte consistência para garantir que Pedidos e Itens não sejam perdidos. Executar um banco de dados dentro do Kubernetes adiciona alta complexidade operacional, exigindo que o time gerencie backups e atualizações manualmente.

## Decisão
O banco de dados PostgreSQL 16 será provisionado como uma **instância gerenciada ou dedicada fora do cluster Kubernetes**. A aplicação acessará o banco através da variável de ambiente `DATABASE_URL`, cujas credenciais ficam seguras em uma `Secret` do Kubernetes.

## Consequências
*   **Positivas:** Desacopla a vida dos containers da persistência dos dados; reduz o risco de perda de dados se o cluster for recriado; libera CPU e Memória dos nós do K8s exclusivamente para a API.
*   **Negativas:** Exige configuração de rede para que os pods tenham acesso seguro à instância externa do banco; adiciona uma pequena latência de rede (mitigada mantendo tudo na mesma região).
