# ADR-001 — Utilizar Modular Monolith

- **Status:** Proposto
- **Data:** 2026-09-24

## Contexto

O LifeHub é um projeto pessoal, desenvolvido por uma pessoa, que precisa evoluir continuamente sem introduzir complexidade operacional desnecessária. Ele roda num notebook antigo, com recursos limitados. Ao mesmo tempo, o projeto é um laboratório de aprendizado e deve permitir, no futuro, praticar mensageria, distribuição e extração de serviços.

> ✍️ Complete o contexto com as forças que *você* considera mais relevantes: tamanho da equipe, hardware, tempo disponível, objetivos de aprendizado.

## Decisão

Utilizar um **Modular Monolith**: um único deployable Spring Boot, organizado em módulos por capacidade de negócio, com APIs públicas explícitas, dados privados por módulo e regras de dependência verificadas automaticamente.

## Alternativas consideradas

### A — Microservices desde o início
> ✍️ Por que foi rejeitada? Pense em: operação num notebook, rede entre serviços, consistência de dados, debugging e o tempo gasto em infraestrutura em vez de domínio.

### B — Monólito tradicional em camadas (controller/service/repository)
> ✍️ Por que foi rejeitada? Pense em: o que acontece com o acoplamento quando o sistema tem 10 módulos, e como seria extrair um deles depois.

### C — _(opcional) outra alternativa que você considerou_

## Consequências

### Positivas
- Menor complexidade operacional: um processo e um banco.
- Deploy simples.
- Transações locais: consistência forte sem sagas.
- Refatoração entre módulos ainda é barata.
- Os limites dos módulos preparam uma futura extração de serviços.

### Negativas (o que perdemos)
- Escala apenas como um todo (não por módulo).
- Uma falha grave no processo derruba todos os módulos.
- Os limites dependem de disciplina e de ferramentas. Sem verificação automática, o sistema degrada para uma "big ball of mud".
- _✍️ adicione outras_

### Riscos e mitigação
| Risco | Mitigação |
|---|---|
| Módulos acessarem internals uns dos outros | Spring Modulith ou ArchUnit no build (CARD-003) |
| Módulos lerem tabelas uns dos outros | Tabelas/schemas por módulo; acesso só via API pública |
| `shared` virar depósito de regras de negócio | Regra: `shared` só contém código técnico |
| _✍️_ | |

## Quando revisitar esta decisão

> ✍️ **Pergunta 9 do CARD-000.** Que sinal concreto justificaria extrair um módulo? (Ex.: um módulo com carga ou ciclo de deploy muito diferente dos outros, ou necessidade de isolar falhas de um processamento pesado.)
