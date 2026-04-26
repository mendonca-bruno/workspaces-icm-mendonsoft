# Proposed Architecture — ClifNotes (raiz)

## Visão geral

ClifNotes é um **meta-vault**: 9 sub-workspaces ICM aninhados (ilhas auto-contidas) + arquivos órfãos na raiz. O retrofit dá uma camada L0/L1 de roteamento meta + 2 silos novos (`reference/`, `templates/`) para os órfãos + `_archive/` para o overlay antigo.

**Princípio guia:** ilhas não são desmontadas. O novo L0 é um **roteador entre worlds**, não um substituto.

```
ClifNotes/                                                    (target após apply)
├── CLAUDE.md                                                 ← L0 NOVO (roteador meta)
├── CONTEXT.md                                                ← L1 NOVO
├── CLAUDE.md.original                                        ← L0 antigo preservado
├── reference/                                                ← silo NOVO
│   ├── CONTEXT.md
│   ├── community-digest.md
│   ├── mapa-completo.md
│   ├── resource-index.md
│   └── skills-field-manual.md
├── templates/                                                ← silo NOVO
│   ├── CONTEXT.md
│   ├── animation-spec-template.md
│   ├── claude-md-examples.md
│   ├── folder-organization-guide.md
│   ├── starter-client-management.md
│   ├── starter-code-project.md
│   └── starter-content-pipeline.md
├── _archive/                                                 ← silo NOVO
│   └── 2026-04-25-classrooms-overlay/                        ← Classrooms-icm-overlay/
├── Classrooms/                                               ← ILHA (leave)
├── Interpreted-Context-Methdology-main/                      ← ILHA (leave)
├── workspace-blueprint/                                      ← ILHA (leave)
├── files/                                                    ← ILHA (leave)
└── _used/                                                    ← ILHA (leave)
```

## Silos propostos

### 1. `reference/` — referência / lookup pontual

**Propósito:** acesso aos 4 documentos panorâmicos do ClifNotes (mapa cruzado, community digest, resource index 40+ links, skills field manual).

**Contrato L2 resumido:**
```
Inputs: ./<nome>.md (carregar APENAS a seção relevante via grep + Read offset/limit)
Process: identificar pergunta → grep → carregar seção → responder/copiar
Outputs: n/a (silo de consulta)
Layer Distribution: 60% infra (grep+offset), 10% AI (síntese pontual)
```

### 2. `templates/` — biblioteca de templates copy-paste

**Propósito:** boilerplates práticos (5 CLAUDE.md examples, animation spec, 3 workflow starters por perfil).

**Contrato L2 resumido:**
```
Inputs: ./<nome>.md (carregar 1 template por vez)
Process: identificar caso de uso → escolher template → copiar → personalizar variáveis
Outputs: novo workspace fora do ClifNotes (não in-place)
Layer Distribution: 60% infra (copy-paste), 10% AI (substituição de [PLACEHOLDER])
```

### 3. `_archive/` — meta-artefatos de retrofits anteriores

**Propósito:** preservar overlays e manifests de rollback de operações passadas. Acesso esporádico, função puramente de seguro.

**Sem CONTEXT.md** — silo passivo. Convenção de naming: `YYYY-MM-DD-<descritor>/`.

## Roteamento meta (no novo CLAUDE.md raiz)

Não é silo, é tabela no L0. O novo `CLAUDE.md` lista os 9 sub-workspaces ICM como destinos navegáveis, com uma frase descrevendo o que cada um oferece e quando entrar.

## Trade-offs

| Decisão | Alternativa | Por que escolhi essa |
|---------|------------|----------------------|
| 2 silos novos (não 1 nem 3) | (a) 1 silo `library/` único; (b) 3 silos separando até "examples" de "starters" | 2 silos refletem 2 modos de trabalho distintos (lookup vs copy-paste). 1 silo confundiria o padrão. 3 silos seria over-engineering para 11 arquivos. |
| Renomear arquivos strippando prefixos | Manter `clief_notes_resource_index_v1.md` etc. | Prefixos ficam redundantes uma vez dentro de `reference/` (`reference/clief-notes-...` vira pleonasmo). Sufixo `_v1` perde valor sem `_v2` para contrastar. |
| `_archive/` como silo passivo (sem CONTEXT.md) | Silo com L2 contrato | Meta-artefatos não têm "processo" — apenas existem. CONTEXT.md vazio seria ruído. |
| Mover overlay como diretório atômico | Mover arquivo por arquivo | Manifest de rollback do Classrooms referencia paths internos do overlay; quebrar a estrutura quebra a opção de rollback. `move-as-dir` preserva integridade. |
| `CLAUDE.md.original` na raiz (não em `_archive/`) | Mover para `_archive/2026-04-26-original-claude-md.md` | Semântica: o `.original` é uma referência de "versão anterior do L0 atual" — útil ter perto do `CLAUDE.md` novo para diff visual. Não é meta-artefato de processo. |
| Não corrigir `mapa_completo_clifnotes.md` (referencia paths obsoletos do Classrooms) | Atualizar inline durante o move | Escopo deste retrofit é **estrutura**, não conteúdo. Corrigir vira tarefa de stage separado. Registro como follow-up no apply-log. |

## Conflitos com ICM pré-existente

| Conflito | Resolução |
|----------|-----------|
| `CLAUDE.md` raiz pré-existe (4.5 KB, escrito intencionalmente) | **Decisão usuário (opção C):** rename para `CLAUDE.md.original` antes de gerar novo. Linha no CSV: `source_path=CLAUDE.md, destination_path=CLAUDE.md.original, action=move`. |
| 9 sub-workspaces ICM aninhados | **Decisão usuário:** `action=leave` para todos. Listados no CSV apenas para registro/awareness. |

## Anti-patterns checklist

- [x] Novo `CLAUDE.md` ≤1.000 tokens (planejado: ~600 palavras = ~780 tok)
- [x] `CONTEXT.md` raiz ≤500 tokens (planejado: ~280 palavras)
- [x] `CONTEXT.md` de silos ≤500 tokens (planejado: ~220 cada)
- [x] Nenhum silo vazio (`reference/` 4 arquivos, `templates/` 6 arquivos, `_archive/` 1 dir)
- [x] Sem cross-silo coupling implícito
- [x] Sem skills inventadas
- [x] ICM pré-existente preservado via rename para `.original`
- [x] Stage 05 só roda com `APPROVED.md`
- [x] `rollback-manifest.json` será escrito antes dos moves
