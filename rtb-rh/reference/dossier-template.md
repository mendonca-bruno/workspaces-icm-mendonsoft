# Template — Dossiê de KB

Use este molde ao criar um arquivo novo em `kb/<tech>/<sistema>/<nome>.md`. Frontmatter é **obrigatório**.

```markdown
---
title: <título descritivo curto>                # ex.: "Schema de Rubricas — Folha"
area: folha                                      # folha | ponto | beneficios | admissao | demissao
systems: [folha-mensal]                          # do area-mapping.md, kebab-case
entities: [RUBRICA, EVENTO, MOVIMENTO_FOLHA]     # tabelas/jobs/mappings/endpoints cobertos
symptoms: [valor-de-rubrica-divergente, rubrica-nao-calculou]   # ids do _taxonomies/symptoms.md
typical_queries:                                 # snippet curto, não query completa
  - "SELECT * FROM RUBRICA WHERE COD_RUB = ?"
  - "SELECT m.* FROM MOVIMENTO_FOLHA m JOIN RUBRICA r ON m.COD_RUB = r.COD_RUB"
related:                                         # paths absolutos a partir de kb/
  - java/folha/fluxo-fechamento-mensal.md
  - tibco/folha/integracao-rubricas.md
source: ../_sources/java/folha/ddl-rubricas-2025-08.sql
source_sha256: <sha256 do arquivo em _sources/>
last_distilled: 2026-04-26
---

# <Título>

## Resumo

(2-3 frases: o que este dossiê cobre, em qual sistema, para quê.)

## Tabelas e relacionamentos

(Liste as tabelas principais. Para cada: nome, propósito, PK, campos críticos. NÃO copie o DDL inteiro — referencie `_sources/...`.)

| Tabela | Propósito | PK | Campos críticos |
|--------|-----------|----|----|
| RUBRICA | Catálogo de códigos de pagamento e desconto | COD_RUB | TIPO, NATUREZA, FLAG_ATIVO |
| ... | | | |

## FKs e joins críticos

(As combinações que aparecem em 80% das queries reais.)

```sql
-- Movimentos de uma rubrica em um período
JOIN MOVIMENTO_FOLHA m ON m.COD_RUB = r.COD_RUB
WHERE m.DT_REF BETWEEN ? AND ?
```

## Regras de negócio inferidas

(O que o DDL/código diz que NÃO é óbvio: CHECK constraints, triggers, validações que vivem em código, invariantes históricas. Cite a evidência: linha do DDL, classe Java, etc.)

- `RUBRICA.TIPO IN ('P','D')` — provento ou desconto. Outras valores indicam corrupção (já visto em INC...).
- `MOVIMENTO_FOLHA.VLR_RUB` é `NUMBER(15,2)` mas trigger zera se `RUBRICA.FLAG_ATIVO = 'N'` no momento do cálculo.

## Padrões de uso (queries típicas em produção)

(O que o time roda toda hora. Útil para o Stage 04 reusar.)

```sql
-- Diagnóstico: rubrica calculou para um funcionário?
SELECT *
  FROM MOVIMENTO_FOLHA
 WHERE NUM_MATRICULA = :matricula
   AND COD_RUB       = :rubrica
   AND DT_REF        = :dt_ref;
```

## Casos referenciados

(Cases que tocaram essas entidades. Auto-populado pelo Stage 06; pode ser editado à mão.)

- `cases/INC1234567.md` — rubrica de 13o zerada
- `cases/INC1234890.md` — divergencia em RUBRICA tipo D
```
