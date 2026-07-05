# Quart + Postgres Boilerplate

A minimal, production-shaped starting point for building **async** web apps with
[Quart](https://quart.palletsprojects.com/) and **PostgreSQL** — fully containerized with
Docker, so there's nothing to install on your host.

> 📚 **Part of the [FromZero](https://github.com/FromZeroCourses) course _From Flask to Quart_.**
> This is the finished application you build in Chapter 4 (*A Quart Database Counter*) — kept
> here as a standalone boilerplate you can clone or fork to start your own Quart + Postgres
> projects.
>
> **Course materials:** [`from-flask-to-quart-course`](https://github.com/FromZeroCourses/from-flask-to-quart-course)
> *(a dedicated course website is coming — this link will point there once it's live).*

## What's inside

| | |
|---|---|
| [Quart](https://quart.palletsprojects.com/) | async, Flask-compatible web framework |
| [SQLAlchemy 2.0](https://www.sqlalchemy.org/) (async) + [asyncpg](https://github.com/MagicStack/asyncpg) | async ORM / Core over PostgreSQL |
| [Alembic](https://alembic.sqlalchemy.org/) | database migrations |
| [pytest](https://docs.pytest.org/) + pytest-asyncio | async test suite, with a fresh isolated database per test |
| [uv](https://docs.astral.sh/uv/) | fast Python packaging |
| Docker + Docker Compose | everything runs in containers |

Python 3.12+. The example feature is a database-backed **counter** (the `counter/` package) —
swap it for your own models and views.

## Quick start

Everything runs in Docker — you don't need Python, Postgres, or uv on your machine.

### 1. Start the stack

```bash
docker compose up -d --build
```

This builds the `web` image and starts two containers: **`app_web_1`** (Quart, on port 5000)
and **`app_db_1`** (Postgres 16, on port 5432).

### 2. Apply the database migrations

```bash
docker compose exec web uv run alembic upgrade head
```

### 3. Open the app

Visit **http://localhost:5000** — every refresh bumps the counter.

### 4. Run the tests

```bash
docker compose run --rm web uv run pytest
```

Add `-s` to also see the fixtures' setup/teardown output (a fresh database is created and
destroyed for every test):

```bash
docker compose run --rm web uv run pytest -s
```

## Project structure

```
.
├── manage.py              # QUART_APP entry point — builds the app via create_app()
├── application.py         # the app factory: registers the counter blueprint,
│                          #   opens/closes the async DB engine on before/after-serving
├── settings.py            # env-driven config (DB URL, secret key, …)
├── db.py                  # async SQLAlchemy engine + table metadata
├── counter/               # the example feature — replace with your own
│   ├── models.py          #   counter_table (SQLAlchemy Core)
│   └── views.py           #   the "/" route that increments + renders the counter
├── migrations/            # Alembic environment
│   └── versions/          #   create-counter-table migration
├── tests/
│   ├── conftest.py        # fixtures: a fresh test DB + app + client per test
│   └── test_counter.py    # example tests (response + direct DB assertions)
├── docker-compose.yml     # web + db services
├── Dockerfile             # uv-based Python 3.12 image
├── pyproject.toml         # dependencies (managed by uv)
├── uv.lock
└── .quartenv             # local env vars (QUART_APP, DB credentials, SECRET_KEY)
```

## Using it as a boilerplate

To start a new project from this skeleton:

1. **Grab it** — click **"Use this template"** on GitHub, or clone and re-init:
   ```bash
   git clone https://github.com/FromZeroCourses/quart-postgres-boilerplate.git my-app
   cd my-app && rm -rf .git && git init
   ```
2. **Rename** to taste — the `counter/` package, the `counter-app` name in `pyproject.toml`,
   and the `/counter_app` paths in the `Dockerfile` / `docker-compose.yml`.
3. **Replace** `counter/` with your own models (`models.py`) and routes (`views.py`), and
   register your blueprint in `application.py`.
4. **Create your first migration** for the new tables:
   ```bash
   docker compose run --rm web uv run alembic revision --autogenerate -m "create my tables"
   docker compose exec web uv run alembic upgrade head
   ```
5. **Change the secrets** before deploying anywhere real — `SECRET_KEY` in `.quartenv` and the
   database credentials in `.quartenv` / `docker-compose.yml`.

## License

Provided as course material for _From Flask to Quart_ by [FromZero](https://github.com/FromZeroCourses).
Use it freely as a starting point for your own projects.
