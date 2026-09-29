# CARD-001 — Modelo de Domínio e Bounded Contexts

| Campo | Valor |
|---|---|
| Fase | F0 — Pensar o sistema |
| Tipo | Documentação e modelagem. **Nenhuma linha de código de produção.** |
| Status | 🟡 Em andamento |
| Timebox | 2 a 3 sessões |

## Objetivo
Modelar o domínio do MVP: a linguagem, os agregados, as invariantes e os eventos. Com isso, confirmar (ou corrigir) se os módulos definidos no CARD-000 são mesmo os limites certos, **antes** de o CARD-003 transformá-los em pacotes Java.

## Por que esse card existe
No CARD-000 você definiu os módulos pela intuição: "tarefas aqui, calendário ali". Neste card você testa essa intuição com o **comportamento** do sistema. Quem muda o quê, o que precisa ser consistente ao mesmo tempo, que regras nunca podem ser quebradas.

Quem modela só tabelas termina com entidades cheias de getters e setters e as regras espalhadas pelos services. Quem modela comportamento termina com código em que a regra mora junto do dado que ela protege. É a diferença entre `task.setStatus(DONE)` e `task.complete()`.

## Pré-requisitos
- CARD-000 concluído (módulos, matriz de dependências, ADR-001 a ADR-003).

## Conceitos para estudar
- **Linguagem ubíqua:** um mesmo termo, com um mesmo significado, na conversa, na documentação e no código.
- **Subdomínios core, de suporte e genéricos:** onde vale investir o seu melhor esforço e onde usar uma biblioteca pronta.
- **Bounded context vs. módulo:** nem sempre é 1:1. Um contexto pode ter mais de um módulo.
- **Entidade vs. value object:** identidade vs. valor. `TaskId` e `DueDate` são value objects?
- **Agregado e raiz do agregado:** a fronteira de consistência. A regra prática é *uma transação altera um agregado*.
- **Invariante:** algo que precisa ser verdade sempre, não só "na maioria das vezes".
- **Referência por ID:** agregados apontam para outros agregados pelo ID, nunca pelo objeto.
- **Eventos de domínio:** algo que *aconteceu*, nomeado no passado (`TaskCompleted`).
- **Event storming (versão solo e leve):** eventos → comandos → agregados → políticas.
- **Context map:** como os contextos se relacionam (customer/supplier, conformist, anti-corruption layer, published language).

Leitura sugerida: *Domain-Driven Design Distilled*, do Vaughn Vernon (curto, com os capítulos 2 a 5 cobrindo o card inteiro), e o artigo "Effective Aggregate Design", do mesmo autor.

## Perguntas que você precisa responder

### Linguagem
1. **"Evento" já significa duas coisas no LifeHub:** um compromisso do calendário e um evento de domínio (`TaskCreated`). Como você vai nomear cada um, em português e no código, para que essa ambiguidade nunca apareça?
2. **O que é um "prazo"?** Uma data ("sexta-feira") ou um instante ("sexta às 14h")? Uma tarefa "para sexta" vence à meia-noite de qual fuso horário? E se o usuário mudar de fuso?

### Agregados e invariantes
3. **Quais são as invariantes de `Task`?** Uma tarefa concluída pode voltar para TODO? Pode ser criada com prazo no passado? Pode ser editada depois de concluída?
4. **A etiqueta é um agregado próprio ou um value object dentro de `Task`?** Se o usuário renomear a etiqueta "trabalho", ela muda em todas as tarefas? O que cada modelo implica?
5. **A ordem no Kanban pertence a quem?** Mover um card muda a posição dos outros cards da coluna. Se a posição é um atributo de cada `Task`, reordenar altera N agregados numa transação. Se existe um agregado `Board`/`Column`, ele vira um ponto de contenção. Qual você escolhe, e o que isso implica para o optimistic locking do CARD-015?
6. **Por que `Task` guarda só o `UserId` do dono, e não o objeto `User`?** O que quebraria se guardasse o objeto?
7. **Exclusão:** excluir uma tarefa é *hard delete* ou *soft delete*? O que acontece com o lembrete, o registro de auditoria e, no pós-MVP, com as notas que a citam?

### Eventos e lembretes
8. **Liste os eventos de domínio do MVP**, no passado, com quem publica, quem consome e o payload. Qual é o payload **mínimo** para que `notifications` agende um lembrete sem precisar chamar a API de `tasks` de volta?
9. **Quem é dono da intenção "me avise 15 minutos antes"?** É um atributo da tarefa/compromisso (quem cria) ou do lembrete (`notifications`)? Se o usuário mudar o prazo, como o lembrete fica sabendo?

### Contextos
10. **`auth` e `users` são dois bounded contexts ou um só ("Identidade")?** Hoje o login atravessa dois módulos: e-mail em `users`, credenciais em `auth`. Isso é saudável? Onde a senha deveria morar?
11. **Qual é o subdomínio *core* do LifeHub?** Qual é genérico, a ponto de você usar pronto (Spring Security, por exemplo) em vez de modelar à mão? Onde o seu esforço de modelagem rende mais aprendizado?
12. **O resultado bate com a matriz de dependências do CARD-000?** Se não bater, o que muda: a matriz ou o modelo?

## Decisões arquiteturais deste card
- [ ] Se algum limite de módulo mudar (por exemplo, `auth` + `users` virando um contexto só), registrar num novo ADR.
- [ ] Se a posição no Kanban justificar um agregado próprio, registrar a decisão e o impacto em concorrência.

## Implementação
Nenhum código de produção. É permitido esboçar assinaturas em pseudocódigo nos documentos, para mostrar o comportamento:

```text
Task
  + complete()          // invariante: não pode concluir o que já está concluído
  + reschedule(dueDate) // publica TaskRescheduled
  + moveTo(status, position)
```

## Testes
Ainda não há código, mas as invariantes **são** os testes futuros. Para cada invariante, escreva o nome do teste que a protegeria:

```text
should_not_complete_task_already_done
should_reject_event_ending_before_it_starts
should_not_list_tasks_from_another_user
```

Essa lista vai direto para os CARDs 014 a 018.

## Documentação — entregáveis
- [ ] `docs/domain/glossary.md` — linguagem ubíqua: termo em português, nome no código, definição, contexto a que pertence.
- [ ] `docs/domain/event-storming.md` — linha do tempo do MVP: eventos → comandos → agregados → políticas ("quando X, então Y"). Mermaid ou uma foto de post-its.
- [ ] `docs/domain/aggregates.md` — para cada agregado do MVP: raiz, entidades, value objects, invariantes, comandos e eventos.
- [ ] `docs/domain/context-map.md` — bounded contexts, subdomínios (core/suporte/genérico) e relações, em Mermaid.
- [ ] `docs/architecture/architecture.md` — matriz e tabela de módulos atualizadas, se o modelo mudou alguma coisa.
- [ ] `docs/study/CARD-001.md` — o que você aprendeu e por quê.
- [ ] `README.md` — incluir `docs/domain/` na estrutura e no índice.

## Erros comuns
- **Modelo anêmico:** entidades que são só dados com setters, e toda a regra no service.
- **Verbos de CRUD em vez de verbos do domínio:** `updateTask` esconde o que aconteceu. `complete`, `reschedule` e `moveTo` revelam.
- **Agregados grandes demais:** um `User` que contém as tarefas, as notas e os eventos. Toda alteração trava tudo.
- **Modelar tabelas e não comportamento:** se o documento parece um diagrama ER, você modelou o banco, não o domínio.
- **Event storming que vira fluxo de tela:** "usuário clica no botão" não é um evento de domínio.
- **DDD pesado em subdomínio genérico:** autenticação não merece a mesma modelagem que tarefas.
- **Ignorar o tempo:** prazo, fuso horário e "hoje" são as maiores fontes de bug em apps de produtividade.

## Dicas
- Comece pelos **eventos**, no passado, sem ordem. Depois ordene numa linha do tempo e pergunte, para cada um: "que comando causou isso?" e "quem precisa reagir?".
- Para achar o agregado, pergunte: **"o que precisa estar consistente *imediatamente*, na mesma transação?"** O resto pode ser eventual.
- Se uma regra envolve dois agregados, provavelmente ela é uma **política** reagindo a um evento, e não uma transação só.
- Use o seu uso real: pense no que você faria no LifeHub numa segunda-feira de manhã.

## Desafios extras
- **Teste de estresse do modelo:** aplique ao seu modelo uma funcionalidade pós-MVP, como "colunas personalizadas no Kanban" ou "tarefas recorrentes". O modelo absorve a mudança ou precisa ser reescrito? Registre o resultado.
- **Row Level Security:** no `aggregates.md`, marque quais agregados precisariam de política RLS no PostgreSQL (ADR-002) e em qual coluna.

## Critérios de aceite
- [ ] As 12 perguntas estão respondidas.
- [ ] O glossário cobre todos os termos do MVP, sem ambiguidade (inclusive "evento").
- [ ] Cada agregado do MVP tem raiz, invariantes, comandos e eventos.
- [ ] Cada evento de domínio tem um publicador e pelo menos um consumidor (ou está marcado como "futuro").
- [ ] O context map e a matriz de dependências do CARD-000 concordam.
- [ ] Cada invariante tem um nome de teste.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
Ao terminar, envie o repositório. A revisão vai procurar: agregados grandes demais, invariantes que ninguém protege, eventos sem consumidor, termos ambíguos e contradições entre o modelo e a matriz de dependências.

_Anotações do review:_