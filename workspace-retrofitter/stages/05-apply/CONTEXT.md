# Stage 05: Apply

Aplicar a migração in-place no diretório alvo. Só roda com aprovação humana explícita. Toda operação é reversível.

## Inputs

| Fonte | Arquivo / Localização | Escopo | Por quê |
|-------|----------------------|--------|---------|
| Overlay | `<target>-icm-overlay/APPROVED.md` | Existência (conteúdo irrelevante) | Gate humano obrigatório |
| Overlay | `<target>-icm-overlay/file-mapping.csv` | Completo (versão pós-edição do usuário) | Fonte de verdade dos movimentos |
| Overlay | `<target>-icm-overlay/CLAUDE.md`, `CONTEXT.md`, `<silo>/CONTEXT.md` | Conteúdo final | Para escrever no alvo |
| Usuário | `--target=<path>` | Caminho | Confirmação de qual diretório aplicar |

## Process

1. **Pré-flight** (sem modificar): `APPROVED.md` existe; CSV passa schema; nenhuma linha `manual-review` pendente; cada `source_path` ainda existe; nenhum `destination_path` colide (exceto conflitos ICM listados no MIGRATION-PLAN).
2. **Rollback manifest:** escrever `rollback-manifest.json` no overlay (array de `{action, source, destination, timestamp_utc}`) ANTES de qualquer movimento.
3. **Criar pastas** dos silos no alvo.
4. **Mover arquivos** linha por linha (`action`: `move`/`copy`/`leave`). Logar cada operação em `apply-log.md`.
5. **Escrever arquivos ICM** do overlay no alvo: `CLAUDE.md`, `CONTEXT.md` raiz, `<silo>/CONTEXT.md`.
6. **Pós-flight:** validar arquivos ICM presentes; contagem final ≈ inicial + ICM novos.
7. Resumo em `apply-log.md`.

## Modo rollback

Invocado com `--rollback`: lê `rollback-manifest.json`, executa em ordem inversa, loga em `rollback-log.md`. Não exige `APPROVED.md`.

## Checkpoints

| Após | Apresenta | Humano Decide |
|------|-----------|---------------|
| 1 | Pré-flight: N moves, K conflitos, R pendentes | Continua ou aborta |
| 7 | Resumo + caminho do manifest | Confirma ou solicita rollback |

## Audit

| Verificação | Aprovação |
|-------------|-----------|
| `APPROVED.md` presente | Existe no overlay |
| Sem `manual-review` pendente | Todas resolvidas |
| Manifest escrito antes dos moves | Mtime anterior |
| Nenhum arquivo perdido | Cada source: existe em destino OU em source |
| `.git/` intacto | Byte-idêntica |

## Outputs

| Artefato | Localização | Formato |
|----------|-------------|---------|
| Manifesto de rollback | `<target>-icm-overlay/rollback-manifest.json` | JSON: array de operações reversíveis |
| Log de aplicação | `output/apply-log.md` | Markdown: cada operação + status |
| Log de rollback (se aplicado) | `output/rollback-log.md` | Markdown: cada reversão + status |

## Hand-off

Após sucesso, o alvo **é** um workspace ICM. Overlay pode ser arquivado/deletado sem afetar o alvo.
