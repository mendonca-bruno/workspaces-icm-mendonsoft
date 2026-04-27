# kb/INDEX.md — Índice agregado de dossiês

Tabela auto-gerada pelo Stage 06 (manutenção) a partir do frontmatter de todos os dossiês em `kb/<tech>/<sistema>/*.md`. **Não editar à mão.**

## Como funciona

Stage 06 lê o frontmatter de cada `.md` em `kb/<tech>/<sistema>/` e renderiza a tabela abaixo. Se você precisa atualizar este índice, rode o gatilho `manutencao` (Stage 06).

## Tabela de dossiês

| Dossiê | Área | Sistemas | Entidades | Sintomas | last_distilled |
|--------|------|----------|-----------|----------|----------------|
| _(vazio — workspace recém-bootstrapped; rode Stage 01 com a primeira fonte)_ | | | | | |

## Como o Stage 03 usa este arquivo

1. Lê esta tabela primeiro (rápido — apenas frontmatters agregados).
2. Filtra por `Sistemas` ∩ ticket.systems_involved e `Sintomas` ∩ ticket.symptoms_normalized.
3. Carrega 1-3 dossiês candidatos full.
4. Não varre dossiês não-listados aqui (se um dossiê novo não aparece, rode `manutencao`).

## Schema esperado do frontmatter (para cada dossiê)

```yaml
---
title: <título descritivo>
area: <folha|ponto|beneficios|admissao|demissao>
systems: [<sistema>]
entities: [<TABELA|JOB|MAPPING|ENDPOINT>]
symptoms: [<sintoma normalizado da taxonomia>]
typical_queries: [<SELECT exemplo curto>]
related: [<path/para/outro-dossie.md>]
source: ../_sources/<tech>/<sistema>/<arquivo>
source_sha256: <hash>
last_distilled: <YYYY-MM-DD>
---
```
