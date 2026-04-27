# Progress

## Fase Atual

Bootstrap — workspace recém-scaffoldado. Aguardando ingestão das primeiras fontes (Stage 01) e bootstrap manual de cases históricos.

## Última Sessão

- **Data:** 2026-04-26
- **Realizado:** Scaffold inicial do workspace (CLAUDE.md, CONTEXT.md, 6 stages, taxonomias seed, templates de reference, estrutura de pastas).
- **Decisões:** Ver `DECISIONS.md` ADR-001..ADR-005.

## Perguntas Abertas

- [ ] Quais 5-10 sistemas-pivô por área RH? Preencher `kb/_taxonomies/area-mapping.md` antes da primeira ingestão real.
- [ ] Qual é o threshold mínimo do score de clarificação no Stage 02? (sugestão inicial: 3 de 4 dimensões preenchidas)
- [ ] Há retenção legal/compliance que impede arquivar matrículas em `cases/`? Confirmar antes de bootstrap de cases históricos.

## Próximos Passos

1. Bruno preenche `kb/_taxonomies/area-mapping.md` com os sistemas reais por área (15 min).
2. Selecionar 1 fonte real (DDL de RUBRICA da folha, por exemplo) e rodar Stage 01 ponta-a-ponta como smoke test.
3. Bootstrap manual de 5-10 cases históricos memoráveis em `cases/INC*.md` para o Stage 03 ter o que matchear.
4. Rodar Stage 03 com 1 ticket conhecido e verificar que cases aparecem antes de dossiês.

## Métricas (vazias até primeira semana de uso real)

| Métrica | Atual | Alvo |
|---------|-------|------|
| Tickets processados | 0 | — |
| % match em cases no Stage 03 | — | subir ao longo do tempo |
| % tickets que dispararam clarificação no Stage 02 | — | 20-40% |
| Playbooks promovidos | 0 | > 0/mês após mês 2 |
| Dossiês com 0 hits após 60 dias | — | minimizar |

## Padrões de Edição Recorrentes

Rastreie edits que você faz repetidamente. Indicam que algo no source (`reference/*`, taxonomias, dossiês) precisa ser corrigido na origem (princípio edit-source do paper ICM).

| Stage | O Que Você Edita Repetidamente | Sugestão de Fix no Source |
|-------|-------------------------------|--------------------------|
| | | |

---

*Atualize este arquivo ao final de cada sessão de trabalho. Leva 2 minutos e economiza 20 minutos na próxima sessão.*
