# Work Modes — ClifNotes (raiz)

Modos detectados a partir do conteúdo dos 11 arquivos do escopo + função meta da raiz como hub para 9 workspaces ICM aninhados.

## Modos detectados

| Modo | Evidências | Sugestão de silo / mecanismo |
|------|-----------|-----------------------------|
| **referência / lookup pontual** | 4 docs raiz têm formato de "manual / index / digest / mapa": tabelas de conteúdo, links, leituras panorâmicas. Padrão "achar um item específico" (ex: "qual MCP server o Jake recomenda para X?"). | Silo `reference/` |
| **biblioteca de templates copy-paste** | 6 arquivos em `workflow/` são literalmente boilerplates: folder structures + CLAUDE.md prontos com placeholders `[YOUR NAME]`. Padrão "copiar template, editar variáveis, usar". | Silo `templates/` |
| **navegação meta para sub-workspaces ICM** | 9 sub-workspaces ICM aninhados (Classrooms, ICM-main, workspace-blueprint, vault-toolkit/architectures × 3). Padrão "preciso entender X → vou para o workspace Y". | Tabela de roteamento no novo `CLAUDE.md` raiz (sem silo dedicado) |
| **arquivamento de meta-artefatos** | Overlay do retrofit anterior + backup manual em `_used/`. Material que não está em uso ativo mas vale preservar. | Silo `_archive/` (novo) e `_used/` (mantido como ilha) |

## Modos NÃO detectados

| Modo | Por quê |
|------|---------|
| escrita / produção textual | Usuário não produz aulas/posts neste diretório (ele produz dentro dos sub-workspaces se acaso). A raiz é um *vault*, não um *studio*. |
| código-fonte | Único `package.json` está dentro de ilha leave (`workspace-blueprint/claude-office-skills-ref/`). Fora do escopo. |
| client-delivery | Sem pastas com nomes próprios; sem contratos. |
| pesquisa / research | Sem `notes/`, `sources/`, PDFs externos no escopo (PDFs estão dentro de `_used/pdfs/` — ilha). |

## Ambiguidade

| Métrica | Valor |
|---------|-------|
| Arquivos ambíguos | 0 / 11 (0%) |

Padrão é claríssimo: 4 docs raiz são overview/lookup e 6 do workflow/ são templates. CLAUDE.md raiz é o único caso especial (conflito ICM, decidido como rename para `.original`).
