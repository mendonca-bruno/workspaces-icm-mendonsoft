# kb/tibco/ — Processos TIBCO BW

Roteador para dossiês de processos TIBCO BusinessWorks que fazem integrações entre sistemas RTB-RH.

## Sistemas cobertos

| Sistema | Área | Dossiês |
|---------|------|---------|
| _(vazio)_ | | |

## O que costuma virar dossiê separado

- **Processo BW de ponta-a-ponta**: `<sistema>/processo-<nome>.md` (ex.: `integracao-fornecedor-vr/processo-envio-carga.md`).
- **Conjunto de subprocessos cooperantes**: agrupar em 1 dossiê quando compartilham contexto e nomeações.
- **Configuração de filas/tópicos JMS**: `<sistema>/filas-<dominio>.md`.
- **Mapping XSLT crítico**: pode entrar como seção dentro do dossiê do processo, ou como dossiê próprio se reusado.

## Pontos de atenção em ingestão

- Sources BW são tipicamente arquivos `.xml`, `.process`, `.bwm`. Mantenha em `_sources/tibco/<sistema>/` com nomes originais.
- Destile o **fluxo lógico** (qual a entrada, qual o destino, transformações principais), não o XML linha-a-linha.
- Sempre documente: fila/tópico de entrada, fila/tópico de saída, error queue, idempotência.
