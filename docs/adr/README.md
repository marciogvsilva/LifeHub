# Architecture Decision Records

Decisões arquiteturais relevantes são registradas aqui, no formato de Michael Nygard: **Contexto → Decisão → Alternativas → Consequências**.

## Regras
- Um ADR por decisão. Numeração sequencial (`ADR-NNN-titulo-curto.md`).
- **ADRs não são editados depois de aceitos.** Se a decisão mudar, crie um novo ADR com status "Substitui ADR-00X" e marque o antigo como "Substituído".
- Todo ADR lista as **alternativas rejeitadas** e **o que se perde** com a escolha.

## Status possíveis
`Proposto` → `Aceito` → (`Substituído` | `Descontinuado`)

## Índice

| ADR | Título | Status |
|---|---|---|
| [ADR-001](ADR-001-modular-monolith.md) | Utilizar Modular Monolith | Aceito |
| [ADR-002](ADR-002-multiusuario.md) | Multiusuário com isolamento por usuário | Aceito |

## Template

```markdown
# ADR-NNN — Título

- **Status:** Proposto
- **Data:** AAAA-MM-DD

## Contexto
Qual problema ou força está exigindo uma decisão?

## Decisão
O que foi decidido, em uma ou duas frases.

## Alternativas consideradas
### Alternativa A — por que foi rejeitada
### Alternativa B — por que foi rejeitada

## Consequências
### Positivas
### Negativas (o que perdemos)
### Riscos e como mitigá-los
```
