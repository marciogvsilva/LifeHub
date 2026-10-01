# Estudo — CARD-002

Guia de apoio para o [CARD-002 — Repositório, convenções e estrutura do monorepo](../cards/CARD-002.md).

- A **Parte 1** resume os conceitos do card, com exemplos do LifeHub.
- A **Parte 2** apoia cada pergunta: o que considerar, as opções e o que cada uma implica. **Não traz a resposta.**
- A **Parte 3** é sua: o que aprendeu e por quê.

---

# Parte 1 — Conceitos

## Monorepo × polyrepo
| | Monorepo | Polyrepo |
|---|---|---|
| Repositórios | Um para backend, frontend e infra | Um por parte |
| Mudança que atravessa partes | Um commit, um PR | Vários PRs coordenados |
| CI | Precisa decidir o que rodar quando só uma parte muda | Cada repositório tem o seu, naturalmente separado |
| Versões | Tudo evolui junto | Cada parte tem a sua versão; precisa de compatibilidade entre elas |

> 💡 Monorepo não é "um projeto só". Dentro dele, backend e frontend continuam com builds independentes (Maven e npm).

## Conventional Commits
Formato: `tipo(escopo opcional): descrição no imperativo`.

```text
feat(tasks): adiciona conclusão de tarefa
fix(auth): retorna erro genérico em credencial inválida
docs(card-002): registra convenção de commits
build: adiciona starter webmvc
```

Tipos mais usados: `feat` (funcionalidade), `fix` (correção), `docs`, `refactor` (sem mudar comportamento), `test`, `build` (dependências, Maven), `ci`, `chore` (manutenção).

> 💡 O ganho real aparece depois: `git log --oneline --grep "^feat"` lista as funcionalidades; ferramentas geram changelog e sugerem a próxima versão SemVer a partir dos tipos.

## Estratégias de branch
| Estratégia | Como funciona | Combina com |
|---|---|---|
| **Trunk-based** | Commits pequenos direto na principal (ou branches de horas) | Pessoa só, CI forte |
| **Branch por card** | Uma branch por card, merge ao concluir | Review por card, histórico agrupado |
| **GitFlow** | `develop`, `release/*`, `hotfix/*`... | Releases formais e várias pessoas; pesado para um projeto solo |

## Como o Git trata arquivos movidos
O Git **não registra** renomeações. Ele guarda só o conteúdo de cada commit. `git mv a b` é um atalho para mover o arquivo e atualizar o índice.

Na hora de mostrar o histórico, o Git **deduz** que houve renomeação comparando o conteúdo dos arquivos apagados e criados (por padrão, a partir de 50% de semelhança). Por isso:
- `git log --follow caminho/novo` segue o arquivo através da renomeação;
- mover **e** reescrever o arquivo no mesmo commit pode quebrar a dedução.

## `.editorconfig`
Um arquivo na raiz que quase todos os editores respeitam (IntelliJ e VS Code com extensão):

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true

[*.java]
indent_style = ...
```

Ele define **como o editor escreve**. Não formata código que já existe.

## `.gitattributes`
Já existe no repositório:
```text
/mvnw text eol=lf
*.cmd text eol=crlf
```
Força o fim de linha **no checkout**, independentemente do sistema. O `mvnw` é um script de shell: com `CRLF` (Windows), o Linux não consegue executá-lo. O `.cmd` é para o Windows, que espera `CRLF`.

> 💡 `.gitattributes` e `.editorconfig` se complementam: um garante o que o Git entrega, o outro, o que o editor escreve.

## SemVer
`MAJOR.MINOR.PATCH`:
- **MAJOR:** quebra compatibilidade;
- **MINOR:** funcionalidade nova compatível;
- **PATCH:** correção.

Enquanto a versão é `0.x`, a especificação diz que **qualquer coisa pode mudar a qualquer momento**: a API pública ainda não é estável. A `1.0.0` é a promessa de estabilidade.

---

# Parte 2 — Apoio às perguntas

### 1. Monorepo ou polyrepo?
**O que considerar:**
- Quantas mudanças vão atravessar backend e frontend ao mesmo tempo? (Um endpoint novo e a tela que o usa.)
- O CI do CARD-006: num monorepo, o GitHub Actions pode filtrar por caminho (`on.push.paths`). Num polyrepo, cada repositório já roda só o seu.
- O deploy do CARD-007 vai subir backend, frontend e banco **juntos** num compose. Onde esse compose mora em cada opção?

**Pergunta-teste:** você consegue imaginar o backend do LifeHub sendo usado por outro frontend, ou o frontend por outro backend?

### 2. Backend na raiz ou em `backend/`?
| | Raiz | `backend/` |
|---|---|---|
| Comandos | `./mvnw ...` direto | `cd backend && ./mvnw ...` ou `./backend/mvnw -f backend/pom.xml` |
| Frontend | Em `frontend/`, convivendo com `src/` e `pom.xml` do backend na raiz | `frontend/` e `backend/` lado a lado, simétricos |
| CI | Sem `working-directory` | Precisa de `working-directory: backend` |
| `.gitignore` | `target/` | Continua funcionando (o padrão vale em qualquer nível) |

**Pergunta-teste:** quem abre o repositório pela primeira vez entende, pela raiz, que existe um frontend?

### 3. Branches e pull requests sozinho
**O que um PR oferece mesmo sem outra pessoa:**
- um lugar para a descrição e o checklist do card;
- o CI rodando **antes** do merge;
- um ponto natural para o review técnico (você manda o link do PR).

**O custo:** mais passos para cada mudança pequena.

### 4. Convenção de commit
**Decisões menores que fazem diferença:**
- **Idioma:** os documentos estão em português. Commits em português combinam; em inglês, combinam com o código (nomes de classes) e com o portfólio internacional.
- **Escopo:** o nome do módulo (`feat(tasks)`) diz *onde*; o card (`docs(card-002)`) diz *quando*. Dá para usar os dois: o card no corpo da mensagem.
- **Commits antigos:** não se reescreve histórico já publicado. A convenção vale daqui para frente.

### 5. Versionamento
**O que considerar:**
- O roadmap chama o MVP de **v1.0**. O que vem antes dele?
- Faz sentido uma `0.1.0` no fim da F1 (sistema vazio no ar) e versões `0.x` a cada fase?
- Versão no `pom.xml` (`0.0.1-SNAPSHOT`) e no `package.json` do frontend: andam juntas ou separadas?
- **Tags do Git** (`git tag v0.1.0`) marcam a versão no histórico.

### 6. `.editorconfig`: tabs ou espaços
**O que considerar:**
- O Initializr gerou o `pom.xml` e as classes Java **com tabs**.
- A maioria dos guias de estilo Java (Google, por exemplo) usa espaços; o código do próprio Spring usa tabs.
- O ecossistema TypeScript/Prettier usa espaços por padrão.
- Mudar de tabs para espaços nos arquivos existentes gera um diff grande, sem mudança real. Se for mudar, faça num commit só para isso.

---

# Parte 3 — O que aprendi (preencher ao longo do card)

## Decisões e por quê
_Para cada pergunta, a decisão tomada e o motivo em uma ou duas linhas._

## Insights

## Dúvidas para o review
