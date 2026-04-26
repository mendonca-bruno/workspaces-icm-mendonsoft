# Stage 06: Questionário

Construir o questionário de onboarding que hidrata as variáveis placeholder do workspace gerado.

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| Output da descoberta | `../02-discovery/output/workflow-map.md` | Seção "Variáveis Específicas" | Variáveis que precisam de perguntas |
| Output do scaffolding | `../04-scaffolding/output/` | Todos os arquivos com patterns `{{PLACEHOLDER}}` | Saber onde cada placeholder vive |
| Template | `../../references/templates/workspace-context-template.md` | Arquivo completo | Referência de formato |

## Processo

1. Ler a seção de variáveis específicas do mapa de workflow.
2. Escanear todos os arquivos markdown no workspace gerado buscando patterns `{{PLACEHOLDER}}`. Construir lista completa.
3. Dividir variáveis em dois grupos:
   - **Nível de sistema:** Coisas que permanecem iguais entre execuções (identidade do projeto, tech stack, convenções, ferramentas, preferências de workflow). Estas viram perguntas de setup.
   - **Por execução:** Coisas que mudam a cada vez que a pipeline roda (nome do ticket, feature, branch, escopo). Estas NÃO viram perguntas de setup. O CONTEXT.md do stage de entrada deve coletá-las conversacionalmente no início de cada execução.
4. Para cada placeholder de nível de sistema, escrever uma pergunta:
   - Texto da pergunta que uma pessoa não-técnica pode entender
   - O(s) placeholder(s) que popula
   - Os arquivos onde esses placeholders aparecem
   - Tipo de input (texto livre, múltipla escolha, sim/não)
   - Um default sensato ou exemplo para o usuário pular se não se importar
5. Para perguntas de tech stack, oferecer opções baseadas na análise do stage 01:
   - Se o stack já foi detectado, confirmar ao invés de perguntar
   - Se não foi detectado, oferecer opções comuns por categoria
6. Para perguntas de convenções de código, pedir exemplos concretos ao invés de descrições:
   - "Cole um trecho de código que siga suas convenções" extrai regras pattern-matchable
   - "Descreva suas convenções" extrai abstração que o agente precisa interpretar
7. Se um campo pode ser derivado de outra resposta, listar como campo derivado sob a pergunta fonte. O agente preenche campos derivados sem perguntar.
8. Escrever TODAS as perguntas como lista numerada flat — sem agrupamentos por categoria. O usuário deve poder responder tudo em uma única mensagem.
9. Adicionar lógica condicional para stages opcionais: perguntas sim/não que removem pastas ou blocos `{{?SECTION}}`.
10. Adicionar processo de duas passagens para convenções de código:
    - Primeira passagem: coletar respostas e popular regras
    - Segunda passagem: apresentar regras geradas ao usuário para revisão
11. Verificar que todo placeholder de nível de sistema tem pergunta correspondente.
12. Verificar que placeholders por-execução são tratados pelos CONTEXT.md dos stages, não pelo questionário.
13. Executar as verificações de auditoria abaixo. Se alguma falhar, revisar.
14. Escrever o questionário.

## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| Cobertura de placeholder | Todo placeholder de nível de sistema no workspace tem pergunta correspondente |
| Separação por-execução | Nenhuma variável por-execução aparece no questionário |
| Estrutura flat | Todas as perguntas estão em lista numerada única sem agrupamentos |
| Defaults presentes | Toda pergunta tem default sensato ou exemplo |
| Stack pre-populated | Dados detectados no stage 01 aparecem como defaults nas perguntas de stack |
| Duas passagens | Convenções de código usam processo de review antes de finalizar |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Questionário | `output/questionnaire.md` | Questionário flat, all-at-once, para a pasta setup/ do novo workspace |
