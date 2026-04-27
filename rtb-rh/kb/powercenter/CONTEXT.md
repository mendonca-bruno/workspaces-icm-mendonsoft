# kb/powercenter/ — Mappings PowerCenter

Roteador para dossiês de mappings/workflows PowerCenter que fazem ETL entre bases.

## Sistemas cobertos

| Sistema | Área | Dossiês |
|---------|------|---------|
| _(vazio)_ | | |

## O que costuma virar dossiê separado

- **Workflow PWC de ponta-a-ponta**: `<sistema>/wf-<nome>.md` (origem, mapping principal, target, schedule).
- **Mapping reusado por múltiplos workflows**: dossiê próprio em `<sistema>/mapping-<nome>.md`.
- **Conjunto de mappings que compartilham source ou target**: agrupar.

## Pontos de atenção em ingestão

- Sources tipicamente `.XML` exportados do Designer. Mantenha em `_sources/powercenter/<sistema>/` com nomes originais.
- Destile: source qualifier (que tabelas/queries lê), transformations críticas (lookup, expression, aggregator), target (tabela/arquivo destino), schedule.
- Sempre documente: pré-condições de execução (jobs upstream), pós-execução (triggers downstream), rejection rules.
