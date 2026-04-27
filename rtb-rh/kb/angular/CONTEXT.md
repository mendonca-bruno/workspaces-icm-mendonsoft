# kb/angular/ — Frontends Angular

Roteador para dossiês de aplicações Angular (portais e ferramentas internas) do RTB-RH.

## Sistemas cobertos

| Sistema | Área | Dossiês |
|---------|------|---------|
| _(vazio)_ | | |

## O que costuma virar dossiê separado

- **Módulo funcional do portal**: `<sistema>/modulo-<nome>.md` (ex.: `portal-holerite/modulo-visualizacao.md`).
- **Conjunto de telas relacionadas**: agrupar.
- **Integrações HTTP críticas**: dossiê de "API consumida" descrevendo os endpoints chamados, headers, tratamento de erro.

## Pontos de atenção em ingestão

- Tickets de Angular tipicamente são "tela não carrega" / "campo X não aparece". Geralmente o root cause está no backend Java (que retorna erro/vazio) — Stage 03 deve seguir `related:` para dossiês Java.
- Não é necessário destilar componentes triviais (header, footer). Foque no que tem regra de negócio.
- Sempre documente: que API o módulo consome, que sistema downstream realmente alimenta.
