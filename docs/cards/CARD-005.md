# CARD-005 — Frontend React + TypeScript chamando o backend

| Campo | Valor |
|---|---|
| Fase | F1 — Walking skeleton |
| Tipo | **Código.** Primeiro frontend. |
| Status | ⬜ A fazer |
| Timebox | 2 sessões |

## Objetivo
Criar o frontend com Vite, React e TypeScript, e fazer uma tela que chama um endpoint real do backend e mostra o resultado, tratando carregamento e erro.

## Por que esse card existe
Um walking skeleton precisa atravessar **todas** as camadas: navegador → frontend → backend → banco. É neste card que aparecem os problemas de integração (CORS, URLs, autenticação) enquanto ainda são baratos de resolver.

## Pré-requisitos
- CARD-004 concluído.
- Node.js LTS instalado no WSL.

## Conceitos para estudar
- **Vite:** servidor de desenvolvimento, build e variáveis de ambiente (`import.meta.env`).
- **TypeScript em modo `strict`.**
- **CORS:** origem, requisição *preflight* (`OPTIONS`) e por que o navegador bloqueia (e o `curl` não).
- **Proxy do Vite vs. CORS no backend:** duas formas de resolver o mesmo problema em desenvolvimento.
- **React Query (TanStack Query):** cache, estados `isPending`/`isError`, *retry*.
- **Material UI:** tema e componentes básicos.
- **ESLint e Prettier.**

## Perguntas que você precisa responder
1. **Proxy do Vite ou CORS no backend?** E em produção, com o Nginx servindo frontend e backend na **mesma origem** (CARD-007), o CORS ainda é necessário?
2. **Qual endpoint o frontend vai chamar?** O `/actuator/health` é de operação, não de produto. Vale criar um endpoint público, como `GET /api/system/info` (versão, ambiente)? Em qual módulo ele mora?
3. **Como liberar esse endpoint sem abrir mais do que o necessário** na sua `SecurityFilterChain`?
4. **Organização das pastas do frontend:** por tipo (`components/`, `hooks/`) ou por feature (`tasks/`, `calendar/`), espelhando os módulos do backend?
5. **npm ou pnpm?** E como fixar a versão do Node para o CI (`.nvmrc`, `engines`)?
6. **Por que usar React Query para uma única chamada?** O que ele faz que um `useEffect` com `fetch` não faz?

## Implementação
1. Criar `frontend/` com o template `react-ts` do Vite.
2. Configurar ESLint e Prettier, e o TypeScript em modo `strict`.
3. Criar o endpoint público no backend (pergunta 2) com teste.
4. Resolver a comunicação em desenvolvimento (pergunta 1).
5. Criar a tela de status com React Query e Material UI: mostra os dados quando o backend está no ar e uma mensagem amigável quando não está.

## Testes
- Backend: teste do novo endpoint (público, status 200, formato da resposta).
- Frontend: testes automatizados entram no CARD-006. Aqui, teste manual: derrube o backend e veja a mensagem de erro.

## Documentação — entregáveis
- [ ] `README.md` — "Como rodar" com o frontend.
- [ ] `docs/study/CARD-005.md` — o que você aprendeu e por quê, incluindo o erro de CORS que você viu (se viu).

## Erros comuns
- Liberar CORS para qualquer origem (`*`) "porque é só desenvolvimento" e esquecer em produção.
- URL do backend fixa no código em vez de variável de ambiente ou proxy.
- Tratar só o caminho feliz: tela em branco quando o backend cai.
- Desligar o `strict` do TypeScript no primeiro erro de tipo.

## Critérios de aceite
- [ ] As 6 perguntas estão respondidas.
- [ ] Com o backend no ar, a tela mostra os dados do endpoint.
- [ ] Com o backend fora, a tela mostra uma mensagem de erro, não uma tela em branco.
- [ ] `npm run build` (ou equivalente) termina sem erros de tipo nem de lint.

## Definition of Done
- [ ] Tudo commitado.
- [ ] Review técnico feito e os pontos levantados foram tratados ou registrados.

## Review técnico
_Anotações do review:_
