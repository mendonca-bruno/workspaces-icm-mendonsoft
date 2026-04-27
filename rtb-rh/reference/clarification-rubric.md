# Rubrica de Clarificação — Stage 02

Define o **score de clarificação** que decide se o pipeline avança ou para para coletar mais informação. Score de 0 a 4. Threshold default: **3**.

## As 4 dimensões

| # | Dimensão | Pergunta de validação | Pontuação |
|---|----------|----------------------|-----------|
| 1 | Sistema identificado | Pelo menos 1 sistema do `kb/_taxonomies/area-mapping.md` foi nomeado ou implicado pelo solicitante? | 0 ou 1 |
| 2 | Sintoma normalizado | Pelo menos 1 sintoma da `kb/_taxonomies/symptoms.md` foi matched (não `symptom_unknown`)? | 0 ou 1 |
| 3 | Janela temporal | Há indicação de **quando** o problema começou ou em qual ciclo (ex.: "fechamento de março", "desde ontem", "no envio de hoje")? | 0 ou 1 |
| 4 | Identificador afetado | Há matrícula, CPF, ID de processo, nome de job, OU descrição clara da abrangência (ex.: "todos os funcionários da CCT X")? | 0 ou 1 |

**Score total = soma das 4.**

## Decisão

| Score | Ação |
|-------|------|
| 4 | Avança para Stage 03 |
| 3 | Avança para Stage 03 (default threshold) |
| 2 | **PARA**. Gerar perguntas para as 2 dimensões com 0. |
| 0 ou 1 | **PARA**. Gerar perguntas para todas as dimensões com 0; mencionar que o ticket está muito aberto. |

## Como gerar as perguntas

Para cada dimensão com 0, escolher pergunta padrão da tabela abaixo. Adaptar terminologia ao contexto se possível.

### Dimensão 1 — Sistema

> "Você sabe qual sistema está apresentando o problema? Por exemplo: portal de holerite, ponto eletrônico, portal de benefícios, workflow de admissão/demissão. Se não souber o nome do sistema, descreva onde você estava (URL, tela, ou nome do programa)."

### Dimensão 2 — Sintoma

> "Pode descrever exatamente o que está acontecendo? Por exemplo:
> - Apareceu uma mensagem de erro? (Cole a mensagem exata.)
> - O valor que apareceu é diferente do esperado? (Qual valor apareceu vs. qual era o esperado?)
> - Algo deveria ter acontecido e não aconteceu? (O quê?)"

### Dimensão 3 — Janela temporal

> "Quando o problema começou? Foi:
> - Após algum evento específico (fechamento da folha, mudança de mês, deploy)?
> - De repente, em um momento específico (ontem, hoje pela manhã)?
> - Recorrente, em momentos previsíveis (sempre no início do mês)?"

### Dimensão 4 — Identificador afetado

> "O problema afeta:
> - Um funcionário específico? (Pode informar a matrícula?)
> - Um grupo de funcionários? (Quais? Por exemplo: toda a CCT X, todos os admitidos em data Y, todos com benefício Z.)
> - Todos os funcionários do sistema?"

## Como ler a resposta do solicitante

1. Bruno cola a resposta na seção `## Resposta` do `02-clarification.md`.
2. Re-disparar `clarificar <ticket-id>`.
3. O Stage 02 reprocessa: `input.md` + `02-clarification.md` (incluindo Resposta) → recalcula score.
4. Loop até score ≥ threshold ou Bruno decide forçar avanço (raro — registra como `forced: true` no `02-clarified.md`).

## Anti-pattern: agente improvisar resposta para ticket vago

Se score < threshold, **NÃO PRODUZIR** `02-clarified.md` nem `03-search-result.md`. O pipeline literalmente não tem o que buscar. Improvisar = enganar.

## Configuração do threshold

Default = 3. Pode ser ajustado em `DECISIONS.md` (criar ADR específico). Sinais para ajustar:
- < 10% dos tickets disparam clarificação → threshold baixo demais (agente permissivo). Subir para 4.
- > 60% disparam → threshold alto (agente exige perfeição). Descer para 2 e melhorar taxonomias.
