# Changelog — Projeto Centro Operacional Dlean

> Registro de tudo que acontece no projeto, em ordem cronológica inversa (mais recente no topo).
> Formato: `- AAAA-MM-DD · [quem] · o que aconteceu`

## Registro

- 2026-09-30 · Adapta (consultor Navaar) · Regularização: registrada a divergência da F1-T003 (implementada em 2026-09-17 sem o ciclo de autorização exigido). Task colocada **em regularização** — a liberação exige evidências de TDD, teste humano do champion e aceite do consultor. Estado reconciliado em `STATUS.md`, `fase.md` e `03_documentos/liberacao-de-tasks.md`.
- 2026-09-30 · Adapta (consultor Navaar) · Publicado o **ciclo de liberação de tasks** (`03_documentos/liberacao-de-tasks.md`): autorização explícita → execução → evidências de TDD → teste humano do champion → recibo de liberação. Vale para a F1-T003 e todas as posteriores. AGENTS.md e CLAUDE.md atualizados; ambiente único pinado (Skip 52812, `05-Sistema/p-gina-em-branco-ai2rz8hkd`); tasks bloqueadas ganharam insumos embutidos (nada de trava sem pergunta).
- 2026-09-30 · Adapta (consultor Navaar) · Observado o projeto externo **"Compass 2.1 — Painel de Entregas"** (repositório `dantedlean/compass-2-1---painel-de-entregas-fffb0lb7b`, publicado em produção em 30/09) — fora do recorte da Fase 1 (faturamento/entregas). Decisão do consultor pendente: registrar como projeto paralelo fora de escopo ou trazer ao plano. Também apontado `.env` commitado no repo público (só URL interna; retirada recomendada).
- 2026-09-17 · Ethos · Task F1-T002 concluída: caminho principal de entrada implementado no projeto-base Centro Operacional Dlean, fixture válida gera contexto único em `aguardando_engenharia`, reenvio é idempotente com histórico `ingestion_replayed`; TypeScript, lint, build, domínio e preview público validados; teste humano aprovado por Andre.
- 2026-08-26 · Ethos · F1-T001 concluída: decisões de entrada documentadas (fonte: MaxiProd; campos: pedido, cliente, data, produtos, local, quantidade; elegibilidade: automática; visibilidade: admin/gestores veem todas, demais veem as suas; fixture: Navaar; pré-fluxo: PV automático no Compass, desenho padrão reutilizado ou novo projeto para Engenharia).
- 2026-08-26 · Ethos · Plugin Adapta Cliente configurado para Centro Operacional Dlean; repositório `fnavaar/dantedlean-adapta-cliente` validado; projeto Skip 52812 criado.
- 2026-08-17 · Adapta Labs · Repositório publicado em `https://github.com/fnavaar/dantedlean-adapta-cliente`.
- 2026-08-17 · Adapta Labs · Handoff realizado com bypass autorizado e ressalvas.
