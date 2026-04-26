# Detection Heuristics — Como Classificar o Diretório Alvo

Heurísticas usadas em stage 02 (Classify) para determinar tipo de workspace e modos de trabalho presentes. Aplicadas sobre o output de stage 01 (`directory-tree.md` + `file-inventory.json`).

## Tipo macro de workspace

Decisão em ordem (primeira que casar vence):

| Sinal | Tipo |
|---|---|
| `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `Gemfile`, `composer.json` na raiz ou ≤2 níveis | **code** |
| ≥60% dos arquivos são `.md`, `.txt`, `.docx`, `.pdf`, `.rtf` | **content** |
| Mistura: tem manifest de código + ≥30% de arquivos textuais | **hybrid** |
| Mistura caótica: nenhuma extensão domina, sem manifest | **personal-mess** |
| Vazio ou só uns poucos arquivos | **greenfield** |

## Modos de trabalho detectáveis

Após classificar o tipo macro, escanear sub-pastas em busca de modos de trabalho. Cada modo presente vira candidato a silo.

| Modo | Sinais |
|---|---|
| **escrita** | Pastas `drafts/`, `posts/`, `articles/`, `content/`. Arquivos `voice.md`, `style.md`. Concentração de `.md` longos (>300 palavras). |
| **produção** | Pastas `builds/`, `dist/`, `output/`, `deliverables/`, `assets/`. Volume de binários (`.mp4`, `.pdf`, `.pptx`, `.png`). |
| **distribuição** | Pastas `social/`, `newsletters/`, `events/`. Arquivos com prefixo de plataforma (`twitter-`, `linkedin-`). |
| **pesquisa** | Pastas `notes/`, `research/`, `sources/`, `references/`. PDFs externos, screenshots, bookmarks. |
| **código-fonte** | Pastas `src/`, `lib/`, `app/`, `cmd/`. Arquivos `.py`, `.ts`, `.js`, `.go`, `.rs` etc. em volume. |
| **documentação técnica** | Pasta `docs/` com `.md`/`.mdx`. ADRs em `decisions/`. README ≥200 linhas. |
| **dados** | Pastas `data/`, `etl/`, `pipelines/`, `notebooks/`. `.ipynb`, `.csv`, `.parquet`, `.sql`. |
| **cliente/projeto** | Pastas com nomes próprios (empresas/clientes). Estrutura repetida dentro delas. |
| **administração** | Contratos, invoices, propostas. `.pdf`/`.docx` com nomes formais. |
| **bagunça** | ≥30% dos arquivos não casam com nenhum dos modos acima. |

## Sinais de tech stack (quando tipo = code ou hybrid)

Reuso de heurística de [workspace-builder-v2 stages/01-analysis](../../workspace-builder-v2/stages/01-analysis/CONTEXT.md):

| Manifest | Linguagem | Frameworks típicos a procurar |
|---|---|---|
| `package.json` | JS/TS | Next.js, React, Vue, Svelte, Express, Hono |
| `pyproject.toml` / `requirements.txt` | Python | FastAPI, Django, Flask, Pandas, PyTorch |
| `go.mod` | Go | Gin, Echo, Cobra |
| `Cargo.toml` | Rust | Axum, Actix, Tauri |
| `pom.xml` / `build.gradle` | Java/Kotlin | Spring, Quarkus |
| `Gemfile` | Ruby | Rails, Sinatra |
| `composer.json` | PHP | Laravel, Symfony |
| `.csproj` / `.sln` | C# | ASP.NET |

Detectar também: test runners (`pytest.ini`, `jest.config.*`, `vitest.config.*`), CI (`.github/workflows/`, `.gitlab-ci.yml`), containers (`Dockerfile`, `docker-compose.yml`), linters (`.eslintrc`, `ruff.toml`, `.prettierrc`).

## Sinais de contexto ICM já existente

Procurar antes de qualquer reestruturação:

- `CLAUDE.md` na raiz ou subpastas
- `CONTEXT.md` na raiz ou subpastas
- `.claude/` (configs do Claude Code)
- `.cursorrules`, `.cursor/rules/`
- `.github/copilot-instructions.md`
- `AGENTS.md`, `AI.md`, `INSTRUCTIONS.md`

Se houver, **catalogar** no scan e **respeitar** no design — nunca sobrescrever sem checkpoint humano.

## Maturidade do diretório

| Nível | Sinais |
|---|---|
| **greenfield** | <10 arquivos, sem manifest, sem estrutura clara |
| **early** | Conteúdo presente, sem hierarquia, sem docs |
| **established** | Hierarquia + algum doc + algum padrão de naming |
| **mature** | Hierarquia + docs + (testes ou processo escrito) + naming consistente |

A maturidade afeta quão agressivo deve ser o redesenho. **Mature** quase sempre só precisa de arquivos ICM sobrepostos. **Personal-mess** precisa de redesenho profundo.

## Output esperado deste módulo

Stage 02 produz um `triage-report.md` com:

```
- tipo macro: <code|content|hybrid|personal-mess|greenfield>
- modos de trabalho detectados: [<lista>]
- tech stack (se aplicável): <linguagens, frameworks>
- contexto ICM pré-existente: [<arquivos encontrados>]
- maturidade: <greenfield|early|established|mature>
- arquivos ambíguos / sem categoria: <contagem + amostras>
```
