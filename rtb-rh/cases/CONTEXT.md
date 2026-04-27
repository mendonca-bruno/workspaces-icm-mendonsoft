# cases/ — Registry tipado de tickets resolvidos

Cada caso é 1 arquivo `INC<numero>.md` com frontmatter obrigatório. Stage 03 (busca) consulta `cases/INDEX.md` ANTES de qualquer dossiê de KB. Stage 05 (pós-mortem) escreve aqui quando ticket é resolvido.

## Como navegar

1. Use `INDEX.md` (tabela agregada). Não varra os arquivos individuais.
2. Filtre por `area`, `systems_involved`, ou `symptoms`.
3. Carregue 1-3 casos candidatos full.
4. Veja `dossiers_cited:` no frontmatter para puxar dossiês de KB relacionados.

## Schema obrigatório do frontmatter

Veja `reference/case-template.md` para o template completo. Resumo dos campos:

| Campo | Tipo | Obrigatório? | Descrição |
|-------|------|--------------|-----------|
| `ticket_id` | string | sim | INC* ou número do ticket no sistema de origem |
| `created` | date | sim | YYYY-MM-DD da resolução |
| `area` | enum | sim | folha\|ponto\|beneficios\|admissao\|demissao |
| `systems_involved` | list | sim | sistemas tocados na resolução (do `area-mapping.md`) |
| `symptoms` | list | sim | sintomas normalizados (da `symptoms.md`) |
| `entities` | list | recomendado | tabelas/jobs/mappings tocados |
| `root_cause` | string | sim | descrição curta da causa raiz |
| `resolution_query` | string \| YAML literal | sim para casos com fix SQL | query/queries que resolveram |
| `resolution_steps` | list | sim | passos da resolução |
| `dossiers_cited` | list | recomendado | dossiês de KB usados na resolução |
| `duration_min` | int | opcional | tempo gasto |
| `reuse_count` | int | sim (inicial 0) | quantas vezes este caso foi citado em outras resoluções |
| `promoted_to` | string \| null | sim (inicial null) | path do playbook se foi promovido |

## Ciclo de vida

```
ticket resolvido
   │
   ▼ (Stage 05)
cases/INC<id>.md (singular)
   │
   ▼ (Stage 06 detecta cluster ≥3 com mesmos symptoms+systems)
proposta de promoção
   │
   ▼ (humano aprova)
kb/_playbooks/<sintoma>.md (criado)
   │
   ▼
cases originais marcados com `promoted_to: <playbook>`
   │
   ▼ (após 6 meses sem reuse)
candidatos a `cases/_archive/`
```

## Anti-pattern: cases/ virar dump

- **Sintoma**: arquivos sem frontmatter completo, sem `resolution_query`, com `root_cause` vago.
- **Mitigação**: Stage 05 tem `Verify` que rejeita arquivo sem schema. Bruno preenche os campos quando fecha o ticket — ~5 min — não depois.
