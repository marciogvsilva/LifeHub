# Requisitos

> Status: **rascunho — CARD-000**

## 1. Escopo do MVP (Release 1)

O MVP é a **primeira versão que você usa no dia a dia**. O que não estiver na lista abaixo está fora dele.

| Área | Dentro do MVP |
|---|---|
| Conta | cadastro, login, logout |
| Tarefas | criar, editar, concluir, prioridade, prazo, etiquetas |
| Kanban | colunas TODO → IN_PROGRESS → DONE, mover cards |
| Calendário | criar, editar e listar eventos |
| Notas | criar, editar, buscar |
| Técnico | PostgreSQL, Liquibase, Docker, Swagger/OpenAPI |

### Fora do MVP (explicitamente)
- Finanças (fora do projeto como um todo)
- MFA, recuperação de senha, gestão de dispositivos → Release de segurança
- Recorrência de eventos, relacionamentos entre tarefas/notas/eventos → Release de produtividade
- Notificações, RabbitMQ → Release de mensageria
- Documentos, IA, RAG, agentes, MCP → releases posteriores
- _✍️ Complete com o que mais você decidir cortar._

> **Teste do MVP:** se um item não for necessário para você abrir o LifeHub amanhã de manhã e planejar o dia, ele não é MVP.

## 2. User stories do MVP

Formato: *Como usuário, quero ___ para ___.* Critério de aceite em uma linha.

### Conta
- **US-01** — Como usuário, quero me cadastrar para ter meus dados isolados. _Aceite: e-mail único, senha armazenada com hash._
- **US-02** — Como usuário, quero fazer login para acessar meus dados. _Aceite: credenciais inválidas retornam erro genérico._

### Tarefas
- **US-03** — Como usuário, quero criar uma tarefa com título, prazo e prioridade para não esquecê-la.
- **US-04** — Como usuário, quero ver as tarefas do dia para saber por onde começar.
- **US-05** — _✍️ escreva as demais._

### Kanban
- **US-0X** — _✍️_

### Calendário
- **US-0X** — _✍️_

### Notas
- **US-0X** — _✍️_

## 3. Atributos de qualidade

> ✍️ **Pergunta 8 do CARD-000.** Ordene por importância (1 = mais importante) e justifique. O que você sacrifica primeiro?

| Atributo | O que significa no LifeHub | Prioridade |
|---|---|---|
| Manutenibilidade | Evoluir sem reescrever; módulos com limites claros | |
| Segurança | Dados pessoais protegidos; agentes com ações controladas | |
| Disponibilidade | O sistema está no ar quando você precisa | |
| Durabilidade dos dados | Nada se perde (backup e restore testados) | |
| Performance | Respostas rápidas num hardware modesto | |
| Custo operacional | Roda num notebook antigo, sem serviços pagos obrigatórios | |

_Justificativa:_

### Metas mensuráveis (opcional, desafio extra)
- _Exemplo de SLO caseiro: "disponível 95% do tempo em que estou acordado"._
- _Exemplo de RPO: "perco no máximo 24h de dados"._

## 4. Restrições

- Hardware: notebook antigo (_✍️ RAM / CPU / disco_)
- Rede: doméstica (_✍️ acesso só na LAN ou também externo?_)
- Tempo: projeto pessoal, sessões de estudo (_✍️ quantas horas por semana?_)
- Stack definida: Java 21, Spring Boot, React, PostgreSQL (ver [architecture.md](architecture.md))
