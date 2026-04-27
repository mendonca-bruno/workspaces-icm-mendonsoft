# kb/controlm/ — Jobs Control-M

Roteador para dossiês de jobs e folders Control-M que orquestram batch RTB-RH.

## Sistemas cobertos

| Sistema | Área | Dossiês |
|---------|------|---------|
| _(vazio)_ | | |

## O que costuma virar dossiê separado

- **Folder Control-M**: `<sistema>/folder-<nome>.md` — fluxo completo de um conjunto de jobs relacionados (com diagrama de dependências).
- **Job crítico isolado**: `<sistema>/job-<nome>.md` quando merece atenção própria (ex.: job de fechamento de folha).
- **Família de jobs cíclicos**: agrupar.

## Pontos de atenção em ingestão

- Sources tipicamente `.JIL` (Job Information Language) ou XML exportado. Mantenha em `_sources/controlm/<sistema>/`.
- Destile: condições de entrada (pré-jobs), comando executado, pós-condições, hosts, calendário, retentativas.
- Sempre documente: o que dispara o job, o que ele dispara, error handling, janela de execução esperada.
