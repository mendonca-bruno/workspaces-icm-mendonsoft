# Silo Archetypes — Catálogo de Padrões

Cada arquétipo descreve um tipo de silo que pode aparecer no diretório alvo. Use como cardápio em stage 03 (Design). **Não force arquétipos que não correspondem ao trabalho real do alvo.**

Cada arquétipo lista: quando aparece, sinais detectáveis no scan, contrato L2 típico, e exemplos de L3 que costumam acompanhá-lo.

---

## writing-room

**Quando aparece:** trabalho de criação textual (artigos, drafts, roteiros, documentação narrativa).

**Sinais no scan:**
- ≥40% de arquivos `.md` / `.txt` / `.docx` em pastas como `drafts/`, `posts/`, `content/`, `articles/`
- Presença de subpastas tipo `final/`, `published/`, `archive/`
- Arquivos `voice.md`, `style-guide.md`, `tone.md` em qualquer lugar

**Contrato L2 típico:**
```
Inputs: docs/voice.md, docs/style-guide.md, docs/audience.md
Process: brief → outline → draft → review → final
Outputs: drafts/<slug>-draft.md, final/<slug>-final.md
```

**L3 que costuma acompanhar:** `voice.md`, `style-guide.md`, `audience.md`, `editorial-calendar.md`.

---

## production

**Quando aparece:** transformação de drafts/specs em deliverables (vídeos, slides, apps demo, PDFs, builds).

**Sinais no scan:**
- Pastas `builds/`, `output/`, `deliverables/`, `dist/`, `assets/`
- Arquivos binários (`.mp4`, `.pdf`, `.pptx`, `.png` em volume)
- Pasta `src/` ao lado de `assets/` ou `output/`

**Contrato L2 típico:**
```
Inputs: writing-room/final/, docs/tech-standards.md, docs/design-system.md
Process: brief → spec → build → output
Outputs: output/<slug>-v<n>.<ext>
```

**L3 típico:** `tech-standards.md`, `component-library.md`, `design-system.md`, `brand-assets.md`.

---

## community

**Quando aparece:** distribuição multi-formato (newsletters, social, eventos, repurposing).

**Sinais no scan:**
- Pastas `social/`, `newsletters/`, `events/`, `posts/`
- Arquivos com prefixo de plataforma: `twitter-`, `linkedin-`, `instagram-`
- Calendário ou planilhas de cadência

**Contrato L2 típico:**
```
Inputs: writing-room/final/ (voice), production/output/ (assets), docs/platforms.md
Process: pick source → adapt to platform → schedule
Outputs: content/<platform>/<slug>.md
```

**L3 típico:** `platforms.md`, `calendar-rules.md`, `tone-per-platform.md`.

---

## research

**Quando aparece:** coleta e síntese de informação externa antes de produção.

**Sinais no scan:**
- Pastas `notes/`, `research/`, `references/`, `sources/`
- Muitos PDFs, screenshots, links salvos
- Arquivos com nomes tipo `<topic>-notes.md`, `<topic>-summary.md`

**Contrato L2 típico:**
```
Inputs: docs/research-questions.md, sources/
Process: collect → tag → synthesize → handoff
Outputs: synthesis/<topic>.md, briefs/<topic>-brief.md
```

**L3 típico:** `research-questions.md`, `source-evaluation.md`, `glossary.md`.

---

## code-src

**Quando aparece:** repositório de código (qualquer linguagem).

**Sinais no scan:**
- `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`
- Pastas `src/`, `lib/`, `app/`, `cmd/`
- Arquivos de tests ao lado (`tests/`, `__tests__/`, `_test.go`)

**Contrato L2 típico:**
```
Inputs: docs/architecture.md, docs/conventions.md, schemas/
Process: feature brief → design → implement → test → review
Outputs: src/<feature>/, tests/<feature>.test.<ext>
```

**L3 típico:** `architecture.md`, `conventions.md`, `api-contracts.md`, `data-models.md`.

**Importante:** raramente é o silo único. Costuma vir com `code-docs` e às vezes `data-pipeline`.

---

## code-docs

**Quando aparece:** documentação técnica do projeto, separada do código fonte.

**Sinais no scan:**
- Pasta `docs/` com `.md`/`.mdx`/`.rst`
- `README.md` extenso na raiz
- ADRs (Architecture Decision Records) em `docs/decisions/` ou `adr/`
- Site de docs (Docusaurus, MkDocs, Sphinx configs)

**Contrato L2 típico:**
```
Inputs: code-src/ (para validar exemplos), docs/style.md
Process: gap analysis → outline → write → publish
Outputs: docs/<section>/<page>.md
```

**L3 típico:** `style.md`, `terminology.md`, `audience.md`.

---

## data-pipeline

**Quando aparece:** processamento de dados estruturados (ETL, análise, ML).

**Sinais no scan:**
- Notebooks `.ipynb`, scripts `.py`/`.R` em `pipelines/`, `etl/`, `notebooks/`
- Pasta `data/` com `raw/`, `processed/`, `output/`
- Configs de orquestração (Airflow, Prefect, Dagster)

**Contrato L2 típico:**
```
Inputs: data/raw/, schemas/, docs/data-contracts.md
Process: extract → validate → transform → load → report
Outputs: data/processed/, reports/<date>-<topic>.md
```

**L3 típico:** `data-contracts.md`, `schema-registry.md`, `quality-rules.md`.

---

## client-delivery

**Quando aparece:** trabalho organizado por cliente/projeto, com handoffs externos.

**Sinais no scan:**
- Pastas com nomes de empresa/cliente
- Subpastas repetidas: `<client>/briefs/`, `<client>/deliverables/`
- Contratos, propostas, SOWs em `.pdf`/`.docx`

**Contrato L2 típico:**
```
Inputs: clients/<name>/brief.md, docs/process.md
Process: discovery → proposal → execute → deliver → invoice
Outputs: clients/<name>/deliverables/, clients/<name>/invoices/
```

**L3 típico:** `process.md`, `pricing.md`, `templates/proposal.md`, `templates/sow.md`.

---

## archive (silo especial)

**Quando aparece:** sempre que há arquivos legados, completos, ou sem destino ativo.

**Uso:** destino-padrão para arquivos que o stage 03 não consegue mapear com confiança. Marcar como `manual review` no `file-mapping.csv` e mover para `_legacy/` ou `archive/` em vez de inventar silo artificial.

---

## Regras de combinação

| Arquétipos comuns juntos | Padrão |
|---|---|
| writing-room + production + community | Content creator / DevRel team (= [workspace-blueprint](../../../../workspace-blueprint/)) |
| code-src + code-docs | Software project clean |
| code-src + code-docs + data-pipeline | Data platform / ML project |
| client-delivery + writing-room + production | Agency / freelancer |
| research + writing-room | Newsletter / journalist |

**Regra:** se nenhuma combinação encaixa, prefira menos silos. Um diretório com 30 arquivos raramente precisa de mais de 2 silos.
