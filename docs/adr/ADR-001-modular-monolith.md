# ADR-001 — Utilizar Modular Monolith

- **Status:** Aceito
- **Data:** 2026-09-24

## Contexto

O LifeHub é um projeto pessoal, desenvolvido por uma pessoa, que precisa evoluir continuamente sem introduzir complexidade operacional desnecessária. Ele roda num notebook antigo, com recursos limitados. Ao mesmo tempo, o projeto é um laboratório de aprendizado e deve permitir, no futuro, praticar mensageria, distribuição e extração de serviços.

Forças mais relevantes:
- **Equipe:** uma pessoa, em sessões de estudo. Cada hora gasta em infraestrutura sai do tempo de domínio.
- **Hardware:** o servidor é um notebook com 8 GB de RAM e Celeron de 2ª geração. Uma JVM e um PostgreSQL cabem; várias JVMs, um broker e um service discovery não cabem. O desenvolvimento é feito num PC mais forte.
- **Aprendizado:** o objetivo é praticar limites de módulo, DDD e, mais tarde, mensageria e extração de serviços, com cada complexidade chegando quando for necessária.
- **Futuro:** a hospedagem pode migrar para a nuvem (AWS), e os limites de módulo mantêm essa porta aberta.

## Decisão

Utilizar um **Modular Monolith**: um único deployable Spring Boot, organizado em módulos por capacidade de negócio, com APIs públicas explícitas, dados privados por módulo e regras de dependência verificadas automaticamente.

## Alternativas consideradas

### A — Microservices desde o início

> ✍️ Por que foi rejeitada? Pense em: operação num notebook, rede entre serviços, consistência de dados, debugging e o tempo gasto em infraestrutura em vez de domínio.

**Por que foi rejeitada:** microservices são um próximo nível, que não se justifica num projeto pequeno e sem necessidade de alta escalabilidade.

### B — Monólito tradicional em camadas (controller/service/repository)

> ✍️ Por que foi rejeitada? Pense em: o que acontece com o acoplamento quando o sistema tem 10 módulos, e como seria extrair um deles depois.

**Por que foi rejeitada:** o monólito modular oferece o desacoplamento que um hub precisa, já que ele terá diversas funcionalidades desenvolvidas aos poucos. Além disso, a arquitetura modular é um interesse de aprendizado que quero preservar.

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
- Todo deploy sobe a aplicação inteira. Uma mudança pequena num módulo reinicia todos.
- Todos os módulos compartilham a mesma JVM e os mesmos 8 GB do notebook. Um módulo pesado (extração de documentos, embeddings) disputa recursos com o resto.
- Uma única stack: todos os módulos usam a mesma versão de Java e Spring.

### Riscos e mitigação
| Risco | Mitigação |
|---|---|
| Módulos acessarem internals uns dos outros | Spring Modulith ou ArchUnit no build (CARD-003) |
| Módulos lerem tabelas uns dos outros | Tabelas/schemas por módulo; acesso só via API pública |
| `shared` virar depósito de regras de negócio | Regra: `shared` só contém código técnico |
| Processamento pesado (extração de texto, embeddings) degradar a aplicação no notebook | Rodar de forma assíncrona e com concorrência limitada; é o primeiro candidato a extração (ver abaixo) |
| Dependências cíclicas surgirem com o tempo | Matriz de dependências em [architecture.md §4](../architecture/architecture.md) verificada no build |

## Quando revisitar esta decisão

> ✍️ **Pergunta 9 do CARD-000.** Que sinal concreto justificaria extrair um módulo? (Ex.: um módulo com carga ou ciclo de deploy muito diferente dos outros, ou necessidade de isolar falhas de um processamento pesado.)

**Resposta:** extrair um módulo só quando houver uma destas evidências, medida e não suposta:
- **Recurso incompatível:** o módulo precisa de um hardware que o servidor não tem. Por exemplo, a IA com modelo local precisaria da GPU do PC.
- **Impacto medido:** um processamento pesado (extração de documentos, embeddings) faz a aplicação sair das metas de desempenho ([requirements.md §3](../architecture/requirements.md)) mesmo depois de otimizado e tornado assíncrono.
- **Ciclo de vida diferente:** o módulo precisa de deploy, escala ou disponibilidade independentes do resto.

Querer praticar microservices é um motivo válido num laboratório. Mas precisa ser registrado como tal, num novo ADR, e não disfarçado de necessidade técnica.
