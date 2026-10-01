# CARD-002 — Repositório, convenções e estrutura do monorepo

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | Configuração do repositório. Pouco código, muita organização. |
| Status | ⬜ A fazer |
| Timebox | 1 sessão |

## Objetivo
Deixar o repositório pronto para receber backend, frontend e infraestrutura lado a lado, com convenções que mantêm o histórico legível daqui a 30 cards.

## Por que esse card existe
O histórico do Git é documentação. `git log` com "ajustes", "subindo próximo card" e "correções" não conta nada daqui a seis meses. Também é agora, com quase nenhum código, que mover pastas custa menos.

## Pré-requisitos
- CARD-000 concluído.

## Conceitos para estudar
- **Monorepo vs. polyrepo:** um repositório para tudo ou um por parte do sistema.
- **Conventional Commits:** `tipo(escopo): descrição` (`feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `ci`, `build`).
- **Estratégia de branches:** trunk-based development vs. GitFlow vs. uma branch por card.
- **`git mv` e `git log --follow`:** como o Git detecta arquivos movidos e como seguir o histórico deles.
- **`.editorconfig`:** encoding, fim de linha e indentação iguais em qualquer editor.
- **`.gitattributes`:** o que o `text eol=lf` que já existe no repositório faz.
- **SemVer:** `MAJOR.MINOR.PATCH`, e o que significa estar em `0.x`.

## Perguntas que você precisa responder
1. **Monorepo ou polyrepo?** O que cada opção muda para o CI (CARD-006) e para o deploy (CARD-007)?
2. **Backend na raiz ou em `backend/`?** Hoje o projeto Maven está na raiz, mas o README promete `backend/`. Onde o frontend do CARD-005 vai morar em cada opção?
3. **Como você vai trabalhar com branches?** Sozinho, abrir pull request faz sentido? O que um PR oferece além do merge (pense no review técnico de cada card)?
4. **Qual convenção de commit?** Em que idioma? Quais escopos (os módulos? os cards?)?
5. **Como versionar?** Quando sai a `v0.1.0`? E a `v1.0.0` (dica: o roadmap chama o MVP de v1.0)?
6. **O que vai no `.editorconfig`?** O `pom.xml` do Initializr usa tabs. Java com tabs ou espaços? E o TypeScript do CARD-005?

## Decisões arquiteturais deste card
- [ ] Se a resposta da pergunta 1 ou 2 tiver consequências relevantes para o deploy, registrar num ADR-004.

## Implementação
1. Aplicar a decisão da pergunta 2. Se mover, use `git mv` e confirme que o histórico continua com `git log --follow`.
2. Criar o `.editorconfig`.
3. Revisar o `.gitignore` para a estrutura nova (os caminhos `target/` e `!**/src/main/**/target/` continuam certos?).
4. Adicionar ao `README.md` uma seção **"Como rodar"** com, no máximo, três comandos.
5. _(opcional)_ Criar `.github/pull_request_template.md` com um checklist do card (critérios de aceite, DoD).
6. Registrar a convenção de commits e de branches num `CONTRIBUTING.md` curto.

## Testes
- `./mvnw verify` passa a partir do novo local do projeto.

## Documentação — entregáveis
- [ ] `README.md` — estrutura atualizada e seção "Como rodar".
- [ ] `CONTRIBUTING.md` — commits, branches, versionamento.
- [ ] `docs/study/CARD-002.md` — o que você aprendeu e por quê.

## Erros comuns
- Mover e editar muito os mesmos arquivos no mesmo commit: o Git identifica renomeações por semelhança de conteúdo e pode deixar de ligar o histórico. Mova num commit e edite no seguinte.
- Criar agora todas as pastas do README (`infra/`, `ansible/`...). O Git não versiona pasta vazia, e o README diz que elas nascem quando o card precisar.
- Adotar uma convenção que você não vai seguir. Melhor simples e real do que completa e ignorada.

## Critérios de aceite
- [ ] As 6 perguntas estão respondidas.
- [ ] `./mvnw verify` passa.
- [ ] `git log --follow` mostra o histórico dos arquivos movidos (se houve mudança).
- [ ] Os commits deste card já seguem a convenção escolhida.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
