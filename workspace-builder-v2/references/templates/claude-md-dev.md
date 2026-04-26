# {{NOME_PROJETO}}

{{DESCRICAO_CURTA}}

## Stack

- **Backend**: {{BACKEND_STACK}}
- **Banco**: {{DATABASE}}
- **Frontend**: {{FRONTEND_STACK}}
- **Deploy**: {{DEPLOY_TARGET}}

## Convenções de Código

### {{LINGUAGEM_PRINCIPAL}}

- Nomes de variáveis, funções, classes: {{NAMING_LANGUAGE}}
- Mensagens de erro, logs e respostas ao usuário: {{UI_LANGUAGE}}
- {{CONVENTION_1}}
- {{CONVENTION_2}}
- {{CONVENTION_3}}

### Formatação

- Linter/formatter: {{LINTER}}
- Line length: {{LINE_LENGTH}}
- Aspas: {{QUOTE_STYLE}}

## Comandos

```bash
{{CMD_DEV}}                    # Servidor de desenvolvimento
{{CMD_TEST}}                   # Todos os testes
{{CMD_LINT}}                   # Lint e format
{{CMD_BUILD}}                  # Build de produção
{{CMD_MIGRATION}}              # Migrações (se aplicável)
```

## Contexto por Domínio

### Hierarquia de carregamento

Sempre: CLAUDE.md → 1 contexto principal → playbook (se aplicável) → auxiliar (se necessário)

### Contextos Principais (carregar UM por tarefa)

| Contexto | Quando |
|----------|--------|
| `stages/[stage-name]/CONTEXT.md` | Trabalho dentro de um stage específico |
| `shared/` | Quando precisar de types/schemas compartilhados |
| `docs/architecture.md` | Para decisões de arquitetura |
| `docs/conventions.md` | Para verificar padrões de código |

### Roteamento

| Tarefa | Ir Para |
|--------|---------|
{{ROUTING_TABLE}}

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
{{LOADING_TABLE}}

## Regras de Economia de Tokens

- NÃO abrir todos os contextos — carregar apenas o que a matriz indica
- NÃO ler diretórios de stages não relacionados à tarefa atual
- NÃO ler `tests/`, `docs/` por padrão — só quando a tarefa exigir
- Preferir ler 1 arquivo específico a varrer diretórios inteiros

## Stage Handoffs

Cada stage escreve seu output na sua pasta `output/`. O próximo stage lê de lá. Se você editar um output entre stages, o próximo stage pega suas edições.

{{?INTEGRATION_CONTRACT}}
## Integration Contract

Contrato formal: `{{CONTRACT_PATH}}`

**Regra:** antes de implementar qualquer mudança, avaliar se afeta integração com {{INTEGRATION_TARGET}}.
{{/INTEGRATION_CONTRACT}}

{{?SKILLS}}
## Skills e Hooks

| Skill | Uso |
|-------|-----|
{{SKILLS_TABLE}}
{{/SKILLS}}
