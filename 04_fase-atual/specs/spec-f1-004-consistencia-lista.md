# SPEC-1-004 — Validação estrutural e fila de inconsistências da lista

**Fase:** 1  
**Status:** bloqueada  
**Dono:** PCP, com Engenharia como autoridade para equivalência técnica  
**Origem no escopo:** Fase 1; RQ-004; RQ-003; RQ-013  
**Degrau da solução:** construção mínima de validações determinísticas sobre a lista versionada — não corrige cadastro, não escolhe substituto e não calcula estoque.

## Contexto e decisões fechadas

- **Estado atual:** não existe catálogo de materiais, regra de unidade, fila de inconsistências ou classificação de linhas no sistema fonte. O repositório está no estado de template.
- **Estado desejado:** cada linha de uma versão recebe uma classificação explícita: `valida`, `nao_reconhecida`, `incompleta`, `divergente` ou `duplicada`, com regra/motivo, versão, ator e timestamp. Somente linhas válidas ficam disponíveis para as fases posteriores.
- **Decisões já fechadas:** linha inconsistente não pode ser tratada como disponível; não há correção automática, substituição técnica ou passagem silenciosa; exceção aprovada registra autor, motivo e escopo.
- **Bloqueios:** **BLOQUEIO HUMANO:** fonte oficial do cadastro, política de unidade conversível, regra para código novo, duplicidade e equivalência/substituição. Sem essas decisões, o sistema deve sinalizar e parar.

## Resultado observável

Ao abrir uma versão de lista, PCP vê a classificação de cada linha, filtra inconsistências e identifica o motivo/próximo passo. Um caso com código reconhecido e unidade correta é `valida`; campos ausentes, código desconhecido, unidade divergente ou repetição ficam fora do conjunto liberado. Nenhuma linha é enviada a estoque, compra ou separação por esta SPEC.

## Limites e dependências

- **Inclui:** validação determinística de presença, tipo, código, quantidade, unidade, duplicidade e referência de catálogo; fila de inconsistência; motivo e responsável; reprocessamento após correção.
- **Fora de escopo:** editar cadastro mestre; escolher substituto; conversão de unidade sem regra aprovada; cálculo de saldo; lista de compras; separação; aprovação técnica da Engenharia.
- **Entradas e pré-condições:** versão criada pela SPEC-1-003; catálogo/regras de unidade aprovados ou fixture de catálogo; usuário PCP/Engenharia conforme ação.
- **Saídas/artefatos:** classificação por linha; conjunto de linhas válidas; fila de inconsistências; eventos de revisão; relatório de cobertura.
- **Dependências e responsáveis:** PCP coordena; Engenharia decide equivalência; responsável pelo cadastro confirma código/unidade; TI/cliente confirma a fonte consultável.
- **Atores e permissões mínimas:** sistema aponta; PCP corrige/reencaminha; Engenharia decide equivalência; ninguém altera a linha de origem durante a validação.
- **Superfícies/arquivos/configurações afetadas:** tabela/detalhe da lista; regras e resultados de validação; fila; histórico; fixture de catálogo.
- **Risco e plano B:** se o catálogo não responder, todas as linhas dependentes ficam `nao_reconhecida` ou `bloqueada_por_fonte`, nunca `valida`; usar catálogo fixture somente na demonstração.
- **Rollback ou reversão:** recalcular a mesma versão preservando resultado anterior; nova versão de lista reinicia a validação; aprovação excepcional é evento separado e não altera o payload original.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Lista versionada → validador | Versão da SPEC-1-003 | `line_id`, `code`, `quantity`, `unit`, `version_id` | Serviço interno; usuário só consulta/decide exceção | Recalcular versão/result hash sem duplicar resultado | Manter resultado anterior e registrar falha |
| Cadastro de materiais → validador | **BLOQUEIO:** fonte e contrato não confirmados | Código, unidade e estado do cadastro; campos/mapeamento pendentes | **BLOQUEIO:** leitura autorizada | Timeout não valida; retry controlado | Classificar dependência externa e atribuir responsável |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-014 | Código, quantidade positiva e unidade presente/compatível | `valida` | Compatibilidade depende da regra aprovada | RQ-004 |
| RN-1-015 | Código ausente, quantidade inválida ou unidade ausente | `incompleta` | PCP corrige na origem | RQ-004 |
| RN-1-016 | Código não encontrado na fonte | `nao_reconhecida` | Engenharia/cadastro decide; sem substituição automática | RQ-004 |
| RN-1-017 | Unidade/código diverge do cadastro ou há repetição | `divergente`/`duplicada` | Política de agregação é bloqueio; padrão seguro é sinalizar | RQ-004 |
| RN-1-018 | Fonte de cadastro indisponível | Classificar como `nao_reconhecida` com motivo técnico `fonte_indisponivel`; a versão fica pendente | Reprocessar após disponibilidade | Riscos do escopo |

## Fluxo e regras

1. O sistema recebe uma versão e congela a referência da validação.
2. Para cada linha, verifica campos obrigatórios e tipos.
3. Consulta o cadastro aprovado, quando disponível, sem transformar timeout em resultado válido.
4. Classifica a linha e registra regra, evidência, timestamp e versão da fonte.
5. Exibe válidas e inconsistentes separadamente, com filtro por motivo/responsável.
6. PCP/Engenharia corrige ou decide a exceção; o sistema reprocessa sem apagar a classificação anterior.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Código conhecido, quantidade positiva e unidade compatível | Linha `valida`, visível no conjunto liberável | Erro técnico mantém pendência |
| Limite | Código ausente/quantidade zero/unidade ausente | `incompleta` com motivo específico | Corrigir origem e criar nova versão/reprocessar |
| Cadastro | Código desconhecido ou unidade divergente | `nao_reconhecida`/`divergente`, sem passagem | Encaminhar ao dono do cadastro/Engenharia |
| Duplicidade | Duas linhas com mesma identidade de material | `duplicada` até regra aprovada | Não somar silenciosamente |
| Falha de fonte | Catálogo indisponível | Não validar como disponível; classificar `nao_reconhecida` com motivo técnico e retry | Reprocessar após fonte recuperada |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; SPEC-1-003; `02-Escopo-Definitivo.md` Fase 1; `requisitos.md` RQ-004; fonte de catálogo apenas se aprovada.
2. **Alterar somente:** classificação, fila, evidência e reprocessamento da versão.
3. **Não alterar:** cadastro mestre, lista de origem, substitutos, estoque, compras, separação, liberação ou regra de agregação não aprovada.
4. **Executar nesta ordem:** confirmar catálogo/fixture; implementar regras mínimas; testar os cinco estados e falha de fonte; executar TDD.
5. **Parar e pedir validação quando:** código novo, unidade convertível, duplicidade ou substituição exigir decisão humana; não escolher um comportamento padrão silencioso.
6. **Estado válido ao parar:** toda linha tem estado e motivo; somente `valida` pode ser consumida pela fase seguinte, ainda sem cálculo de estoque.

## Checklist de execução

- [ ] Fonte/fixture de catálogo e regra de unidade foram aprovadas.
- [ ] Toda linha recebe classificação, motivo, versão e timestamp.
- [ ] Linhas inconsistentes ficam fora do conjunto válido.
- [ ] Código desconhecido, unidade divergente e duplicidade não são corrigidos silenciosamente.
- [ ] Falha de catálogo bloqueia/reprocessa sem marcar linha válida.
- [ ] Fila possui responsável e próximo passo.

## Critérios de aceite

- [ ] **CA-1-016:** casos válido, incompleto, não reconhecido, divergente e duplicado são classificados conforme fixture e regra documentada.
- [ ] **CA-1-017:** nenhuma linha inconsistente entra no conjunto liberável sem decisão registrada.
- [ ] **CA-1-018:** timeout/indisponibilidade do catálogo não vira saldo ou validade presumida.
- [ ] **CA-1-019:** reprocessamento preserva o resultado anterior e relaciona o novo resultado à mesma versão.
- [ ] **CA-1-020:** cada exceção tem motivo, responsável e próximo passo.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Não existe validador/fila no estado atual | Abrir aplicação base com lista de teste | Não há classificação nem bloqueio; CA-1-016 a CA-1-020 falham | Captura e commit base |
| GREEN | Fixture com cinco classes e catálogo indisponível | Executar validação sobre linhas válidas, incompletas, desconhecidas, divergentes e repetidas; simular timeout | Cada linha recebe classe/motivo; nenhuma inconsistente passa; timeout fica bloqueado | Relatório, capturas e logs |
| REFACTOR/REGRESSÃO | Reprocessamento e exceção | Corrigir uma linha, reprocessar, registrar decisão humana e executar `pnpm lint`/`pnpm build` | Histórico preservado, estado atual coerente e pacote passa | Histórico, captura da fila e saída dos comandos |

**Dados/fixtures:** catálogo sintético sem dados pessoais; cinco linhas cobrindo as classes; uma repetição e um caso de timeout.

**Caminhos de erro obrigatórios:** ausência de campo, valor inválido, código desconhecido, unidade divergente, duplicidade, catálogo indisponível, permissão negada e tentativa de substituir automaticamente.

**Evidência exigida:** classificação por linha, fonte/revisão do catálogo, fila de inconsistências, decisão excepcional e saída de `pnpm lint`/`pnpm build`.

## Handoff e operação

- **Como demonstrar:** PCP abre uma lista, filtra as cinco classes, trata um caso e mostra que os demais continuam bloqueados.
- **Como operar depois:** PCP coordena; cadastro corrige códigos; Engenharia decide equivalências; nenhuma correção automática substitui a origem.
- **Como monitorar:** quantidade por classe, tempo em exceção, falhas de catálogo e reprocessamentos.
- **Pendência conhecida:** fonte do cadastro, unidade, duplicidade e substituição precisam de decisão antes do aceite produtivo.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T010 | Confirmar catálogo, unidade, duplicidade, código novo e política de substituição | PCP | SPEC-1-004 § Contexto e decisões fechadas; § Dados e integrações | Fonte/fixture e políticas de classificação registradas | Conferir cinco classes e catálogo indisponível | Fixture de catálogo, regras e dono da exceção | Autorização humana; dono do cadastro disponível | bloqueada |
| F1-T011 | Implementar classificação determinística e fila de inconsistências | Ethos | SPEC-1-004 § Fluxo e regras; CA-1-016 e CA-1-017; TDD GREEN | Cinco classes atribuídas; inconsistências fora do conjunto válido | TDD GREEN com cinco classes | Relatório por linha, motivo, responsável e filtro | F1-T009 e F1-T010 concluídas | bloqueada |
| F1-T012 | Implementar falha de catálogo, reprocessamento, exceção e prova final | Ethos | SPEC-1-004 § Fluxo e regras; CA-1-018 a CA-1-020; TDD REFACTOR/REGRESSÃO | Timeout não valida; reprocessamento preserva; exceção tem dono | TDD de timeout, correção, exceção e regressão | Logs, histórico, fila e build/lint | F1-T011 concluída; fixture de falha disponível | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
