# Template — Stage CONTEXT.md

Use este molde ao criar um stage novo. Cópia adaptada de `Interpreted-Context-Methdology-main/workspaces/workspace-builder-v2/references/templates/stage-context-template.md`.

```markdown
# Stage NN: <NomeDoStage>

<1-2 frases descrevendo o que este stage faz e onde se encaixa no pipeline.>

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| <stage anterior ou recurso> | `<path>` | <arquivo completo / seção X> | <razão para carregar> |

## Processo

1. <Passo 1 — ação concreta>
2. <Passo 2>
3. <Passo 3>
...
N. **[Checkpoint]** — Apresentar X ao Bruno; aguardar decisão Y.

## Checkpoints

(Omitir se stage é totalmente automatizado.)

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| <N> | <o que mostrar> | <que decisão tomar> |

## Verify

(Verificações automatizáveis. Cada uma com comando concreto.)

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| <nome> | `<comando shell ou inspeção>` | <critério de pass> |

## Audit

(Verificações de qualidade que requerem julgamento humano.)

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| <nome> | <critério qualitativo> |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| <nome> | `<path>` | <descrição do conteúdo/schema> |
```

## Princípios para escrever um stage novo

1. **Inputs explícitos**: nunca "ler tudo que precisar". Listar cada arquivo, com escopo (arquivo completo? seção? frontmatter apenas?). Isso é o paper ICM §recursive routing aplicado.
2. **Processo numerado**: cada passo é uma ação verificável. Se um passo faz mais de uma coisa, separar.
3. **Checkpoint só onde precisa**: stages mecânicos (regenerar índice, calcular hash) não precisam. Stages com julgamento (clarificar, promover) precisam.
4. **Verify automatizável > Audit qualitativo**: prefira algo que dê pra rodar e responder pass/fail. Audit é fallback.
5. **Outputs com path absoluto e schema claro**: o próximo stage precisa saber onde ler e o que esperar.
6. **Não duplique conteúdo de outro stage**: se vai precisar dos mesmos inputs, referencie pelo path; o agente carrega na hora.
