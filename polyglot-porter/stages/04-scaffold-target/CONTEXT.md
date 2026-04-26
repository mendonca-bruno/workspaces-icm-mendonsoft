# Stage 04: Scaffold Target

Gerar o esqueleto do target: build files, dep manifest, layout de pastas, ICM L0/L1 na linguagem alvo. Sem código de domínio ainda.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Stage 03 | `../03-pattern-mapping/output/APPROVED.md` | Existência | Gate de execução |
| Stage 03 | `../03-pattern-mapping/output/translation-map.csv` | Completo (versão pós-checkpoint #2) | Que arquivos criar |
| Stage 02 | `../02-target-discovery/output/target-decisions.md` | Completo | Versões e flavors |
| Stage 02 | `../02-target-discovery/output/dependency-map.json` | Completo | Lista de pacotes a declarar |
| Stage 02 | `../02-target-discovery/output/reference-distillation.md` | Completo (se existir) | Layout/naming/libs custom — precedência sobre templates default |
| Templates | `../../shared/templates/<target-lang>/` | Apenas a pasta do target-lang | Esqueletos paramétricos (fallback se sem references) |

## Process

1. Verificar existência de `APPROVED.md` em stage 03. Abortar se ausente.
2. Determinar overlay path: `<source_dir>-port-<target_lang>/` (sibling do source). Se já existir, sufixar com timestamp.
3. Criar pasta overlay e estrutura idiomática. **Precedência:** se `reference-distillation.md` existe e define layout, usar. Senão, fallback para layout do `target-decisions.md` § "Layout" (que vem do archetype/golden).
4. Renderizar build file (pom.xml | .csproj+.sln | pyproject.toml) com:
   - Versões travadas
   - Dependências do `dependency-map.json` (incluindo libs custom detectadas em references)
   - Plugin de testes (preferindo framework usado pelas references se houver)
5. Renderizar ICM L0/L1 do target:
   - `<overlay>/CLAUDE.md` — descreve o projeto migrado, link de volta para source
   - `<overlay>/CONTEXT.md` — routing simples para módulos do target
6. Criar pastas vazias para cada `target_type` distinto no CSV. **Nomes seguem `reference-distillation.md`** se definido (ex: `Application/`, `Infrastructure/`, `Web/` em vez de `Controllers/`+`Services/` planos), senão archetype default.
7. Renderizar `MIGRATION-MANIFEST.md` no overlay:
   - Source path
   - Target stack-flavor
   - Data da migração
   - Link para `translation-map.csv`
   - Como rodar build/testes
8. Renderizar CI workflow (`.github/workflows/ci.yml` ou equivalente) baseado em template do target.
9. Não criar código de domínio (deixar para stage 05). Mas gerar arquivo placeholder em cada pasta principal com comentário `// TODO: populated by stage 05`.

## Checkpoints

Sem gate humano interno (próximo é após stage 06).

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| Overlay criado em path correto | Sibling do source |
| Build file parseia | `mvn validate` / `dotnet restore --dry-run` / `python -c "import tomllib; tomllib.load(open('pyproject.toml','rb'))"` |
| Pastas idiomáticas existem | Match com archetype |
| `MIGRATION-MANIFEST.md` presente | — |
| CI workflow YAML válido | parse OK |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Skeleton do target | `<source>-port-<target-lang>/` (overlay) | Estrutura completa com build + L0/L1 + pastas |
| Log de scaffold | `output/scaffold-log.md` | MD: lista de arquivos criados, versões travadas, comandos de validação |

## Hand-off

Stage 05 lê CSV (autoridade) + scaffold + golden-examples para popular cada arquivo.
