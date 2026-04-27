# Mapa Área RH × Sistemas

Lista canônica das 5 áreas de People & Culture e os sistemas-pivô de cada uma. Preencher com os nomes reais dos sistemas usados na F1rst/Santander.

**Como usar:** Stage 02 (clarificação) consulta este mapa quando o solicitante mencionou uma área mas não um sistema específico. Stage 03 (busca) usa para limitar o escopo de dossiês carregados.

**Convenção de nomes:** kebab-case, sem acentos. Ex.: `folha-mensal`, não `Folha Mensal`.

---

## Folha

| Sistema | Tecnologia principal | Função |
|---------|----------------------|--------|
| folha-mensal | Java / Oracle | Cálculo da folha mensal |
| folha-13 | Java / Oracle | Cálculo do 13º salário |
| ferias | Java / Oracle | Cálculo de férias |
| portal-holerite | Angular / Java | Portal de visualização do contracheque |
| _(adicionar conforme inventário real)_ | | |

## Ponto

| Sistema | Tecnologia principal | Função |
|---------|----------------------|--------|
| ponto-eletronico | Java / Oracle | Registro e cálculo de jornada |
| integracao-catraca | TIBCO BW | Integração com catracas físicas |
| _(adicionar)_ | | |

## Benefícios

| Sistema | Tecnologia principal | Função |
|---------|----------------------|--------|
| beneficios-portal | Angular / Java | Portal de benefícios do funcionário |
| integracao-fornecedor-vr | TIBCO BW / PowerCenter | Envio de carga para fornecedor de VR/VA |
| _(adicionar)_ | | |

## Admissão

| Sistema | Tecnologia principal | Função |
|---------|----------------------|--------|
| workflow-admissao | Java / Angular | Workflow de aprovações de admissão |
| integracao-erp-admissao | TIBCO BW / PowerCenter | Carga do funcionário no ERP central |
| _(adicionar)_ | | |

## Demissão

| Sistema | Tecnologia principal | Função |
|---------|----------------------|--------|
| workflow-demissao | Java / Angular | Workflow de aprovações de demissão |
| baixa-sistemas-downstream | Control-M / TIBCO BW | Propagação da baixa para sistemas downstream |
| _(adicionar)_ | | |

---

## Sistemas transversais

| Sistema | Tecnologia | Função |
|---------|-----------|--------|
| controlm | Control-M | Orquestração de jobs batch (toda área) |
| _(adicionar barramentos, datalakes, etc.)_ | | |
