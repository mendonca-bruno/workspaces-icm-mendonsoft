# Questionário de Onboarding: Workspace Builder v2

Leia este arquivo quando o usuário digitar "setup" ou "configurar". Faça TODAS as perguntas abaixo em uma única passada conversacional. O usuário deve poder responder tudo em uma mensagem. Estas respostas alimentam a Análise (Stage 01) e a Descoberta (Stage 02) — não são substituições de placeholder.

---

### Q1: Para qual projeto/domínio é este workspace?
- Exemplos: "API de delivery com otimização de rotas", "pipeline de dados de vendas", "SaaS de gerenciamento de tarefas"
- Tipo: texto livre
- Propósito: Nomear o workspace e definir o escopo

### Q2: Já existe código? Se sim, qual o caminho do projeto?
- Exemplos: "Sim, em D:\projetos\meu-app" ou "Não, é greenfield"
- Tipo: texto livre
- Propósito: Se há código, o Stage 01 (Análise) escaneia o codebase. Se não, pula direto para o Stage 02 (Descoberta).

### Q3: Descreva o workflow de desenvolvimento em uma frase.
- Exemplos: "Receber pedidos, geocodificar endereços, otimizar rotas e distribuir para motoboys via app."
- Tipo: texto livre
- Propósito: Dá ao Stage 02 um ponto de partida para aprofundar

### Q4: Qual o tech stack? (linguagens, frameworks, banco, ferramentas)
- Exemplos: "Python + FastAPI + PostgreSQL + Docker" ou "Node.js + Next.js + Prisma + Vercel"
- Tipo: texto livre
- Propósito: Se não há codebase para escanear, isso define o stack. Se há codebase, será confirmado pela análise.

### Q5: Quem vai usar este workspace e qual o nível com ferramentas AI?
- Tipo: texto livre
- Propósito: Calibra a complexidade dos contratos e a quantidade de checkpoints/review gates.

### Q6: Quantos stages aproximadamente, e algum é opcional?
- Tipo: texto livre (número ou lista breve + quais são opcionais)
- Propósito: Ajuda os stages 01 e 02 a fazer as perguntas certas
- Nota: Isto é uma estimativa. Os stages reais serão refinados durante a descoberta.

### Q7: Usa CI/CD? Se sim, qual plataforma?
- Exemplos: "GitHub Actions", "GitLab CI", "Só local por enquanto", "Não sei"
- Tipo: texto livre
- Propósito: Define se o Stage 05 (Testing) gera workflows de CI e para qual plataforma.

---

## Após o Onboarding

Se o usuário forneceu caminho de projeto existente (Q2):
> "Entendido. Vou começar analisando o codebase existente. Iniciando Stage 01 — Análise."
> → Ir para `stages/01-analysis/CONTEXT.md`

Se é projeto greenfield (Q2):
> "Entendido. Como não há código ainda, vamos direto para a descoberta do workflow. Iniciando Stage 02 — Descoberta."
> → Ir para `stages/02-discovery/CONTEXT.md`
