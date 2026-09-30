# STATUS — Projeto Centro Operacional Dlean

> **Atualizado em:** 2026-09-30 · **Por:** Adapta (consultor Navaar)
> O painel do projeto: fase atual, progresso e o que precisa de atenção.

## Onde estamos

- **Fase atual:** Fase 1 — entrada de demanda, projeto e lista de materiais · aberta em 2026-08-26
- **Objetivo desta fase:** conduzir um pedido/projeto de teste até a validação da Engenharia e uma lista de materiais versionada, registrando o baseline.
- **No prazo?** Em andamento — F1-T001 e F1-T002 concluídas e liberadas.
- **Atenção (regularização em curso):** a F1-T003 foi implementada em 2026-09-17 sem o ciclo de autorização. Está **em regularização** — a liberação exige evidências de TDD, teste humano do champion e aceite do consultor. O ciclo de liberação passa a valer para esta e todas as tasks posteriores (ver `03_documentos/liberacao-de-tasks.md`).

## Progresso da fase

- **Tasks:** 2/15 liberadas (13%) · 1 em regularização (F1-T003) · 12 bloqueadas
- **Próxima task:** definida após a decisão do consultor sobre a liberação da F1-T003; nenhuma task posterior começa antes disso.

## Travas ativas

| Trava | Desde | Quem resolve | O que destrava (responder junto com a abertura da task) |
|---|---|---|---|
| F1-T004 — decisões da Engenharia | 2026-08-26 | Consultor (Navaar) + Engenharia | Checklist mínimo de aprovação, alçada substituta, fonte/formato do projeto e matriz de permissões da Engenharia |
| F1-T007 — contrato da lista | 2026-08-26 | Consultor (Navaar) | Versionamento, formato da origem e papel permitido da IA (sugestão ou não uso) |
| F1-T010 — catálogo e substituição | 2026-08-26 | PCP | Unidade de medida, regra de duplicidade e política de substituição |
| F1-T013 — baseline e métricas | 2026-08-26 | PCP | Intervalo, marcos, população, timezone, meta e responsável pelo baseline |

## Entregas concluídas

| Fase | O que foi entregue | Fechada em |
|---|---|---|
| F1-T001 | Contrato de entrada, campos mínimos, papéis, fixture e pré-fluxo pedido → projeto | 2026-08-26 |
| F1-T002 | Caminho principal de entrada e vínculo idempotente do contexto, com fixture, histórico e preview público validados | 2026-09-17 |

## Próxima reunião

a definir — demonstração: entrada de demanda, validação automática, idempotência e estados (F1-T002 + F1-T003).
