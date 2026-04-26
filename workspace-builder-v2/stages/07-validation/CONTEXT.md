# Stage 07: Validação

Verificar o workspace gerado contra convenções ICM/MWP, integridade de contratos, infraestrutura de testes, e corrigir problemas antes do workspace ser entregue.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Output do scaffolding | `../04-scaffolding/output/` | Workspace gerado completo | O workspace a validar |
| Output do testing | `../05-testing/output/` | Infraestrutura de testes | Testes e CI a validar |
| Output do questionário | `../06-questionnaire/output/questionnaire.md` | Arquivo completo | Verificar cobertura de placeholders |
| Referência | `../../references/conventions-reference.md` | Arquivo completo | Regras para validar contra |

## Processo

Executar cada verificação abaixo. Registrar pass/fail e problemas encontrados.

### Verificações ICM Padrão

1. **Integridade de referências cruzadas.** Todo caminho de arquivo mencionado em qualquer tabela de Inputs de CONTEXT.md deve apontar para um arquivo real no workspace gerado. Listar referências quebradas.

2. **Sem dependências circulares.** Traçar o grafo de referências. Se Stage A referencia Stage B, Stage B não pode referenciar Stage A (diretamente ou através de outros stages). Desenhar grafo e confirmar que é um DAG.

3. **Cobertura de placeholders.** Escanear todos os arquivos markdown buscando patterns `{{PLACEHOLDER}}`. Todo placeholder deve ter uma pergunta correspondente no questionário. Toda pergunta deve mapear para pelo menos um arquivo. Listar orphans.

4. **Validade de seções condicionais.** Todo bloco `{{?SECTION}}...{{/SECTION}}` deve envolver uma seção completa (heading + todo conteúdo abaixo). Nenhum wrapping inline. Flagear violações.

5. **Cadeia de handoff de stages.** Verificar que a cadeia é contínua: o local de output do Stage N deve corresponder ao que a tabela de Inputs do Stage N+1 referencia. Listar a cadeia e flagear gaps.

6. **Pureza de CONTEXT.md.** Verificar que nenhum CONTEXT.md contém conteúdo de referência (definições, regras extensas, exemplos, guidelines). Devem conter apenas: título, descrição, tabela de Inputs, passos de Processo, tabela de Checkpoints (opcional), tabela de Verify (opcional), tabela de Audit (opcional), tabela de Outputs.

7. **Checkpoints em stages criativos.** Verificar que stages fazendo trabalho criativo ou de decisão têm pelo menos um checkpoint. Verificar que tabelas de checkpoint referenciam números de passos válidos.

8. **Audits em stages de build.** Verificar que stages de build têm seção Audit com condições específicas e não ambíguas.

### Verificações de Código (Novas)

9. **Integridade de seções Verify.** Todo stage com seção Verify tem um script correspondente em `_scripts/`. O script verifica os critérios definidos no contrato.

10. **Validade do CI pipeline.** O workflow de CI:
    - Parseia sem erros YAML
    - Referencia todos os verify scripts
    - Inclui steps de lint, test, e build (quando aplicável)
    - Define triggers corretos (push, PR)

11. **Presença de PROGRESS.md.** Workspaces code-producing têm PROGRESS.md no root com template completo (fase atual, última sessão, próximos passos, padrões de edição).

12. **Shared constants/types.** Workspaces code-producing definem tipos ou schemas compartilhados em `shared/`. Nenhum tipo é duplicado entre stages.

13. **Executabilidade de scripts.** Scripts em `_scripts/` são marcados como executáveis. Scripts shell têm shebang (`#!/bin/bash` ou `#!/usr/bin/env bash`).

14. **Setup de ambiente.** Se o projeto precisa de variáveis de ambiente:
    - `.env.example` existe com todas as variáveis documentadas
    - Nenhum secret real está no workspace (sem .env commitado)
    - `_config/environments.md` documenta diferenças entre ambientes

15. **Documentação.** Workspaces code-producing têm:
    - `docs/architecture.md` — estrutura e decisões
    - `docs/conventions.md` — regras de código
    - `docs/testing.md` — guia de testes

### Verificações de Qualidade

16. **Contagem de linhas.** Flagear CONTEXT.md acima de 80 linhas. Flagear referências acima de 200 linhas.

17. **Convenções de naming.** Todas as pastas e arquivos usam lowercase-com-hifens. Pastas de stage usam números com zero-padded (01-, 02-). Diretórios output/ vazios têm .gitkeep.

18. **Scan de qualidade.** Verificar: em dashes (substituir por --), jargão sem explicação, problemas de formatação markdown.

Se problemas forem encontrados, corrigir nos arquivos do workspace e re-executar as verificações que falharam para confirmar a correção.

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Relatório de validação | `output/validation-report.md` | Checklist com pass/fail por verificação, problemas encontrados, correções aplicadas |
