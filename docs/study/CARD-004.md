# Estudo — CARD-004

Guia de apoio para o [CARD-004 — PostgreSQL no Docker Compose + Liquibase](../cards/CARD-004.md).

- A **Parte 1** resume os conceitos do card, com exemplos do LifeHub.
- A **Parte 2** apoia cada pergunta: o que considerar, as opções e o que cada uma implica. **Não traz a resposta.**
- A **Parte 3** é sua: o que aprendeu e por quê.

---

# Parte 1 — Conceitos

## Docker Compose
Descreve **vários containers** e como eles se relacionam num arquivo (`compose.yaml`).

Peças que este card usa:
| Peça | Para quê |
|---|---|
| `image: postgres:<versão>` | A imagem, com versão **fixa** |
| `environment` / `env_file` | Configuração (usuário, senha, nome do banco) |
| `volumes` | Onde os dados sobrevivem quando o container é recriado |
| `ports` | Qual porta do seu computador aponta para a do container |
| `healthcheck` | Um comando que diz se o serviço está **pronto**, não só "rodando" |

> 💡 **Rodando ≠ pronto.** O container do PostgreSQL sobe em um segundo, mas o banco leva mais alguns para aceitar conexões. O `healthcheck` (com `pg_isready`, por exemplo) é o que mede "pronto".

## Volumes
- **Volume nomeado:** gerenciado pelo Docker (`docker volume ls`). Sobrevive a `docker compose down`, mas **não** a `docker compose down -v`.
- **Bind mount:** uma pasta do seu computador montada no container.

## Suporte a Docker Compose do Spring Boot
Com a dependência `spring-boot-docker-compose`, ao iniciar a aplicação o Boot:
- encontra o `compose.yaml`, sobe os serviços e espera ficarem prontos;
- descobre a URL, o usuário e a senha do banco e **configura o `DataSource` sozinho** (*service connection*).

Ela é pensada para **desenvolvimento**: por padrão, não vai para o jar empacotado.

## Liquibase
- **Changelog:** o arquivo (ou árvore de arquivos) com todas as mudanças do banco.
- **Changeset:** uma mudança, identificada por `autor` + `id` + caminho do arquivo.
- **`DATABASECHANGELOG`:** tabela onde o Liquibase registra cada changeset aplicado, com um **checksum** do conteúdo.
- **`DATABASECHANGELOGLOCK`:** impede duas instâncias de migrar ao mesmo tempo.

> ⚠️ **Changeset aplicado não se edita.** O checksum muda e o Liquibase recusa a inicialização (*validation failed*). A correção é um changeset **novo**. A mesma regra dos ADRs aceitos.

Formatos possíveis: XML, YAML, JSON ou **SQL formatado**:
```sql
--liquibase formatted sql

--changeset autor:identificador
-- o SQL da mudança

--rollback -- como desfazer (opcional)
```

## `ddl-auto`
| Valor | O Hibernate... |
|---|---|
| `create` / `create-drop` | apaga e recria as tabelas |
| `update` | tenta alterar as tabelas para bater com as entidades |
| `validate` | só confere se as tabelas batem com as entidades, e falha se não baterem |
| `none` | não faz nada |

## Schemas no PostgreSQL
Um banco tem vários **schemas** (namespaces). `tasks.task` e `calendar.task` seriam tabelas diferentes.
- `CREATE SCHEMA nome;`
- `search_path`: em que schemas o PostgreSQL procura uma tabela quando o nome vem sem prefixo.
- Permissões por schema: `GRANT USAGE ON SCHEMA ... TO ...`.

## UUID × sequência
| | `bigint` + sequência | UUID v4 (aleatório) | UUID v7 (ordenado pelo tempo) |
|---|---|---|---|
| Tamanho | 8 bytes | 16 bytes | 16 bytes |
| Gerado | Pelo banco, ao inserir | Em qualquer lugar | Em qualquer lugar |
| Índice B-tree | Inserção sempre no fim | Inserção espalhada pelo índice | Inserção quase sempre no fim |
| Na URL | Previsível (`/tasks/42` → tente `/tasks/43`) | Imprevisível | Revela a hora de criação |

> 💡 O UUID v7 está padronizado na RFC 9562. Verifique o suporte nativo na versão do PostgreSQL e do Hibernate que você vai usar.

## HikariCP
O pool de conexões padrão do Spring Boot. Abrir uma conexão com o banco é caro; o pool mantém algumas abertas e as empresta.

---

# Parte 2 — Apoio às perguntas

### 1. `ddl-auto` com Liquibase
**O que considerar:**
- Se os dois alteram o schema, quem é a **fonte da verdade**? O que aparece no `DATABASECHANGELOG` e o que não aparece?
- O `update` do Hibernate não apaga colunas nem renomeia (renomear vira "criar outra"). O que acontece com os dados?
- Qual valor transforma uma entidade fora de sincronia com o banco num **erro na inicialização** em vez de num erro em produção?
- Enquanto não houver entidades (este card), o valor importa?

### 2. UUID ou sequência?
**Perguntas-teste:**
- Você quer gerar o ID **antes** de salvar? (Útil para publicar o evento `TaskCreated` com o ID, ou para o frontend criar algo otimista.)
- IDs aparecem em URLs (`/api/tasks/{id}`). Com cadastro aberto (ADR-003), um ID previsível é um risco?
- O notebook tem disco possivelmente lento (HDD). Qual opção é mais amigável ao índice?

### 3. Changelogs com um schema por módulo
**Estruturas possíveis:**
```text
db/changelog/
├── db.changelog-master.(xml|yaml)   ← inclui os demais
├── 000-schemas/                     ← cria todos os schemas
├── tasks/
├── calendar/
└── ...
```
ou cada módulo cria o próprio schema no primeiro changeset dele.

**O que considerar:**
- A ordem importa? O `master` inclui na ordem em que você listar (ou por nome, com `includeAll`).
- Se um dia um módulo for extraído (ADR-001), o que é mais fácil levar junto?
- Onde ficam as tabelas do próprio Liquibase (`DATABASECHANGELOG`)? Investigue `spring.liquibase.liquibase-schema` e `default-schema`.

### 4. YAML, XML ou SQL formatado?
| | XML/YAML | SQL formatado |
|---|---|---|
| Independente do banco | Sim (o Liquibase gera o SQL) | Não |
| Recursos do PostgreSQL (tsvector, pgvector, PL/pgSQL) | Precisam de `<sql>` embutido | Naturais |
| Rollback automático | Para muitas operações | Você escreve |
| Aprendizado de SQL | Escondido | Explícito |

**Pergunta-teste:** o LifeHub vai trocar de banco algum dia? (A stack e o roadmap dizem PostgreSQL, pgvector e PL/pgSQL.)

### 5. Suporte a Docker Compose do Boot ou manual?
| | Suporte do Boot | Manual (`docker compose up`) |
|---|---|---|
| Rodar a aplicação | Um comando | Dois |
| Configuração do `DataSource` em dev | Automática | Você escreve |
| Deploy (CARD-007) | Não se aplica (só dev) | O mesmo compose pode servir de base |
| Entendimento | Mágica a investigar | Explícito |

### 6. Um usuário de banco ou um por schema?
**O que considerar:**
- Com um usuário por schema, o **PostgreSQL** recusa um `SELECT` de `dashboard` em `tasks.task`. A regra deixa de depender só do teste de modularidade.
- O custo: uma aplicação só (monólito) com **vários** `DataSource`s, um por módulo. Como as transações funcionam entre eles?
- Meio-termo: um usuário para a aplicação, mas sem permissão de `DDL` (só o Liquibase cria tabelas, com outro usuário). Que problema isso resolve?

### 7. Tamanho do pool no notebook
**Onde investigar:** a página "About Pool Sizing" do HikariCP, que explica por que pools **menores** costumam ser mais rápidos.

**O que considerar:**
- Quantos núcleos tem o Celeron? Quantas consultas o banco consegue executar **de verdade** ao mesmo tempo?
- Cada conexão consome memória no PostgreSQL. Qual o impacto na meta de 4 GB?
- Quantos usuários simultâneos o LifeHub terá na prática?

---

# Parte 3 — O que aprendi (preencher ao longo do card)

## Decisões e por quê

## Insights

## Dúvidas para o review
