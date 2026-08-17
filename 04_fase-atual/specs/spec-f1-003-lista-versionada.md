# SPEC-1-003 — Recebimento e versionamento da lista de materiais

**Fase:** 1  
**Status:** bloqueada  
**Dono:** Engenharia, com PCP como acompanhamento  
**Origem no escopo:** Fase 1; RQ-003; RQ-013; D-001  
**Degrau da solução:** construção mínima de registro/versionamento no módulo independente, reutilizando o cliente PocketBase existente; a leitura de SolidWorks e qualquer conector de IA ficam atrás de contrato e gate humano.

## Contexto e decisões fechadas

- **Estado atual:** não há coleção, tela, importador ou histórico de lista no sistema. O repositório contém somente o cliente PocketBase e um helper genérico de IA, sem agente, fonte ou fluxo de materiais configurado.
- **Estado desejado:** depois da aprovação da Engenharia, uma lista estruturada é recebida, validada estruturalmente, vinculada ao contexto, identificada por versão/origem e preservada sem apagar a versão anterior. Uma nova versão reabre as validações afetadas.
- **Decisões já fechadas:** projeto não aprovado não recebe lista; lista sem código, quantidade ou unidade não avança; histórico não é sobrescrito; interpretação técnica do desenho continua humana.
- **Bloqueios:** **BLOQUEIO HUMANO:** formato/caminho de exportação SolidWorks, campos aceitos, fonte oficial, permissões e papel exato da IA. A fixture estruturada é o único caminho demonstrável até esses itens serem confirmados.

## Resultado observável

Engenharia envia uma fixture de lista para um contexto aprovado e vê a versão `v1`, origem, revisão, timestamp e linhas. Reenviar a mesma lista é idempotente; enviar uma revisão diferente cria `v2` e mantém `v1` consultável. Contexto devolvido/bloqueado ou arquivo estruturalmente inválido não gera lista utilizável.

## Limites e dependências

- **Inclui:** recebimento estruturado; checagem de contexto aprovado; identificação de origem/revisão; checksum; versão; preservação de histórico; validação estrutural mínima; encaminhamento à SPEC-1-004.
- **Fora de escopo:** interpretar desenho livre; corrigir cadastro; decidir equivalência/substituição; consultar estoque; gerar compras/separação; publicar em ERP/Compass; aceitar saída de IA sem confirmação humana.
- **Entradas e pré-condições:** contexto `aprovado` da SPEC-1-002; payload estruturado ou fixture; `project_id`, `source_ref`, `source_revision`, `received_at`, checksum e linhas com código, quantidade e unidade.
- **Saídas/artefatos:** lista vinculada; versão monotônica por contexto; origem/revisão; linhas; status estrutural; eventos de recebimento e reprocessamento; pendências detalhadas.
- **Dependências e responsáveis:** Engenharia é dona da origem e revisão; PCP acompanha; TI/cliente confirma exportação, acesso e permissões; SPEC-1-004 classifica consistência sem alterar a lista de origem.
- **Atores e permissões mínimas:** Engenharia pode enviar/substituir por nova revisão; PCP pode consultar e solicitar reprocessamento; nenhum papel pode apagar uma versão aceita; administrador técnico não pode aprovar conteúdo.
- **Superfícies/arquivos/configurações afetadas:** formulário/importador da lista; entidades de lista/versão/linha; schema PocketBase; histórico; fixture de teste; eventual adaptador externo somente após aprovação.
- **Risco e plano B:** se SolidWorks/API não estiver disponível, usar fixture estruturada assinada pelo responsável e marcar `source_type=fixture`; não chamar isso de integração produtiva.
- **Rollback ou reversão:** uma versão nova não apaga a anterior; versão inválida fica `rejeitada`/`pendente`; reprocessamento usa a mesma origem/checksum; correção gera revisão nova.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Fixture → lista | Fixture assinada é fonte só no teste | `context_id`, `project_id`, `source_ref`, `source_revision`, `received_at`, `checksum`, `lines[]` | Usuário Engenharia autenticado | Mesmo checksum/origem não cria nova versão; retry seguro | Estado pendente/rejeitada e motivo |
| SolidWorks/origem → lista | **BLOQUEIO:** formato e canal não aprovados | Exportação estruturada, campos e caminho pendentes | **BLOQUEIO:** permissão de leitura/transferência | Retry e timeout serão definidos no contrato | Bloquear recebimento produtivo; preservar erro |
| Assistência de IA → rascunho | **BLOQUEIO:** ferramenta, modelo e papel não confirmados | Sugestões nunca são fonte final; cada campo precisa de confirmação humana | Sem segredo no cliente; conector pendente | Sem chamada automática neste recorte | Se indisponível, seguir por entrada estruturada; não preencher silenciosamente |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-009 | Contexto não está `aprovado` | Recusar recebimento utilizável e informar estado | Reenvio só após nova aprovação | RQ-002/RQ-003 |
| RN-1-010 | Payload contém campos/linhas mínimos | Criar versão e encaminhar para consistência | Linha inválida fica pendente, sem apagar payload | RQ-003 |
| RN-1-011 | Mesmo contexto, origem e checksum | Retornar versão existente e registrar reenvio idempotente | Conflito de checksum abre exceção | AC-009 |
| RN-1-012 | Revisão nova do projeto/lista | Criar nova versão, preservar anterior e reabrir validações | Não sobrescrever uma versão anterior | RQ-003 |
| RN-1-013 | Sugestão de IA sem confirmação | Manter rascunho não aceito | Engenharia confirma ou descarta campo a campo | D-001 |

## Fluxo e regras

1. O usuário seleciona um contexto aprovado e a origem/revisão da lista.
2. O sistema verifica estado do contexto e presença dos metadados de origem.
3. O sistema calcula/recebe checksum e compara com versões existentes.
4. O sistema registra a nova versão ou retorna a existente quando a submissão é idempotente.
5. O sistema valida estrutura mínima e encaminha cada linha à SPEC-1-004.
6. O usuário consulta a versão anterior, a nova versão e os motivos de qualquer bloqueio.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Contexto aprovado e fixture estruturada válida | `v1` registrada, vinculada e disponível para consistência | Falha de persistência mantém tentativa repetível |
| Limite | Linha sem código, quantidade ou unidade | Versão registrada como pendente/rejeitada; não avança como lista válida | Corrigir na origem e reenviar como revisão |
| Reenvio idempotente | Mesmo checksum e origem | Não duplica versão; mostra referência existente | Conflito de metadados abre exceção |
| Revisão nova | `source_revision`/checksum diferente | Cria `v2`, preserva `v1` e reabre validação | Falha não altera `v1` |
| IA não confirmada | Sugestão de campo não revisada por Engenharia | Campo não entra na versão aceita | Usuário confirma manualmente ou descarta |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; SPEC-1-002; `02-Escopo-Definitivo.md` Fase 1; `requisitos.md` RQ-003/RQ-004; `src/lib/pocketbase/client.ts`, `schema.json` e `skipAi.ts`.
2. **Alterar somente:** recebimento estruturado, entidades de versão/linha, histórico e validação de encaminhamento.
3. **Não alterar:** conector SolidWorks, ERP, Compass, estoque, compra, separação, liberação, modelo/agente de IA ou segredos.
4. **Executar nesta ordem:** confirmar contrato/fixture; validar estado aprovado; registrar checksum; criar versão append-only; validar estrutura; testar duplicidade/revisão; executar TDD.
5. **Parar e pedir validação quando:** formato, campos, canal, credenciais ou papel da IA não estiverem decididos; não converter a lacuna em um adapter inventado.
6. **Estado válido ao parar:** versões de teste são consultáveis e nenhuma versão anterior é perdida; versões pendentes não alimentam estoque/compras.

## Checklist de execução

- [ ] Formato, campos, origem e permissão de entrada foram aprovados ou a execução está explicitamente limitada à fixture.
- [ ] Apenas contexto aprovado recebe lista utilizável.
- [ ] Versões, origem, revisão, checksum e timestamp são persistidos.
- [ ] Reenvio idempotente não duplica; revisão nova preserva anterior.
- [ ] Estrutura inválida fica pendente/rejeitada com motivo.
- [ ] IA, se autorizada, só produz sugestão e exige confirmação humana.

## Critérios de aceite

- [ ] **CA-1-011:** fixture válida em contexto aprovado cria uma versão rastreável com linhas e metadados.
- [ ] **CA-1-012:** reenvio idempotente não cria nova versão nem apaga histórico.
- [ ] **CA-1-013:** revisão diferente cria nova versão, preserva a anterior e reabre validações.
- [ ] **CA-1-014:** contexto não aprovado e payload estruturalmente inválido não produzem lista utilizável.
- [ ] **CA-1-015:** qualquer sugestão de IA permanece não aceita até confirmação humana explícita.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Não há modelo de lista/versionamento no sistema atual | Abrir aplicação base e tentar enviar fixture para contexto aprovado | Não existe fluxo nem histórico; CA-1-011 a CA-1-015 falham | Captura e commit base |
| GREEN | Fixture válida, duplicada, inválida e revisada | Enviar `bom-v1`; repetir mesmo checksum; enviar linha sem unidade; enviar `bom-v2` | `v1` única; inválida pendente; `v2` nova com `v1` preservada | Capturas, IDs, versões e histórico |
| REFACTOR/REGRESSÃO | Estado não aprovado, falha e sugestão IA | Tentar enviar para devolvido; simular falha; inserir sugestão não confirmada; executar `pnpm lint`/`pnpm build` | Sem avanço indevido, sem perda de versão e pacote passa | Logs/capturas e saída dos comandos |

**Dados/fixtures:** `bom-v1` com duas linhas válidas; cópia idêntica; uma linha sem unidade; `bom-v2` com alteração de quantidade; nenhum desenho ou dado pessoal real.

**Caminhos de erro obrigatórios:** contexto não aprovado, campo ausente, formato inválido, duplicidade, revisão conflitante, falha de persistência, permissão negada e IA sem confirmação.

**Evidência exigida:** versões visíveis, checksum/origem, histórico preservado, bloqueios e saída de `pnpm lint`/`pnpm build`.

## Handoff e operação

- **Como demonstrar:** Engenharia aprova contexto, envia `bom-v1`, repete, envia revisão e abre o histórico.
- **Como operar depois:** Engenharia é responsável pela origem e revisão; PCP acompanha pendências; nenhuma versão é apagada.
- **Como monitorar:** versões pendentes, falhas de entrada, reenvios idempotentes e revisões sem validação.
- **Pendência conhecida:** fechar formato SolidWorks, campos, origem, permissões e papel da IA.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T007 | Confirmar contrato da lista, versionamento, formato da origem e papel permitido da IA | Consultor | SPEC-1-003 § Contexto e decisões fechadas; § Dados e integrações | Campos, origem, permissões e IA estão decididos | Conferir `bom-v1`, `bom-v2` e IA não confirmada | Contrato/fixture aprovado, checksum e decisão de IA | Autorização humana; responsável da origem disponível | bloqueada |
| F1-T008 | Implementar recebimento estruturado e registro da primeira versão | Ethos | SPEC-1-003 § Fluxo e regras; CA-1-011; TDD GREEN | Contexto aprovado recebe `v1` rastreável | TDD GREEN de fixture válida | ID, checksum, linhas, captura e comandos | F1-T005 e F1-T007 concluídas | bloqueada |
| F1-T009 | Implementar idempotência, revisão, inválidos e controle de IA | Ethos | SPEC-1-003 § Fluxo e regras; CA-1-012 a CA-1-015; TDD REFACTOR/REGRESSÃO | Reenvio não duplica; revisão preserva anterior; inválida/IA não confirmada não é aceita | TDD de reenvio, `bom-v2`, inválida e IA + regressão | Histórico, estados pendentes e saída dos comandos | F1-T008 concluída; fixtures disponíveis | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
