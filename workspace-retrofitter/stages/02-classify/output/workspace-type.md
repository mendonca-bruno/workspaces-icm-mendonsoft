# Workspace Type — ClifNotes (raiz)

| Campo | Valor |
|-------|-------|
| **tipo macro** | `content` (no escopo do retrofit; visão global é meta-vault) |
| **maturidade** | `mature` (visão global) / `early` (escopo do retrofit) |
| **stack técnico** | n/a no escopo (Node.js só dentro de ilha `workspace-blueprint`) |
| **ICM pré-existente** | 1 conflito (CLAUDE.md raiz) + 9 ilhas com ICM próprio |

## Justificativa

**tipo = content.** No escopo do retrofit (após excluir as 5 ilhas), 100% dos 11 arquivos são `.md`. Sem manifests de stack ativos no caminho a redesenhar.

**maturidade dual:**
- **Visão global = `mature`**: o ClifNotes raiz já contém 3 sub-workspaces ICM completamente formados (`Interpreted-Context-Methdology-main/`, `workspace-blueprint/`, `files/vault-toolkit/`) com hierarquia, naming e padrões. É um **meta-vault** funcional.
- **Visão do escopo retrofit = `early`**: a parte "não-ilha" (raiz + `workflow/`) tem 11 arquivos sem hierarquia, com CLAUDE.md raiz desatualizado (referencia `Classrooms/the_foundation.md` que foi movido no retrofit anterior).

O retrofit deste alvo é, fundamentalmente, **dar uma camada L0/L1 de roteamento meta** que conecte os documentos soltos da raiz aos sub-workspaces ICM, mais 2 silos novos (`reference/`, `templates/`) para os arquivos órfãos.

## Conteúdo identificado

### Arquivos a redesenhar (11)

| Arquivo | Tipo de conteúdo |
|---------|------------------|
| `CLAUDE.md` (raiz, 4.5 KB) | ICM L0 obsoleto — caminhos quebrados após retrofit do Classrooms |
| `mapa_completo_clifnotes.md` | Mapa cruzando cursos Skool × arquivos locais (referência cruzada) |
| `clief-notes-community-digest.md` | Resumo dos posts mais engajados da comunidade Skool (24.4k membros) |
| `clief_notes_resource_index_v1.md` | Resource Index — 40+ links curados (Anthropic repos, MCP, Remotion, etc.) |
| `clief_notes_skills_field_manual_v1.md` | Manual das 10 capabilities do Claude (Projects, Skills, MCP, Memory, etc.) |
| `workflow/folder-organization-guide.md` | Guia do 3-layer system (CLAUDE/CONTEXT/Skills) com routing tables |
| `workflow/production-claude-md-examples.md` | 5 CLAUDE.md prontos por perfil (creator, dev, agency, etc.) |
| `workflow/animation-spec-templates.md` | Template de spec para animation videos |
| `workflow/workflow-starter-content-pipeline.md` | Boilerplate folder + CLAUDE.md para content pipeline |
| `workflow/workflow-starter-code-project.md` | Boilerplate para code project |
| `workflow/workflow-starter-client-management.md` | Boilerplate para client management |

### Ilhas (action=leave)

5 sub-trees auto-organizados, alguns já com ICM completo. **Não tocar.** Apenas referenciar no roteamento do novo L0.

### Movimento especial

`Classrooms-icm-overlay/` (15 arquivos, 51 KB): overlay/manifest do retrofit anterior. Mover como diretório para `_archive/2026-04-25-classrooms-overlay/` preservando `rollback-manifest.json`.
