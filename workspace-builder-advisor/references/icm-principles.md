# Princípios ICM — Interpreted Context Methodology

Resumo dos cinco princípios fundamentais do paper *Interpretable Context Methodology* (arXiv:2603.16021v2). Use como rubrica ao aconselhar qualquer decisão do builder.

Fonte: `Interpreted-Context-Methdology-main/arXiv-2603.16021v2/ICM__1_.tex`, seção 3.1.

---

## 1. One stage, one job

> *"Each stage in a workspace handles a single step of the workflow and writes its output to its own folder. This follows McIlroy's Unix principle and Parnas's information-hiding criterion."*

**Implicação para o builder:**
- Se o `Process` de um stage tem 3+ ações independentes, é sinal de decomposição insuficiente. Divida.
- Stage 02 (discovery) frequentemente tenta absorver decisões que pertencem a Stage 03 (mapping). Resista.

---

## 2. Plain text as the interface

> *"Stages communicate through markdown and JSON files. No binary formats, no database connections, no proprietary serialization. This follows Kernighan and Pike's argument that text is the universal interface."*

**Implicação para o builder:**
- Todo handoff é arquivo `.md` ou `.json`. Se a tentação for "guardar no Postgres", escreva num arquivo.
- O próprio advisor segue isso: produz `advice-*.md`, sem hooks.

---

## 3. Layered context loading

> *"Agents load only the context they need for the current stage, following the principle that less irrelevant context means better model performance."*

**As 5 camadas:**
- **L0** (~800 tok) `CLAUDE.md` — identidade global ("Onde estou?")
- **L1** (~300 tok) `CONTEXT.md` raiz — roteador ("Para onde vou?")
- **L2** (200-500 tok) `CONTEXT.md` de stage — contrato ("O que faço?")
- **L3** (variável) referências estáveis — voice, conventions, decision-points ("Que regras se aplicam?")
- **L4** (variável) artefatos per-run — output do stage anterior ("Com que estou trabalhando?")

**Implicação para o builder:**
- A tabela `Inputs` em cada `CONTEXT.md` de stage é **a válvula de controle**. Especifique exatamente quais arquivos de L3 e L4 carregar.
- Nunca diga "carregue tudo".

**Implicação para o advisor:**
- `references/` é L3. `phases/*/output/` é L4. Não misture.

---

## 4. Every output is an edit surface

> *"The intermediate output of each stage is a file a human can open, read, edit, and save before the next stage runs. This implements Horvitz's mixed-initiative principles and Shneiderman's direct manipulation paradigm."*

**Implicação para o builder:**
- Outputs devem ser legíveis em editor de texto, não binários ou estruturas que requerem parsing.
- Existe **review gate** humano entre cada stage. Stages criativos devem ter checkpoint explícito.

**Implicação para o advisor:**
- `advice-*.md` é editável pelo usuário antes de ser usado no builder.

---

## 5. Configure the factory, not the product

> *"A workspace is set up once with the user's preferences, brand, style, and structural decisions. After that, each run of the pipeline produces a new deliverable using the same configuration. This follows the continuous delivery principle that production pipelines should be repeatable."*

**Distinção crítica:**
- **Factory** = L3 (estável entre runs): conventions, voice, schemas, principles
- **Product** = L4 (muda a cada run): output de stage, brief atual, código gerado

**Implicação para o builder:**
- Stage 06 (questionnaire) faz exatamente essa separação: placeholders system-level (factory) viram perguntas; per-execution (product) ficam no CONTEXT.md do stage de entrada.
- Confundir os dois é o erro mais comum em Stage 06.

**Implicação para o advisor:**
- `references/` (factory) nunca depende de `phases/*/output/` (product).

---

## Critérios de qualidade derivados

Um workspace está bem estruturado se:

1. **Decomposição**: cada stage faz uma coisa.
2. **Transparência**: qualquer arquivo pode ser aberto e entendido em editor de texto.
3. **Contexto focado**: L3 e L4 separados; tabela `Inputs` explícita.
4. **Edit surfaces**: humano pode inspecionar/editar entre stages.
5. **Estabilidade**: factory (L3) separada de product (L4).
6. **Acessibilidade**: não-técnico consegue editar `.md` e mudar comportamento.

Use estes 6 critérios ao revisar qualquer decisão do builder.
