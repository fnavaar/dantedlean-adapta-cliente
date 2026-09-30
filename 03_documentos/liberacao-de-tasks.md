# Liberação de tasks — protocolo do projeto

> Publicado em 2026-09-30 pelo consultor (Adapta). Serve para regularizar a **F1-T003** e
> para liberar **todas as tasks posteriores**. Nenhuma task é liberada fora deste ciclo.

## O ciclo de uma task (vale para esta e para as posteriores)

1. **Autorização explícita** — antes de qualquer código, o consultor (Navaar) autoriza a task
   por escrito (linha no `changelog.md`). **Uma task por vez.**
2. **Execução conforme a SPEC** — implementar apenas o recorte da task, parando nos pontos de
   parada descritos em `04_fase-atual/fase.md`.
3. **Evidências** — rodar/demonstrar o TDD da SPEC (GREEN; e REFACTOR/REGRESSÃO quando
   previsto) e registrar: capturas, logs, saída dos comandos e histórico idempotente.
   **Sem evidência não há liberação.**
4. **Teste humano** — o champion (André) testa e aprova por escrito no `changelog.md`.
5. **Liberação (recibo)** — preencher o recibo abaixo, registrar a liberação no `changelog.md`
   e atualizar o status em `fase.md` e `STATUS.md`.
6. **Próxima task** — só então voltar ao passo 1.

**Insumos que faltam nunca viram trava muda:** quando uma task precisar de uma decisão ou
informação de alguém, a pergunta é embutida na própria task e respondida junto com a abertura
dela (ver quadro abaixo).

## F1-T003 — regularização (histórico e decisão)

- Implementada em **2026-09-17 15:23** (commit `cdccbe5`, no sistema `p-gina-em-branco-ai2rz8hkd`:
  `domain.ts`, `fixtures.ts`, `Index.tsx` — bordas e estados da entrada de demanda) **antes** do
  ciclo de autorização. O estado oficial declarava: "a próxima task elegível é F1-T003 e exige
  novo ciclo de análise/autorização".
- Não havia registro de autorização prévia, evidências de TDD nem teste humano para esta task.
- **Decisão do consultor (2026-09-30): aceite retroativo com ressalvas** — a implementação é
  mantida. Para a **liberação final**, restam dois itens (registrar no `changelog.md`):
  - **Evidências de TDD reexecutadas** das bordas (reenvio não duplica; inválida não avança;
    falha é recuperável e sem efeitos de compra/liberação) — com capturas/logs;
  - **Teste humano do champion (André)** com data e resultado.
- Feitos os dois itens, preencher o recibo abaixo e marcar F1-T003 como **liberada**.

## Insumos embutidos — respondidos junto com a abertura da task

| Task | Insumo | Pessoa | Situação |
|---|---|---|---|
| F1-T003 | Tratamento do ciclo retroativo | Consultor (Navaar) | **RESPONDIDO 2026-09-30:** aceitar retroativamente com ressalvas (manter implementação; reexecutar provas + teste humano para liberar) |
| F1-T004 | Checklist, alçada substituta, fonte/formato do projeto e matriz de permissões | Consultor + Engenharia | pendente — qual checklist mínimo? quem aprova na ausência? qual fonte/formato do projeto? |
| F1-T007 | Contrato da lista | Consultor (Navaar) | pendente — versionamento, formato da origem, papel da IA (sugestão com revisão × não uso) |
| F1-T010 | Catálogo e substituição | PCP | pendente — unidade por item, regra de duplicidade, política de substituição |
| F1-T013 | Baseline | PCP | pendente — intervalo, marcos, população, timezone, meta e responsável |

## Recibo de liberação (modelo — preencher por task)

- **Task:**
- **Liberada em:**
- **Autorização prévia (data e quem):**
- **Evidências de TDD (o que foi rodado/demonstrado):**
- **Teste humano (quem, quando, resultado):**
- **Aceite do consultor:**
- **Próxima task elegível:**

### Recibo — F1-T003 (em preenchimento)

- **Task:** F1-T003 — bordas, duplicidade, pendência, cancelamento e prova final da entrada
- **Liberada em:** pendente (libera quando os dois itens abaixo forem registrados)
- **Autorização prévia (data e quem):** retroativa — aceite do consultor Navaar em 2026-09-30 (com ressalvas; violação original de 2026-09-17 registrada acima)
- **Evidências de TDD (o que foi rodado/demonstrado):** PENDENTE — reexecutar GREEN de bordas + REFACTOR/REGRESSÃO com capturas/logs
- **Teste humano (quem, quando, resultado):** PENDENTE — André
- **Aceite do consultor:** aceito retroativamente com ressalvas — 2026-09-30
- **Próxima task elegível:** definir após a liberação (F1-T005 depende também de F1-T004)

## Registro de liberações

| Task | Data | Autorização prévia | Evidências de TDD | Teste humano | Aceite do consultor | Situação |
|---|---|---|---|---|---|---|
| F1-T001 | 2026-08-26 | registrada (decisões documentadas) | registro da decisão + fixture | — (task de decisão do consultor) | consultor (própria task) | liberada |
| F1-T002 | 2026-09-17 | fluxo da decomposição | lint/build/domínio/preview registrados | André, 2026-09-17, aprovado | consultor (revisão de estado) | liberada |
| F1-T003 | pendente | retroativa — aceite 2026-09-30 (com ressalvas) | PENDENTE (reexecução) | PENDENTE (André) | **aceito retroativamente com ressalvas — 2026-09-30** | **em liberação — falta evidências + teste humano** |

## Ambiente de trabalho (pinado)

- Sistema único: `05-Sistema/p-gina-em-branco-ai2rz8hkd` — Skip 52812 ("Página em Branco" →
  Centro Operacional Dlean). Repositório único do projeto: este.
- Isolamento de dados não é isolamento de infraestrutura: fixtures sintéticas isolam dados, não
  justificam criar ambiente novo.
- **Proibido** criar repositório, projeto Skip ou app novo para o escopo do plano sem autorização
  explícita do consultor. Projetos paralelos existentes são registrados aqui como
  "fora de escopo" **antes** de irem a produção.

## Fora de escopo — registro de projetos paralelos

| Projeto | Repositório | Situação | Decisão do consultor |
|---|---|---|---|
| Compass 2.1 — Painel de Entregas | `dantedlean/compass-2-1---painel-de-entregas-fffb0lb7b` | publicado em produção em 2026-09-30; escopo de faturamento/entregas | **2026-09-30: só higiene por enquanto** — retirada do `.env` pedida ao Dante; destino (paralelo × plano) decide depois |
