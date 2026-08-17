# SPEC-1-005 — Registro de baseline e marcos do primeiro ciclo

**Fase:** 1  
**Status:** bloqueada  
**Dono:** PCP e Diretoria, com validação do Consultor/cliente  
**Origem no escopo:** Fase 1; RQ-012; RQ-013; D-006  
**Degrau da solução:** construção mínima de eventos e relatório demonstrável no módulo independente — mede o processo e não promete redução antes de intervalo, baseline e meta serem aprovados.

## Contexto e decisões fechadas

- **Estado atual:** o sistema fonte é um template sem eventos de negócio, marcos, relatório ou série histórica. A discussão do cliente cita 21/26 dias e ganho aproximado de cinco dias, mas não fecha intervalo, população, meta ou fonte oficial.
- **Estado desejado:** cada projeto de teste registra marcos reproduzíveis, permite distinguir intervalo local e lead time total, mostra dados ausentes/exceções e entrega um baseline antes de declarar ganho.
- **Decisões já fechadas:** não estimar timestamp ausente; projetos com exceção permanecem identificáveis; a fase 1 mede desde o primeiro ciclo; velocidade do agente não é critério de aceite.
- **Bloqueios:** **BLOQUEIO HUMANO:** escolher 21 versus 26 dias, início/fim oficial, definição de “mesmo dia”, população/amostra, meta e responsável pelo veredito. Sem isso, o relatório fica em `baseline_pendente_aprovacao`.

## Resultado observável

Para a amostra acordada, PCP abre um relatório com os marcos `projeto_validado`, `lista_recebida_validada`, `materiais_liberados` e `inicio_producao`, exibindo duração local, duração total quando possível, casos excluídos e ausência de dados. O relatório não afirma que a meta foi atingida enquanto D-006 não estiver aprovada.

## Limites e dependências

- **Inclui:** captura append-only de marcos; identificação de projeto/versão; ator, origem e timestamps; cálculo reproduzível de intervalos; baseline; filtros por exceção; relatório de qualidade da medição.
- **Fora de escopo:** prometer redução de lead time; alterar datas históricas; completar timestamp por estimativa; construir planejamento fabril; criar loop de valor; validar todas as fases.
- **Entradas e pré-condições:** eventos das SPECs 1-001 a 1-004; amostra e intervalo aprovados; timezone/sistema de relógio definidos; usuário autorizado para registrar ou corrigir evento por novo evento.
- **Saídas/artefatos:** eventos; baseline por caso e agregado; relatório; lista de dados ausentes; decisão de medição pendente/aprovada; evidência da demonstração.
- **Dependências e responsáveis:** PCP registra/valida marcos; Diretoria/cliente aprova métrica e meta; Consultor valida interpretação; Engenharia/Produção fornecem eventos posteriores quando aplicável.
- **Atores e permissões mínimas:** áreas registram seus próprios eventos; PCP consulta/agrega; Diretoria/cliente lê e valida métrica; ninguém edita timestamp já registrado sem evento de correção autorizado.
- **Superfícies/arquivos/configurações afetadas:** registro de eventos; relatório de baseline; filtros; schema; fixture de marcos; nenhuma publicação externa.
- **Risco e plano B:** se marcos históricos não existirem, registrar somente casos prospectivos e marcar cobertura insuficiente; não retroestimar.
- **Rollback ou reversão:** evento incorreto não é apagado; registrar correção/revogação com autor, motivo e referência; recalcular relatório a partir da trilha válida.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Sistema de demanda/lista → eventos | Eventos internos das SPECs 1-001 a 1-004 | `entity_id`, `event_type`, `occurred_at`, `recorded_at`, `actor`, `source`, `version_id`, `correlation_id` | Sessão autenticada conforme área | Mesmo `event_id` não duplica; correção é novo evento | Evento rejeitado com motivo e sem cálculo silencioso |
| Produção/áreas → marcos posteriores | **BLOQUEIO:** fonte oficial e integração não confirmadas | Eventos dos marcos; contrato e permissões pendentes | **BLOQUEIO:** origem/autorização pendentes | Retry idempotente por `event_id` | Falta de marco aparece como dado ausente |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-019 | Marco tem entidade, tipo, ator e timestamp válidos | Persistir evento e incluí-lo no cálculo aplicável | Timestamp inconsistente fica pendente | RQ-012 |
| RN-1-020 | Marco ausente ou inválido | Não estimar; marcar cobertura insuficiente | Responsável pode registrar evento real depois | D-006 |
| RN-1-021 | Caso tem exceção | Manter caso identificável e separar no relatório | Inclusão na meta depende de regra aprovada | RQ-012 |
| RN-1-022 | Intervalo/meta ainda não aprovados | Exibir baseline descritivo e estado `baseline_pendente_aprovacao` | Não declarar ganho | D-006 |

## Fluxo e regras

1. PCP inicia um caso de medição para o projeto de teste.
2. Cada área registra os marcos quando o evento real ocorre.
3. O sistema valida ordem, identidade, versão e timestamp sem substituir o evento.
4. O relatório calcula somente intervalos definidos e mostra a fórmula/fonte.
5. O responsável revisa amostra, exceções e cobertura; o estado fica pendente até decisão de métrica.
6. Depois da aprovação humana de D-006, a mesma estrutura pode comparar ciclos, sem reescrever o baseline.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Quatro marcos válidos e caso sem exceção | Relatório reproduzível com intervalos local/total conforme configuração | Dado divergente fica sinalizado |
| Limite | Marco posterior sem anterior | Sem duração inventada; caso com cobertura insuficiente | Registrar marco faltante quando ocorrer |
| Exceção | Caso devolvido/bloqueado ou com revisão | Caso aparece separado com motivo e não é apagado | PCP decide inclusão somente após regra aprovada |
| Correção | Timestamp digitado incorretamente | Novo evento de correção referencia o anterior | Relatório recalcula usando evento vigente e mantém trilha |
| Duplicidade | Mesmo `event_id` recebido duas vezes | Um evento efetivo e registro idempotente | Conflito de payload vira exceção |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; SPECs 1-001 a 1-004; `02-Escopo-Definitivo.md` Fase 1; `requisitos.md` RQ-012; `STATUS.md` e decisões pendentes.
2. **Alterar somente:** eventos, cálculo reproduzível, baseline e relatório da Fase 1.
3. **Não alterar:** datas históricas, meta não aprovada, sistemas externos, lead time prometido, loops, compras ou produção.
4. **Executar nesta ordem:** fechar métrica; validar relógio/amostra; registrar eventos; calcular baseline; demonstrar caso completo/incompleto/exceção; executar TDD.
5. **Parar e pedir validação quando:** intervalo, meta, fonte de marco, timezone ou população não estiverem confirmados; não preencher ausência por inferência.
6. **Estado válido ao parar:** baseline descritivo preservado, cobertura conhecida e nenhum ganho declarado sem decisão humana.

## Checklist de execução

- [ ] Intervalo, marcos, população, timezone, meta e responsável foram aprovados.
- [ ] Eventos têm ator, origem, timestamp e identidade.
- [ ] Duplicidade e correção preservam trilha.
- [ ] Relatório diferencia intervalo local, total, casos excluídos e dados ausentes.
- [ ] Baseline é reproduzível e não declara ganho prematuramente.
- [ ] Evidência foi revisada por PCP/Diretoria/cliente.

## Critérios de aceite

- [ ] **CA-1-021:** os quatro marcos do escopo podem ser registrados e consultados por projeto/versão.
- [ ] **CA-1-022:** o relatório calcula somente intervalos cujas definições estejam aprovadas e mostra a origem de cada valor.
- [ ] **CA-1-023:** timestamp ausente, caso com exceção e conflito de evento ficam identificáveis, sem estimativa silenciosa.
- [ ] **CA-1-024:** baseline da amostra é reproduzível a partir dos eventos e permanece separado de qualquer meta futura.
- [ ] **CA-1-025:** o sistema não declara redução de cinco dias nem atendimento de “mesmo dia” antes da aprovação de D-006.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Não existe coleta/relatório de marcos no sistema atual | Abrir aplicação base e tentar localizar relatório de ciclo | Nenhum evento/relatório; CA-1-021 a CA-1-025 falham | Captura e commit base |
| GREEN | Casos completo, incompleto, exceção, duplicado e corrigido | Registrar fixtures de quatro marcos; omitir um; repetir `event_id`; registrar correção; gerar relatório | Cálculo reproduzível; ausência/exceção visíveis; duplicidade controlada; sem ganho não aprovado | Relatório, trilha e capturas |
| REFACTOR/REGRESSÃO | Recalcular e validar qualidade do pacote | Reexecutar relatório sem mudar dados; executar `pnpm lint` e `pnpm build` | Mesmo resultado; nenhum evento perdido; comandos passam | Hash/saída do relatório, logs e comandos |

**Dados/fixtures:** projetos sintéticos com quatro marcos, um caso sem marco inicial, um com exceção e um evento repetido; nenhum dado pessoal real.

**Caminhos de erro obrigatórios:** timestamp ausente/inválido, ordem impossível, projeto desconhecido, duplicidade, correção, permissão negada e integração posterior indisponível.

**Evidência exigida:** relatório exportável/consultável, trilha de eventos, cálculo/fórmula, cobertura da amostra, exceções e saída de `pnpm lint`/`pnpm build`.

## Handoff e operação

- **Como demonstrar:** PCP registra os marcos de dois casos, abre o baseline e mostra um caso excluído por falta de dado.
- **Como operar depois:** cada área registra seu marco; PCP revisa; Diretoria/cliente aprova a interpretação antes de comparar ciclos.
- **Como monitorar:** cobertura de marcos, casos com dados faltantes, correções, duplicidades e idade do baseline pendente.
- **Pendência conhecida:** D-006 precisa fechar intervalo, meta, população e veredito; sem isso a SPEC permanece bloqueada.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T013 | Confirmar intervalo, marcos, população, timezone, meta e responsável | PCP | SPEC-1-005 § Contexto e decisões fechadas; § Dados e integrações | D-006 registrado e aprovado antes de declarar ganho | Conferir casos completo, incompleto e com exceção | Registro da métrica, amostra, fórmula e responsável | Dados de amostra e responsáveis disponíveis | bloqueada |
| F1-T014 | Implementar eventos e relatório de baseline reproduzível | Ethos | SPEC-1-005 § Fluxo e regras; CA-1-021, CA-1-022 e CA-1-024; TDD GREEN | Quatro marcos consultáveis; baseline reproduz fórmula aprovada | TDD GREEN de caso completo e relatório | Trilha, relatório, cálculo e captura | F1-T006, F1-T009, F1-T012 e F1-T013 concluídas | bloqueada |
| F1-T015 | Implementar ausência/exceção, duplicidade, correção e prova final | Ethos | SPEC-1-005 § Fluxo e regras; CA-1-023 e CA-1-025; TDD REFACTOR/REGRESSÃO | Ausências visíveis; correção preserva trilha; nenhum ganho não aprovado | TDD de marco ausente, exceção, repetição e correção + regressão | Relatório de cobertura, histórico e comandos | F1-T014 concluída; fixtures disponíveis | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
