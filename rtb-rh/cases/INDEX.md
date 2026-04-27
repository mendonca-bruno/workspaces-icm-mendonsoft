# cases/INDEX.md — Índice agregado de cases

Tabela auto-gerada pelo Stage 06 (manutenção) a partir do frontmatter de todos os arquivos `INC*.md` em `cases/`. **Não editar à mão.**

## Como funciona

Stage 06 lê o frontmatter de cada `cases/INC*.md` e renderiza a tabela abaixo. Se você fechou um ticket recentemente e ele não aparece aqui, rode `manutencao` (Stage 06).

## Tabela de cases

| Ticket | Área | Sistemas | Sintomas | Entities | Root Cause (resumo) | reuse_count | promoted_to |
|--------|------|----------|----------|----------|---------------------|-------------|-------------|
| _(vazio — bootstrap manual de 5-10 cases históricos é o próximo passo recomendado)_ | | | | | | | |

## Como o Stage 03 usa este arquivo

1. Lê esta tabela primeiro, antes de qualquer coisa em `kb/`.
2. Filtra por `Sistemas` ∩ ticket.systems_involved E `Sintomas` ∩ ticket.symptoms_normalized.
3. Calcula score de match: ≥2 sintomas em comum + ≥1 sistema em comum = **hit forte** → carrega caso full + dossiês citados → pula para Stage 04.
4. Sem hit forte → segue para `kb/INDEX.md`.

## Como o Stage 06 detecta clusters para promoção

Agrupa cases por `(area, systems_involved, symptoms)`. Se um agrupamento tem ≥3 cases criados nos últimos 90 dias e nenhum `promoted_to`, propõe `kb/_playbooks/<sintoma>.md`.
