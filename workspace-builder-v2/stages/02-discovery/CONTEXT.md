# Stage 02: Descoberta

Entender o workflow de desenvolvimento através de conversa com o usuário, enriquecida pelos insights da análise de codebase.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Stage anterior | `../01-analysis/output/analysis-report.md` | Arquivo completo | Insights do codebase para guiar a conversa |
| Usuário | (conversa) | Descrição completa do workflow | O domínio para construir o workspace |
| Referência | `../../references/conventions-reference.md` | Arquivo completo | Conhecer os patterns MWP para descobrir |
| Referência | `../../references/examples/fullstack-api-summary.md` | Arquivo completo | Exemplo de workspace dev completo |
| Referência | `../../references/examples/python-pipeline-summary.md` | Arquivo completo | Exemplo de pipeline de dados |

## Processo

1. Ler o relatório de análise do stage anterior. Resumir os achados para o usuário.
2. Pedir ao usuário para descrever o workflow de desenvolvimento ponta a ponta:
   - O que inicia um ciclo de trabalho? (ticket, PR, bug report, feature request)
   - Quais são as fases de desenvolvimento? (planning, implementation, review, deploy)
   - O que produz cada fase?
3. Identificar os stages distintos. Onde termina uma tarefa e começa outra? Procurar por pontos naturais de handoff onde um humano revisaria antes de continuar.
4. Para cada stage, perguntar:
   - O que entra? (arquivos, input do usuário, output do stage anterior)
   - O que sai? (o artefato que esse stage produz)
   - O que o agente precisa saber? (material de referência, regras, constraints)
   - Que verificações automáticas podem validar o output? (testes, lint, type-check)
5. Identificar contexto compartilhado:
   - Tipos/schemas usados entre múltiplos stages
   - Configurações globais (env vars, constants, configs)
   - Documentação de arquitetura que informa todas as fases
6. Identificar detalhes específicos do usuário:
   - Tech stack (linguagens, frameworks, versões)
   - Convenções de código (naming, formatting, patterns)
   - Ferramentas de CI/CD preferidas
   - Estratégia de branching (trunk-based, gitflow, feature branch)
7. Identificar stages opcionais:
   - Stages que alguns projetos podem não precisar (ex: deploy se é lib)
   - Stages condicionais (ex: migration só quando modelos mudam)
8. Identificar pré-requisitos de ferramentas:
   - Para cada stage, que ferramentas externas são necessárias?
   - Qual é obrigatória vs. opcional?
   - Skills existentes que cobrem essas ferramentas
9. Identificar o nível de automação desejado por stage:
   - **Totalmente automático** — agente executa sem pausa
   - **Review gate** — agente apresenta output, humano aprova antes de prosseguir
   - **Colaborativo** — humano e agente trabalham juntos
10. Identificar necessidades de rastreamento de estado:
    - O projeto precisa de PROGRESS.md entre sessões?
    - Precisa de DECISIONS.md para registrar decisões arquiteturais?
    - Precisa de changelog automático?
11. **[Checkpoint]** — Apresentar o mapa de workflow ao usuário. Perguntar: Todos os stages foram capturados? Algum input ou output faltando? Algum stage para combinar ou separar?
12. Executar as verificações de auditoria abaixo. Se alguma falhar, revisar antes de salvar.
13. Escrever o mapa de workflow.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 10 | Mapa de workflow: todos os stages com inputs/outputs, contexto compartilhado, variáveis, ferramentas, níveis de automação | Se a divisão de stages, pontos de handoff e contexto compartilhado estão corretos |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Clareza de stages | Todo stage tem responsabilidade única e artefato de output nomeado |
| Cadeia input/output | Todo input de stage é fornecido pelo usuário ou produzido por stage anterior |
| Contexto compartilhado | Recursos cross-stage (types, configs, docs) listados separados dos específicos |
| Cobertura de variáveis | Todo detalhe específico do usuário é capturado como variável nomeada |
| Níveis de automação | Todo stage tem nível de automação definido (automático, review gate, colaborativo) |
| Insights da análise | Achados do stage 01 refletidos no mapa de workflow |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Mapa de workflow | `output/workflow-map.md` | Doc estruturado: stages com inputs/outputs, contexto compartilhado, variáveis, ferramentas, níveis de automação, stages opcionais |
