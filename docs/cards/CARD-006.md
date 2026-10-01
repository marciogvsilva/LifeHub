# CARD-006 — Testes de integração (Testcontainers) + CI no GitHub Actions

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | **Código e automação.** |
| Status | ⬜ A fazer |
| Timebox | 2 sessões |

## Objetivo
Testar o backend contra um PostgreSQL de verdade e fazer cada push rodar, automaticamente, o build e os testes do backend e do frontend.

## Por que esse card existe
Daqui em diante, cada card adiciona código. Sem CI, um teste quebrado só aparece quando você lembra de rodar. E testar banco com H2 esconde diferenças reais do PostgreSQL (tipos, schemas, funções), justamente o banco que o projeto escolheu.

## Pré-requisitos
- CARD-005 concluído.
- Repositório no GitHub.

## Conceitos para estudar
- **Pirâmide de testes:** unidade, integração, ponta a ponta; custo e velocidade de cada um.
- **Testcontainers:** containers descartáveis nos testes; `@ServiceConnection` no Spring Boot.
- **Surefire vs. Failsafe:** `*Test` vs. `*IT`, fases `test` e `verify` do Maven.
- **Cobertura com JaCoCo:** o que a porcentagem mede e o que ela não mede.
- **GitHub Actions:** workflow, job, step, `actions/setup-java`, `actions/setup-node`, cache de dependências.
- **Vitest e Testing Library:** testes de componentes React.

## Perguntas que você precisa responder
1. **Por que não usar H2 nos testes,** se ele é mais rápido? Cite uma diferença real com o PostgreSQL que afetaria o LifeHub (pense nos schemas por módulo).
2. **Separar testes de unidade e de integração** (Surefire e Failsafe) ou rodar tudo junto? O que muda no tempo de feedback?
3. **Exigir cobertura mínima?** Se sim, quanto, e por que esse número? Se não, como garantir que o código importante está testado?
4. **Quando o CI roda?** Todo push, só pull requests ou os dois? E a branch principal deve aceitar merge com CI vermelho?
5. **Um workflow ou dois** (backend e frontend)? Faz sentido rodar o do frontend quando só o backend mudou?
6. **Como acelerar os testes com Testcontainers** sem subir um container por classe de teste?

## Implementação
1. Adicionar Testcontainers e escrever um teste de integração que sobe o PostgreSQL, deixa o Liquibase aplicar as migrations e verifica que os schemas dos módulos existem.
2. Garantir que o teste de modularidade do CARD-003 roda no `verify`.
3. Configurar Vitest e escrever um teste da tela de status do CARD-005 (dados e erro).
4. Criar o(s) workflow(s) em `.github/workflows/`: Java 21, `./mvnw verify`, Node, lint, build e testes do frontend, com cache.
5. Adicionar o badge do CI ao `README.md`.
6. _(opcional)_ Proteger a branch principal exigindo CI verde.

## Testes
```text
migrations_should_create_module_schemas
modules_should_respect_boundaries
status_page_should_show_backend_info
status_page_should_show_error_when_backend_is_down
```

## Documentação — entregáveis
- [ ] `README.md` — badge do CI e como rodar os testes.
- [ ] `docs/study/CARD-006.md` — o que você aprendeu e por quê.

## Erros comuns
- `mvnw` sem permissão de execução: o CI falha com `Permission denied` (corrigido no CARD-000; confirme que continua `100755`).
- Testes que dependem da ordem de execução ou de dados de outro teste.
- Perseguir 100% de cobertura testando getters.
- Esconder testes lentos com `@Disabled` em vez de resolver.

## Critérios de aceite
- [ ] As 6 perguntas estão respondidas.
- [ ] O teste de migrations roda contra um PostgreSQL real.
- [ ] Um push dispara o CI, que fica verde.
- [ ] Um teste quebrado de propósito deixa o CI vermelho (e depois é corrigido).

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
