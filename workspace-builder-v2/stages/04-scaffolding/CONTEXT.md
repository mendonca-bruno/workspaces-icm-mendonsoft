# Stage 04: Scaffolding

Gerar a estrutura completa de pastas do workspace, arquivos CONTEXT.md, referências, e artefatos específicos para desenvolvimento de código.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Stage anterior | `../03-mapping/output/stage-contracts.md` | Arquivo completo | Contratos a implementar como pastas e arquivos |
| Output da descoberta | `../02-discovery/output/workflow-map.md` | Seções "Pré-requisitos" e "Ferramentas" | Ferramentas que precisam de setup guides |
| Output da análise | `../01-analysis/output/analysis-report.md` | Seção "Tech Stack" | Stack detectado para gerar configs apropriados |
| Template | `../../references/templates/stage-context-template.md` | Arquivo completo | Template para CONTEXT.md de stage |
| Template | `../../references/templates/workspace-claude-template.md` | Arquivo completo | Template para o CLAUDE.md do workspace |
| Template | `../../references/templates/workspace-context-template.md` | Arquivo completo | Template para o CONTEXT.md do workspace |
| Template | `../../references/templates/progress-template.md` | Arquivo completo | Template para PROGRESS.md |
| Template | `../../references/templates/claude-md-dev.md` | Arquivo completo | Template CLAUDE.md otimizado para dev |

## Processo

1. Ler os contratos de stage do output do mapeamento.
2. Criar a estrutura de pastas do workspace:
   - Root: CLAUDE.md, CONTEXT.md, PROGRESS.md
   - `stages/` com uma subpasta numerada por stage, cada uma contendo CONTEXT.md, output/, e references/
   - `shared/` para recursos cross-stage (types, schemas, constants)
   - `_scripts/` para automação local (verificação, build, deploy)
   - `_config/` para configuração do workspace (tech-stack.md, environments.md)
   - `docs/` para documentação do workspace (architecture.md, conventions.md, testing.md)
   - `skills/` se alguma skill foi selecionada durante a descoberta
3. Poplar cada stage CONTEXT.md usando o template, preenchido com inputs, processo e outputs do contrato. Para cada stage, determinar:
   - **Checkpoints:** O stage se beneficia de revisão humana entre passos? Stages criativos ou de decisão devem ter checkpoint. Stages lineares podem pular.
   - **Verify:** O stage produz output que pode ser verificado automaticamente? Se sim, incluir comandos e critérios. Gerar script correspondente em `_scripts/`.
   - **Audit:** O stage precisa de verificações manuais de qualidade? Incluir tabela com condições específicas.
   - Deletar seções Checkpoints, Verify, ou Audit do template se o stage não precisar.
4. Criar o CLAUDE.md do workspace usando o template dev:
   - Mapa de pastas, triggers, tabela de roteamento
   - Seção "O Que Carregar" mapeando cada tarefa ao seu conjunto mínimo de arquivos
   - Matriz de roteamento por domínio (se aplicável)
   - Regras de economia de tokens
5. Criar o CONTEXT.md do workspace usando o template:
   - Tabela de roteamento de tarefas
   - Recursos compartilhados
6. Criar PROGRESS.md no root usando o template:
   - Fase atual, última sessão, perguntas abertas, próximos passos
   - Seção "Padrões de Edição Recorrentes" (para tracking edit-to-source)
7. Criar arquivos de referência placeholder para cada stage com variáveis `{{PLACEHOLDER}}`.
8. Para workspaces code-producing, criar:
   - `shared/types/` ou `shared/schemas/` com placeholder para tipos compartilhados
   - `shared/constants/` para configurações compartilhadas
   - `_config/tech-stack.md` com detalhes do stack detectado
   - `_config/environments.md` com configuração por ambiente
9. Para workspaces code-producing, criar `docs/`:
   - `docs/architecture.md` — template para documentação de arquitetura
   - `docs/conventions.md` — convenções de código do projeto
   - `docs/testing.md` — guia de testes (preenchido no stage 05)
10. Criar `_scripts/`:
    - `verify-[stage-name].sh` para cada stage com seção Verify
    - `run-all-checks.sh` que executa todas as verificações sequencialmente
    - Scripts de utilidade para tarefas mecânicas (file I/O, data movement)
11. Gerar `.gitignore` apropriado para o stack detectado.
12. Gerar `.env.example` se o projeto precisa de variáveis de ambiente.
13. Se skills foram selecionadas durante a descoberta, criar pasta `skills/`:
    - Copiar skills locais ou documentar comando de clone
    - Remover referências duplicadas
    - Atualizar tabelas de Inputs dos stage CONTEXT.md
14. Adicionar .gitkeep em todos os diretórios output/.
15. Executar as verificações de auditoria abaixo. Se alguma falhar, corrigir antes de salvar.
16. Escrever tudo em output/.

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Estrutura de pastas | Todo stage tem CONTEXT.md, output/, e references/ |
| Fidelidade contratual | Todo CONTEXT.md de stage corresponde aos contratos do Stage 03 |
| Sintaxe de placeholders | Todos os placeholders usam formato `{{SCREAMING_SNAKE_CASE}}` |
| Cobertura .gitkeep | Todo diretório output/ contém um .gitkeep |
| Tamanho CONTEXT.md | Nenhum CONTEXT.md excede 80 linhas |
| Convenções de naming | Todas as pastas e arquivos usam lowercase-com-hifens |
| PROGRESS.md | Root contém PROGRESS.md com template completo |
| Verify scripts | Todo stage com seção Verify tem script correspondente em `_scripts/` |
| Shared directory | Workspaces code-producing têm `shared/` com types ou schemas |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Workspace gerado | `output/` | Estrutura completa de pastas com todos os arquivos. Pronto para testing e questionnaire. |
