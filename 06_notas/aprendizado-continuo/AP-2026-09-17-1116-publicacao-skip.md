# AP-2026-09-17-1116 — Publicação no Skip depende da versão interna do builder

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T002 / SPEC-1-001
- Sinal: commit enviado ao `main` do GitHub não atualizou automaticamente o bundle do preview; a publicação exigiu sincronização pelo agente do Skip e gerou o commit interno `318b18e`.
- Evidência: `06_notas/debug/debug-2026-09-16-publicacao-f1-t002.md`, commit GitHub `c96d55b`, preview público `https://centro-operacional-dlean-9f3f5--preview.goskip.app`.
- Regra reutilizável: após publicar código por GitHub em um projeto Skip, confirmar a versão interna publicada e o bundle do preview; se a URL servir o asset anterior, solicitar sincronização/publicação no builder antes do teste humano.
- Quando aplicar: qualquer alteração feita fora do fluxo de publicação interno do Skip.
- Quando não aplicar: quando o próprio Skip confirmar a versão publicada e o preview contiver os marcadores da alteração.
- Confiança: alta — causa reproduzida e corrigida no ciclo da F1-T002.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
