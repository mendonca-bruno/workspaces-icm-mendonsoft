# Referência de Convenções MWP — Atualizada para Código

Ponteiro para as convenções core do Interpreted Context Methodology (ICM), com extensões para workspaces de desenvolvimento de código.

---

## Convenções Core (do ICM Original)

### Pattern 1: Contratos de Stage
Cada stage tem um CONTEXT.md com seções formais: Inputs, Processo, Outputs. O contrato define O QUÊ e QUANDO, não COMO.

### Pattern 2: Carregamento Seletivo
Nunca carregue tudo. Para cada tarefa, o CLAUDE.md especifica exatamente quais arquivos carregar e quais pular. O agente carrega apenas o mínimo necessário.

### Pattern 3: Referências Unidirecionais
Stage A pode referenciar output do Stage B, mas nunca o contrário. O grafo de dependência é um DAG (Directed Acyclic Graph). Um fato tem uma casa canônica.

### Pattern 4: Skills Bundled
Skills são copiadas para dentro do workspace, não referenciadas externamente. O workspace é auto-contido.

### Pattern 5: Sintaxe de Placeholders
Variáveis de usuário usam `{{SCREAMING_SNAKE_CASE}}`. Seções condicionais usam `{{?SECTION}}...{{/SECTION}}`. Seções condicionais envolvem blocos completos (heading + conteúdo).

### Pattern 6: Checkpoints
Stages criativos ou de decisão pausam para revisão humana. Tabela com: após qual passo, o que o agente apresenta, o que o humano decide.

### Pattern 7: Pré-requisitos de Ferramentas
Cada ferramenta tem: qual stage precisa, se é obrigatória ou opcional, o que faz. Setup guides vivem em `references/` do stage relevante.

### Pattern 8: Audit
Verificações de qualidade antes do output ser escrito. Tabela com: check e condição de aprovação. Condições são específicas e não ambíguas.

---

## Extensões para Código (v2)

### Pattern 9: Seções Verify
Todo stage que produz output verificável automaticamente tem uma seção `## Verify` no contrato. Diferente do Audit (manual), o Verify define:
- Comandos automatizados a executar
- Critérios de pass/fail
- Script correspondente em `_scripts/`

### Pattern 10: State Tracking (PROGRESS.md)
Workspaces de código mantêm um `PROGRESS.md` no root que persiste estado entre sessões:
- Fase atual do pipeline
- Resumo da última sessão
- Decisões tomadas
- Próximos passos
- Padrões de edição recorrentes (para detectar melhorias source-level)

### Pattern 11: Automação Local (_scripts/)
Tarefas mecânicas (file I/O, verificações, data movement) são scripts no `_scripts/`:
- `verify-[stage-name].sh` — verificações por stage
- `run-all-checks.sh` — executa tudo
- Scripts de utilidade para tarefas repetitivas
O agente AI foca em raciocínio, não em file I/O.

### Pattern 12: Shared Types/Schemas
Workspaces de código definem tipos e schemas compartilhados em `shared/`:
- `shared/types/` — Interfaces, tipos, modelos
- `shared/schemas/` — Database schemas, API specs
- `shared/constants/` — Configurações compartilhadas
Um tipo tem uma casa canônica. Stages referenciam, não duplicam.

### Pattern 13: Dependency Graph Awareness
O mapeamento distingue três tipos de dependência:
- **File dependencies** — Stage N lê output do stage N-1
- **Package dependencies** — node_modules, venvs, packages compartilhados
- **Schema dependencies** — Types, APIs, database schemas compartilhados

### Pattern 14: Níveis de Automação por Stage
Cada stage define seu nível:
- **Totalmente automático** — agente executa sem pausa
- **Review gate** — agente apresenta, humano aprova
- **Colaborativo** — humano e agente trabalham juntos

### Pattern 15: Análise Pré-Discovery
Para workspaces de código, o primeiro stage analisa o codebase existente antes da conversa. Insights da análise alimentam a descoberta como defaults e contexto.
