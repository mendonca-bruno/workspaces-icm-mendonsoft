# Pontos de Decisão Críticos do Builder

Os 10 momentos onde o usuário do `workspace-builder-v2` mais frequentemente toma decisões subótimas. O coaching do advisor (em `phases/01-coaching/playbook.md`) deve **sempre** endereçar os pontos relevantes.

---

## Mapa do Fluxo do Builder (onde o advisor entra)

```
USER digita "setup" no builder
    ↓
setup/questionnaire.md (Q1-Q7)            ← ADVISOR: advice-q1-q7.md
    ↓
[Q2 Sim] Stage 01 análise                 ← ADVISOR: advice-stage-01.md
[Q2 Não] (pula)
    ↓
Stage 02 discovery (conversa)             ← ADVISOR: advice-stage-02.md
    ↓ [checkpoint]
Stage 03 mapping (contratos+DAG)          ← ADVISOR: advice-stage-03.md
    ↓ [checkpoint]
Stage 04 scaffolding                      ← ADVISOR: advice-stage-04.md
    ↓
Stage 05 testing                          ← ADVISOR: advice-stage-05.md
    ↓
Stage 06 questionnaire                    ← ADVISOR: advice-stage-06.md
    ↓
Stage 07 validation                       ← ADVISOR: advice-stage-07.md
    ↓
Workspace pronto
```

---

## Os 10 Pontos Críticos

### DP-1. Q2 ambíguo: greenfield vs codebase legado a refatorar

**Sintoma:** "Tenho código legado mas vou reescrever tudo."

**Risco:** análise desnecessária OU perda de insights úteis.

**Heurística:** "Você quer reutilizar patterns/infra do legado?"
- Sim → analise. Stage 01 vai detectar conventions a manter.
- Não, vou jogar fora → greenfield. Pule Stage 01.

**Princípio:** *Configure the factory* — só reaproveite o que vai sobreviver.

---

### DP-2. Q6 subestimado: "3 stages" quando são 7

**Sintoma:** usuário responde "uns 3 stages" para um projeto que claramente tem 6-8.

**Risco:** Stage 02 acaba reabrindo discussão e gera retrabalho.

**Heurística:** mostre exemplos. "API fullstack típica = 5 stages (planning, contracts, implementation, testing, deployment). Pipeline de dados = 4 (ingestion, transformation, analysis, reporting). Onde seu projeto se encaixa?"

**Princípio:** *One stage, one job* — se cada stage faz uma coisa, geralmente são 5-7 para qualquer workflow real.

---

### DP-3. Stage 02 — confundir "fases" com "stages"

**Sintoma:** o usuário propõe "fase de desenvolvimento" como um único stage.

**Risco:** stages mal-decompostos no Stage 03.

**Heurística:** "Onde um humano pausaria para revisar antes de continuar?" — esses são os boundaries de stage. Fase = conceito de gestão; Stage = unidade técnica de transformação.

**Princípio:** *Every output is an edit surface* — boundaries de stage são os edit surfaces.

---

### DP-4. Stage 02 — automação sem checkpoints em decisões criativas

**Sintoma:** usuário define todo stage como "totalmente automático".

**Risco:** stages que envolvem decisão de design rodam sem revisão e produzem outputs inadequados que contaminam stages seguintes.

**Heurística:** para cada stage, perguntar: "há decisão de design aqui ou é mecânico?"
- Decisão → review gate
- Mecânico/determinístico → automático
- Combinação → colaborativo

**Princípio Clif:** stage-gate. **Princípio ICM:** *every output is an edit surface*.

---

### DP-5. Stage 02 — input/output bagunçado entre stages

**Sintoma:** stage propõe ler de 3+ stages anteriores não-contíguos.

**Risco:** dependências circulares escondidas; stages mal-separados.

**Heurística:** desenhe grafo de inputs/outputs antes de avançar para Stage 03. Se um stage tem múltiplas linhas vindo de stages distantes, refatore (provavelmente é um stage que deveria ser dois).

**Princípio:** *Plain text as interface* + DAG (sem ciclos).

---

### DP-6. Stage 03 — Verify recebendo critério subjetivo

**Sintoma:** "Stage X passa Verify se o output tem qualidade aceitável."

**Risco:** verify scripts impossíveis de implementar; testes fracos.

**Heurística:** para cada Verify, pergunte: "há comando shell que retorna pass/fail?" Se não, mova para `Audit` (revisão humana).

**Critério:**
- `Verify` = automatizável (lint, type-check, tests, schema validation)
- `Audit` = humano lê e aprova

**Princípio Clif:** 60/30/10 — qualidade subjetiva é IA/humano, não regra.

---

### DP-7. Stage 04 — nomenclatura inconsistente de pastas

**Sintoma:** mistura `stages/01-name/`, `phases/phase-1/`, `01_name/`.

**Risco:** scaffolding inconsistente; quebra de referências em handoffs.

**Heurística:** padrão do builder é `stages/NN-name/` (zero-padded, hífen, lowercase). Não invente.

**Princípio:** consistency > criatividade. (Convenção MWP — ver `{BUILDER_PATH}/references/conventions-reference.md`.)

---

### DP-8. Stage 04 — shared/ inconsistente

**Sintoma:** tipos compartilhados duplicados em múltiplos stages, ou em pastas com nomes diferentes (`types/`, `contracts/`, `shared/types/`).

**Risco:** "uma fonte canônica" violada — mudança em um lugar não propaga.

**Heurística:** padrão fixo:
- `shared/types/` para TypeScript
- `shared/schemas/` para Pydantic, JSON schemas, SQL
- `shared/constants/` para configs estáticas

Um tipo = uma casa.

**Princípio:** *Plain text + canonical sources* (Pattern 3 do MWP).

---

### DP-9. Stage 06 — placeholders system-level vs per-execution

**Sintoma:** questionário inclui pergunta como "qual o nome do ticket?" (per-execution).

**Risco:** questionário poluído com perguntas que mudam a cada run.

**Heurística:** regra binária:
- Constante entre runs (tech stack, conventions, brand) → pergunta no questionário (factory)
- Muda a cada run (ticket name, feature, branch) → CONTEXT.md do stage de entrada coleta conversacionalmente (product)

**Princípio ICM #5:** *Configure the factory, not the product*. Erro mais comum do Stage 06.

---

### DP-10. Stage 07 — corrigir no validation o que vem de mapping

**Sintoma:** muitos checks falhando no Stage 07 → tentação de patch direto nos arquivos do workspace gerado.

**Risco:** correção superficial; problema raiz (Stage 03 mapping incompleto) persiste.

**Heurística:** se 5+ checks falham, **volte ao Stage 03**. Se 1-2 falham, conserte localmente.

**Princípio:** root cause > sintoma. (Anti-padrão Clif: "automatizar juízo antes de entender o trabalho".)

---

## Como o playbook usa esta lista

Cada seção do `phases/01-coaching/playbook.md` referencia **pelo menos um DP** acima. O coaching de Q1-Q7 cobre DP-1 e DP-2. O coaching dos stages cobre DP-3 a DP-10. Sem cross-reference para DP, a recomendação fica genérica e perde força.
