# Workspace Retrofitter

Adapta um diretório existente para virar um workspace ICM otimizado para agentes de IA. Sem criar nada do zero — engenharia reversa + camadas L0-L4 + framework 60/30/10.

## O problema que resolve

Você já tem uma pasta. Pode ser um repo de código, uma coleção de drafts, sua pasta `Downloads/` cheia de coisas, ou um projeto antigo. Quando um agente de IA entra ali, ele tenta carregar tudo, queima 40-50k tokens em ruído, e perde o fio. O retrofitter transforma essa pasta em uma estrutura onde cada interação carrega apenas 2-8k tokens focados.

## Como funciona

5 stages em pipeline. Você confirma duas vezes antes de qualquer arquivo ser movido.

```
01-scan  →  02-classify  →  [confirmar]  →  03-design  →  04-generate  →  [APPROVED.md]  →  05-apply
```

| Stage | O que faz | Toca no alvo? |
|-------|-----------|---------------|
| 01-scan | Inventário read-only | Não |
| 02-classify | Diagnostica tipo, modos, aplica triage 60/30/10 | Não |
| 03-design | Propõe silos e mapeia cada arquivo para destino | Não |
| 04-generate | Renderiza CLAUDE.md, CONTEXT.md, silos em pasta paralela `<target>-icm-overlay/` | Não |
| 05-apply | Move arquivos in-place (com rollback) | **Sim**, após `APPROVED.md` |

## Como usar

1. Abra este workspace no Claude Code (entre na pasta `workspace-retrofitter/`).
2. Diga `setup` → responda o questionário ([setup/questionnaire.md](setup/questionnaire.md)).
3. Diga `scan --target=<caminho-absoluto-do-seu-diretorio>` → roda stage 01.
4. Continue stage por stage. Após stage 02, leia `triage-report.md` e edite se quiser.
5. Após stage 04, inspecione `<target>-icm-overlay/`. Edite `file-mapping.csv` se quiser ajustar.
6. Para autorizar a aplicação: crie `<target>-icm-overlay/APPROVED.md` (qualquer conteúdo).
7. Diga `apply` → stage 05 move os arquivos e escreve os arquivos ICM no alvo.

Para reverter: `rollback` (lê `<target>-icm-overlay/rollback-manifest.json`).

## Garantias

- Nada é apagado. Arquivos sem destino claro vão para `_legacy/`.
- `.git/`, `node_modules/`, `.venv/` e similares nunca são tocados.
- `rollback-manifest.json` é escrito **antes** do primeiro movimento.
- Stage 05 recusa rodar sem `APPROVED.md`.
- Arquivos ICM pré-existentes do alvo são detectados e flagueados como conflito (não sobrescritos sem aprovação).

## O que esperar do output

Depois do stage 05, seu diretório alvo terá:

```
<target>/
├── CLAUDE.md           ← mapa global, ≤1.000 tokens
├── CONTEXT.md          ← roteador de tarefas, ≤500 tokens
├── <silo-1>/
│   ├── CONTEXT.md      ← contrato L2 do silo, ≤500 tokens
│   ├── docs/           ← regras persistentes (L3)
│   └── work/           ← seus arquivos existentes, organizados (L4)
├── <silo-2>/...
└── _legacy/            ← arquivos sem destino claro
```

Quantos silos, quais nomes, e quais arquivos vão para cada um — tudo derivado do que **já existe** no seu diretório, não de um template fixo.

## Onde o retrofitter se encaixa entre os builders

| Builder | Quando usar |
|---------|-------------|
| [workspace-builder](../workspace-builder/) | Você quer criar um workspace genérico do zero. |
| [workspace-builder-v2](../workspace-builder-v2/) | Você quer criar um workspace para um **projeto novo de código**. |
| [workspace-builder-advisor](../workspace-builder-advisor/) | Você está usando o builder e quer coaching/segunda opinião. |
| **workspace-retrofitter** | Você **já tem** um diretório e quer adaptá-lo. |

## Referências

- [Interpreted Context Methodology — paper](../../arXiv-2603.16021v2/ICM__1_.tex)
- [Classrooms/the_foundation.md](../../../Classrooms/the_foundation.md)
- [files/vault-toolkit/constraints/06-layer-triage.md](../../../files/vault-toolkit/constraints/06-layer-triage.md)
- [workspace-blueprint](../../../workspace-blueprint/) — exemplo canônico de workspace ICM completo
