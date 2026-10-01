# CARD-004 — PostgreSQL no Docker Compose + Liquibase

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | **Código e infraestrutura.** |
| Status | ⬜ A fazer |
| Timebox | 2 sessões |

## Objetivo
Subir o PostgreSQL com Docker Compose, reativar as dependências de banco comentadas desde o CARD-000 e criar, pelo Liquibase, a estrutura de **um schema por módulo**.

## Por que esse card existe
O princípio 3 da visão diz que "o banco faz parte da arquitetura": toda mudança estrutural passa pelo Liquibase. Este card cria esse caminho antes da primeira tabela de negócio, para que nenhuma tabela nasça "na mão".

## Pré-requisitos
- CARD-003 concluído.
- Docker funcionando no WSL.

## Conceitos para estudar
- **Docker Compose:** serviços, volumes nomeados, `healthcheck`, variáveis de ambiente.
- **Suporte a Docker Compose do Spring Boot:** o Boot pode subir o `compose.yaml` sozinho ao iniciar a aplicação.
- **Liquibase:** changelog, changeset, a tabela `DATABASECHANGELOG`, e por que um changeset aplicado nunca é editado.
- **`spring.jpa.hibernate.ddl-auto`:** `none`, `validate`, `update`, `create`.
- **Schemas no PostgreSQL:** `CREATE SCHEMA`, `search_path`, permissões por schema.
- **HikariCP:** o pool de conexões e o seu tamanho.
- **Segredos fora do código:** variáveis de ambiente e o `.env` que já está no `.gitignore`.

## Perguntas que você precisa responder
1. **Qual valor de `ddl-auto` com Liquibase, e por quê?** O que dá errado se o Hibernate e o Liquibase mexerem no schema ao mesmo tempo?
2. **Chave primária: UUID ou sequência (`bigint`)?** Pense em índices, em IDs expostos na URL e em gerar o ID antes de salvar (útil para eventos). O que é UUIDv7?
3. **Como organizar os changelogs com um schema por módulo?** Um changelog mestre que inclui um por módulo? Quem cria o schema: o próprio módulo ou um changeset inicial?
4. **Changelog em YAML, XML ou SQL formatado?** O que você ganha e perde escrevendo SQL puro?
5. **Subir o banco pelo suporte a Docker Compose do Boot ou manualmente?** O que cada opção muda no dia a dia e no deploy do CARD-007?
6. **Um usuário de banco para a aplicação toda ou um por schema?** Qual torna a regra "ninguém mexe nas tabelas dos outros" verificável pelo próprio PostgreSQL?
7. **Qual tamanho de pool faz sentido no notebook** (Celeron, 8 GB)? Mais conexões é sempre melhor?

## Decisões arquiteturais deste card
- [ ] Registrar as convenções de banco (chaves, nomes, changelogs) em `docs/database/README.md`.
- [ ] Se a pergunta 6 levar a usuários por schema, registrar num ADR.

## Implementação
1. Criar o `compose.yaml` com o PostgreSQL: versão fixa (nada de `latest`), volume nomeado e `healthcheck`.
2. Usuário e senha por variáveis de ambiente, com um `.env.example` versionado e o `.env` real ignorado.
3. Reativar no `pom.xml` as dependências de banco comentadas e adicionar o Liquibase.
4. Criar o changelog mestre e a primeira migration: **os schemas dos módulos do MVP**. Nenhuma tabela de negócio ainda.
5. Configurar `ddl-auto` conforme a pergunta 1.

## Testes
- O `contextLoads` continua passando (com o banco no ar).
- Testes de integração com Testcontainers ficam para o CARD-006.

## Documentação — entregáveis
- [ ] `docs/database/README.md` — convenções decididas neste card.
- [ ] `README.md` — "Como rodar" atualizado com o banco.
- [ ] `docs/study/CARD-004.md` — o que você aprendeu e por quê.

## Erros comuns
- Imagem `postgres:latest`: o banco muda de versão sozinho num `docker pull`.
- Senha no `application.properties` commitado.
- Editar um changeset já aplicado em vez de criar um novo.
- Deixar o Hibernate criar tabelas "só para testar".
- Sem volume: `docker compose down` apaga os dados.

## Critérios de aceite
- [ ] As 7 perguntas estão respondidas.
- [ ] `docker compose up` sobe o banco, e o healthcheck fica `healthy`.
- [ ] A aplicação sobe conectada ao banco e o Liquibase aplica a migration.
- [ ] `\dn` no `psql` lista os schemas dos módulos.
- [ ] Nenhum segredo está versionado.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
