# Stage 05: Testing

Gerar a infraestrutura de testes, configuração de CI/CD e quality gates para o workspace gerado.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Output da análise | `../01-analysis/output/analysis-report.md` | Seção "Infraestrutura de Testes" | O que já existe no projeto |
| Output do scaffolding | `../04-scaffolding/output/` | Workspace gerado | O workspace para adicionar testes |
| Referência | `../../references/code-patterns.md` | Seção "Testing Patterns" | Patterns de teste conhecidos por stack |
| Referência | `../../references/templates/ci-workflow-template.md` | Arquivo completo | Template para CI |

## Processo

1. Ler a seção de testes do relatório de análise (stage 01).
2. Ler o workspace gerado do stage 04.
3. Determinar a estratégia de testes baseada no tipo de workspace:
   - **Workspace code-producing:** Testes unitários para scripts, testes de integração para pipelines
   - **Workspace content-producing com stages de código:** Verificação de build, testes de render
   - **Workspace híbrido:** Ambos
4. Para cada stage que tem seção Verify no seu contrato:
   a. Verificar que o script de verificação existe em `_scripts/`
   b. Se não existe, gerar o script
   c. Garantir que o script verifica os critérios definidos no contrato
   d. Adicionar tratamento de erros e output formatado (pass/fail)
5. Gerar scripts de smoke test para cada stage code-producing:
   - Verifica que o output existe no formato esperado
   - Verifica que dependências estão resolvidas
   - Verifica que linting passa
   - Verifica que type-checking passa (se aplicável)
6. Gerar configuração de CI:
   a. Detectar plataforma preferida (GitHub Actions, GitLab CI, ou local-only)
   b. Para GitHub Actions: `.github/workflows/verify.yml`
   c. Incluir: verificação de lint, type-check, execução de testes, verificação de build
   d. Configurar matrix strategy se múltiplas versões de runtime
7. Gerar configuração de pre-commit hooks se aplicável:
   a. `.pre-commit-config.yaml` ou equivalente
   b. Hooks de lint, format, e type-check
8. Atualizar `_scripts/run-all-checks.sh`:
   a. Executar todos os verify scripts em sequência
   b. Reportar resultado consolidado (pass/fail por stage)
   c. Falhar rápido ou continuar (configurável)
9. Criar `docs/testing.md` no workspace gerado:
   - Como rodar os testes localmente
   - Como os testes rodam no CI
   - Como adicionar novos testes
   - Padrões de nomenclatura para testes
   - Estrutura de fixtures/mocks
10. Criar template de teste para cada stage code-producing:
    - Template com imports, setup, teardown
    - Exemplo de teste para o tipo de output do stage
11. Se o workspace usa contratos/schemas compartilhados, gerar testes de contrato:
    - Verificar que schemas nos `shared/` são válidos
    - Verificar que stages que consomem schemas produzem output compatível
12. Executar as verificações de auditoria abaixo. Se alguma falhar, corrigir.
13. Escrever tudo em output/.

## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| Scripts existem | `ls _scripts/verify-*.sh` | Todo stage com Verify tem script correspondente |
| Scripts executam | `bash _scripts/run-all-checks.sh --dry-run` | Dry run completa sem erros de sintaxe |
| CI config válida | Validar YAML do workflow | YAML parseia sem erros |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Cobertura de verify | Todo stage com seção Verify tem script correspondente em `_scripts/` |
| Completude do CI | CI workflow referencia todos os verify scripts e reporta pass/fail |
| Permissões de script | Todos os shell scripts têm permissão de execução (chmod +x) |
| Docs de testing | `docs/testing.md` cobre: rodar local, rodar CI, adicionar testes |
| Templates de teste | Todo stage code-producing tem template de teste |
| Testes de contrato | Se existem schemas compartilhados, existem testes de contrato |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Infraestrutura de testes | `output/` | CI configs, verify scripts, templates de teste, guia de testes, testes de contrato |
