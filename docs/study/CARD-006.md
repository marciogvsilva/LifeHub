# Estudo — CARD-006

Guia de apoio para o [CARD-006 — Testes de integração (Testcontainers) + CI no GitHub Actions](../cards/CARD-006.md).

- A **Parte 1** resume os conceitos do card, com exemplos do LifeHub.
- A **Parte 2** apoia cada pergunta: o que considerar, as opções e o que cada uma implica. **Não traz a resposta.**
- A **Parte 3** é sua: o que aprendeu e por quê.

---

# Parte 1 — Conceitos

## Pirâmide de testes
```text
        /\        Ponta a ponta: poucos, lentos, frágeis, mais realistas
       /  \
      /----\      Integração: banco real, contexto Spring, HTTP
     /      \
    /--------\    Unidade: muitos, rápidos, isolados
```
- **Unidade:** uma classe ou função, sem Spring nem banco. Milissegundos.
- **Integração:** várias peças juntas (repositório + banco, controller + segurança). Segundos.
- **Ponta a ponta:** o sistema inteiro, pelo navegador.

> 💡 A pirâmide é sobre **proporção e custo**, não uma regra. Num sistema sem regra de negócio (como agora), quase tudo o que existe para testar é integração.

## Testcontainers
Sobe containers Docker **durante os testes** e os descarta no fim.

No Spring Boot, `@ServiceConnection` liga o container à aplicação: o Boot lê a URL, o usuário e a senha do container e configura o `DataSource` sozinho, sem propriedades à mão.

```java
// formato geral, não a sua solução
@Container
@ServiceConnection
static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:<mesma versão do compose>");
```

> ⚠️ Use **a mesma versão** de imagem do `compose.yaml`. Testar com uma versão e rodar com outra recria o problema que o Testcontainers resolve.

**Requisito:** Docker acessível. No WSL, o Docker precisa estar rodando no ambiente onde os testes executam. No GitHub Actions, o `ubuntu-latest` já tem Docker.

## Surefire × Failsafe
| | Surefire | Failsafe |
|---|---|---|
| Fase do Maven | `test` | `integration-test` e `verify` |
| Arquivos (padrão) | `*Test`, `*Tests` | `*IT` |
| Se um teste falha | Interrompe o build na hora | Termina a fase `post-integration-test` (limpeza) e falha no `verify` |

`./mvnw test` roda só o Surefire; `./mvnw verify` roda os dois.

## Cobertura com JaCoCo
Mede **quais linhas e ramos foram executados** durante os testes.

> ⚠️ Executar não é verificar. Um teste sem `assert` que chama todo o código dá 100% de cobertura e não protege nada.

## GitHub Actions
```text
.github/workflows/backend.yml
└── workflow  (on: push, pull_request)
    └── job   (runs-on: ubuntu-latest)
        └── steps
            ├── actions/checkout
            ├── actions/setup-java   (java-version, distribution, cache: maven)
            └── run: ./mvnw verify
```
- **Cache:** `setup-java` e `setup-node` têm a opção `cache` para não baixar as dependências a cada execução.
- **Filtro por caminho:** `on.push.paths` roda o workflow só quando certos arquivos mudam.
- **`working-directory`:** necessário se o projeto não estiver na raiz (CARD-002).

## Vitest e Testing Library
- **Vitest:** executor de testes integrado ao Vite (mesma configuração, mesmo TypeScript).
- **Testing Library:** testa componentes **como o usuário os vê**: procura textos e papéis (`getByRole`), não classes CSS.
- Para simular o backend: *mock* do `fetch` ou uma biblioteca como o MSW (Mock Service Worker).

---

# Parte 2 — Apoio às perguntas

### 1. Por que não H2?
**Liste o que o LifeHub já usa ou vai usar do PostgreSQL e verifique se o H2 tem:**
- `CREATE SCHEMA` e o comportamento do `search_path` (CARD-004);
- changesets em **SQL formatado** com sintaxe do PostgreSQL, se você escolheu esse formato;
- `tsvector` e busca full-text (CARD-018);
- `pgvector` (F7);
- funções em PL/pgSQL;
- tipos como `jsonb` e `uuid`.

**Pergunta-teste:** um teste que passa no H2 e falha no PostgreSQL protege o quê?

### 2. Separar unidade e integração?
**O que considerar:**
- Quanto tempo leva o `./mvnw test` hoje? E com um container subindo?
- Você roda testes enquanto programa? Qual comando você quer que seja rápido?
- No CI, o tempo importa menos, mas a **ordem** importa: falhar rápido nos testes de unidade economiza minutos.
- Hoje quase não há teste de unidade. A separação é para agora ou para quando houver regra de negócio?

### 3. Cobertura mínima?
| | A favor | Contra |
|---|---|---|
| Exigir um número (ex.: 80%) | Evita regressão de cobertura; simples de verificar | Incentiva testes vazios para bater a meta; pune código trivial não testado |
| Não exigir | Foco em testar o que importa | Depende de disciplina |

**Alternativas ao número global:** exigir só em pacotes de domínio; usar a cobertura como **informação** no PR, sem bloquear. Lembre-se: "testes proporcionais ao risco" é um princípio da [visão](../architecture/vision.md).

### 4. Quando o CI roda?
**O que considerar:**
- Se você trabalha direto na branch principal (CARD-002), só rodar em PR não roda nunca.
- Push **e** PR na mesma branch rodam duas vezes.
- **Proteção de branch:** o GitHub pode exigir CI verde antes do merge. Faz sentido se você usa PRs.

### 5. Um workflow ou dois?
| | Um workflow | Dois workflows (com filtro por caminho) |
|---|---|---|
| Simplicidade | Maior | Menor |
| Tempo quando só o backend muda | Roda tudo | Roda só o backend |
| Badge no README | Um | Dois |
| Mudança que toca os dois | Natural | Os dois rodam |

### 6. Acelerar o Testcontainers
**Padrões para investigar:**
- **Container estático compartilhado:** um container para todas as classes de teste (padrão "singleton container").
- **Uma classe base ou configuração de teste reutilizável** com o `@ServiceConnection`.
- **Cache de contexto do Spring:** testes com a mesma configuração reaproveitam o mesmo `ApplicationContext`. Configurações diferentes (`@MockitoBean` diferentes, por exemplo) criam contextos novos, e cada um pode subir outro container.
- **Reuso entre execuções** (`testcontainers.reuse.enable`): só para a sua máquina, nunca no CI.

---

# Parte 3 — O que aprendi (preencher ao longo do card)

## Decisões e por quê

## Insights

## Dúvidas para o review
