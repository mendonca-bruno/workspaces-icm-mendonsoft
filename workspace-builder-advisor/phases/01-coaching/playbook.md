# Coaching Playbook — Setup (Q1-Q7) + Stages (01-07)

Conteúdo de coaching consolidado. Use junto com `references/decision-points.md` (DPs) e `references/voice/format-patterns.md` (estrutura de cada output).

Referências usadas:
- `../../references/icm-principles.md` (ICM #1-#5)
- `../../references/clif-rubric.md` (60/30/10, L0-L4, voice 3-arch, stage-gate, anti-patterns)
- `../../references/decision-points.md` (DP-1 a DP-10)

---

# Parte A — Setup do Builder (Q1-Q7)

## Q1 — Para qual projeto/domínio é este workspace?

**Claim:** Nome de domínio em 3-5 palavras, sem mencionar tecnologia.
**Reason:** O nome vira o L0 (CLAUDE.md) do workspace gerado — deve ser estável, não acoplado a stack que pode mudar (ICM #5, factory vs product).
**How to apply:** Use o "Problema" do `advice-refined-brief.md` reduzido a 3-5 palavras. Exemplos bons: "Wallet de motoboys", "Pipeline de ingestão de vendas". Ruins: "Sistema Python FastAPI" (acopla a stack).

**Alerta:** Se o usuário descreve projeto com adjetivos vagos ("plataforma incrível", "solução completa"), peça concretude antes de aceitar Q1.

---

## Q2 — Já existe código? Se sim, qual o caminho?

**Claim:** Resposta binária com path absoluto OU declaração explícita de greenfield.
**Reason:** Q2 é a **bifurcação crítica** do pipeline (Stage 01 é executado ou pulado). Ambiguidade aqui contamina todos os 7 stages (DP-1).
**How to apply:** Se há código legado mas o usuário vai jogar fora, responda "Não, é greenfield". Se vai reaproveitar patterns/infra, responda "Sim, em {{path}}". Não responda "tenho código mas vou refatorar" — escolha um lado.

**Alerta:** DP-1 — esta é a pergunta mais frequentemente respondida ambiguamente. Force decisão binária antes de seguir.

---

## Q3 — Descreva o workflow em uma frase.

**Claim:** Uma frase com verbos de ação no presente, ligando entrada → transformação → saída.
**Reason:** Stage 02 (discovery) usa Q3 como ponto de partida; frase mal-formulada gera discovery confusa (ICM #1, one stage one job).
**How to apply:** Estrutura: "{{Recebe X}} → {{transforma em Y}} → {{entrega Z}}". Exemplo: "Recebe pedidos, otimiza rotas, distribui para motoboys via app push." Evite frases com mais de uma conjunção "e".

**Alerta:** Se a frase precisa de 3+ "e", o workflow é grande demais para um workspace só (Clif: silos auto-contidos). Considere separar em dois workspaces.

---

## Q4 — Qual o tech stack?

**Claim:** Lista concreta de linguagens + frameworks + banco + CI. Sem hedging.
**Reason:** Stack define geração de testes (Stage 05) e CI config (Stage 07). Vagueza propaga (60/30/10 — não é decisão de IA, é de time).
**How to apply:** Se houver Stage 01 (Q2=Sim), o builder vai detectar e confirmar; aqui a resposta serve só para confirmação. Se greenfield, escreva: "{{linguagem}} {{versão}} + {{framework}} + {{banco}} + {{ferramenta auxiliar}}". Exemplo: "Python 3.12 + FastAPI + PostgreSQL + Alembic + ruff".

**Alerta:** Não inclua ferramentas que você não vai usar. Adicionar "Docker" sem ter certeza força Stage 05 a gerar configs que serão removidas.

---

## Q5 — Quem usa e qual nível com AI?

**Claim:** Perfil em 1 frase: papel + experiência com AI + preferência de automação.
**Reason:** Calibra quantidade de checkpoints (Stage 02 passo 9) e nível de detalhe em docs/ (Stage 04). Princípio ICM #4 — edit surfaces existem para humanos; quanto menos experiência, mais edit surfaces.
**How to apply:** Exemplo: "Devs backend, experientes com AI, preferem automação onde possível, review gates só em decisões de schema." Curto e direto.

**Alerta:** Resposta tipo "todos os devs da empresa" é ruim — perfil heterogêneo gera workspace com checkpoints inconsistentes. Escolha o perfil dominante.

---

## Q6 — Quantos stages, e algum opcional?

**Claim:** Estimativa entre 5 e 8 para qualquer projeto não-trivial. Liste opcionais explicitamente.
**Reason:** Usuários subestimam consistentemente. "One stage, one job" (ICM #1) força granularidade que parece exagerada à primeira vista (DP-2).
**How to apply:** Comparar com exemplos do builder:
- API fullstack: 5 (planning, contracts, implementation, testing, deployment)
- Pipeline de dados: 4 (ingestion, transformation, analysis, reporting)
- Content production: 5 (research, script, production, review, publish)

Se usuário responde "2-3 stages", desafie: "Há decisão de design intermediária? Há review gate? Esses são boundaries."

**Alerta:** DP-2. Subestimar Q6 gera retrabalho garantido em Stage 02. Errar para mais é barato; errar para menos é caro.

---

## Q7 — CI/CD?

**Claim:** Plataforma específica OU "só local por enquanto".
**Reason:** Stage 05 (testing) gera workflows específicos por plataforma. Resposta "talvez no futuro" deixa Stage 05 sem direção (60/30/10 — CI é determinístico, não IA).
**How to apply:** Se já existe CI no projeto (Stage 01 detectou), confirme. Se greenfield e o time tem GitHub: "GitHub Actions". Se não tem certeza ainda: "Só local por enquanto" — isso evita gerar configs que ninguém vai usar.

**Alerta:** "Não sei" é resposta ruim. Force escolha: GitHub Actions, GitLab CI, ou só local. Pode mudar depois; agora precisa de direção para Stage 05.

---

# Parte B — Stages do Builder (01-07)

## Stage 01 — Análise (codebase scan)

**O que esperar:** O builder escaneia o codebase em `{path Q2}`, detecta stack, estrutura, maturidade, conventions. Termina apresentando um relatório no Passo 12 (checkpoint).

**Pontos de decisão:**

### DP relevante: Validação do checkpoint Passo 12

**Claim:** Não aprove cego. Verifique se a maturidade classificada faz sentido.
**Reason:** Stage 02 usa o relatório como base — erro aqui vira mau discovery (ICM #1).
**How to apply:** No checkpoint, compare a classificação (Greenfield/Early/Established/Mature) com sua percepção. Se discordar, edite o `analysis-report.md` antes de prosseguir (ICM #4).

### DP-1: Codebase legado a descartar

**Claim:** Se o relatório identifica patterns que você vai descartar, marque-os explicitamente.
**Reason:** Stage 04 vai tentar manter conventions detectadas; o que você não quer manter precisa estar fora.
**How to apply:** Edite `analysis-report.md` riscando seções que não se aplicam.

### Anti-padrão a evitar

Aceitar o relatório porque "parece detalhado". Volume não é qualidade. Se o stack tem 12 ferramentas listadas mas só 4 são usadas de verdade, edite.

---

## Stage 02 — Discovery (conversa de workflow)

**O que esperar:** O builder faz 11 passos de conversa, identificando stages, inputs/outputs, contextos compartilhados, variáveis, níveis de automação. Termina em checkpoint apresentando `workflow-map.md`.

**Pontos de decisão:**

### DP-3: Fase ≠ Stage

**Claim:** Trate "fase" (gerencial) e "stage" (técnico) como conceitos diferentes.
**Reason:** Confundir gera stages mal-decompostos (ICM #1).
**How to apply:** No Passo 3 do builder, pergunte a si mesmo: "Onde um humano pausaria para revisar antes de continuar?" Esses são os boundaries de stage. "Desenvolvimento" não é stage; "implementação do módulo X" pode ser.

### DP-4: Nível de automação por stage

**Claim:** Para cada stage, classifique: automático / review gate / colaborativo. Não default para automático.
**Reason:** Stages criativos sem checkpoint geram outputs ruins que contaminam stages seguintes (ICM #4 + stage-gate Clif).
**How to apply:** No Passo 9 do builder, para cada stage, responda à pergunta: "há decisão de design ou é mecânico?" Decisão → review gate. Mecânico → automático. Combinação → colaborativo.

### DP-5: Inputs/outputs cruzados

**Claim:** Se um stage tem input vindo de stage não-contíguo, refatore.
**Reason:** Pode esconder dependência circular ou mal-decomposição (ICM #1, plain text + DAG).
**How to apply:** Após Passo 4 do builder, desenhe o grafo. Se Stage 5 lê output de Stage 1 e Stage 3 simultaneamente, considere se Stage 1 e 3 deveriam ser um stage só, OU se Stage 5 deveria ser dois.

### Anti-padrão a evitar

Definir "todos os stages são automáticos". Workspace sem review gate é workspace sem edit surface (viola ICM #4). Mínimo 1 review gate por workspace.

---

## Stage 03 — Mapping (contratos formais + DAG)

**O que esperar:** Builder transforma `workflow-map.md` em contratos formais com Inputs/Process/Outputs/Verify/Audit. Desenha DAG. Termina em checkpoint do diagrama.

**Pontos de decisão:**

### DP-6: Verify vs Audit

**Claim:** Coloque em `Verify` apenas critérios automatizáveis (comando shell pass/fail).
**Reason:** "Qualidade aceitável" não é verify — é audit (60/30/10 — qualidade subjetiva é humano, não regra).
**How to apply:** Para cada `Verify` proposto, pergunte: "qual o comando shell?" Se não consegue formular, mova para `Audit`.

### DP-5 (continuação): Validação do DAG

**Claim:** No checkpoint do Passo 9, exija o diagrama ASCII visualmente.
**Reason:** DAG é a estrutura central — bug aqui vira bug em todos os stages downstream (ICM #1).
**How to apply:** Se o diagrama tem seta voltando, é ciclo: refatore antes de aprovar. Se um output não é consumido por ninguém, ou o stage é desnecessário, ou falta um stage final.

### DP-8: Fontes canônicas

**Claim:** Cada tipo/schema/constante deve viver em uma única casa em `shared/`.
**Reason:** Pattern 3 do MWP — uma casa, múltiplas referências.
**How to apply:** Antes de aprovar contratos, pergunte: "este tipo aparece em mais de um stage como definição (não referência)?" Se sim, mova para `shared/types/` ou `shared/schemas/`.

### Anti-padrão a evitar

Tentar fazer Verify cobrir aspectos subjetivos com regex frouxo ("output deve conter mais de 100 palavras"). Verify é binário; quando não é, vire Audit.

---

## Stage 04 — Scaffolding (gerar estrutura)

**O que esperar:** Builder cria toda a árvore de pastas do workspace alvo, popula CONTEXT.md de stages com placeholders, cria `shared/`, `_scripts/`, `_config/`, `docs/`.

**Pontos de decisão:**

### DP-7: Naming conventions

**Claim:** Mantenha `stages/NN-name/` (zero-padded, hífen, lowercase). Sem variações.
**Reason:** Convenção MWP — consistência > criatividade.
**How to apply:** Se o builder gerou pasta com nome inconsistente, renomeie antes de prosseguir. Verifique também que cada `output/` tem `.gitkeep`.

### DP-8: Estrutura do shared/

**Claim:** Use `shared/types/`, `shared/schemas/`, `shared/constants/`. Não invente nomes.
**Reason:** Padrão estabelecido do builder; CI scripts esperam essas pastas (ICM #5 — factory padronizada).
**How to apply:** Após Stage 04, verifique a árvore. Se aparece `contracts/`, `interfaces/`, ou similar fora de `shared/`, consolide.

### Princípio ICM #3 — Tabela Inputs explícita

**Claim:** Cada `CONTEXT.md` de stage gerado deve ter tabela `Inputs` listando arquivos exatos.
**Reason:** Sem tabela explícita, agente carrega tudo (viola layered context loading).
**How to apply:** Abra cada CONTEXT.md gerado. Se tabela `Inputs` está com placeholders genéricos, preencha com paths reais antes do Stage 05.

### Anti-padrão a evitar

CONTEXT.md de stage com mais de 80 linhas. Se está longo, conteúdo de referência foi misturado com contrato — mova referência para `references/`.

---

## Stage 05 — Testing (verify scripts + CI)

**O que esperar:** Builder gera `_scripts/verify-*.sh`, `_scripts/run-all-checks.sh`, `.github/workflows/verify.yml`, `docs/testing.md`.

**Pontos de decisão:**

### DP-6 (continuação): Verify scripts inviáveis

**Claim:** Se um stage marcou Verify mas o script não tem como ser implementado, volte ao Stage 03.
**Reason:** Tratar Stage 05 como "vou bolar algum teste" gera testes fracos (60/30/10).
**How to apply:** Para cada `verify-NN.sh` gerado, leia o script. Se ele apenas faz `echo "ok"` ou tem grep frouxo, o Verify do Stage 03 estava errado.

### DP-4 (continuação): CI alinhado com automação

**Claim:** Stages marcados como "review gate" no Stage 02 não devem ser executados em CI.
**Reason:** CI roda automaticamente; review gate exige humano. Inconsistente.
**How to apply:** No `.github/workflows/verify.yml`, verifique que os steps cobrem apenas stages automáticos. Review gates ficam fora.

### Princípio Clif: stage-gate

**Claim:** `run-all-checks.sh` deve refletir a ordem dos stages.
**Reason:** Falha em stage anterior deve curto-circuitar verificação dos seguintes (drift cumulativo).
**How to apply:** Configure fail-fast no script. Modo "rodar tudo mesmo com falha" só serve para diagnóstico, não para validação.

### Anti-padrão a evitar

CI que "passa" porque não testa nada de verdade. `assertTrue(True)` em pytest, ou `exit 0` no verify script. Reveja cada teste e exija que ele falharia se o output estivesse errado.

---

## Stage 06 — Questionnaire (onboarding do workspace gerado)

**O que esperar:** Builder escaneia placeholders `{{PLACEHOLDER}}` no workspace gerado, separa em system-level vs per-execution, gera questionário flat para o end-user.

**Pontos de decisão:**

### DP-9: System-level vs per-execution (o ponto mais crítico)

**Claim:** Aplique a regra binária: "constante entre runs?" Se sim, vira pergunta. Se não, fica no CONTEXT.md do stage de entrada.
**Reason:** Princípio ICM #5 — configure the factory, not the product. Erro mais frequente do Stage 06.
**How to apply:** Para cada placeholder, pergunte: "se eu rodar a pipeline 10 vezes, isso muda?" Se sim → per-execution → não vai para questionário. Se não → system-level → vai.

Exemplos:
- `{{PROJECT_NAME}}` → system-level (não muda).
- `{{TECH_STACK}}` → system-level.
- `{{TICKET_ID}}` → per-execution (muda a cada run).
- `{{FEATURE_DESCRIPTION}}` → per-execution.

### Princípio ICM #4: Edit surface

**Claim:** Antes de finalizar o questionário, leia como se fosse o end-user. Toda pergunta clara? Sem jargão técnico não-explicado?
**Reason:** Questionário é edit surface para non-técnicos; não pode parecer schema YAML.
**How to apply:** Reescreva qualquer pergunta que use termo técnico sem definir. Ex: "Qual o ORM?" → "Qual ferramenta gerencia o acesso ao banco? (ex: SQLAlchemy, Prisma)".

### Anti-padrão a evitar

Questionário com 30+ perguntas. Se passou de 15, alguns placeholders são per-execution disfarçados, OU múltiplas perguntas estão capturando a mesma informação. Consolide.

---

## Stage 07 — Validation (18 checks)

**O que esperar:** Builder roda 18 verificações ICM/código (cross-references, DAG, placeholders, CONTEXT.md purity, naming, etc.). Reporta pass/fail. Corrige inline ou pede correção.

**Pontos de decisão:**

### DP-10: Falha em massa = problema upstream

**Claim:** Se 5+ checks falham, **volte ao Stage 03 (mapping)**, não conserte localmente.
**Reason:** Validation expõe problemas; corrigir só os sintomas mascara o root cause (anti-padrão Clif: "automatizar juízo antes de entender o trabalho").
**How to apply:** Se contagem de falhas > 5, abandone correções pontuais. Reabra `stage-contracts.md`, identifique o erro estrutural, e re-rode Stages 04-07.

### Princípio ICM #4: Edit antes de ship

**Claim:** Mesmo passando os 18 checks, leia o CLAUDE.md e o CONTEXT.md raiz do workspace gerado.
**Reason:** Checks pegam estrutura; não pegam clareza para humanos.
**How to apply:** Faça o teste do "novo dev": se um colega chegasse hoje, ele entenderia o workspace só lendo CLAUDE.md + CONTEXT.md? Se não, edite antes de declarar pronto.

### Princípio Clif: stage-gate final

**Claim:** Workspace só está pronto quando: 18 checks passam + CLAUDE.md e CONTEXT.md raiz lidos por humano + 1 stage rodado com placeholder real.
**Reason:** Validação automatizada cobre estrutura; teste end-to-end cobre semântica.
**How to apply:** Antes de finalizar, popule um placeholder, rode mentalmente um stage do workspace gerado, veja se chega a output. Se trava, ajuste.

### Anti-padrão a evitar

Tratar `validation-report.md` como checklist a marcar. Cada falha aponta para uma classe de problema; trate como sintoma, não como item.

---

# Cross-references rápidas

| Tema | Onde aparece |
|------|--------------|
| Bifurcação Q2 / codebase ambíguo | Q2, Stage 01 |
| Subestimação de stages | Q6, Stage 02 |
| Verify vs Audit | Stage 03, Stage 05 |
| System-level vs per-execution placeholders | Stage 04, Stage 06 |
| Falha em validation = problema upstream | Stage 03, Stage 07 |
| Naming conventions | Stage 04, Stage 07 |
| Edit surface humano | ICM #4 — em todos os checkpoints |
| 60/30/10 (triagem IA) | Q3, Q4, Stage 03, Stage 05 |
