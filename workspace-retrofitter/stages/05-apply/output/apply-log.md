# Apply Log — ClifNotes raiz (2026-04-26)

**Target:** `C:\Users\bruno\Downloads\ClifNotes`
**Overlay:** `C:\Users\bruno\Downloads\ClifNotes-icm-overlay`
**Authorized by:** brunooms21@gmail.com (verbal: "aplly")
**Rollback manifest:** `<overlay>/rollback-manifest.json`

## Pré-flight

| Check | Result |
|-------|--------|
| `APPROVED.md` presente | ✓ |
| 11 sources atómicas existem | ✓ |
| Source `Classrooms-icm-overlay/` existe | ✓ |
| Destinos novos sem colisão (4 verificados) | ✓ |
| 5 ilhas intactas (existência) | ✓ |

## Operações executadas (7 fases, set -e)

### [1] Resolver conflito ICM PRIMEIRO

| Source | Destination | Status |
|--------|-------------|--------|
| `CLAUDE.md` (4.5 KB) | `CLAUDE.md.original` | ✓ (preserva L0 anterior antes de gerar novo) |

### [2] mkdir 3 silos

`reference/`, `templates/`, `_archive/` ✓

### [3] move 4 docs raiz → `reference/`

| Source | Destination |
|--------|-------------|
| `clief-notes-community-digest.md` | `reference/community-digest.md` ✓ |
| `clief_notes_resource_index_v1.md` | `reference/resource-index.md` ✓ |
| `clief_notes_skills_field_manual_v1.md` | `reference/skills-field-manual.md` ✓ |
| `mapa_completo_clifnotes.md` | `reference/mapa-completo.md` ✓ |

### [4] move 6 `workflow/*` → `templates/`

| Source | Destination |
|--------|-------------|
| `workflow/folder-organization-guide.md` | `templates/folder-organization-guide.md` ✓ |
| `workflow/production-claude-md-examples.md` | `templates/claude-md-examples.md` ✓ |
| `workflow/animation-spec-templates.md` | `templates/animation-spec-template.md` ✓ |
| `workflow/workflow-starter-content-pipeline.md` | `templates/starter-content-pipeline.md` ✓ |
| `workflow/workflow-starter-code-project.md` | `templates/starter-code-project.md` ✓ |
| `workflow/workflow-starter-client-management.md` | `templates/starter-client-management.md` ✓ |

### [5] rmdir `workflow/`

✓ Removida (estava vazia após os 6 moves).

### [6] move-as-dir `Classrooms-icm-overlay/` → `_archive/2026-04-25-classrooms-overlay/`

✓ Operação atômica de pasta inteira; estrutura interna e `rollback-manifest.json` do retrofit anterior preservados (15 arquivos).

### [7] write 5 ICM novos

| Source (overlay) | Destination | Layer |
|------------------|-------------|-------|
| `CLAUDE.md` | `CLAUDE.md` | L0 ✓ |
| `CONTEXT.md` | `CONTEXT.md` | L1 ✓ |
| `reference/CONTEXT.md` | `reference/CONTEXT.md` | L2 ✓ |
| `templates/CONTEXT.md` | `templates/CONTEXT.md` | L2 ✓ |
| `_archive/README.md` | `_archive/README.md` | meta (silo passivo) ✓ |

## Pós-flight

| Check | Result |
|-------|--------|
| Integridade byte-a-byte (vs inventário stage 01) | ✓ 11/11 |
| 6 sources removidos da raiz / `workflow/` | ✓ |
| `Classrooms-icm-overlay/` ausente da raiz | ✓ |
| `workflow/` ausente da raiz | ✓ |
| 5 ilhas intactas (count batendo) | ✓ Classrooms 11, ICM-main 292, workspace-blueprint 194, files 52, _used 6 |
| 4 ICM raiz/silos presentes | ✓ |
| `CLAUDE.md.original` preservado (4.479 B) | ✓ |
| `_archive/2026-04-25-classrooms-overlay/` preservado | ✓ 15 arquivos (rollback-manifest interno + 14 ICM/meta do overlay anterior) |

## Estado final do alvo (top level)

```
ClifNotes/
├── CLAUDE.md                                  (3.810 B — novo L0 meta-vault)
├── CLAUDE.md.original                         (4.479 B — L0 anterior preservado)
├── CONTEXT.md                                 (1.922 B — novo L1)
├── reference/                                 (4 docs + CONTEXT.md)
├── templates/                                 (6 boilerplates + CONTEXT.md)
├── _archive/                                  (silo passivo)
│   ├── README.md
│   └── 2026-04-25-classrooms-overlay/         (15 arquivos do overlay anterior)
├── Classrooms/                    [ILHA]      (11 files, intacta)
├── Interpreted-Context-Methdology-main/ [ILHA] (292 files, intacta)
├── workspace-blueprint/           [ILHA]      (194 files, intacta)
├── files/                         [ILHA]      (52 files, intacta)
└── _used/                         [ILHA]      (6 files, intacta)
```

## Reversão

```
apply --rollback
```
Lê `<overlay>/rollback-manifest.json` em ordem inversa: deleta os 5 ICM novos, restaura `CLAUDE.md.original` → `CLAUDE.md`, move arquivos de volta para raiz/`workflow/` (recriando `workflow/`), restaura `Classrooms-icm-overlay/` na raiz, remove as 3 pastas vazias.

## Follow-ups recomendados (NÃO executados — fora do escopo)

1. **Atualizar `reference/mapa-completo.md`**: corrigir paths obsoletos do `Classrooms/` (era pré-retrofit). Trocar por `Classrooms/learning/01-foundation.md` etc. Tarefa de conteúdo, não de estrutura.
2. **Auditar referências cruzadas** em `reference/resource-index.md` e `reference/skills-field-manual.md` para paths que possam ter mudado.
3. **Decidir destino final do overlay arquivado**: depois de algumas semanas sem precisar de rollback do Classrooms, pode-se deletar `_archive/2026-04-25-classrooms-overlay/` para liberar 51 KB.

## Status final

✅ **APPLY CONCLUÍDO** — ClifNotes raiz é agora um meta-vault ICM funcional. 5 ilhas preservadas como destinos navegáveis pelo novo `CLAUDE.md` raiz. Overlay (`ClifNotes-icm-overlay/`) pode ser arquivado/deletado; preservar o `rollback-manifest.json` interno se quiser opção de reversão futura.
