# Pending Updates

Flags abertas pelo Stage 05 (pós-mortem) e Stage 02 (clarificação) para o Stage 06 (manutenção) reconciliar. Cada flag indica que a KB tem inconsistência ou lacuna conhecida.

## Como adicionar uma flag

Append uma seção `### <slug-curto>` com o schema abaixo. Não edite flags abertas por outro stage — apenas adicione novas ou marque como `resolved`.

```markdown
### dossier-rubrica-flag-ativo-divergente

- **Tipo:** `dossier-inconsistency` | `taxonomy-gap` | `playbook-candidate` | `system-not-mapped`
- **Aberto por:** Stage 05 / Stage 02 / Stage 06
- **Aberto em:** YYYY-MM-DD
- **Evidência:** `tickets/INC1234567/` ou `cases/INC1234567.md`
- **Descrição:** O dossiê `kb/java/folha/schema-rubricas.md` diz que `FLAG_ATIVO` só pode mudar via stored procedure, mas vimos DBA editar diretamente.
- **Ação sugerida:** atualizar regra inferida no dossiê + adicionar nota em `## Lições` do caso fonte.
- **Status:** `open` | `in-progress` | `resolved`
- **Resolvido em:** (preencher quando `resolved`)
```

## Flags abertas

_(Nenhuma — workspace recém-bootstrapped.)_

## Tipos de flag

| Tipo | Quando usar | Quem resolve |
|------|-------------|--------------|
| `dossier-inconsistency` | Resolução real contradisse o que o dossiê diz | Bruno (re-rodar Stage 01 com source atualizado) |
| `taxonomy-gap` | Stage 02 encontrou sintoma `symptom_unknown` ou sistema não-mapeado | Bruno (editar `_taxonomies/`) |
| `playbook-candidate` | Stage 05 detectou cluster mas Bruno adiou promoção | Stage 06 reconsidera após N dias |
| `system-not-mapped` | Stage 01 detectou sistema novo que não está em `area-mapping.md` | Bruno (preencher `area-mapping.md`) |
