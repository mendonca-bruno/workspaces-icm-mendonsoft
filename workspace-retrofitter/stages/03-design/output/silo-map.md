# Silo Map — ClifNotes (raiz)

## Silos novos

| Silo | Propósito | Modos cobertos | Arquivos | Padrão de acesso |
|------|-----------|----------------|----------|------------------|
| `reference/` | Documentos panorâmicos: mapa cruzado, community digest, resource index, skills manual | referência/lookup pontual | 4 | Random access via grep |
| `templates/` | Boilerplates copy-paste: 5 CLAUDE.md examples, animation spec, 3 workflow starters | biblioteca de templates | 6 | Copy → editar → usar fora |
| `_archive/` | Meta-artefatos de retrofits passados (overlay + rollback manifest) | arquivamento | 1 dir (Classrooms-icm-overlay completo) | Esporádico, só para rollback |

## Não-silo (raiz)

| Item | Função |
|------|--------|
| `CLAUDE.md` (novo) | L0 — mapa global + roteador para sub-workspaces ICM |
| `CONTEXT.md` (novo) | L1 — roteador de tarefas |
| `CLAUDE.md.original` | Backup do L0 anterior (preservado por decisão usuário) |

## Ilhas (action=leave) — listadas no L0 como destinos

| Ilha | É ICM root? | Descrição (vai para CLAUDE.md novo) |
|------|-------------|-------------------------------------|
| `Classrooms/` | sim | Cursos do Skool reorganizados (Foundation, Building Stack, Playbooks, Vault, Archive) |
| `Interpreted-Context-Methdology-main/` | sim | Framework ICM completo + retrofitter + 5 builders/workspaces |
| `workspace-blueprint/` | sim | Workspace de exemplo "Acme DevRel" — referência viva |
| `files/` | parcial | Biblioteca vault-toolkit: 8 constraints + 3 architectures ICM + skill starters |
| `_used/` | não | Backup manual (zips, exports, PDFs) — não tocar |
