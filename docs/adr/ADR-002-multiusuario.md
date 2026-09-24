# ADR-002 — Multiusuário com isolamento por usuário

- **Status:** Aceito
- **Data:** 2026-09-24

## Contexto

O LifeHub é de uso pessoal, mas também serve como portfólio e deve ser hospedado online no futuro. A escolha entre usuário único e multiusuário é a mais cara de mudar depois: ela afeta o modelo de dados (toda tabela tem `user_id`?), a autenticação e a autorização, a auditoria e o isolamento de dados no RAG.

A decisão de ser multiusuário foi tomada em [vision.md §3](../architecture/vision.md). Este ADR registra o porquê e as consequências.

## Decisão

O LifeHub é **multiusuário**. Todo dado de negócio pertence a um usuário, identificado por `user_id`, e só esse usuário tem acesso a ele. Não há compartilhamento de dados entre usuários.

## Alternativas consideradas

### A — Usuário único
Mais simples: sem cadastro, sem isolamento, autenticação mínima.
**Por que foi rejeitada:** não combina com a hospedagem online nem com o uso como portfólio. Também tiraria do projeto a prática de Spring Security, RBAC e isolamento de dados. E adicionar `user_id` em todas as tabelas depois seria uma migração cara.

### B — Multi-tenant com um schema (ou banco) por usuário
Isolamento físico dos dados de cada usuário.
**Por que foi rejeitada:** complexidade desproporcional para poucos usuários. Cada migration do Liquibase teria de rodar em N schemas, e o consumo de recursos no notebook cresceria com o número de usuários.

## Consequências

### Positivas
- Pronto para a hospedagem online sem reescrever o modelo de dados.
- Pratica autenticação, autorização e isolamento de dados de verdade.
- A auditoria sempre sabe quem fez cada ação.

### Negativas (o que perdemos)
- Toda tabela de negócio carrega `user_id`, e toda consulta precisa filtrar por ele.
- Cadastro e login fazem parte do MVP, mesmo que no início só exista um usuário.
- Mais casos de teste: cada funcionalidade precisa provar que um usuário não vê os dados de outro.

### Riscos e mitigação
| Risco | Mitigação |
|---|---|
| Uma consulta sem filtro por `user_id` vazar dados de outro usuário | O `userId` vem sempre do contexto de segurança, nunca da requisição; testes de isolamento por módulo; avaliar Row Level Security do PostgreSQL |
| O RAG devolver trechos de notas ou documentos de outro usuário | Filtrar por `user_id` na busca vetorial (pgvector), antes de montar o contexto do modelo |
| Tools de IA agirem sobre dados de outro usuário | Tools chamam as APIs públicas dos módulos, que aplicam a mesma autorização |
