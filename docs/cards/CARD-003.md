# CARD-003 — Backend por módulo + verificação dos limites

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | **Código.** Primeira aplicação HTTP do LifeHub. |
| Status | ⬜ A fazer |
| Timebox | 2 a 3 sessões |

## Objetivo
Colocar o backend no ar respondendo HTTP, com os módulos do MVP como pacotes Java e as regras de dependência do CARD-000 **verificadas por teste**. Ao final, quebrar uma regra de propósito precisa fazer o build falhar.

## Por que esse card existe
O ADR-001 diz que os limites dos módulos "dependem de disciplina e de ferramentas" e que, sem verificação automática, o sistema vira uma *big ball of mud*. Este card transforma a matriz de dependências de documento em teste.

## Pré-requisitos
- CARD-002 recomendado. As tarefas T1 e T2 não dependem dele: se o projeto for movido para `backend/` depois, um `git mv` leva o código junto.
- CARD-001 recomendado, mas não bloqueante: se a modelagem mudar algum módulo (por exemplo, juntar `auth` e `users`), renomear um pacote vazio custa minutos.

## Conceitos para estudar
- **Starters do Spring Boot 4:** o starter web agora se chama `spring-boot-starter-webmvc`, e cada tecnologia tem o próprio starter de teste. O seu `pom.xml` já segue esse padrão (`security-test`, `validation-test`).
- **Spring Boot Actuator:** endpoints de operação (`health`, `info`, `metrics`) e como expor só o necessário.
- **Spring Security por padrão:** o que a auto-configuração faz quando você não configura nada, e o que muda quando você declara um `SecurityFilterChain`.
- **401 vs. 403:** "não sei quem você é" vs. "sei quem você é, e você não pode".
- **Package-by-feature:** pacotes por capacidade de negócio (revisão do CARD-000).
- **Spring Modulith:** cada subpacote direto do pacote principal é um módulo; os subpacotes dele são internos por padrão; `ApplicationModules.verify()` detecta ciclos e acessos a internos.
- **ArchUnit:** regras de arquitetura escritas como testes JUnit.
- **`package-info.java`:** documentação e anotações no nível do pacote.

## Perguntas que você precisa responder
1. **Spring Modulith ou ArchUnit?** O que cada um verifica sozinho e o que você precisaria escrever à mão? Qual deles conhece a sua matriz de dependências (quem pode depender de quem)?
2. **Por que, ao adicionar o starter web, todo endpoint passa a pedir login?** De onde vem a senha que aparece no log?
3. **Com a sua `SecurityFilterChain`, uma rota protegida acessada sem credenciais retorna 401 ou 403?** Explique o porquê pela filter chain, não pelo resultado do teste.
4. **Quais endpoints do Actuator expor?** O que um `/actuator/env` exposto revelaria? E o `/actuator/heapdump`?
5. **O que entra em `shared` agora?** Nada é uma resposta válida. Qual é o critério para algo entrar?
6. **Como um módulo expõe a sua API pública?** Se o `dashboard` precisar de uma classe que está em `tasks/internal`, qual é o caminho certo, e qual é o atalho errado?

## Decisões arquiteturais deste card
- [ ] Escolha entre Spring Modulith e ArchUnit (pergunta 1). Se for diferente do que o ADR-001 sugere, registrar.

## Implementação — tarefas em ordem

### T1 — HTTP no ar
- Adicionar `spring-boot-starter-webmvc` e `spring-boot-starter-actuator` (e o starter de teste correspondente ao webmvc).
- Expor apenas o endpoint `health`.
- **Pronto quando:** a aplicação sobe e **continua rodando**, e `curl localhost:8080/actuator/health` responde `{"status":"UP"}`.

### T2 — Segurança mínima e explícita
- Declarar a sua própria `SecurityFilterChain`: `/actuator/health` liberado, todo o resto exige autenticação.
- **Pronto quando:** health responde 200 sem login, e uma rota qualquer responde 401 ou 403 (pergunta 3).

### T3 — Os módulos do MVP como pacotes
- Criar em `br.com.lifehub`: `auth`, `users`, `tasks`, `calendar`, `notes`, `dashboard`, `audit` e `shared`.
- Em cada um, um `package-info.java` com a frase "é dono de" da tabela de módulos.
- `notifications`, `documents` e `ai` **não** entram agora: nascem nas fases deles.

### T4 — Verificação dos limites
- Adicionar o Spring Modulith ou o ArchUnit (pergunta 1).
- Escrever o teste de modularidade.
- **Pronto quando:** o teste passa e roda no `./mvnw verify`.

### T5 — Quebrar de propósito
- Criar uma violação: uma classe em `tasks/internal` usada por `calendar`, ou um ciclo `tasks ↔ calendar`.
- Rodar o teste, copiar a mensagem de erro para o seu estudo e desfazer a violação.

### T6 _(opcional)_ — Documentação gerada
- Gerar o diagrama dos módulos (o Spring Modulith tem o `Documenter`) e comparar com a matriz do CARD-000.

## Testes
Nomes sugeridos (ajuste à sua convenção):
```text
health_should_be_public
any_other_route_should_require_authentication
modules_should_respect_boundaries
```

## Documentação — entregáveis
- [ ] `docs/architecture/architecture.md` — registrar a ferramenta escolhida no §1 (onde hoje diz "entra no CARD-003").
- [ ] `docs/study/CARD-003.md` — o que você aprendeu e por quê, incluindo a mensagem de erro da T5.

## Erros comuns
- Desligar a segurança para "facilitar" (`exclude = SecurityAutoConfiguration.class`). Ela volta mais difícil depois.
- Expor todos os endpoints do Actuator com `include=*`.
- Criar classes vazias em todos os pacotes "para o teste ter o que verificar". Pacote com `package-info.java` basta.
- Escrever teste que só passa: se a T5 não fizer o teste falhar, o teste não protege nada.

## Critérios de aceite
- [ ] As 6 perguntas estão respondidas.
- [ ] A aplicação fica no ar e o health responde sem login.
- [ ] Qualquer outra rota exige autenticação, coberta por teste.
- [ ] Os 8 pacotes de módulo existem, cada um com `package-info.java`.
- [ ] O teste de modularidade passa, e falhou na T5.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
