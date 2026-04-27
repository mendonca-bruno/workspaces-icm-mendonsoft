# Voice — Format Patterns

Padrões estruturais por tipo de output do advisor.

---

## Padrão central: Claim → Reason → How-to-apply

Toda recomendação do advisor segue este shape:

```
**Claim:** [afirmação direta, 1 frase]
**Reason:** [princípio ICM ou Clif que sustenta, 1 frase, citar fonte]
**How to apply:** [ação concreta no builder, 1-2 frases]
```

Exemplo:

> **Claim:** Marque Stage 03 como review gate, não automático.
> **Reason:** Definição de schema é decisão de design (princípio ICM #4 — every output is an edit surface; DP-4).
> **How to apply:** No Stage 02 do builder, Passo 9, responder "review gate" para o nível de automação do Stage 03.

---

## Formato `advice-refined-brief.md`

Schema obrigatório:

```markdown
# Refined Brief — {{nome do projeto}}

## Problema (1 frase)
{{problema central que o sistema resolve}}

## Escopo
- In: {{lista do que está dentro}}
- Out: {{lista do que está fora}}

## Stakeholders
- {{quem usa, quem mantém, quem consome o output}}

## Triagem 60/30/10
- 60% Tradicional: {{o que é determinístico/lookup}}
- 30% Rule-based: {{o que é if/then/template}}
- 10% IA: {{o que genuinamente precisa de julgamento}}

## Predição Q1-Q7 (input para o builder)
- Q1 (domínio): {{resposta sugerida}}
- Q2 (codebase): {{resposta sugerida + justificativa de bifurcação}}
- Q3 (workflow em 1 frase): {{resposta sugerida}}
- Q4 (tech stack): {{resposta sugerida}}
- Q5 (perfil do usuário): {{resposta sugerida}}
- Q6 (# de stages): {{estimativa + justificativa}}
- Q7 (CI/CD): {{resposta sugerida}}

## Stage-gate de prontidão
- [ ] Problema em 1 frase
- [ ] Triagem 60/30/10 feita
- [ ] Predição Q1-Q7 escrita
- [ ] Cada predição tem justificativa em 1 frase
```

---

## Formato `advice-q1-q7.md`

Schema obrigatório:

```markdown
# Advice — Q1-Q7 do workspace-builder-v2

Para cada pergunta:

## Q{{N}}: {{título}}

**Resposta recomendada:** {{texto literal a colar no builder}}

**Justificativa (Claim → Reason → How):**
- Claim: {{...}}
- Reason: {{princípio ICM/Clif + DP relevante}}
- How: {{como evitar a armadilha desta pergunta}}

**Alerta (se aplicável):** {{anti-padrão a evitar}}

---

## Stage-gate de prontidão
- [ ] 7 respostas com justificativa
- [ ] Bifurcação Q2 explícita (greenfield vs existing)
- [ ] Alerta Q6 presente (não subestime stages)
- [ ] DP-1 e DP-2 endereçados
```

---

## Formato `advice-stage-NN.md`

Schema obrigatório:

```markdown
# Advice — Stage {{NN}} ({{nome}}) do workspace-builder-v2

## O que esperar neste stage
{{1-2 frases descrevendo o job do stage no builder}}

## Pontos de decisão críticos
Para cada DP relevante (mínimo 3):

### DP-{{X}}: {{título do ponto}}
- Claim: {{...}}
- Reason: {{princípio + fonte}}
- How to apply: {{ação no checkpoint do stage}}

## Anti-padrão a evitar
{{descrição em 1 frase + por que falha}}

## Stage-gate de prontidão
- [ ] 3+ pontos de decisão endereçados
- [ ] Princípio ICM/Clif citado em cada um
- [ ] Anti-padrão listado
- [ ] Ação concreta no checkpoint do stage definida
```

---

## Regras transversais de formatação

- Markdown padrão. Sem HTML.
- Bullets aninhados no máximo 2 níveis.
- Tabelas só quando comparação 2D real (linhas × colunas com semântica).
- Code fences só para paths, comandos, ou snippets de schema. Não para enfatizar texto.
- Negrito só para `Claim`, `Reason`, `How to apply`, `Alerta`. Não para frases inteiras.
- Cite fontes por path absoluto ou por ID de princípio (ex: "ICM #3", "DP-6").
