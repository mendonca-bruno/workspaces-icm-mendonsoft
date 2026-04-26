# Golden Example — Full Stack FastAPI Template

Repositório de referência canônica para Python/FastAPI. Stage 02 cita como template; stage 06 usa shape para cross-check.

## URL

https://github.com/tiangolo/full-stack-fastapi-template (anteriormente `full-stack-fastapi-postgresql`)

## Stack

- Python 3.11+
- FastAPI
- SQLModel (wrapper sobre SQLAlchemy 2 + Pydantic)
- PostgreSQL
- Alembic (migrations)
- pytest

(Para projetos sem SQLModel, swap para SQLAlchemy 2 puro + Pydantic.)

## Expected shape

```
full-stack-fastapi-template/
├── pyproject.toml
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    (FastAPI app + lifespan)
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── api.py             (APIRouter aggregator)
│   │   │       ├── deps.py            (Depends factories)
│   │   │       └── routes/
│   │   │           ├── users.py
│   │   │           ├── items.py
│   │   │           └── login.py
│   │   ├── core/
│   │   │   ├── config.py              (Settings via pydantic-settings)
│   │   │   ├── security.py
│   │   │   └── db.py
│   │   ├── models.py                  (SQLModel classes — entities + schemas)
│   │   ├── crud.py                    (data access functions)
│   │   └── alembic/
│   │       └── versions/
│   └── tests/
│       ├── conftest.py
│       └── api/
│           └── routes/
│               ├── test_users.py
│               └── test_items.py
└── docker-compose.yml
```

## Componentes representativos

| Conceito | Arquivo | Padrão |
|---|---|---|
| Entity + schema | `app/models.py` | SQLModel = entity + Pydantic |
| Router | `app/api/v1/routes/items.py` | `APIRouter()` + `@router.get/post/put/delete` |
| DI factory | `app/api/v1/deps.py` | `def get_db() -> Generator[Session, None, None]` + `SessionDep = Annotated[Session, Depends(get_db)]` |
| Service-ish | `app/crud.py` | funções (não classes) |
| Settings | `app/core/config.py` | `class Settings(BaseSettings)` |
| Tests | `tests/api/routes/test_users.py` | `client: TestClient` injectado via fixture |

## Casos canários para L3 equivalence

| Caso | Endpoint | Espera |
|---|---|---|
| Login | `POST /api/v1/login/access-token` form | 200 + JWT |
| Listar items | `GET /api/v1/items?skip=0&limit=10` | 200 + lista paginada |
| Buscar item | `GET /api/v1/items/{id}` | 200 ou 404 |
| Criar item | `POST /api/v1/items` body=ItemCreate | 200 + ItemPublic |
| Update item | `PUT /api/v1/items/{id}` | 200 + atualizado |
| Delete item | `DELETE /api/v1/items/{id}` | 200 + msg |

## Por que este exemplo

- Mantido por Sebastián Ramírez (criador do FastAPI).
- Já tem auth (JWT), CRUD completo, migrations, testes — exemplo end-to-end.
- Estrutura bem alinhada com práticas modernas de FastAPI.
- SQLModel reduz duplicação entity/schema (relevante migrando de Spring onde DTO + Entity são separados).

## Caveat para migração

- Se source não tem auth (Spring sem Spring Security), ignorar `security.py`/`login.py`.
- Se preferir SQLAlchemy 2 puro (entity separada de schema Pydantic), o template `cookiecutter-fastapi-sqlalchemy` ou estrutura manual é mais próxima do "espelho 1:1" do JPA.

## Variante Django

Para target `python/django`, usar como referência:
- https://github.com/wsvincent/awesome-django (lista curada)
- https://github.com/cookiecutter/cookiecutter-django (cookiecutter oficial)

## Não vendor

URL + esta descrição apenas.
