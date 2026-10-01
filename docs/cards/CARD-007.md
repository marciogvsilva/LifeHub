# CARD-007 — Primeiro deploy no notebook

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | **Infraestrutura e operação.** |
| Status | ⬜ A fazer |
| Timebox | 2 a 3 sessões |

## Objetivo
Colocar o LifeHub "vazio" no ar no notebook, com health check, logs estruturados e backup **testado**. É o checkpoint da fase F1.

## Por que esse card existe
Rodar em produção é uma habilidade, e ela fica mais barata de aprender quando o sistema ainda é pequeno. Também é aqui que as metas do `requirements.md` §3 (p95 < 500 ms, stack ≤ 4 GB, RPO 24 h, RTO 1 h) começam a ser medidas na máquina real.

## Pré-requisitos
- CARD-006 concluído.
- Pendências do notebook resolvidas: tipo de disco (HDD ou SSD) e estado da bateria (`requirements.md` §4 e §5).

## Conceitos para estudar
- **Imagem Docker do backend:** Dockerfile multi-stage vs. Buildpacks (`./mvnw spring-boot:build-image`).
- **JVM em container:** como a JVM enxerga o limite de memória; `-XX:MaxRAMPercentage`.
- **Compose de produção:** `restart`, limites de memória, `depends_on` com `condition: service_healthy`.
- **Logs estruturados:** o suporte nativo do Spring Boot (`logging.structured.format.console`) e rotação de logs do Docker.
- **Backup:** `pg_dump`, agendamento (cron ou timer do systemd), cópia para fora do notebook, restore.
- **Linux como servidor:** SSH com chave, usuário sem root, notebook que não suspende ao fechar a tampa.

## Perguntas que você precisa responder
1. **Qual sistema operacional no notebook,** e por quê? Com 8 GB e um Celeron, quanto sobra para o LifeHub?
2. **Onde a imagem é construída?** Compilar no Celeron é lento. Se construir no PC, como a imagem chega ao notebook sem um registry (`docker save` | `ssh` | `docker load`)? Ou vale usar um registry?
3. **Quanta memória cada container recebe** para caber na meta de 4 GB? Como você vai medir (`docker stats`)?
4. **Quem serve o frontend em produção:** um Nginx na frente de tudo ou o próprio backend servindo os arquivos estáticos? Qual resolve a pergunta 1 do CARD-005 (CORS)?
5. **Backup:** com que frequência (RPO de 24 h), para onde (fora do notebook) e quantas cópias guardar?
6. **Como provar que o backup funciona?** Quanto tempo leva o restore, e isso cabe no RTO de 1 h?
7. **O que acontece quando o notebook reinicia?** Tudo volta sozinho?

## Implementação
1. Preparar o notebook: SO, SSH com chave, Docker, energia (tampa, bateria).
2. Gerar a imagem do backend (pergunta 2) e do frontend.
3. Criar o compose de produção com limites de memória, `restart` e healthchecks.
4. Configurar logs estruturados e rotação.
5. Criar o script de backup, agendá-lo e copiar os arquivos para fora do notebook.
6. **Fazer um restore de verdade** num banco vazio e cronometrar.
7. Reiniciar o notebook e confirmar que tudo volta sozinho.
8. Medir a memória total (meta: ≤ 4 GB) e o tempo de resposta do health.

## Testes
- Restore cronometrado (pergunta 6).
- Reinício do notebook sem intervenção manual (pergunta 7).

## Documentação — entregáveis
- [ ] `docs/ops/runbook.md` — como fazer deploy, ver logs, fazer backup e restaurar.
- [ ] `docs/architecture/requirements.md` — resultados medidos ao lado das metas.
- [ ] `docs/study/CARD-007.md` — o que você aprendeu e por quê.

## Erros comuns
- Backup no mesmo disco do banco: se o disco morrer, os dois morrem juntos.
- Backup nunca restaurado: não é backup, é esperança.
- JVM sem limite de memória dentro de um container limitado: o container é morto (OOM) sem aviso claro.
- Expor portas do notebook para a internet. O ADR-003 exige rede local até as mitigações existirem.

## Critérios de aceite
- [ ] As 7 perguntas estão respondidas.
- [ ] O LifeHub está no ar no notebook, acessível pela rede local.
- [ ] O backup roda sozinho e está fora do notebook.
- [ ] Um restore foi feito e cronometrado.
- [ ] Depois de reiniciar o notebook, tudo volta sem intervenção.
- [ ] A memória medida está registrada ao lado da meta.

✅ **Checkpoint da F1:** um sistema "vazio", mas no ar no seu servidor e com backup.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
