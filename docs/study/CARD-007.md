# Estudo — CARD-007

Guia de apoio para o [CARD-007 — Primeiro deploy no notebook](../cards/CARD-007.md).

- A **Parte 1** resume os conceitos do card, com exemplos do LifeHub.
- A **Parte 2** apoia cada pergunta: o que considerar, as opções e o que cada uma implica. **Não traz a resposta.**
- A **Parte 3** é sua: o que aprendeu e os números medidos.

---

# Parte 1 — Conceitos

## Imagem Docker do backend: duas formas
| | Dockerfile multi-stage | Buildpacks (`./mvnw spring-boot:build-image`) |
|---|---|---|
| Quem escreve a receita | Você | O Spring Boot + Paketo |
| Controle | Total | Configurável, mas opinativo |
| Boas práticas (usuário sem root, camadas, JVM ajustada) | Você aplica | Vêm prontas |
| Aprendizado | Explícito | Investigar o que foi feito |

**Multi-stage:** um estágio com JDK e Maven compila; o estágio final só tem JRE e o jar. A imagem final fica menor e sem ferramentas de build.

**Camadas:** o Spring Boot separa dependências (mudam pouco) do seu código (muda sempre). Com camadas separadas, uma mudança no código não obriga a reenviar todas as dependências.

## JVM em container
- Desde o Java 10, a JVM **enxerga o limite de memória do container** (não a memória do computador).
- Por padrão, o heap máximo é **25%** dessa memória. O resto fica para metaspace, threads, buffers e o próprio sistema.
- `-XX:MaxRAMPercentage=` ajusta essa porcentagem.

> ⚠️ Se o processo inteiro (heap + não-heap) passar do limite do container, o kernel mata o processo (**OOM kill**). No log da aplicação não aparece nenhuma exceção; aparece só em `docker inspect` ou `dmesg`.

## Compose de produção
| Peça | Para quê |
|---|---|
| `restart: unless-stopped` (ou `always`) | O container volta sozinho após falha ou reinício |
| Limite de memória (`mem_limit` ou `deploy.resources.limits.memory`) | Um serviço não consome a memória dos outros |
| `depends_on` com `condition: service_healthy` | O backend só sobe quando o banco está **pronto** |
| `healthcheck` | O Docker sabe se o serviço está saudável |

> 💡 `restart` só funciona após reiniciar o computador se o **serviço do Docker** iniciar com o sistema (`systemctl enable docker`).

## Logs estruturados
Log estruturado é **JSON** (um objeto por linha), não texto livre. Ferramentas como o Loki (F9) leem campos (`level`, `logger`, `traceId`) em vez de procurar texto.

O Spring Boot tem suporte nativo: investigue `logging.structured.format.console` e os formatos disponíveis (ECS, Logstash, GELF).

**Rotação:** por padrão, o Docker guarda os logs de cada container **sem limite**. Num disco pequeno, isso enche o disco. Investigue as opções `max-size` e `max-file` do driver de log.

## Backup do PostgreSQL
- **`pg_dump`:** cópia lógica de um banco. No formato custom (`-Fc`), é comprimida e restaurável com `pg_restore`, inclusive em partes.
- **`pg_dumpall`:** inclui papéis (usuários) e configurações globais, que o `pg_dump` não leva.
- **Agendamento:** `cron` ou **timer do systemd** (com log no `journalctl` e "executar o que perdeu" se a máquina estava desligada: investigue `Persistent=true`).
- **Regra 3-2-1:** 3 cópias, em 2 mídias diferentes, 1 fora do local.

## RPO e RTO (revisão do CARD-000)
- **RPO** (24 h): quanto dado você aceita perder → define a **frequência** do backup.
- **RTO** (1 h): quanto tempo você aceita para restaurar → só se conhece **medindo** um restore.

## Linux como servidor
- **SSH com chave** e login por senha desabilitado.
- **Usuário sem root** para rodar a aplicação; `sudo` só quando preciso.
- **Tampa do notebook:** por padrão, fechar a tampa suspende. Investigue `HandleLidSwitch` no `logind.conf`.
- **Atualizações de segurança automáticas** (`unattended-upgrades` no Debian/Ubuntu).

---

# Parte 2 — Apoio às perguntas

### 1. Qual sistema operacional?
**O que considerar:**
- Com 8 GB, cada centena de MB do sistema conta. Uma instalação **sem interface gráfica** (server) usa uma fração de uma desktop.
- **Debian** (estável, mínimo) e **Ubuntu Server** (mais documentação, ciclo LTS) são as escolhas comuns com Docker. Há outras, como distribuições específicas para containers.
- O Celeron de 2ª geração é x86-64: as imagens que você constrói no PC (também x86-64) rodam nele sem conversão.

**Medição útil:** depois de instalar, quanto de memória o sistema usa parado (`free -h`)? Quanto sobra para a meta de 4 GB da stack?

### 2. Onde construir a imagem?
| Opção | Como a imagem chega ao notebook | Custo |
|---|---|---|
| Construir no notebook | Já está lá | Compilação lenta no Celeron; ferramentas de build no servidor |
| Construir no PC + `docker save \| ssh \| docker load` | Arquivo via SSH | Transfere a imagem inteira toda vez |
| Construir no PC (ou no CI) + registry | `docker pull` | Configurar um registry (GitHub Container Registry, por exemplo) |

**Pergunta-teste:** no F10, o roadmap prevê "GitHub Actions → imagem → registry → servidor". Qual opção agora deixa esse caminho mais curto?

### 3. Memória por container
**Monte uma tabela antes de configurar:**
| Serviço | Limite | Medido (`docker stats`) |
|---|---|---|
| PostgreSQL | ? | ? |
| Backend (JVM) | ? | ? |
| Nginx / frontend | ? | ? |
| **Total** | **≤ 4 GB** | ? |

**O que considerar:**
- O limite da JVM não é só o heap (Parte 1).
- O PostgreSQL usa memória para cache (`shared_buffers`) e por conexão (pool do CARD-004).
- Meça **parado** e **sob uso**. Uma ferramenta simples de carga (como `hey` ou `ab`) ajuda.

### 4. Quem serve o frontend?
| | Nginx na frente | Backend servindo os estáticos |
|---|---|---|
| Containers | Mais um (leve) | Nenhum a mais |
| Mesma origem (sem CORS) | Sim, o Nginx roteia `/` e `/api` | Sim |
| TLS no futuro (F10) | Natural no Nginx | No Spring, mais trabalho |
| Build | Frontend e backend independentes | O build do backend precisa incluir o frontend |

Releia a sua resposta da pergunta 1 do CARD-005: ela continua válida?

### 5. Frequência e destino do backup
**O que considerar:**
- O RPO de 24 h define o **mínimo**. Com o banco pequeno, um backup mais frequente custa quase nada.
- **Fora do notebook:** o PC (está sempre ligado?), um HD externo, um armazenamento em nuvem gratuito?
- **Retenção:** se um bug corromper dados e você só perceber três dias depois, o backup de ontem já contém o dado corrompido. Quantos dias guardar?
- O backup contém dados pessoais de todos os usuários (ADR-002). Precisa ser **criptografado** antes de sair do notebook?

### 6. Provar que o backup funciona
**Roteiro de teste:**
1. Pegue o backup mais recente **do destino externo** (não do notebook).
2. Restaure num banco **vazio** (outro container).
3. Confira: as tabelas e os registros estão lá? O `DATABASECHANGELOG` do Liquibase também?
4. Cronometre do início ao fim, incluindo encontrar e baixar o arquivo.
5. Escreva os passos no runbook, para que o "você do futuro", com pressa, consiga repetir.

### 7. O que acontece quando o notebook reinicia?
**Verifique em ordem:**
- O sistema volta sozinho depois de uma queda de energia? (Existe uma opção na BIOS para ligar ao receber energia.)
- O serviço do Docker inicia com o sistema?
- Os containers têm `restart` configurado?
- O backend espera o banco ficar pronto?
- Os lembretes atrasados serão enviados (decisão do CARD-000)? Ainda não existem, mas registre isso como teste futuro.

---

# Parte 3 — O que aprendi (preencher ao longo do card)

## Decisões e por quê

## Números medidos
| Métrica | Meta | Medido |
|---|---|---|
| Memória total da stack | ≤ 4 GB | |
| p95 do health | < 500 ms | |
| Tempo de restore | ≤ 1 h | |
| Memória do sistema parado | — | |

## Insights

## Dúvidas para o review
