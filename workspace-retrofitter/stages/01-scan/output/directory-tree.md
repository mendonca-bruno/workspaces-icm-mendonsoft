# Directory Tree — `C:\Users\bruno\Downloads\ClifNotes`

**Target:** raiz `ClifNotes/`. Profundidade ≤ 4. Ilhas ICM listadas mas **não traversed**.

```
ClifNotes/
├── CLAUDE.md                                       4,479 B   ← ⚠ ICM raiz pré-existente
├── clief-notes-community-digest.md                13,878 B
├── clief_notes_resource_index_v1.md               16,213 B
├── clief_notes_skills_field_manual_v1.md          26,912 B
├── mapa_completo_clifnotes.md                     10,880 B
├── workflow/                                       (6 files, 44,006 B)
│   ├── animation-spec-templates.md                 6,189 B
│   ├── folder-organization-guide.md                8,619 B
│   ├── production-claude-md-examples.md           11,457 B
│   ├── workflow-starter-client-management.md       6,154 B
│   ├── workflow-starter-code-project.md            6,899 B
│   └── workflow-starter-content-pipeline.md        4,688 B
├── Classrooms/                          [ILHA — leave]    11 files, 220 KB
├── Classrooms-icm-overlay/              [MOVE p/ _archive] 15 files, 51 KB
├── Interpreted-Context-Methdology-main/ [ILHA — leave]   292 files, 2.7 MB
├── workspace-blueprint/                 [ILHA — leave]   194 files, 3.3 MB
├── files/                               [ILHA — leave]    52 files, 352 KB
└── _used/                               [ILHA — leave]     6 files, 1.5 MB
```

## Contagens — alvo efetivo do retrofit

| Métrica | Valor |
|---------|-------|
| Arquivos a redesenhar | 11 (1 conflito ICM + 4 .md raiz + 6 workflow) |
| Tamanho total alvo | 116,388 B (~114 KB) |
| Subpastas a criar | 3 (`reference/`, `templates/`, `_archive/`) |

## Por extensão (alvo efetivo)

| Ext | Qtd | Tamanho |
|-----|-----|---------|
| `.md` | 11 | 116,388 B |

## Ilhas detectadas (action=leave) — agregados apenas

| Caminho | Arquivos | Tamanho | Motivo |
|---------|----------|---------|--------|
| `Classrooms/` | 11 | 220 KB | Workspace ICM recém-retrofitado |
| `Interpreted-Context-Methdology-main/` | 292 | 2.7 MB | Repo todo do framework ICM (CLAUDE.md raiz + 5 sub-workspaces) |
| `workspace-blueprint/` | 194 | 3.3 MB | Workspace ICM de exemplo (CLAUDE.md raiz + 1 sub-workspace) |
| `files/` | 52 | 352 KB | Biblioteca vault-toolkit auto-organizada (3 architectures ICM internas) |
| `_used/` | 6 | 1.5 MB | Backup manual (zips, exports) |

## Movimento especial

| Caminho | Ação | Destino |
|---------|------|---------|
| `Classrooms-icm-overlay/` | move-as-dir | `_archive/2026-04-25-classrooms-overlay/` |

## Exclusões aplicadas

Defaults (`.git/`, `node_modules/`, `.venv/`, etc.) — `.git/` não existe na raiz.
Adicionais por decisão do usuário: 5 ilhas ICM (Classrooms, ICM-main, workspace-blueprint, files, _used).

## ICM pré-existente — alerta

| Arquivo | Tamanho | Status |
|---------|---------|--------|
| `CLAUDE.md` (raiz do alvo) | 4,479 B | **CONFLITO** — escrito intencionalmente. Decisão do usuário (opção C): renomear para `CLAUDE.md.original` antes de gerar novo. |

Conteúdo nota: o L0 atual referencia `Classrooms/the_foundation.md` (caminho **obsoleto** — agora em `Classrooms/learning/01-foundation.md` após retrofit anterior). Confirma necessidade de regeneração.
