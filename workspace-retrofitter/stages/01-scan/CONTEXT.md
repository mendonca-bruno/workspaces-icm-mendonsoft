# Stage 01: Scan

Análise read-only do diretório alvo. Nenhuma inferência, nenhuma modificação. Apenas inventário factual.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Usuário | `--target=<path>` | Caminho absoluto do diretório alvo | O que vamos analisar |
| Setup | `../../setup/questionnaire.md` (respostas) | Restrições do usuário | Pastas a ignorar, .git, segredos |

## Process

1. Verificar que o caminho alvo existe e é diretório.
2. Ler `.gitignore`, `.dockerignore` e o questionnaire para construir lista de exclusão (sempre incluir: `.git/`, `node_modules/`, `.venv/`, `__pycache__/`, `dist/`, `build/`, `target/`).
3. Walk recursivo: para cada arquivo registrar caminho relativo, extensão, tamanho em bytes, data de modificação.
4. Contar agregados por extensão e por pasta de primeiro/segundo nível.
5. Detectar manifests de tech stack na raiz e até 2 níveis: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `Gemfile`, `composer.json`, `*.csproj`.
6. Detectar contexto ICM pré-existente: `CLAUDE.md`, `CONTEXT.md`, `.claude/`, `.cursorrules`, `AGENTS.md`. Catalogar com caminho e tamanho.
7. Renderizar árvore de diretórios (profundidade ≤4) em formato markdown.
8. Escrever os outputs.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|------------------|---------------|
| 8 | `directory-tree.md` em formato visual + contagens | Se algo importante foi excluído por engano |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Caminho alvo é válido | Existe e é diretório legível |
| `.git/` não foi escaneado | Nenhuma entrada em `file-inventory.json` começa com `.git/` |
| Inventário não-vazio | ≥1 arquivo listado (ou alvo é greenfield e isso é confirmado) |
| Arquivos ICM pré-existentes catalogados | Campo `existing_icm_files` em `file-inventory.json` |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Árvore visual | `output/directory-tree.md` | Markdown com `tree`-style |
| Inventário | `output/file-inventory.json` | JSON com array de `{path, ext, size, mtime}` + agregados |
| Manifests detectados | `output/tech-manifests.json` | JSON: `{manifest_path, type, declared_deps[]}` |

## Hand-off

Stage 02 (Classify) lê os 3 outputs deste stage. Não modifique outputs após Stage 02 começar.
