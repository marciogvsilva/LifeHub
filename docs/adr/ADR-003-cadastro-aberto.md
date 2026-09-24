# ADR-003 — Cadastro aberto, com papéis ADMIN e USER

- **Status:** Aceito
- **Data:** 2026-09-24

## Contexto

O [ADR-002](ADR-002-multiusuario.md) definiu o LifeHub como multiusuário, mas não definiu **quem pode criar conta**. Isso importa porque:
- o servidor é um notebook com 8 GB de RAM, e o LifeHub deve ser exposto na internet no futuro;
- a IA pode usar um provider pago na nuvem, com cota;
- segurança é o atributo de qualidade nº 1 ([requirements.md §3](../architecture/requirements.md));
- o CARD-012 (RBAC) precisa de papéis definidos, e até aqui só existia o "usuário comum".

## Decisão

O **cadastro é aberto**: qualquer pessoa com acesso ao LifeHub pode criar uma conta.

O RBAC tem dois papéis:
- **USER:** acessa apenas os próprios dados.
- **ADMIN:** tudo o que o USER faz, mais desativar contas e ver o uso de recursos por usuário. O ADMIN **não** lê os dados de negócio dos outros usuários. O primeiro ADMIN é criado por configuração na inicialização.

Enquanto o acesso for só pela rede local ([requirements.md §4](../architecture/requirements.md)), o risco é baixo. **Antes de expor o LifeHub na internet, as mitigações abaixo são obrigatórias.**

## Alternativas consideradas

### A — Só o admin cria contas
**Por que foi rejeitada:** o cadastro aberto é mais simples de usar e demonstrar como portfólio, e o risco dele é controlável com as mitigações abaixo.

### B — Cadastro por convite
Link de convite de uso único, gerado pelo admin.
**Por que foi rejeitada:** exige token de convite, expiração e envio, trabalho extra sem necessidade no MVP. Continua sendo a saída se as mitigações não bastarem.

## Consequências

### Positivas
- Qualquer pessoa pode experimentar o LifeHub sem depender do admin, o que ajuda no portfólio.
- O RBAC do CARD-012 ganha um caso real: dois papéis com permissões diferentes.

### Negativas (o que perdemos)
- Contas criadas por desconhecidos consomem recursos do notebook e, com a IA ligada, a cota do provider.
- Exige mitigações de abuso que um cadastro fechado dispensaria.

### Riscos e mitigação
| Risco | Mitigação |
|---|---|
| Criação de contas em massa (bots) | Limite de tentativas por IP no cadastro e no login; verificação de e-mail quando o envio de e-mail existir (fase F4) |
| Um usuário esgotar o disco ou a memória do notebook | Cotas por usuário (armazenamento de documentos, quantidade de itens); ADMIN pode desativar contas |
| Um usuário esgotar a cota do provider de IA | Limite de uso de IA por usuário; a IA é opcional e pode ficar desligada |
| Exposição na internet antes das mitigações | Acesso só pela rede local até que todas as mitigações desta tabela estejam implementadas |

## Quando revisitar esta decisão

Se as mitigações não contiverem o abuso, mudar para cadastro por convite (alternativa B), num novo ADR que substitua este.
