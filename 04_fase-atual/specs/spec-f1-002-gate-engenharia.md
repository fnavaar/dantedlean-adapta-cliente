# SPEC-1-002 — Fila e gate de validação da Engenharia

**Fase:** 1  
**Status:** bloqueada  
**Dono:** Engenharia  
**Origem no escopo:** Fase 1; RQ-002; RQ-013  
**Degrau da solução:** construção mínima na mesma superfície de contexto da SPEC-1-001, reutilizando autenticação e histórico do sistema existente — não substitui SolidWorks nem cria aprovação automática.

## Contexto e decisões fechadas

- **Estado atual:** o repositório não possui fila, projeto de domínio, decisão de Engenharia ou transição persistida; a página inicial é vazia e o PocketBase só declara usuários.
- **Estado desejado:** Engenharia consulta uma fila de contextos, revisa os dados/anexos disponíveis e registra `aprovado`, `devolvido` ou `bloqueado` com motivo, responsável e timestamp. Apenas `aprovado` pode alimentar a SPEC de lista de materiais.
- **Decisões já fechadas:** a decisão é humana; projeto não aprovado não gera lista de materiais, compras, separação ou liberação; projeto devolvido volta para correção e reenvio.
- **Bloqueios:** **BLOQUEIO HUMANO:** checklist mínimo de aprovação, origem/formato do projeto, permissões e alçada substituta quando o responsável estiver ausente. **BLOQUEIO DE CLIENTE:** acesso aos anexos e fonte SolidWorks ainda não foi confirmado.

## Resultado observável

Um engenheiro abre um contexto `aguardando_engenharia`, vê o pacote disponível, escolhe uma decisão e informa o motivo. O histórico prova quem decidiu e quando. Um contexto `devolvido` ou `bloqueado` não consegue iniciar o recebimento da lista; somente um contexto `aprovado` habilita a próxima SPEC.

## Limites e dependências

- **Inclui:** fila; visualização do contexto; registro de decisão; motivos obrigatórios; estados e transições; reenvio após correção; trilha de auditoria; bloqueio de avanço.
- **Fora de escopo:** desenhar produto; editar SolidWorks; interpretar desenho automaticamente; aprovar por IA; aprovar comercialmente; gerar compra; verificar estoque; liberar produção.
- **Entradas e pré-condições:** contexto criado pela SPEC-1-001; referência de projeto/anexo ou indicação explícita de ausência; usuário com papel Engenharia; checklist aprovado pelo responsável humano.
- **Saídas/artefatos:** decisão de Engenharia; motivo; ator; timestamps; versão do contexto/projeto revisada; evento que habilita ou impede a SPEC-1-003.
- **Dependências e responsáveis:** Engenharia é dona do veredito técnico; PCP mantém a fila e acompanha devoluções; Comercial corrige origem comercial; TI/cliente confirma acesso aos arquivos e autenticação.
- **Atores e permissões mínimas:** PCP pode visualizar, atribuir e devolver por pendência operacional; Engenharia pode decidir; Comercial pode corrigir dados de origem sem alterar decisão; somente Engenharia autorizada pode aprovar. A matriz real de permissões é bloqueio até validação.
- **Superfícies/arquivos/configurações afetadas:** tela de fila e detalhe de contexto; domínio de decisão; regras de transição; histórico; contrato de anexo/referência de projeto.
- **Risco e plano B:** sem acesso ao arquivo original, usar referência/fixture de projeto aprovada e marcar `bloqueado_por_fonte`; nunca tratar ausência de anexo como aprovação.
- **Rollback ou reversão:** não sobrescrever decisão anterior; uma correção cria nova revisão e novo evento; aprovação revogada deve registrar motivo e retornar o contexto a `bloqueado` ou `aguardando_engenharia` conforme regra aprovada.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Contexto da SPEC-1-001 → fila | Contexto interno e histórico | `context_id`, dados mínimos, referência do projeto, estado, versão | Sessão autenticada; PCP/Engenharia conforme papel | Abrir novamente não duplica a decisão; salvar decisão exige idempotência por comando | Erro mantém estado anterior e mostra tentativa |
| SolidWorks/origem → detalhe de projeto | **BLOQUEIO:** formato, caminho e API não confirmados | Referência, revisão, anexos e checksum somente após contrato | **BLOQUEIO:** credencial e permissão de leitura | Timeout não muda estado; retry controlado | Estado `bloqueado_por_fonte`, responsável e próximo passo |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-1-005 | Contexto elegível chega sem decisão | Estado `aguardando_engenharia` e fila visível | Falta de dados vira `bloqueado` | RQ-002 |
| RN-1-006 | Engenharia confirma checklist e pacote | Estado `aprovado`; evento habilita recebimento de lista | Checklist incompleto não pode aprovar | Fase 1 |
| RN-1-007 | Há correção necessária | Estado `devolvido`, motivo e próximo passo obrigatórios | Reenvio cria nova revisão, preservando anterior | RQ-002 |
| RN-1-008 | Fonte/anexo indisponível ou acesso negado | Estado `bloqueado`, sem avanço | PCP pode reprocessar após acesso confirmado | Riscos do escopo |

## Fluxo e regras

1. PCP encaminha um contexto elegível para a fila.
2. Engenharia abre o contexto e confirma a revisão/pacote que está sendo avaliado.
3. O sistema exige checklist e decisão; não aceita salvar aprovação sem os campos obrigatórios.
4. Engenharia escolhe `aprovado`, `devolvido` ou `bloqueado`, informa motivo quando aplicável e confirma.
5. O sistema registra autor, timestamp, revisão e evento; bloqueia a próxima etapa nos estados não aprovados.
6. Uma correção gera nova submissão sem apagar a decisão anterior.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Pacote completo e checklist confirmado | `aprovado`, evento rastreável e SPEC-1-003 habilitada | Se falhar ao salvar, estado anterior permanece |
| Limite | Anexo ausente, projeto incompleto ou checklist incompleto | `bloqueado` ou `devolvido`, com motivo e sem habilitar lista | PCP/Engenharia corrigem e reenviam |
| Reenvio | Contexto devolvido com nova revisão | Nova decisão vinculada à revisão nova; histórico anterior preservado | Conflito de revisão vira pendência |
| Sem permissão | Usuário não é Engenharia autorizada tenta aprovar | Ação negada e evento de segurança sem alterar estado | Encaminhar ao responsável correto |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; SPEC-1-001; `02-Escopo-Definitivo.md` Fase 1; `requisitos.md` RQ-002; código atual de `Index.tsx` e schema PocketBase.
2. **Alterar somente:** fila, detalhe, decisão e transições da Engenharia.
3. **Não alterar:** arquivos SolidWorks, ERP, estoque, compras, separação, liberação, credenciais, permissões fora do recorte ou qualquer aprovação automática.
4. **Executar nesta ordem:** validar checklist e papéis; criar estados; criar decisão append-only; bloquear avanço; demonstrar aprovado/devolvido/bloqueado; executar TDD.
5. **Parar e pedir validação quando:** checklist, substituto, fonte do projeto, acesso ou regra de revogação não estiverem confirmados.
6. **Estado válido ao parar:** decisões já registradas permanecem auditáveis; estados não aprovados não alimentam a lista.

## Checklist de execução

- [ ] Checklist mínimo e responsável substituto foram aprovados.
- [ ] Fila mostra contexto, revisão, responsável, idade e estado.
- [ ] Aprovação exige checklist, autor e timestamp.
- [ ] Devolução/bloqueio exige motivo e próximo passo.
- [ ] Estados não aprovados impedem a SPEC-1-003.
- [ ] Histórico e evidências de permissão foram anexados.

## Critérios de aceite

- [ ] **CA-1-006:** Engenharia consegue aprovar um contexto completo e o sistema registra autor, timestamp, revisão e evento.
- [ ] **CA-1-007:** contexto devolvido ou bloqueado não habilita recebimento/versionamento de lista.
- [ ] **CA-1-008:** aprovação sem checklist ou por usuário sem permissão é recusada sem mudar o estado.
- [ ] **CA-1-009:** reenvio após correção preserva a decisão anterior e cria referência de revisão nova.
- [ ] **CA-1-010:** falha de leitura do projeto não é interpretada como aprovação.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Não existe fila nem gate no estado atual | Abrir `/` no commit base e tentar encaminhar uma demanda de teste para Engenharia | Não há fila/estado; CA-1-006 a CA-1-010 não são atendidos | Captura da tela base e referência do commit |
| GREEN | Decisões e permissão | Criar contexto de teste; aprovar com Engenharia; repetir com usuário PCP e com checklist incompleto; devolver um segundo caso | Aprovação válida avança; demais tentativas são negadas ou ficam devolvidas/bloqueadas com motivo | Capturas, histórico e matriz de usuário/papel |
| REFACTOR/REGRESSÃO | Histórico, revisão e falha de fonte | Reenviar versão corrigida; simular anexo indisponível; executar `pnpm lint` e `pnpm build` | Histórico não é apagado, fonte indisponível bloqueia e pacote passa nos comandos | Logs, capturas e saída dos comandos |

**Dados/fixtures:** quatro contextos sem dados pessoais reais: completo, checklist incompleto, anexo ausente e devolvido com revisão nova.

**Caminhos de erro obrigatórios:** acesso negado, checklist ausente, anexo indisponível, conflito de revisão, falha de persistência e tentativa de aprovação automática.

**Evidência exigida:** captura da fila, decisão, histórico append-only, resposta negativa de permissão e saída de `pnpm lint`/`pnpm build`.

## Handoff e operação

- **Como demonstrar:** PCP encaminha um contexto; Engenharia aprova um, devolve outro e bloqueia um terceiro por falta de fonte.
- **Como operar depois:** Engenharia decide; PCP acompanha pendências; correções sempre voltam como nova revisão.
- **Como monitorar:** fila por estado, idade da pendência, devoluções sem reenvio e tentativas negadas.
- **Pendência conhecida:** checklist, alçada substituta, fonte/formato SolidWorks e permissões precisam de decisão antes do aceite.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T004 | Confirmar checklist, alçada substituta, fonte/formato do projeto e matriz de permissões | Consultor | SPEC-1-002 § Contexto e decisões fechadas; § Dados e integrações | Checklist, estados e acesso à fonte registrados | Exercitar pacote completo, incompleto e fonte indisponível | Checklist, matriz de papéis e fixture/referência | Autorização humana; responsável técnico disponível | bloqueada |
| F1-T005 | Implementar fila e caminho principal de decisão da Engenharia | Ethos | SPEC-1-002 § Fluxo e regras; CA-1-006 e CA-1-007; TDD GREEN | Aprovação válida avança; não aprovado não habilita lista | TDD GREEN de aprovado, devolvido e bloqueado | Fila, decisão, autor, timestamp e evento | F1-T002 e F1-T004 concluídas | bloqueada |
| F1-T006 | Implementar permissões, checklist obrigatório, revisão e falhas do gate | Ethos | SPEC-1-002 § Fluxo e regras; CA-1-008 a CA-1-010; TDD REFACTOR/REGRESSÃO | Ações indevidas são recusadas; revisão preserva histórico; fonte ausente bloqueia | TDD de permissão/fonte + REFACTOR/REGRESSÃO | Respostas negativas, histórico e logs | F1-T005 concluída; usuários de teste autorizados | bloqueada |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
