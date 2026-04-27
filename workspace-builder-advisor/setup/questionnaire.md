# Questionário de Onboarding: Workspace Builder Advisor

Leia este arquivo quando o usuário digitar `setup` ou `configurar`. Faça as 3 perguntas em uma única passada conversacional. As respostas decidem o roteamento; **não substituem placeholders** em arquivos do advisor.

---

### A1: Descreva sua ideia em 1-3 frases.

- Aceite ideia bruta. Pode ser vaga ("um sistema de wallet", "uma API de notificações").
- Tipo: texto livre.
- Propósito: alimenta `phases/00-idea-refinement/` se o usuário escolher refinar antes de ir ao builder.

### A2: Onde você está em relação ao `workspace-builder-v2`?

- Opções:
  - "Não rodei nada ainda — quero refinar a ideia primeiro"
  - "Vou começar o setup agora (Q1-Q7)"
  - "Estou prestes a entrar no Stage NN" (NN = 01 a 07)
  - "Não sei — escaneie o builder pra mim" (dispara `diagnose`)
- Tipo: múltipla escolha + texto opcional.
- Propósito: decisão binária de roteamento (ver `CONTEXT.md`).

### A3: Qual o caminho absoluto do `workspace-builder-v2`?

- Default sugerido: `c:\Users\bruno\Downloads\ClifNotes\Interpreted-Context-Methdology-main\workspaces\workspace-builder-v2\`
- Tipo: caminho absoluto.
- Propósito: define `BUILDER_PATH` para o trigger `diagnose`. Se o usuário aceitar o default, prosseguir sem perguntar de novo.

---

## Após o Onboarding

Aplicar a tabela de decisão de `CONTEXT.md`:

- A2 = "refinar" → "Vamos refinar sua ideia. Carregando `phases/00-idea-refinement/CONTEXT.md`."
- A2 = "começar setup" → "Vamos preparar suas respostas Q1-Q7. Carregando `phases/01-coaching/CONTEXT.md` modo setup."
- A2 = "estou no Stage NN" → "Vamos preparar coaching para o Stage NN. Carregando `phases/01-coaching/CONTEXT.md` modo stage."
- A2 = "não sei" → executar trigger `diagnose` (escanear `{A3}/stages/*/output/`).

Salvar `BUILDER_PATH = {A3}` para uso em `diagnose` e referências futuras.
