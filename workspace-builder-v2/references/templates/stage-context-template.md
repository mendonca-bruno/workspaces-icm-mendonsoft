# Stage {{STAGE_NUMBER}}: {{STAGE_NAME}}

{{STAGE_DESCRIPTION}}

## Inputs

| Fonte | Arquivo/Localização | Escopo | Por quê |
|-------|---------------------|--------|---------|
| {{INPUT_SOURCE}} | {{INPUT_PATH}} | {{INPUT_SCOPE}} | {{INPUT_REASON}} |

## Processo

1. {{PROCESS_STEP_1}}
2. {{PROCESS_STEP_2}}
3. {{PROCESS_STEP_3}}

{{?CHECKPOINTS}}
## Checkpoints

| Após Passo | Agente Apresenta | Humano Decide |
|------------|-------------------|---------------|
| {{CHECKPOINT_STEP}} | {{CHECKPOINT_PRESENT}} | {{CHECKPOINT_DECIDE}} |
{{/CHECKPOINTS}}

{{?VERIFY}}
## Verify

| Verificação | Comando | Condição de Aprovação |
|-------------|---------|----------------------|
| {{VERIFY_NAME}} | {{VERIFY_CMD}} | {{VERIFY_CONDITION}} |
{{/VERIFY}}

{{?AUDIT}}
## Audit

| Verificação | Condição de Aprovação |
|-------------|----------------------|
| {{AUDIT_CHECK}} | {{AUDIT_CONDITION}} |
{{/AUDIT}}

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| {{OUTPUT_NAME}} | `output/{{OUTPUT_FILE}}` | {{OUTPUT_FORMAT}} |
