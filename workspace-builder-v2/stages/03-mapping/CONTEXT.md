# Stage 03: Mapeamento

Transformar o mapa de workflow em contratos formais de stages com seções Verify e verificar o grafo de dependências.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Stage anterior | `../02-discovery/output/workflow-map.md` | Arquivo completo | O workflow a formalizar |
| Referência | `../../references/conventions-reference.md` | "Pattern 1: Contratos de Stage" e "Pattern 3: Referências Unidirecionais" | Regras para escrever contratos |

## Processo

1. Ler o mapa de workflow do output da descoberta.
2. Para cada stage, escrever o contrato formal seguindo o pattern de contrato de stage:
   - **Inputs** — Tabela com fonte, arquivo, escopo, razão
   - **Processo** — Passos numerados com ações específicas
   - **Checkpoints** — Pontos de pausa para revisão humana (para stages criativos/complexos)
   - **Verify** — Verificações automatizadas que validam o output
   - **Audit** — Verificações manuais de qualidade
   - **Outputs** — Tabela com artefato, localização, formato
3. Para cada stage, definir a seção **Verify**:
   - Quais outputs de stages anteriores verificar para consistência
   - Que comandos automatizados executar (testes, lint, type-check, build)
   - Critérios de pass/fail para cada verificação
   - Script correspondente em `_scripts/verify-[stage-name].sh`
4. Mapear referências cruzadas: quais stages leem de quais outros stages?
5. Identificar fontes canônicas: onde cada informação vive? (uma casa por fato)
6. Mapear tipos de dependência:
   - **File dependencies** — Stage N lê output do stage N-1
   - **Package dependencies** — Dependências de packages compartilhados
   - **Schema dependencies** — Types/schemas/contratos compartilhados entre stages
7. Verificar referências circulares: desenhar o grafo de dependência e confirmar fluxo unidirecional.
8. Verificar que todo output de stage é consumido por pelo menos um stage downstream (ou é o output final).
9. Identificar stages paralelizáveis: stages sem dependência entre si podem rodar em paralelo.
10. **[Checkpoint]** — Apresentar o diagrama de dependência e contratos ao usuário. Perguntar: O fluxo faz sentido? Alguma conexão faltando?
11. Executar as verificações de auditoria abaixo. Se alguma falhar, revisar antes de salvar.
12. Produzir o documento de contratos com diagrama de dependência.

## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| 9 | Diagrama de dependência e contratos draft para todos os stages | Se o fluxo de dependência e definições de contrato estão corretos |

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Sem referências circulares | Grafo de dependência flui em uma direção apenas |
| Consumo de output | Todo output de stage é lido por pelo menos um stage downstream ou é o entregável final |
| Completude de contrato | Todo stage tem Inputs, Processo, Outputs preenchidos |
| Fontes canônicas | Nenhuma informação é definida como autoritativa em mais de um stage |
| Seções Verify | Todo stage que produz código ou configuração tem uma seção Verify com verificações automatizáveis |
| Tipos de dependência | Dependências de file, package, e schema estão separadas e documentadas |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Contratos de stage | `output/stage-contracts.md` | Blocos formais Inputs/Processo/Verify/Audit/Outputs para todo stage, + diagrama de dependência ASCII, + mapa de dependências por tipo |
