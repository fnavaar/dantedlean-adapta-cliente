# AGENTS.md

Instruções para agentes de código neste repositório. **A fonte de verdade é o
[`CLAUDE.md`](CLAUDE.md)** — leia-o antes de agir. Em resumo:

1. Este é o repositório do projeto de um cliente da consultoria Adapta. O trabalho avança
   **uma fase por vez** (`04_fase-atual/`); o estado vive em `STATUS.md`.
2. Não edite specs nem o plano — dúvidas viram registro no `changelog.md` para o consultor.
3. Task só fecha com o critério de pronto binário cumprido, com evidência.
4. **Ciclo de liberação (obrigatório, vale para esta e todas as tasks posteriores):**
   autorização explícita do consultor → execução → evidências de TDD → teste humano do
   champion → recibo de liberação em `03_documentos/liberacao-de-tasks.md`. Nenhuma task
   começa sem o passo 1; nenhuma é liberada sem os passos 3 e 4. Uma task por vez.
5. **Ambiente único:** o sistema é `05-Sistema/p-gina-em-branco-ai2rz8hkd` (Skip 52812).
   Não crie repositório, projeto Skip ou app novo para o escopo do plano sem autorização
   explícita do consultor. Projeto paralelo é registrado como "fora de escopo" antes de
   ir a produção.
6. Prefira `/adapta-cliente:trabalhar` para abrir a próxima task e
   `/adapta-cliente:finalizar-task` para validar/fechar. Se houver erro tecnico, use
   `/adapta-cliente:destravar-task` antes de tentar concluir.
7. Toda ação relevante → `changelog.md`; progresso → `STATUS.md`.
8. Conteúdo confidencial do projeto; tudo em português.
