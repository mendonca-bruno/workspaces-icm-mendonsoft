# Stage 01: Análise

Analisar o codebase, configuração e ambiente de desenvolvimento existente antes de qualquer conversa com o usuário.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Usuário | (caminho do projeto) | Árvore completa de diretórios | O codebase a ser analisado |
| Referência | `../../references/code-patterns.md` | Arquivo completo | Patterns conhecidos para detectar |

## Processo

1. Escanear a árvore de diretórios do projeto. Registrar linguagens, frameworks e ferramentas detectadas.
2. Identificar arquivos de contexto existentes: CLAUDE.md, CONTEXT.md, .ai-context/, ou similares.
3. Detectar o tech stack:
   - Linguagens (por extensão e configs: package.json, pyproject.toml, go.mod, Cargo.toml)
   - Frameworks (FastAPI, Next.js, Django, Spring, etc.)
   - Package managers (npm, pip, cargo, go modules)
   - Build tools (webpack, vite, esbuild, make, etc.)
4. Analisar a estrutura de diretórios:
   - Monorepo ou single-repo?
   - Multi-package (workspaces npm, Python packages)?
   - Flat ou hierárquico?
   - Separação de concerns (src/, tests/, docs/, config/)?
5. Identificar infraestrutura de testes:
   - Test runners (pytest, jest, vitest, go test)
   - Diretórios de testes e padrão de nomenclatura
   - Configuração de cobertura
   - Mocks/fixtures existentes
6. Identificar CI/CD:
   - .github/workflows/, .gitlab-ci.yml, Jenkinsfile
   - Dockerfile, docker-compose.yml
   - Makefile, scripts de build
7. Detectar convenções de código:
   - Configs de style: ESLint, Prettier, Black, Ruff, clippy
   - Naming patterns (kebab-case, snake_case, camelCase)
   - Padrões de import/export
8. Identificar documentação:
   - README.md, docs/, API docs, changelogs
   - ADRs (Architecture Decision Records)
   - Guias de contribuição
9. Identificar gerenciamento de ambiente:
   - .env files e .env.example
   - Padrões de secrets (vaults, variáveis de ambiente)
   - Configs por ambiente (dev, staging, prod)
10. Identificar integrações e contratos:
    - APIs externas consumidas ou expostas
    - Schemas compartilhados entre serviços
    - Contratos de integração documentados
11. Classificar a maturidade do projeto:
    - **Greenfield** — Sem código ainda, construindo do zero
    - **Early** — Código existe mas sem estrutura ou docs
    - **Established** — Código + estrutura + alguma documentação
    - **Mature** — Código + estrutura + docs + testes + CI
12. **[Checkpoint]** — Apresentar o relatório de análise ao usuário. Perguntar: Isto está correto? Esqueci algo? O que você mudaria?
13. Executar as verificações de auditoria abaixo. Se alguma falhar, revisar antes de salvar.
14. Escrever o relatório de análise.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 11 | Relatório completo: stack, estrutura, maturidade, patterns detectados, gaps | Se a análise está correta e completa |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Detecção de stack | Pelo menos uma linguagem/framework identificada (ou confirmado greenfield) |
| Análise estrutural | Hierarquia de diretórios documentada com profundidade >= 2 |
| Classificação de maturidade | Uma das quatro categorias atribuída com justificativa |
| Contexto existente | Quaisquer CLAUDE.md/CONTEXT.md encontrados são catalogados |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Relatório de análise | `output/analysis-report.md` | Doc estruturado: tech stack, mapa de estrutura, nível de maturidade, patterns existentes, gaps identificados, integrações |
