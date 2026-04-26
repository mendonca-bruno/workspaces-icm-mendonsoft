# ICM Rubric — Princípios Acionáveis

Extrato operacional do paper [ICM__1_.tex](../../../arXiv-2603.16021v2/ICM__1_.tex). Use como checklist ao classificar, projetar e gerar o workspace alvo.

## As 5 camadas

| Camada | Carga | Propósito | Token budget típico |
|---|---|---|---|
| **L0 — CLAUDE.md** | Sempre carregada | Identidade global, mapa, naming | ~800 tok |
| **L1 — CONTEXT.md raiz** | Início de toda sessão | Roteamento de tarefa → estágio | ~300 tok |
| **L2 — stages/<n>/CONTEXT.md** | Quando entra no estágio | Contrato Inputs/Process/Outputs | ~300-500 tok |
| **L3 — references / _config / docs** | Sob demanda do L2 | Regras persistentes, voice, convenções | ~500-2k tok cada |
| **L4 — output / artefatos** | Só estágio atual | Material da execução em curso | variável |

**Regra de seleção:** o L2 declara explicitamente a Inputs table — quais arquivos de L3 e L4 carregar e quais seções importam. Nunca carregar L3 inteiro "por garantia".

## Critérios para decidir camada de um arquivo no diretório alvo

| Pergunta | Se SIM | Se NÃO |
|---|---|---|
| Esta informação muda a cada execução? | L4 | L3 |
| O modelo precisa internalizar como regra? | L3 | L4 |
| Identidade do workspace, mapa global? | L0 | desce |
| Roteamento de tarefa? | L1 | desce |
| Contrato de uma fase específica? | L2 | desce |

## Princípios não-negociáveis

1. **Selectivity, not compression** — não comprimir contexto: selecionar arquiteturalmente o que entra. Modelos degradam com contexto irrelevante no meio (Liu et al., 2024).
2. **Plain text como interface universal** — markdown e JSON. Sem binários, sem databases proprietárias. Qualquer editor abre.
3. **Folder = pipe, file = stream** — herança Unix. Output de um estágio é input do próximo via arquivo.
4. **Human gate mandatório** — cada output é arquivo editável. Humano verifica antes do próximo estágio ler.
5. **Configure a fábrica, não o produto** — L3 é setup persistente; pipeline produz L4 novo a cada execução.
6. **Silo isolado** — cada silo funciona lendo só seu CONTEXT.md + L3 + L4. Sem dependências cruzadas implícitas.

## Anti-padrões a detectar e evitar no workspace gerado

Veja [anti-patterns.md](anti-patterns.md) para a lista completa. Os 3 mais comuns:

- **CLAUDE.md inflado** — vira documentação de projeto em vez de mapa. Limite: ≤1.000 tokens.
- **Instruções ao Claude em vez de contexto do projeto** — "escreva profissionalmente" é inútil; "audiência: CFOs céticos de IA" muda output. Ratio alvo: 80% contexto, 20% comportamento.
- **Silos vazios criados por imitação** — não copie writing-room/production/community se o alvo não faz esse trabalho.

## Ratio de conteúdo recomendado por arquivo gerado

| Arquivo | Work context | Behavioral rules | Routing/Map |
|---|---|---|---|
| CLAUDE.md raiz | 20% | 10% | 70% |
| CONTEXT.md raiz | 30% | 0% | 70% |
| stages/<n>/CONTEXT.md | 80% | 20% | 0% |
| docs/ (L3) | 100% | 0% | 0% |

## Como o retrofitter aplica ICM ao alvo

O processo de 5 estágios do retrofitter é, ele próprio, um pipeline ICM. O alvo do usuário será mapeado para uma estrutura ICM da seguinte forma:

```
target/
├── CLAUDE.md           ← L0 gerada por stage 04
├── CONTEXT.md          ← L1 gerada por stage 04
├── <silo-1>/
│   ├── CONTEXT.md      ← L2
│   ├── docs/           ← L3 (regras persistentes detectadas)
│   └── work/           ← L4 (artefatos existentes do alvo)
├── <silo-2>/
│   └── ...
└── _legacy/            ← arquivos sem destino claro (opcional)
```

A escolha de quantos silos, quais nomes, e quais arquivos vão para cada um é definida em stage 03 (Design), nunca por convenção rígida.
