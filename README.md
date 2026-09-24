# LifeHub

> **Um laboratório pessoal de engenharia de software, produtividade e inteligência artificial.**

O **LifeHub** é uma plataforma pessoal de produtividade e conhecimento que centraliza tarefas, calendário, notas, documentos e outras informações do dia a dia em um único sistema self-hosted.

Mais do que uma aplicação, o LifeHub é um **projeto de aprendizado contínuo**: uma aplicação real que evolui de uma arquitetura simples e bem estruturada até mensageria, RAG, agentes de IA, MCP, observabilidade, automação de infraestrutura e CI/CD.

> **Complexidade deve ser conquistada, não adicionada artificialmente.**

---

## 🎯 Objetivos

1. **Construir uma aplicação realmente útil** para a produtividade pessoal do dia a dia.
2. **Servir como laboratório de engenharia de software**, privilegiando entendimento e qualidade em vez de quantidade de funcionalidades.

Detalhes em [Visão do produto](docs/architecture/vision.md).

---

## 🛠️ Stack

| Camada | Tecnologias |
|---|---|
| Backend | Java 21, Spring Boot, Spring Security, Spring Data JPA, Bean Validation, Maven, Spring AI |
| Frontend | React, TypeScript, React Query, Material UI |
| Banco | PostgreSQL, Liquibase, PL/pgSQL, pgvector |
| Mensageria | RabbitMQ |
| Infra | Docker, Docker Compose, Nginx, Ansible |
| CI/CD | GitHub Actions |
| Observabilidade | Spring Boot Actuator, Prometheus, Grafana, Loki |
| IA | Spring AI, OpenAI, Ollama, embeddings, RAG, Tool Calling, agentes, MCP |

Cada tecnologia entra no projeto **quando resolve um problema real**, seguindo o [roadmap](docs/architecture/roadmap.md).

---

## 📖 Documentação

| Documento | Conteúdo |
|---|---|
| [Visão do produto](docs/architecture/vision.md) | Problema, usuário, objetivos, não-objetivos e princípios |
| [Requisitos](docs/architecture/requirements.md) | Escopo do MVP, user stories e atributos de qualidade |
| [Arquitetura](docs/architecture/architecture.md) | Modular Monolith, módulos, responsabilidades e dependências |
| [Diagramas C4](docs/diagrams/c4.md) | Contexto (nível 1) e Containers (nível 2) |
| [Roadmap](docs/architecture/roadmap.md) | Releases e cards |
| [ADRs](docs/adr/README.md) | Registro de decisões arquiteturais |
| [Cards](docs/cards/) | Unidades de aprendizado e implementação |
| [Estudo](docs/study/) | Anotações e explicações sobre as decisões de cada card |

---

## 🗂️ Estrutura do repositório

```text
lifehub/
├── backend/          # Spring Boot (a partir do CARD-003)
├── frontend/         # React + TypeScript (a partir do CARD-005)
├── api/              # Contratos OpenAPI
├── docs/
│   ├── architecture/ # visão, requisitos, arquitetura, roadmap
│   ├── adr/          # Architecture Decision Records
│   ├── cards/        # cards de aprendizado
│   ├── study/        # anotações de estudo por card
│   ├── diagrams/     # C4 e outros diagramas (Mermaid)
│   └── database/     # modelo de dados e convenções
├── infra/            # Docker Compose, Nginx
├── ansible/          # automação do servidor (fases finais)
├── scripts/
└── .github/          # workflows de CI/CD
```

As pastas são criadas quando o card correspondente precisar delas, e não antes.

---

## 📚 Metodologia

O desenvolvimento acontece em **cards**, um por vez:

```text
Estudar → Projetar → Implementar → Testar → Documentar → Revisar → Evoluir
```

Cada card tem objetivo, conceitos para estudar, perguntas para reflexão, decisões arquiteturais, critérios de aceite, Definition of Done e review técnico.

---

## 🚧 Status

| Item | Status |
|---|---|
| Projeto | Em planejamento |
| Último card concluído | [CARD-000 — Visão do Produto e Definição Arquitetural](docs/cards/CARD-000.md) |
| Próximo card | CARD-001 — Modelo de domínio e bounded contexts |
| Implementação | Base gerada pelo Spring Initializr; nenhuma funcionalidade ainda |

---

> **O objetivo não é terminar o LifeHub rapidamente.**
> **O objetivo é construir um sistema real enquanto se aprende a projetar, implementar, testar, documentar, operar e evoluir software profissionalmente.**

## License

A definir. Se o repositório for público, é bom decidir isso antes do primeiro código.
