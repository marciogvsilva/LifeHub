# Estudo — CARD-000

Anotações, insights e explicações sobre as decisões tomadas durante o CARD-000. Os documentos oficiais registram **o quê** foi decidido; este arquivo registra **por quê** e o que aprendi no caminho.

---

## 1. Por que a aplicação não subia

### O sintoma
```
APPLICATION FAILED TO START
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
Reason: Failed to determine a suitable driver class
```

### A explicação: auto-configuration
O Spring Boot olha o **classpath** e configura sozinho o que encontra.

- Com `spring-boot-starter-data-jpa` e o driver do PostgreSQL no `pom.xml`, ele conclui: "essa aplicação usa banco", e tenta criar um `DataSource` (o pool de conexões, HikariCP) já na inicialização.
- Para isso, precisa de `spring.datasource.url`, usuário e senha. Como o `application.properties` só tinha `spring.application.name`, ele não sabia onde conectar.
- Ele ainda tenta um banco embarcado (H2, HSQL, Derby) como plano B. Como nenhum estava no classpath, falhou.

> 💡 **Insight:** colocar uma dependência no `pom.xml` não é neutro no Spring Boot. Cada starter ativa comportamento. Dependência que ainda não vai ser usada é melhor ficar de fora (ou comentada).

### A decisão
Comentei as dependências de banco (`data-jpa`, `data-jpa-test` e `postgresql`) até o CARD-004, em que o PostgreSQL entra com Docker Compose e Liquibase. A aplicação passou a subir.

### O que ainda acontece: a aplicação sobe e encerra
Sem `spring-boot-starter-webmvc` (ou `web`), não existe servidor HTTP (Tomcat) segurando o processo. O Spring inicia o contexto, não tem nada para esperar e termina. Isso **não é erro**. Quando o primeiro endpoint for criado, o starter web entra.

> 💡 **Como ler o log:** `Started LifehubApplication in 0.499 seconds` significa que o contexto subiu com sucesso. Se o processo termina logo depois sem stack trace, é falta de algo que o mantenha vivo, não uma falha.

### Detalhe: `./mvnw: Permission denied`
O script do Maven Wrapper veio sem permissão de execução, comum quando o projeto é gerado ou copiado pelo Windows no WSL. Soluções:
- local: `chmod +x mvnw`;
- no Git, para quem clonar: `git update-index --chmod=+x mvnw` depois do `git add`.

---

## 2. `.gitignore` e Maven Wrapper

### O que entrou no `.gitignore` e por quê
| Entrada | Por quê |
|---|---|
| `target/` | Saída do build. É regenerada a cada compilação. |
| `.idea`, `.vscode/`, `.settings`... | Configurações pessoais de IDE. Cada dev tem as suas. |
| `.env`, `application-local.*` | **Segredos.** Senha de banco não vai para o repositório nunca. |
| `*.log`, `logs/` | Gerados em tempo de execução. |
| `.DS_Store`, `Thumbs.db`, `*:Zone.Identifier` | Lixo do sistema operacional. O último aparece no WSL ao copiar arquivos do Windows. |

> 💡 **Regra geral:** versiona-se o que é **fonte** (escrito por pessoas e necessário para reconstruir o projeto). Ignora-se o que é **gerado**, **pessoal** ou **secreto**.

### Por que `.mvn/` **não** vai para o `.gitignore`
Minha dúvida foi se `.mvn/` deveria ser ignorada. Não deve:
- `.mvn/wrapper/maven-wrapper.properties` diz ao `mvnw` **qual versão do Maven** baixar (aqui, 3.9.16);
- sem ele, `./mvnw` não funciona para quem clonar o repositório;
- é isso que garante que eu, o CI e qualquer outra pessoa usem **o mesmo Maven**, sem instalar nada.

O que se ignora é só o `.mvn/wrapper/maven-wrapper.jar` (binário), e neste projeto ele nem existe, porque o wrapper usa o modo `only-script`.

---

## 3. Modular Monolith: o que aprendi

### Módulo ≠ camada
- **Camada** (`controller/`, `service/`, `repository/`) organiza por **tipo técnico** de código. Com 10 funcionalidades, tudo fica misturado em cada pasta, e qualquer classe acessa qualquer outra.
- **Módulo** (`tasks/`, `calendar/`, `notes/`) organiza por **capacidade de negócio**. Cada um tem uma API pública e uma parte interna que ninguém de fora toca.

### O teste do "é dono de"
*"Este módulo é dono de ___ e ninguém mais altera ___."*

Minhas primeiras respostas descreviam **o que o módulo faz** ("faz tudo relacionado aos usuários"). O teste pede **quais dados ele possui**. A diferença importa: ser dono de um dado significa que **só aquele módulo escreve nele**. É isso que evita dois módulos disputando a mesma tabela.

> 💡 O `dashboard` quase não tem dados próprios: ele **mostra** dados dos outros. O que ele possui é só a configuração das visões personalizadas.

### Por que Modular Monolith (ADR-001)
- **Contra microservices:** operar vários serviços num notebook de 8 GB, com rede, consistência distribuída e debugging espalhado, consome o tempo que deveria ir para o domínio. É "o próximo nível", não o ponto de partida.
- **Contra camadas:** o hub vai ganhar funcionalidades aos poucos, e o desacoplamento entre módulos é o que permite isso sem virar uma "big ball of mud".
- **O preço:** os limites só existem se forem **verificados** (Spring Modulith ou ArchUnit, no CARD-003). Sem ferramenta, a disciplina se perde.

---

## 4. A matriz de dependências

### Os erros da primeira versão
A primeira matriz tinha todos os módulos dependendo só de `users`. Na revisão apareceram problemas:

1. **`auth` sem dependências.** Para validar o login, ele precisa buscar as credenciais em `users`. Era a dependência mais óbvia, e estava faltando.
2. **`dashboard` só em `users`.** Mas a função dele é agregar `tasks`, `calendar` e `notes`.
3. **`ai` só em `users`.** A própria seção 6 do documento dizia que as tools chamam `tasks` e `calendar`. **A matriz contradizia o diagrama.**
4. **Todos dependendo de `users`.** Na maioria dos casos, o módulo só precisa do `userId` do usuário logado, e isso vem do Spring Security, não do módulo `users`.

> 💡 **Insight:** "saber quem é o dono do dado" não é o mesmo que "depender do módulo de usuários". Depende de `users` só quem precisa de **dados** do usuário (fuso horário para o calendário, e-mail para notificações).

### As regras que ficaram
- Ninguém depende de `auth`, `dashboard`, `ai` ou `notifications`: eles ficam nas **pontas** do grafo.
- `audit` não depende de ninguém: só recebe registros.
- Todos podem usar `shared`, que não pode ter regra de negócio.
- **Sem ciclos.** Se A depende de B e B de A, os dois viram um módulo só, na prática.

### Dependências que vieram das minhas respostas
- **Pergunta 4:** tarefa com data aparece no calendário e gera notificação. Por isso `calendar → tasks` e `notifications` escuta eventos de `tasks`.
- **Pergunta 5:** notas referenciam tarefas (link estilo Markdown, abrindo num popover). Por isso `notes → tasks` (pós-MVP). A direção importa: a nota conhece a tarefa, a tarefa não sabe que foi citada.

### API síncrona × eventos
| Use… | Quando… | Exemplo |
|---|---|---|
| **API síncrona** | quem chama precisa da resposta **agora** | `dashboard` montando a tela; `auth` validando login |
| **Eventos** | quem emite **não precisa saber** quem reage | `tasks` publica "tarefa criada"; `notifications` decide se notifica |

> 💡 Com eventos, criar uma tarefa não falha porque o e-mail falhou, e `tasks` nem sabe que `notifications` existe. O custo é a consistência eventual e um fluxo mais difícil de seguir no debug.

### Como evitar o "módulo Deus" (`ai`)
`ai` depende de quase todos os módulos, e isso é aceitável **se ele usar só as APIs públicas**. Assim, as regras de negócio e a autorização continuam valendo, mesmo quando é a IA agindo. Uma alternativa futura é **inverter a dependência**: cada módulo registra suas tools numa interface definida em `ai`. O acoplamento não some, só troca de lado. Por isso é uma decisão para registrar num ADR na fase de IA.

---

## 5. Decisões de produto e contexto

### Multiusuário (ADR-002)
É a decisão **mais cara de mudar depois**: afeta toda tabela (`user_id`), a autenticação, a auditoria e o RAG. Escolhi multiusuário porque o projeto é portfólio e deve ir para a internet.

> ⚠️ **Maior risco:** uma consulta esquecer o filtro por `user_id` e mostrar dados de outra pessoa. Mitigações: o `userId` vem sempre do contexto de segurança (nunca do corpo da requisição), testes de isolamento e, talvez, Row Level Security no PostgreSQL. No RAG, o filtro por usuário tem que acontecer **antes** de montar o contexto do modelo.

### Duas máquinas, dois papéis
| Máquina | Hardware | Papel |
|---|---|---|
| Notebook | 8 GB de RAM, Celeron de 2ª geração | **Servidor** (produção) |
| PC | 32 GB de RAM, Ryzen 5 5600G, RX 6600 | **Desenvolvimento** e Ollama |

Consequências:
- **LLM local só no PC.** No servidor, a IA provavelmente vai usar um provider na nuvem. A **abstração de provider** permite trocar por configuração, sem mudar código.
- **8 GB é pouco para a stack completa.** PostgreSQL e Spring Boot cabem. RabbitMQ, Prometheus, Grafana e Loki juntos apertam. Reforça a regra do roadmap: cada tecnologia entra quando resolve um problema real.
- **RX 6600 e Ollama:** a placa não tem suporte oficial no ROCm (a plataforma da AMD para rodar na GPU). No Linux, costuma funcionar com `HSA_OVERRIDE_GFX_VERSION=10.3.0`.

### Hospedagem: notebook agora, decisão com dados depois
A decisão foi manter no notebook, otimizando para caber nele, e decidir sobre a nuvem (AWS) **depois de testes**. Para os testes decidirem alguma coisa, é preciso definir antes o que é "rodar bem". Daí as metas em `requirements.md` §3:
- p95 abaixo de 500 ms nas telas principais;
- stack do MVP em até 4 GB;
- RPO de 24 h (quanto dado posso perder);
- RTO de 1 h (quanto tempo levo para restaurar).

> 💡 **SLO, RPO e RTO:** SLO é a meta de qualidade do serviço. RPO (*Recovery Point Objective*) é quanto dado você aceita perder, o que define a frequência do backup. RTO (*Recovery Time Objective*) é quanto tempo você aceita ficar fora do ar até restaurar.

### Ordem dos atributos de qualidade
1. Segurança → 2. Durabilidade → 3. Manutenibilidade → 4. Custo → 5. Performance → 6. Disponibilidade.

> 💡 Ordenar é escolher o que **sacrificar**. Disponibilidade é a primeira: o notebook desligado é aceitável, **desde que nenhum dado nem lembrete se perca**. Perder uma nota é pior do que ficar fora do ar por algumas horas.

---

## 6. Lições gerais do card

- **Documento que se contradiz é pior que documento incompleto.** O maior valor da revisão foi achar a matriz dizendo uma coisa e o diagrama da IA dizendo outra.
- **Decisão sem "por quê" não é decisão.** Um ADR que não lista as alternativas rejeitadas e o que se perde é só uma descrição.
- **Fatos antes de opiniões.** Descobri que o hardware do servidor era bem diferente do que eu tinha escrito, e isso mudou a resposta sobre IA local.
- **Complexidade conquistada, não adicionada.** Até comentar as dependências de banco foi aplicar isso: o banco volta quando o card dele chegar.

## 7. O que o segundo review ensinou

### Evento de domínio não é relógio
Eu tinha escrito que `notifications` reage aos eventos de `tasks`. Mas o evento acontece quando a tarefa é **criada**, e o lembrete precisa disparar quando ela **vence**. Nesse momento, nenhum evento acontece: alguém precisa de um relógio.

A solução: `notifications` guarda cada lembrete com a hora de disparo e a marca "já enviado", e **um único job** envia os vencidos. De brinde, a recuperação ao religar o notebook sai de graça: o que venceu desligado continua "não enviado".

> 💡 **Insight:** eventos dizem **o que mudou**; jobs agendados dizem **que horas são**. Funcionalidades baseadas em tempo quase sempre precisam dos dois.

### "Depois eu decido" é uma decisão
Deixar a pergunta 6 em aberto ("depende da dificuldade") contradizia três decisões já tomadas: o schema por módulo, a mitigação do ADR-001 e a matriz. Fechei: só APIs públicas. Se a performance exigir, a saída legítima é um **read model** (uma cópia dos dados, própria do dashboard, alimentada por eventos), registrada num ADR, e nunca um `JOIN` entre schemas.

### Multiusuário tem duas perguntas, não uma
O ADR-002 respondeu "os dados são isolados?". Faltava "**quem pode entrar?**". Escolhi cadastro aberto ([ADR-003](../adr/ADR-003-cadastro-aberto.md)), com papéis ADMIN e USER. O preço é a proteção contra abuso: limite de tentativas, cotas por usuário e acesso só pela rede local até as mitigações existirem.

> 💡 **ADR aceito não se edita.** A decisão do cadastro foi para um ADR novo (ADR-003), em vez de alterar o ADR-002.

### Documento e código precisam concordar
- O estudo dizia que o driver do PostgreSQL estava comentado, mas ele estava ativo. Agora está comentado de verdade.
- O estudo citava `git update-index --chmod=+x mvnw`, mas o comando não tinha sido aplicado. Sem ele, o CI do CARD-006 falharia com `Permission denied`. Agora foi aplicado.

### Trade-off consciente: auditoria síncrona
Gravar a auditoria na mesma transação garante que ação e registro andam juntos, mas faz todos os módulos dependerem de `audit`. Alternativa para avaliar no CARD-013: publicar um evento e consumi-lo com `@TransactionalEventListener(phase = BEFORE_COMMIT)`. A transação continua a mesma, sem a dependência direta.

## Pendências para revisar
- [x] Revisar tudo que estava marcado como proposta nos documentos (aprovado).
- [ ] Preencher o disco do notebook (HDD ou SSD?) e as horas por semana (`requirements.md` §4 e §5).
- [ ] Verificar a bateria do notebook antes de deixá-lo ligado 24/7.
- [ ] Na fase de IA, avaliar os dados pessoais enviados ao provider na nuvem (a IA é opcional).
- [ ] Reativar as dependências de banco no CARD-004.
- [ ] Decidir se o backend fica na raiz ou em `backend/` (CARD-002/003).
