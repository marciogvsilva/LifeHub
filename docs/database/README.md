# Banco de dados

> Status: **placeholder**. Esta pasta ganha conteúdo a partir do CARD-004 (PostgreSQL + Liquibase).

O que vai morar aqui:
- **Convenções:** nomes de tabelas, colunas, constraints e índices; chaves (UUID vs. sequência).
- **Organização por módulo:** schema por módulo ou prefixo de tabela (decisão pendente, ver [c4.md](../diagrams/c4.md)).
- **Modelo de dados:** diagramas ER por módulo (Mermaid `erDiagram`).
- **Liquibase:** estrutura dos changelogs e regras (um changeset nunca é editado depois de aplicado).
- **PL/pgSQL:** quando usar funções e triggers, e quando manter a regra na aplicação.
