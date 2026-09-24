# CARD-000 — Visão do Produto e Definição Arquitetural

| Campo | Valor |
|---|---|
| Fase | F0 — Pensar o sistema |
| Tipo | Documentação. **Nenhuma linha de código.** |
| Status | 🟡 Em andamento |
| Timebox | 2 a 3 sessões |

## Objetivo
Definir o que o LifeHub é, o que ele não é e como está organizado, com clareza suficiente para que os próximos 30 cards não precisem rediscutir fundamentos.

## Por que esse card existe
A maior parte da dívida técnica de projetos pessoais nasce de decisões que nunca foram tomadas conscientemente. Aqui você treina uma habilidade de sênior: **decidir, registrar o porquê e aceitar os trade-offs.**

## Pré-requisitos
Nenhum. Só um repositório Git vazio.

## Conceitos para estudar
- **Visão de produto e não-objetivos:** o que fica de fora é tão importante quanto o que entra.
- **Requisitos funcionais vs. atributos de qualidade:** disponibilidade, segurança, manutenibilidade, custo do hardware.
- **Modelo C4:** só os níveis 1 (Contexto) e 2 (Containers) por enquanto.
- **ADR:** formato do Michael Nygard, incluindo as alternativas rejeitadas.
- **Modular monolith:** o que torna um módulo *de fato* um módulo, e a diferença entre módulo e camada.

## Perguntas que você precisa responder
Responda por escrito, **no documento indicado**.

| # | Pergunta | Onde responder | Status |
|---|---|---|---|
| 1 | Que problema **seu, de hoje**, o LifeHub resolve melhor que Notion ou Todoist? | [vision.md §2](../architecture/vision.md) | ✅ |
| 2 | Usuário único ou multiusuário? | [vision.md §3](../architecture/vision.md) | ✅ |
| 3 | O que acontece quando o notebook desliga? Isso é aceitável? | [vision.md §6](../architecture/vision.md) | 🟡 parcial |
| 4 | Uma tarefa com data é um evento? Um lembrete pertence a Tasks, Calendar ou Notifications? | [architecture.md §4](../architecture/architecture.md) | ✅ |
| 5 | Notes pode referenciar Tasks? Em que direção vai a dependência? | [architecture.md §4](../architecture/architecture.md) | ✅ |
| 6 | Dashboard tem dados próprios ou só agrega? Pode ler as tabelas dos outros? | [architecture.md §4](../architecture/architecture.md) | ✅ |
| 7 | Como evitar que `ai` vire o "módulo Deus"? | [architecture.md §4](../architecture/architecture.md) | ✅ |
| 8 | Ordene os atributos de qualidade. O que você sacrifica primeiro? | [requirements.md §3](../architecture/requirements.md) | ⬜ |
| 9 | Que sinal concreto justificaria extrair um microservice? | [ADR-001](../adr/ADR-001-modular-monolith.md) | ⬜ |
| 10 | O notebook aguenta rodar um LLM local (Ollama)? | [vision.md §6](../architecture/vision.md) | ✅ |

## Decisões arquiteturais deste card
- [ ] ADR-001 — Modular Monolith (completar alternativas e consequências)
- [ ] _(desafio)_ ADR-002 — Usuário único vs. multiusuário

## Entregáveis
- [ ] `README.md` — revisado
- [ ] `docs/architecture/vision.md` — perguntas 1, 2, 3 e 10 + não-objetivos
- [ ] `docs/architecture/requirements.md` — user stories do MVP + atributos de qualidade
- [ ] `docs/architecture/architecture.md` — tabela "é dono de" + matriz de dependências
- [ ] `docs/diagrams/c4.md` — revisado
- [ ] `docs/adr/ADR-001-modular-monolith.md` — completo
- [ ] `docs/architecture/roadmap.md` — MVP bem delimitado

## Erros comuns
- Detalhar as fases 6 a 10 agora. Elas vão mudar, então só esboce.
- Diagramas bonitos que não registram nenhuma decisão.
- Um ADR que só descreve a escolha, sem as alternativas nem o que você perde com ela.
- Pensar módulos como camadas. Módulos são **capacidades de negócio**.
- Um MVP inflado. Se tudo é MVP, nada é.

## Dicas
- Documento vivo é melhor que documento perfeito.
- Os diagramas estão em Mermaid para evoluírem junto com o código. O GitHub renderiza Mermaid, inclusive C4.
- Para cada módulo, escreva: *"Este módulo é dono de ___ e ninguém mais altera ___."*

## Desafios extras
- Escrever o ADR-002 (usuário único vs. multiusuário).
- Definir um SLO caseiro e um RPO (quanto de dado você aceita perder).

## Critérios de aceite
- [ ] As 10 perguntas estão respondidas.
- [ ] Todo módulo tem um dono claro e dependências explícitas, **sem ciclos**.
- [ ] O ADR-001 tem pelo menos 2 alternativas rejeitadas, com os motivos.
- [ ] O MVP está fechado, com uma lista explícita do que fica fora.

## Definition of Done
- [ ] Tudo commitado num repositório Git.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
Ao terminar, envie os documentos para uma architecture review: contradições entre decisões, o que ficou implícito e os riscos não percebidos.

_Anotações do review:_
