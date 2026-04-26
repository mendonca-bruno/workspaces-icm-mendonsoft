# Golden Examples — Index

Referências canônicas (URL + shape) para cada stack-flavor MVP. Stage 02 cita ao escolher target; stage 06 usa para cross-check de layout.

## MVP

| Stack | Arquivo | URL principal |
|---|---|---|
| Java / Spring Boot | [spring-petclinic.md](spring-petclinic.md) | https://github.com/spring-projects/spring-petclinic |
| .NET / ASP.NET 8 | [eshop-on-web.md](eshop-on-web.md) | https://github.com/dotnet-architecture/eShopOnWeb |
| Python / FastAPI | [full-stack-fastapi-template.md](full-stack-fastapi-template.md) | https://github.com/tiangolo/full-stack-fastapi-template |

## Política

- **Não vendor.** Referenciar via URL + descrição. Se necessário clonar para canary, usar `c:\tmp\<name>/` fora do workspace.
- **Não considerar autoridade absoluta.** Cada projeto tem suas próprias decisões; o golden serve como "shape esperada", não como spec rígida.
- **Cross-check shape.** Stage 06 audita layout do overlay contra as seções "Expected shape" destes arquivos.

## Precedência

Os golden-examples são **fallback default**. Se o usuário fornece `target_references[]` no questionnaire (paths para projetos próprios da target-lang), o destilado dessas references **sobrescreve** os goldens. Ver `_config/reference-ingestion.md` para a ordem completa de precedência (`target_flavor` > user references > golden-examples > archetypes).

Stage 02 só carrega o golden-example se `target_references[]` está vazio ou todas as references foram skipadas.

## Pares canários

| Source → Target | Canário | Aceitação |
|---|---|---|
| Spring PetClinic → ASP.NET | https://github.com/spring-projects/spring-petclinic + eShopOnWeb shape | L1+L2 da rubric |
| Spring PetClinic → FastAPI | spring-petclinic + full-stack-fastapi-template shape | L1+L2 |
| Reverso (ASP.NET → Spring, FastAPI → Spring) | usar mesmos repos | idem |
