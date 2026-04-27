# Rubrica ClifNotes — Heurísticas de Triagem e Estrutura

Princípios complementares à ICM, vindos da metodologia ClifNotes (Jake Van Clief / Eduba). Use como segundo filtro depois de checar `icm-principles.md`.

Fontes:
- `Classrooms/the_foundation.md`
- `Classrooms/Building Your Stack.md`
- `files/vault-toolkit/constraints/06-layer-triage.md` (60/30/10)
- `files/vault-toolkit/constraints/05-voice-architecture.md` (voice 3-arch)
- `workspace-blueprint/CLAUDE.md` (silos auto-contidos)

---

## 1. Framework 60/30/10 (triagem do que precisa de IA)

Toda solução produtiva se distribui assim:

| % | Camada | O que faz | Ferramenta |
|---|--------|-----------|------------|
| ~60% | Tradicional | Determinístico, lookup, cálculo | Planilhas, bancos, código |
| ~30% | Rule-based | If/then, automação, roteamento | Zapier, scripts, templates |
| ~10% | IA genuína | Síntese, julgamento, texto não-estruturado | LLM |

**Triagem em 3 perguntas:**
1. O resultado é determinístico (uma resposta certa, calculável)? → Sim: spreadsheet/DB. PARE.
2. Pode ser expresso como if/then? → Sim: automação simples. PARE.
3. Requer julgamento em informação não-estruturada? → Sim: IA.

**Aplicação no builder:** ao aconselhar Q3 (workflow) e Q4 (stack), alerte se o usuário está empurrando para IA o que é determinístico (lookups, cálculos, validações de schema). Stages que são puramente determinísticos devem ter `Verify` automatizado, não checkpoint criativo.

**Anti-padrão:** "IA fazendo lookup de dados" / "IA calculando totais" → lento, caro, inconsistente.

---

## 2. Padrão L0-L4 (carregamento seletivo)

Mesmo modelo da ICM. Ver `icm-principles.md` seção 3 para definição. Aqui ficam as **heurísticas Clif para alocação de tokens**:

- Roteamento (L0+L1+L2): 10-15% da janela
- Referência (L3): 20-30%
- Material fonte (L4): 30-40%
- Espaço para resposta: 20-30%

**Teste da limpeza:** "% de contexto desperdiçado nesta conversa?" Se > 40%, está carregando demais.

---

## 3. Voice 3-arquivos (arquitetura de tom)

Nunca use um único arquivo de "voz". Use três (já implementado em `references/voice/` deste advisor):

| Arquivo | Função | Tamanho típico | Frequência de mudança |
|---------|--------|----------------|----------------------|
| `tone.md` | Direcional, descreve quando a voz emerge | 20-40 linhas | Raramente |
| `format-patterns.md` | Estrutural, padrões por formato | 1 parágrafo por formato | Cresce com novos formatos |
| `constraints.md` | Don't-list, padrões a evitar | 10-20 itens | Frequentemente |

**Por que três:** cada um tem job, frequência e padrão de carga distintos. Carregar só o necessário = contexto limpo.

---

## 4. Stage-gate (checkpoint humano entre etapas)

Pipeline de 4 estágios canônico:

```
01_briefs/    ← O QUÊ (entrada)
  ↓ [revisão humana]
02_specs/     ← COMO (plano)
  ↓ [revisão humana]
03_builds/    ← Trabalho real
  ↓ [revisão humana]
04_output/    ← Entregáveis
```

**Regra:** todo handoff é checkpoint. A saída do estágio anterior é entrada do próximo. Evita drift acumulado.

**Aplicação no builder:** Stage 02 (discovery) deve perguntar para cada stage proposto: "qual o nível de automação? auto / review gate / colaborativo?" Cada review gate é um stage-gate.

---

## 5. Silos auto-contidos

Workspace em silos: cada silo (workspace, stage, folder) é completamente isolado. Agente em um silo nunca precisa saber o que está em outro. Fluxo é unidirecional.

**Vantagem:** token economy. Agente em Stage 02 carrega só o que Stage 02 precisa.

**Implicação para o builder:** o `O Que Carregar` table do CLAUDE.md raiz é a manifestação concreta deste princípio. Verifique que **nada** é carregado fora da necessidade do stage atual.

---

## 6. Anti-padrões consolidados (don't-list)

❌ **Mega-arquivo de contexto** — tudo em um único `CONTEXT.md` → ruído, ineficiente.

❌ **CLAUDE.md como instruções** — "Você é um copywriter senior. Seja conciso..." → CLAUDE.md é mapa, não persona.

❌ **Skills antes de necessidade real** — construir skill-library teórica → desperdício. Construa em resposta a dor recorrente.

❌ **Contexto nunca atualizado entre sessões** — workspace trava em "week 1 state" → 2 minutos no fim de cada sessão atualizando estado.

❌ **Misturar referência com material fonte sem rótulos** → modelo trata style guide como conteúdo a transformar. Marque `REFERÊNCIA:` vs `FONTE:` vs `STAGE ANTERIOR:`.

❌ **Tentar construir determinístico com IA** → lento, caro, errado. Ver 60/30/10.

❌ **Automatizar juízo antes de entender o trabalho** → "technically correct but qualitatively weak". Faça manual primeiro, automate depois.

❌ **Um workspace para tudo** → contexto que tenta dez coisas, não faz nenhuma. Separe em workspaces quando muda modo mental, público, processo, ou docs de referência.

---

## Quick-reference: heurísticas de decisão

| Decisão | Regra Clif |
|---------|-----------|
| Manter junto ou separar workspaces? | Separe se muda modo/público/processo/docs |
| CONTEXT.md ou skill? | Skill só após 5+ invocações ou dor real |
| Determinístico, regra, ou IA? | Triagem 60/30/10 (3 perguntas) |
| Quanto contexto carregar? | < 40% desperdiçado, L3+L4 ≤ 70% da janela |
| Quando construir estrutura? | Não construa para problema que ainda não tem |
