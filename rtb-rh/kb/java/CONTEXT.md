# kb/java/ — Sistemas Java

Roteador para dossiês de aplicações Java do domínio RTB-RH (backends, APIs REST, serviços de cálculo).

## Sistemas cobertos

| Sistema | Área | Dossiês |
|---------|------|---------|
| _(vazio — primeiros dossiês entram via Stage 01)_ | | |

## Como adicionar um sistema novo

1. Rode `ingerir <fonte>` com o source primário do sistema (DDL, JAR descompactado, repo).
2. Stage 01 detecta sistema candidato e propõe `kb/java/<sistema>/CONTEXT.md` + 1 ou mais dossiês.
3. Após aprovação, este `CONTEXT.md` é atualizado com o novo sistema (manual ou pelo Stage 06).

## O que costuma virar dossiê separado

- **Schema de banco** de uma área: `<sistema>/schema-<entidade-principal>.md` (ex.: `folha-mensal/schema-rubricas.md`).
- **APIs REST expostas**: `<sistema>/api-<dominio>.md`.
- **Regras de negócio críticas**: `<sistema>/regras-<dominio>.md` (ex.: regras de cálculo de hora extra).
- **Integrações com outros sistemas**: melhor documentar no lado da integração (`tibco/`, `powercenter/`) com referência cruzada.
