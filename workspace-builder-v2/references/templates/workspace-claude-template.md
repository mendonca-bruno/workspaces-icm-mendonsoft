# {{NOME_PROJETO}}

{{DESCRICAO_CURTA}}

## Mapa de Pastas

```
{{FOLDER_MAP}}
```

## Triggers

| Keyword | Ação |
|---------|------|
{{TRIGGERS_TABLE}}

## Roteamento

| Tarefa | Ir Para |
|--------|---------|
{{ROUTING_TABLE}}

## O Que Carregar

| Tarefa | Carregar | NÃO Carregar |
|--------|----------|--------------|
{{LOADING_TABLE}}

## Stage Handoffs

Cada stage escreve seu output na sua pasta `output/`. O próximo stage lê de lá. Se você editar um output entre stages, o próximo stage pega suas edições.
