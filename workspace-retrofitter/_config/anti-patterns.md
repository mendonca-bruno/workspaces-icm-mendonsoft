# Anti-Patterns — Lista de Erros a Evitar no Workspace Gerado

Stage 04 (Generate) e stage 05 (Apply) devem rejeitar saídas que violem estes padrões. Stage 03 (Design) deve auditar o design proposto contra esta lista antes de seguir.

## 1. CLAUDE.md inflado

**Sintoma:** CLAUDE.md raiz com mais de 1.000 tokens (~750 palavras), virando documentação de projeto.

**Por quê é ruim:** L0 é carregado em **toda** sessão. Cada token aqui custa em **toda** interação futura.

**Correção:** mover detalhes para CONTEXT.md de silos. CLAUDE.md fica só com mapa, naming e roteamento.

## 2. Instruções ao Claude em vez de contexto do projeto

**Sintoma:** "Escreva profissionalmente", "Seja conciso", "Use linguagem clara".

**Por quê é ruim:** o modelo já sabe escrever. Essas instruções não mudam comportamento. Desperdiçam tokens.

**Correção:** descrever o **projeto**, não o **comportamento**. "Audiência: CFOs céticos de IA" muda output de verdade. Ratio: 80% contexto, 20% comportamento.

## 3. Silos vazios criados por imitação

**Sintoma:** workspace gerado tem `community/` mesmo que o alvo nunca tenha feito trabalho de distribuição.

**Por quê é ruim:** silo vazio é ruído. Roteamento manda agente para lugar sem contexto útil.

**Correção:** stage 03 só propõe silo se stage 02 detectou modo de trabalho correspondente. **Não copiar [workspace-blueprint](../../../../workspace-blueprint/) cegamente.**

## 4. Um único CONTEXT.md para tudo

**Sintoma:** CONTEXT.md de silo tenta cobrir 5 tipos de tarefa diferentes em 800 linhas.

**Por quê é ruim:** carrega contexto irrelevante para cada tarefa específica.

**Correção:** se um silo tem ≥3 tipos de trabalho distintos, ele virou pseudo-workspace. Quebrar em sub-silos com seus próprios CONTEXT.md.

## 5. Contexto que nunca atualiza

**Sintoma:** workspace gerado em um dia, ignorado depois. Documentação fica obsoleta.

**Por quê é ruim:** contexto stale é pior que nenhum contexto. Modelo segue regras desatualizadas.

**Correção:** cada CONTEXT.md de silo declara o que precisa atualizar entre execuções. CLAUDE.md raiz tem rodapé com data da última revisão.

## 6. Skills inventadas antecipadamente

**Sintoma:** workspace gerado lista 10 skills "que poderiam ser úteis".

**Por quê é ruim:** skill sem dor específica nunca é usada. Vira backlog mental.

**Correção:** retrofitter **sugere** skills só onde stage 02 detectou tarefa recorrente que falha com só contexto. Lista mínima > lista compreensiva.

## 7. Cross-silo coupling implícito

**Sintoma:** silo A precisa ler arquivos de silo B sem isso estar declarado no Inputs do CONTEXT.md.

**Por quê é ruim:** quebra o princípio de silo isolado. Agente em A não sabe que precisa de B.

**Correção:** se silo A consome output de silo B, declarar **explicitamente** na Inputs table do A. Fluxo entre silos sempre via arquivo, nunca via "lembre de carregar X também".

## 8. Mover arquivos sem rollback

**Sintoma:** stage 05 (Apply) move arquivos sem gravar manifesto.

**Por quê é ruim:** usuário não consegue reverter se mapping estiver errado.

**Correção:** stage 05 sempre escreve `rollback-manifest.json` **antes** de mover qualquer arquivo. Manifesto lista cada par `(origem, destino)` para reverter.

## 9. Sobrescrever arquivos ICM pré-existentes

**Sintoma:** o alvo já tem CLAUDE.md/CONTEXT.md (criados pelo usuário ou outro builder), e o retrofitter substitui sem checkpoint.

**Por quê é ruim:** apaga trabalho do usuário.

**Correção:** stage 02 detecta arquivos ICM existentes (ver [detection-heuristics.md](detection-heuristics.md#sinais-de-contexto-icm-já-existente)). Stage 04 gera versões em `icm-overlay/` com prefixo `proposed-`, e MIGRATION-PLAN.md flagga conflitos para revisão humana.

## 10. Inflar L1 com regras de comportamento

**Sintoma:** CONTEXT.md raiz vira lista de "do/don't" para o agente.

**Por quê é ruim:** L1 é roteador. Regras de comportamento pertencem a L3 (referências) e são carregadas apenas quando o silo precisa.

**Correção:** L1 só responde "minha tarefa é X → vou para Y, carregando Z". Sem prosa instrutiva.

---

## Audit checklist para stage 03 e 04

Antes de aprovar o design ou os arquivos gerados, verificar cada item:

- [ ] CLAUDE.md gerado tem ≤1.000 tokens
- [ ] CONTEXT.md raiz gerado tem ≤500 tokens
- [ ] Cada CONTEXT.md de silo gerado tem ≤500 tokens
- [ ] Nenhum silo gerado vazio (sem arquivos mapeados ou propostos)
- [ ] Toda dependência cross-silo aparece em Inputs table
- [ ] Skills sugeridas têm justificativa via tarefa detectada
- [ ] Arquivos ICM pré-existentes do alvo aparecem no MIGRATION-PLAN como conflito
- [ ] Stage 05 só roda se `APPROVED.md` existe no overlay
- [ ] `rollback-manifest.json` planejado antes de qualquer move
