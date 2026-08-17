# SPEC-1-001 — Entrada de demanda, elegibilidade e vínculo

**Fase:** 1  
**Status:** bloqueada  
**Dono:** PCP, com Comercial como fonte da demanda  
**Origem no escopo:** Fase 1; RQ-001; RQ-013; D-003  
**Degrau da solução:** construção mínima sobre o repositório React/Vite e o cliente PocketBase já existente — a página inicial está vazia e não há modelo de domínio; não será criada integração externa sem contrato aprovado.

## Contexto e decisões fechadas

- **Estado atual:** `05-Sistema/p-gina-em-branco-ai2rz8hkd/src/pages/Index.tsx` renderiza apenas uma superfície vazia. O cliente PocketBase existe em `src/lib/pocketbase/client.ts`, mas `src/lib/pocketbase/schema.json` contém somente a coleção de usuários. Não há tela, coleção ou fluxo de demanda.
- **Estado desejado:** uma demanda de teste aparece uma única vez, com origem rastreável, dados mínimos visíveis, elegibilidade explícita e vínculo a um contexto de projeto; demandas incompletas, canceladas ou duplicadas não avançam silenciosamente.
- **Decisões já fechadas:** o módulo de materiais permanece independente do Compass; o sistema não substitui ERP, não aprova pedido comercial, não planeja capacidade e não inicia compra, separação ou produção.
- **Bloqueios:** **BLOQUEIO HUMANO:** confirmar fonte oficial dos pedidos, campos mínimos, formato/mapeamento de entrada, permissões e inclusão definitiva do pré-fluxo pedido aprovado → projeto. **BLOQUEIO DE CLIENTE:** `check-cliente.md` ainda está `PENDENTE`; até essa confirmação, somente fixture controlada pode ser usada na demonstração.

## Resultado observável

Na demonstração, o PCP importa ou registra uma demanda de teste por uma entrada controlada, vê o contexto criado uma única vez e consegue distinguir `pendente_dados`, `elegivel`, `duplicada` e `cancelada`. Uma demanda elegível fica vinculada a um identificador interno de contexto e segue para `aguardando_engenharia`; nenhuma demanda sem dados mínimos cria um segundo contexto.

## Limites e dependências

- **Inclui:** entrada controlada de demanda; validação de dados mínimos; idempotência; identificação de duplicidade; criação/vínculo do contexto; estado e motivo; registro de eventos; listagem para PCP.
- **Fora de escopo:** aprovação comercial; sincronização produtiva com ERP/MaxiProd; cálculo de estoque; compras; separação; liberação; interpretação de desenho; autonomia de IA.
- **Entradas e pré-condições:** fixture JSON/CSV versionada ou fonte externa cujo contrato esteja aprovado; usuário autenticado com papel autorizado; `source_event_id` ou identificador equivalente; pedido, cliente, data prometida e produtos.
- **Saídas/artefatos:** contexto de demanda; estado atual; histórico append-only; motivo de pendência/duplicidade; vínculo de projeto quando o identificador estiver disponível; evidência da demonstração.
- **Dependências e responsáveis:** PCP valida regra operacional e trata exceções; Comercial confirma origem e dados do pedido; TI/cliente confirma acesso, autenticação e formato; Engenharia recebe o contexto na SPEC-1-002.
- **Atores e permissões mínimas:** Comercial pode criar/editar demanda em rascunho; PCP pode validar elegibilidade, vincular e reprocessar; Engenharia pode consultar o contexto, sem alterar a decisão desta SPEC; administrador técnico configura ambiente, sem aprovar demanda. **Esses papéis são proposta mínima e precisam de confirmação antes da execução.**
- **Superfícies/arquivos/configurações afetadas:** `src/pages/Index.tsx`; componentes de entrada/listagem; módulo de domínio a criar; `src/lib/pocketbase/schema.json`; regras de acesso da coleção de demandas/contextos; fixture de demonstração fora do fluxo produtivo.
- **Risco e plano B:** se a fonte externa ou a API não estiver disponível, usar fixture controlada com a mesma estrutura canônica e registrar `source_type=fixture`; nunca declarar que houve sincronização com ERP.
- **Rollback ou reversão:** não apagar histórico; invalidar a tentativa ou marcar `cancelada`; reprocessamento com o mesmo `source_event_id` deve retornar o registro existente; vínculo incorreto só pode ser corrigido por novo evento autorizado.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Fixture controlada → módulo de demandas | Fixture é fonte apenas no teste | `source_event_id`, `order_external_id`, `customer`, `promised_date`, `products[]`, `source_type`, `received_at` | Usuário de teste autenticado; sem segredo no repositório | Mesmo `source_event_id` não cria novo contexto; retry manual após erro | Exibir erro, manter tentativa pendente e registrar motivo |
| Fonte ERP/MaxiProd → módulo de demandas | **BLOQUEIO:** fonte e contrato ainda não aprovados | Mapear os mesmos campos canônicos; endpoint, método e payload pendentes | **BLOQUEIO:** credencial, escopo e ambiente pendentes | Timeout, retry e limite de chamadas dependem do contrato aprovado | Bloquear conclusão; não converter ausência em demanda válida |

**Contrato canônico provisório:** `order_external_id`, `customer`, `promised_date` e pelo menos um item em `products[]` são obrigatórios para o teste. Nomes de campos, origem oficial e transformação produtiva permanecem bloqueados até decisão humana; o executor não deve escolher equivalentes.

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-001 | `source_event_id` ou pedido já vinculado ao contexto | Reutilizar o contexto existente e registrar tentativa idempotente | Conflito de dados exige exceção para PCP | RQ-001 / AC-009 |
| RN-1-002 | Falta pedido, cliente, data prometida ou produto | Estado `pendente_dados`, motivo e responsável; não encaminhar | PCP pode corrigir e reenviar | Fase 1 / RQ-001 |
| RN-1-003 | Pedido cancelado ou inelegível | Estado terminal `cancelada`/`inelegivel`, sem novo contexto | Reativação exige novo evento autorizado | RQ-001 |
| RN-1-004 | Dados mínimos válidos e sem duplicidade | Estado `elegivel` e vínculo a contexto; encaminhar para `aguardando_engenharia` | Projeto ausente permanece pendente conforme decisão do pré-fluxo | Fase 1 / RQ-002 |

## Fluxo e regras

1. Usuário autenticado seleciona a entrada controlada e informa/recebe os dados canônicos.
2. O sistema valida presença, tipo e formato dos campos mínimos sem alterar o payload de origem.
3. O sistema procura `source_event_id` e `order_external_id` antes de criar contexto.
4. O sistema cria ou reutiliza o contexto, grava origem, ator, timestamp e motivo do estado.
5. PCP confirma elegibilidade; o estado elegível encaminha o contexto à fila de Engenharia.
6. O usuário consegue consultar a linha de histórico e demonstrar que uma repetição não criou duplicata.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Fixture válida, não repetida | Uma demanda, um contexto, estado `aguardando_engenharia` | Se persistência falhar, tentativa fica pendente e pode ser repetida sem duplicar |
| Limite | Produto vazio, data inválida ou identificador ausente | `pendente_dados` com motivo específico e sem encaminhamento | Corrigir dados e reenviar a mesma tentativa |
| Duplicidade | Mesmo `source_event_id` ou pedido já vinculado | Retorna contexto original, registra evento idempotente e não cria segundo registro | Conflito de campos abre exceção para PCP |
| Fonte indisponível | API não responde ou fixture ilegível | Nenhum estado elegível; erro visível e registro técnico sem segredo | Usar fixture aprovada no modo de demonstração; parar antes de produção |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; `03-Projeto/02-Escopo-Definitivo.md` §§3, 4, Fase 1 e 9; `03-Projeto/requisitos.md` RQ-001/RQ-013; `src/pages/Index.tsx`; `src/lib/pocketbase/schema.json`.
2. **Alterar somente:** a superfície de entrada/listagem e os artefatos de domínio necessários para esta SPEC no submódulo do sistema.
3. **Não alterar:** integrações externas, Compass, compras, estoque, separação, liberação, `.env`, credenciais ou regras sem aprovação; não ativar IA.
4. **Executar nesta ordem:** validar bloqueios; confirmar fixture; criar validação/idempotência; criar estados e histórico; demonstrar principal, limite, duplicidade e falha; executar TDD.
5. **Parar e pedir validação quando:** fonte, campos, papéis, autenticação ou inclusão do pré-fluxo não estiverem confirmados; qualquer ação exigir credencial ou publicação.
6. **Estado válido ao parar:** nenhum dado externo alterado; demandas de teste continuam auditáveis; falhas não criam duplicatas.

## Checklist de execução

- [ ] Fonte, campos mínimos, papéis e modo fixture foram confirmados pelo responsável humano.
- [ ] Uma demanda válida cria somente um contexto e registra origem/ator/timestamp.
- [ ] Dados incompletos, cancelados e duplicados ficam em estados explícitos.
- [ ] Repetição idempotente e falha de persistência foram exercitadas.
- [ ] Evidências da demonstração e do histórico foram anexadas.

## Critérios de aceite

- [ ] **CA-1-001:** uma fixture válida gera exatamente um contexto rastreável, exibido ao PCP com estado `aguardando_engenharia`.
- [ ] **CA-1-002:** reenviar a mesma demanda não cria novo contexto e mostra a referência do contexto existente.
- [ ] **CA-1-003:** entrada incompleta, cancelada ou inelegível não chega à fila de Engenharia e exibe motivo/responsável.
- [ ] **CA-1-004:** a demonstração prova o fluxo e os estados sem chamada produtiva ao ERP/MaxiProd.
- [ ] **CA-1-005:** nenhuma compra, separação, liberação ou aprovação automática ocorre como efeito colateral.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | A página atual não possui o fluxo de demanda | No submódulo, executar `pnpm build` e abrir `/`; tentar localizar o formulário de entrada | A tela está vazia e não é possível cumprir CA-1-001/002 | Captura da tela antes da entrega + commit observado |
| GREEN | Entrada válida, incompleta e duplicada | Abrir o fluxo com `order_external_id=TEST-001`, cliente, data e produto; reenviar; repetir sem cliente | Primeira entrada cria um contexto; repetição reutiliza; entrada incompleta fica pendente com motivo | Capturas, IDs dos registros e log do histórico |
| REFACTOR/REGRESSÃO | Erro de persistência e qualidade do pacote | Simular resposta indisponível sem alterar segredo; executar `pnpm lint` e `pnpm build` | Nenhuma duplicata; erro recuperável; lint/build passam | Log/captura do erro, saída dos comandos e checklist |

**Dados/fixtures:** fixture controlada contendo um pedido de portfólio, um pedido repetido, um pedido sem cliente e um pedido cancelado; nenhum dado pessoal real deve ser versionado.

**Caminhos de erro obrigatórios:** campo ausente, formato inválido, duplicidade, conflito de payload, persistência indisponível, sessão sem permissão e tentativa de acesso direto não autenticada.

**Evidência exigida:** roteiro da demonstração, capturas dos estados, identificador do contexto, histórico de eventos e saída de `pnpm lint`/`pnpm build`.

## Handoff e operação

- **Como demonstrar:** PCP insere a fixture, mostra a demanda elegível, repete o envio e abre o histórico; depois mostra uma entrada pendente.
- **Como operar depois:** Comercial fornece a demanda; PCP valida e trata pendências; nenhum operador usa o modo fixture como sincronização produtiva.
- **Como monitorar:** contador de entradas, duplicidades, pendências e falhas de persistência por período.
- **Pendência conhecida:** confirmar fonte, campos, permissões, pré-fluxo e contrato externo; registrar a decisão antes de remover o status bloqueada.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T001 | Confirmar fonte, campos mínimos, papéis, fixture e decisão sobre pré-fluxo pedido → projeto | Consultor | SPEC-1-001 § Contexto e decisões fechadas; § Dados e integrações | Contrato de entrada e matriz de permissões registrados e aprovados | Conferir contrato canônico e fixture da SPEC | Registro da decisão, fixture sem dado pessoal e matriz de acesso | Autorização humana e participantes disponíveis | bloqueada |
| F1-T002 | Implementar caminho principal de entrada e vínculo idempotente | Ethos | SPEC-1-001 § Fluxo e regras; CA-1-001; TDD GREEN | Fixture válida cria exatamente um contexto em `aguardando_engenharia` | TDD GREEN com fixture válida | Captura, ID, histórico e `pnpm lint`/`pnpm build` | F1-T001 concluída; ambiente autorizado | bloqueada |
| F1-T003 | Implementar bordas, duplicidade, pendência, cancelamento e prova final | Ethos | SPEC-1-001 § Fluxo e regras; CA-1-002 a CA-1-005; TDD REFACTOR/REGRESSÃO | Reenvio não duplica; inválida não avança; falha não gera ação externa | TDD GREEN de bordas + REFACTOR/REGRESSÃO | Capturas, logs e histórico idempotente | F1-T002 concluída; fixtures de erro disponíveis | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
