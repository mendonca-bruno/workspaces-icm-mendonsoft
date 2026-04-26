# Workspace Builder v2 — Roteamento de Tarefas

## O Que Este Workspace Faz

Constrói novos workspaces ICM otimizados para desenvolvimento de código. Pipeline de 7 stages que vai da análise de codebase até a validação final.

---

## Para Onde Ir

| Você Quer... | Ir Para |
|--------------|---------|
| **Analisar** um codebase existente | `stages/01-analysis/CONTEXT.md` |
| **Descobrir** o workflow de desenvolvimento | `stages/02-discovery/CONTEXT.md` |
| **Mapear** contratos formais dos stages | `stages/03-mapping/CONTEXT.md` |
| **Gerar** a estrutura de pastas e arquivos | `stages/04-scaffolding/CONTEXT.md` |
| **Criar testes** e configuração de CI/CD | `stages/05-testing/CONTEXT.md` |
| **Construir** o questionário de onboarding | `stages/06-questionnaire/CONTEXT.md` |
| **Validar** tudo contra as convenções ICM | `stages/07-validation/CONTEXT.md` |

**Não leia tudo.** Identifique sua tarefa, carregue apenas o que precisa.

---

## Fluxo da Pipeline

```
01-analysis → 02-discovery → 03-mapping → 04-scaffolding → 05-testing → 06-questionnaire → 07-validation
  (scan)       (conversa)     (contratos)   (geração)       (qualidade)   (onboarding)      (verificação)
```

---

## Recursos Compartilhados

| Recurso | Localização | Quando Carregar |
|---------|-------------|-----------------|
| Convenções MWP | `references/conventions-reference.md` | Stages 02, 03, 04, 07 |
| Patterns de Código | `references/code-patterns.md` | Stages 01, 05 |
| Exemplos Completos | `references/examples/` | Stage 02 (para referência) |
| Templates | `references/templates/` | Stages 04, 06 |
