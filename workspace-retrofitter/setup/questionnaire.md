# Setup Questionnaire — Antes de Começar o Scan

Responda estas 6 perguntas. Suas respostas viram parte dos inputs de stage 01 (scan) e stage 02 (classify). Salve este arquivo após preencher; o agente lê os campos abaixo da linha `## Suas respostas`.

---

## 1. Caminho do diretório alvo

Caminho absoluto. Ex: `C:\Users\bruno\projects\meu-blog` ou `/home/bruno/work/cliente-x`.

## 2. Propósito do diretório

Em uma frase: o que você faz neste diretório? Ex: "Escrevo posts para meu blog", "Mantenho o código de uma API", "Arquivo de pesquisa pessoal", "Pasta de Downloads que preciso organizar".

## 3. Audiência

Quem usa este diretório?

- Só você
- Você + agente de IA (Claude Code, Cursor, Copilot)
- Você + outras pessoas (colegas, clientes)
- Você + outras pessoas + agentes

## 4. Restrições — pastas a NUNCA tocar

Liste pastas/padrões que devem ser ignorados além do default (`.git/`, `node_modules/`, `.venv/`, `dist/`, `build/`, `__pycache__/`, `target/`).

Exemplo:
- `secrets/`
- `client-deliverables/2024/` (já entregue, congelar)
- `*.key`

## 5. Sensibilidade — algo que o agente NÃO deve ler?

Arquivos com credenciais, dados pessoais de terceiros, contratos sob NDA. Liste padrões. O scan não vai ler conteúdo desses arquivos, apenas registrar nome e tamanho.

## 6. Você já tentou organizar este diretório antes?

- Não, é como nasceu.
- Sim, tem alguma estrutura que eu queria preservar (descreva).
- Sim, e quero refazer do zero (ignorar estrutura atual no design).

---

## Suas respostas

```yaml
target_path: "C:\\Users\\bruno\\Downloads\\ClifNotes\\Classrooms"
purpose: "Arquivos .md com todas as aulas do Skool Clief Notes"
audience: "solo+ai"
ignore_paths: []
sensitive_patterns: []
prior_organization: "redo"
```

---

*Após preencher, diga `scan --target=<caminho>` para iniciar stage 01.*
