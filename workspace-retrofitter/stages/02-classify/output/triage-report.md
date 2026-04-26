# Triage Report — ClifNotes (raiz)

## Tipo & Maturidade

- **Tipo macro:** `content` (no escopo retrofit)
- **Maturidade:** `mature` global / `early` escopo
- **Stack:** n/a no escopo
- **ICM pré-existente:** **1 conflito raiz** (CLAUDE.md, decisão usuário: rename para `.original`) + **9 ilhas ICM** auto-contidas (action=leave)
- **Audiência:** `solo+ai`
- **Estrutura prévia:** mista (raiz desorganizada, ilhas internas organizadas)

## Modos confirmados

1. **referência / lookup pontual** — 4 docs raiz (digest, index, manual, mapa)
2. **biblioteca de templates copy-paste** — 6 arquivos em workflow/
3. **navegação meta para 9 sub-workspaces ICM** — via roteamento no novo L0
4. **arquivamento** — overlay antigo + _used/ (backup manual)

## 60/30/10 Triage

### Por modo

| Modo | Camada predominante | Justificativa |
|------|---------------------|---------------|
| referência | **60 (infra)** + **10 (AI)** | Index + grep cobrem 90% do uso. AI agrega quando o usuário pergunta "qual desses templates serve para X?" |
| templates | **60 (infra)** | Copiar arquivo é file system. AI só na personalização ocasional. |
| navegação meta | **30 (orq)** | Roteador estático no L0. Decisão "tarefa X → vou para workspace Y" é tabela. |
| arquivamento | **60 (infra)** | Storage + naming por data. Sem AI. |

### AI candidates to demote

Nenhum. Tudo no escopo é referência estática ou template — não há prompts/AI usados de forma errada aqui.

### AI candidates to adopt

| Oportunidade | Por quê | Onde implementar |
|--------------|---------|------------------|
| **Atualizar `mapa_completo_clifnotes.md`** com caminhos corretos pós-retrofit do Classrooms | Mapa referencia `Classrooms/the_foundation.md` que não existe mais | **Fora do escopo deste retrofit** — registrar como follow-up no apply-log |
| **Tabela de roteamento meta** entre os 9 sub-workspaces ICM | Hoje só o L0 raiz lista alguns; falta uma tabela "se você quer fazer X, vá para workspace Y" | Novo `CLAUDE.md` raiz (gerado por stage 04) |

## Arquivos ambíguos

| Métrica | Valor |
|---------|-------|
| Total ambíguos | 0 / 11 |
| Ambiguidade | 0% |

## Trade-offs sinalizados para Stage 03

1. **Renomeação dos 4 docs raiz.** Strip dos prefixos redundantes (`clief_notes_`, `clief-notes-`) e do sufixo `_v1`, já que estão dentro de `ClifNotes/reference/`. Default: sim. Se preferir preservar nomes originais, edite o CSV.
2. **Renomeação dos 6 templates.** Strip do prefixo `workflow-starter-` e `production-` (redundantes em `templates/`). Default: sim.
3. **`Classrooms-icm-overlay/` como `move-as-dir`** (operação atômica de pasta inteira) — diferente das ações por arquivo do schema padrão. Stage 05 precisa reconhecer.
4. **5 ilhas com `action=leave`** — listadas no CSV mas stage 05 não toca. Apenas registradas para garantir que o roteador do novo L0 as conhece.
5. **`mapa_completo_clifnotes.md` está parcialmente obsoleto** (caminhos antigos do Classrooms). Não corrigir neste retrofit — escopo é estrutura, não conteúdo. Registrar como follow-up.
