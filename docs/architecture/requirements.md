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
| Dashboard | visão do dia: tarefas com prazo hoje ou atrasadas e eventos de hoje |
| Técnico | PostgreSQL, Liquibase, Docker, Swagger/OpenAPI, auditoria das ações (CARD-013) |

### Fora do MVP (explicitamente)
- Finanças (fora do projeto como um todo)
- MFA, recuperação de senha, gestão de dispositivos → Pós-MVP de segurança
- Recorrência de eventos, relacionamentos entre tarefas/notas/eventos → Pós-MVP de produtividade
- Notificações, RabbitMQ → Release de mensageria
- Documentos, IA, RAG, agentes, MCP → releases posteriores
- App mobile (fora do projeto como um todo; ver [vision.md §5](vision.md))
- Tarefas com data exibidas no calendário e referências entre notas e tarefas → Pós-MVP de produtividade
- Visões personalizadas no dashboard e colunas personalizadas no Kanban → Pós-MVP de produtividade
- Compartilhamento de dados entre usuários → fora do projeto por enquanto
- Anexos em tarefas e notas → Release de documentos

> **Teste do MVP:** se um item não for necessário para você abrir o LifeHub amanhã de manhã e planejar o dia, ele não é MVP.

## 2. User stories do MVP

Formato: *Como usuário, quero ___ para ___.* Critério de aceite em uma linha.

### Conta
- **US-01** — Como visitante, quero me cadastrar para ter meus dados isolados. _Aceite: cadastro aberto ([ADR-003](../adr/ADR-003-cadastro-aberto.md)); e-mail único, senha armazenada com hash; limite de tentativas por IP._
- **US-02** — Como usuário, quero fazer login para acessar meus dados. _Aceite: credenciais inválidas retornam erro genérico._
- **US-03** — Como usuário, quero sair da conta para que ninguém use minha sessão. _Aceite: depois do logout, o token não é mais aceito._

### Tarefas
- **US-04** — Como usuário, quero criar uma tarefa com título, prazo e prioridade para não esquecê-la.
- **US-05** — Como usuário, quero ver as tarefas do dia para saber por onde começar.
- **US-06** — Como usuário, quero editar uma tarefa para corrigir título, prazo ou prioridade. _Aceite: só o dono da tarefa consegue editá-la._
- **US-07** — Como usuário, quero concluir uma tarefa para tirá-la da minha lista. _Aceite: a tarefa concluída some da lista do dia, mas continua consultável._
- **US-08** — Como usuário, quero marcar tarefas com etiquetas e filtrar por elas para agrupar assuntos. _Aceite: uma tarefa pode ter várias etiquetas; o filtro combina com paginação._

### Kanban
- **US-09** — Como usuário, quero ver minhas tarefas num quadro TODO → IN_PROGRESS → DONE para enxergar o andamento. _Aceite: cada tarefa aparece em exatamente uma coluna, de acordo com o status._
- **US-10** — Como usuário, quero mover e reordenar cards para refletir o que estou fazendo. _Aceite: a posição persiste ao recarregar; duas edições simultâneas não se sobrescrevem em silêncio (optimistic locking)._

### Calendário
- **US-11** — Como usuário, quero criar um evento com título, início e fim para registrar compromissos. _Aceite: o fim não pode ser antes do início; horários são salvos em UTC e exibidos no meu fuso horário._
- **US-12** — Como usuário, quero ver os eventos da semana para planejar meus dias.
- **US-13** — Como usuário, quero editar e excluir eventos para manter a agenda atualizada.

### Notas
- **US-14** — Como usuário, quero criar e editar notas em Markdown para registrar ideias.
- **US-15** — Como usuário, quero buscar notas por texto para reencontrar o que escrevi. _Aceite: a busca full-text encontra palavras no título e no corpo._

### Dashboard
- **US-16** — Como usuário, quero ver um resumo do dia para planejar a manhã. _Aceite: mostra as tarefas com prazo hoje, as atrasadas e os eventos de hoje, numa única tela._

## 3. Atributos de qualidade

> ✍️ **Pergunta 8 do CARD-000.** Ordene por importância (1 = mais importante) e justifique. O que você sacrifica primeiro?

**Resposta:**

| Atributo | O que significa no LifeHub | Prioridade |
|---|---|---|
| Segurança | Dados pessoais protegidos; agentes com ações controladas | 1 |
| Durabilidade dos dados | Nada se perde (backup e restore testados) | 2 |
| Manutenibilidade | Evoluir sem reescrever; módulos com limites claros | 3 |
| Custo operacional | Roda num notebook antigo, sem serviços pagos obrigatórios | 4 |
| Performance | Respostas rápidas num hardware modesto | 5 |
| Disponibilidade | O sistema está no ar quando você precisa | 6 |

**Justificativa:**
- **Segurança em 1º:** o sistema é multiusuário, guarda dados pessoais e deve ir para a internet no futuro. Um vazamento entre usuários é o pior cenário.
- **Durabilidade em 2º:** perder notas e tarefas é pior do que ficar fora do ar por algumas horas.
- **Manutenibilidade em 3º:** o projeto é um laboratório que evolui aos poucos, por vários cards e fases.
- **Custo operacional em 4º:** o servidor é o notebook (8 GB de RAM, Celeron) e não há serviços pagos obrigatórios. Isso limita as escolhas de tecnologia.
- **Performance em 5º:** as otimizações são necessárias para caber no notebook, mas o uso pessoal tolera respostas um pouco mais lentas.
- **Disponibilidade é sacrificada primeiro:** o notebook desligado é aceitável ([vision.md §6](vision.md)), desde que nenhum dado nem lembrete se perca.

### Metas mensuráveis
Servem de critério para os testes de desempenho que vão decidir a hospedagem ([vision.md §6](vision.md)).
- **Tempo de resposta:** p95 abaixo de 500 ms nas telas principais (tarefas, Kanban, calendário, dashboard), medido no notebook.
- **Memória:** a stack do MVP (backend + PostgreSQL) usa no máximo 4 GB, deixando folga no notebook.
- **RPO:** perco no máximo 24 h de dados (backup diário).
- **RTO:** consigo restaurar o sistema a partir do backup em até 1 h.
- **Disponibilidade:** sem meta formal.

## 4. Restrições

- Hardware do servidor: notebook com 8 GB de RAM e Celeron de 2ª geração. _✍️ Disco?_
- Hardware de desenvolvimento: PC com 32 GB de RAM, Ryzen 5 5600G e Radeon RX 6600 (ver [vision.md §6](vision.md)).
- Rede: doméstica por enquanto; no futuro, hospedagem online (AWS). Até lá, acesso só pela rede local. Se precisar de acesso externo, usar VPN (WireGuard ou Tailscale) em vez de expor portas do roteador.
- Tempo: projeto pessoal, sessões de estudo (_✍️ quantas horas por semana?_)
- Stack definida: Java 21, Spring Boot, React, PostgreSQL (ver [architecture.md](architecture.md))

## 5. Riscos operacionais

| Risco | Impacto | Mitigação |
|---|---|---|
| Disco do notebook ser HDD | É o maior gargalo do PostgreSQL e pode inviabilizar a meta de p95 < 500 ms | Confirmar o tipo de disco; se for HDD, trocar por SSD antes do deploy (CARD-007) |
| Bateria antiga ligada na tomada 24/7 | Baterias velhas podem inchar | Retirar a bateria ou limitar a carga; se estiver saudável, ela serve de nobreak |
