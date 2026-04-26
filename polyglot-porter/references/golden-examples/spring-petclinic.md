# Golden Example — Spring PetClinic

Repositório de referência canônica para Java/Spring Boot. Stage 02 cita como template; stage 06 usa shape para cross-check.

## URL

https://github.com/spring-projects/spring-petclinic

## Stack

- Java 17+
- Spring Boot 3.x
- Spring Web MVC
- Spring Data JPA + Hibernate
- H2 (default) / MySQL / PostgreSQL profiles
- Maven (default) ou Gradle
- Thymeleaf (UI server-side; ignorar se migrando só API)

## Expected shape (estrutura idiomática)

```
spring-petclinic/
├── pom.xml
├── src/main/java/org/springframework/samples/petclinic/
│   ├── PetClinicApplication.java        @SpringBootApplication
│   ├── owner/
│   │   ├── Owner.java                   @Entity
│   │   ├── OwnerController.java         @Controller (web) ou @RestController
│   │   ├── OwnerRepository.java         interface JpaRepository<Owner, Integer>
│   │   ├── Pet.java                     @Entity
│   │   ├── PetController.java
│   │   ├── PetRepository.java
│   │   └── ...
│   ├── vet/
│   │   ├── Vet.java
│   │   ├── VetController.java
│   │   └── VetRepository.java
│   └── visit/
│       └── Visit.java
├── src/main/resources/
│   ├── application.properties
│   └── db/
└── src/test/java/<espelho>/
    ├── PetClinicIntegrationTests.java   @SpringBootTest
    └── owner/OwnerControllerTests.java  @WebMvcTest
```

## Componentes representativos

| Conceito | Arquivo | Padrão |
|---|---|---|
| Entity com relacionamento | `Owner.java` | `@OneToMany` para `Pet` |
| Controller com path variable | `OwnerController.java` | `@GetMapping("/owners/{ownerId}")` |
| Repository | `OwnerRepository.java` | `extends JpaRepository<Owner, Integer>` + custom `@Query` |
| Validation | `Owner.java` | `@NotEmpty`, `@Digits` |
| Test slice | `OwnerControllerTests.java` | `@WebMvcTest` + `MockMvc` |
| Integration test | `PetClinicIntegrationTests.java` | `@SpringBootTest` |

## Casos canários para L3 equivalence

| Caso | Endpoint | Espera |
|---|---|---|
| Listar owners | `GET /owners?lastName=Davis` | 200 + lista filtrada |
| Buscar owner por ID | `GET /owners/1` | 200 + Owner com pets |
| Criar owner | `POST /owners/new` body=form | 302 redirect ou 201 (se API) |
| Adicionar pet a owner | `POST /owners/{id}/pets/new` | 302 ou 201 |
| Listar vets | `GET /vets.html` ou `GET /api/vets` | 200 + lista |

## Por que este exemplo

- Cobre todas as camadas idiomáticas Spring (controller, service via repository, entity, validation, test).
- Tamanho gerenciável (~3000 LOC).
- Mantido pelo time oficial do Spring.
- Bem documentado em [spring.io/guides](https://spring.io/guides).

## Não vendor

Não copiar o código para dentro deste workspace. Apenas referenciar URL + esta descrição. Se necessário, clonar para `c:\tmp\petclinic-source\` para canary testing.
