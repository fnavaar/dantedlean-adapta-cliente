# Debug Summary — publicação no projeto-base

- **Data:** 2026-09-16
- **Task:** F1-T002
- **Sintoma:** o preview `https://centro-operacional-dlean-9f3f5.goskip.app` mostrava somente o shell `Centro Operacional Dlean`, embora a implementação da entrada de demanda estivesse validada no checkout local.
- **Reprodução:** o `main` do repositório conectado ao Skip estava em `92a4ee9`, com `src/pages/Index.tsx` vazio; a alteração da T02 estava apenas no checkout local. O preview público confirmou a tela antiga.
- **Causa raiz:** a implementação não estava no `main` do repositório que alimenta o projeto-base público.
- **Correção:** a T02 foi reaplicada sobre o `main` remoto de `dantedlean/p-gina-em-branco-ai2rz8hkd` e publicada via conector GitHub no commit `c96d55b`, adicionando `src/lib/demand/domain.ts`, `src/lib/demand/fixtures.ts` e atualizando `src/pages/Index.tsx`.
- **Verificação local:** TypeScript, lint, build, prova de domínio e smoke test passaram; o smoke test criou um contexto único em `aguardando_engenharia` e o reenvio gerou `ingestion_replayed` sem duplicar.
- **Verificação pública:** no momento do registro, o domínio ainda retornava o shell antigo; a publicação do Skip precisa propagar o commit antes do teste humano.
- **Próximo passo:** task F1-T002 fechada após aprovação humana; não iniciar nova task automaticamente.

## Verificação pública posterior

- **Preview:** https://centro-operacional-dlean-9f3f5--preview.goskip.app
- **Resultado:** tela Entrada de demanda publicada; fixture criou 1 contexto em `aguardando_engenharia`; reenvio manteve o mesmo ID e acrescentou `ingestion_replayed`.
- **Gate:** teste humano aprovado por Andre em 2026-09-17.
