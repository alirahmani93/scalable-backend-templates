# django-project-template

A production-grade Django 5+ project template, designed to be consumed via
`django-admin startproject --template=...`.

It scaffolds a modular, typed, container-ready Django codebase with batteries
included:

- **Python 3.12 / Django 5+**
- **PostgreSQL-ready** (`dj-database-url`, `psycopg[binary]`)
- **Modular settings**: `base`, `development`, `staging`, `production`, `test`
- **Docker** multi-stage build + `docker-compose` (web + Postgres)
- **Tooling**: Ruff, MyPy (strict + django-stubs), pytest + pytest-django, pre-commit
- **`src/` layout** with `apps/` namespace for local Django apps
- **Production hardening** in `production.py` (HSTS, secure cookies, fail-fast env validation)
- **`.env`, `.gitignore`, `.aiignore`, `.dockerignore`** — rendered with project-specific values

---

## Layout of this repository

```
django-project-template/
├── README.md               # you are here (template repo + usage)
├── LICENSE
└── project_template/       # <-- passed to --template=...
    ├── .aiignore
    ├── .env                # rendered into new projects (placeholders)
    ├── .gitignore          # copied into new projects (not the repo’s root ignore)
    ├── .pre-commit-config.yaml
    ├── README.md           # short README rendered into each new project
    ├── pyproject.toml
    ├── docker/
    │   ├── Dockerfile
    │   ├── docker-compose.yml
    │   └── .dockerignore
    ├── requirements/
    │   ├── base.in
    │   ├── base.txt
    │   ├── dev.in
    │   └── dev.txt
    └── src/
        ├── manage.py-tpl
        ├── apps/
        │   └── __init__.py-tpl
        └── config/
            ├── __init__.py-tpl
            ├── asgi.py-tpl
            ├── urls.py-tpl
            ├── wsgi.py-tpl
            └── settings/
                ├── __init__.py-tpl
                ├── base.py-tpl
                ├── development.py-tpl
                ├── production.py-tpl
                ├── staging.py-tpl
                └── test.py-tpl
```

### `-tpl` suffix convention

Django’s `startproject` only auto-renames `*.py-tpl` → `*.py`. Non-Python files
keep their final names and are rendered when you pass `--extension` /
`--name` (see below).

---

## Usage

> Replace `your-org` and repo name (`django-template` or `django-project-template`)
> with your GitHub org/user and repository. Replace `myproject` with your project name.

Shared flags (local path **or** remote zip — same flags):

```bash
FLAGS='--extension=py,toml,yml,yaml,md,in \
  --name=Dockerfile,.env,.gitignore,.dockerignore,.aiignore,.pre-commit-config.yaml'
```

What they do:

- `--extension=…` — runs Django’s template engine on those extensions (`pyproject.toml`,
  `docker-compose.yml`, `.pre-commit-config.yaml` via `yaml`, `README.md`, `requirements/*.in`,
  and all `*.py-tpl`).
- `--name=…` — renders files matched **by exact basename** (needed for `Dockerfile` and dotfiles).

### A. Local clone (recommended)

```bash
git clone https://github.com/your-org/django-project-template.git
pip install "django>=5.1"

django-admin startproject myproject \
  --template=django-project-template/project_template \
  $FLAGS

cd myproject
```

### B. Remote template from GitHub (zip)

Django can fetch the archive directly. Use whichever URL style GitHub serves for your default branch:

```bash
django-admin startproject myproject \
  --template=https://github.com/your-org/django-template/archive/main.zip \
  $FLAGS
```

Equivalent long form (explicit branch ref):

```bash
django-admin startproject myproject \
  --template=https://github.com/your-org/django-project-template/archive/refs/heads/main.zip \
  $FLAGS
```

**Important:** With a zip URL, Django treats the **archive root** as the template directory.
This repo keeps the real template under `project_template/`, so the root of the zip is **not**
valid as `--template` by itself. Either:

1. **Download and point at `project_template/` inside the extracted folder** (works with any repo layout):

   ```bash
   curl -L -o tpl.zip https://github.com/your-org/django-template/archive/main.zip
   unzip -q tpl.zip
   django-admin startproject myproject \
     --template=django-template-main/project_template \
     $FLAGS
   ```

   > Folder name may be `django-template-main`, `django-project-template-main`, etc., matching the repo name.

2. **Fork** and move everything under `project_template/` to the repository root so the zip root **is** the template (then a single `--template=<zip>` works without extra paths).

Shell helper (local path):

```bash
new-django-project() {
  django-admin startproject "$1" \
    --template=path/to/django-project-template/project_template \
    --extension=py,toml,yml,yaml,md,in \
    --name=Dockerfile,.env,.gitignore,.dockerignore,.aiignore,.pre-commit-config.yaml
}
```

---

## Template variables (`startproject`)

| Variable | Source |
|----------|--------|
| `{{ project_name }}` | First argument to `startproject` |
| `{{ secret_key }}` | Generated by Django |
| `{{ django_version }}` | Installed Django version |
| `{{ docs_version }}` | Matching Django docs version |

---

## Example generated project layout

```
myproject/
├── .aiignore
├── .env
├── .gitignore
├── .pre-commit-config.yaml
├── README.md
├── pyproject.toml
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── .dockerignore
├── requirements/
│   ├── base.in
│   ├── base.txt
│   ├── dev.in
│   └── dev.txt
└── src/
    ├── manage.py
    ├── apps/
    │   └── __init__.py
    └── config/
        ├── __init__.py
        ├── asgi.py
        ├── urls.py
        ├── wsgi.py
        └── settings/
            ├── __init__.py
            ├── base.py
            ├── development.py
            ├── production.py
            ├── staging.py
            └── test.py
```

`.env` and `pyproject.toml` will contain `myproject` and a generated secret key.

---

## After you generate a project (quick reference)

This section merges the guides that ship inside each new project’s `README.md`.

### Stack

- Python 3.12 · Django 5+ · PostgreSQL 16  
- Docker / Docker Compose · Ruff · MyPy · pytest · django-stubs · pre-commit  

### Local development (host)

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements/dev.txt

# Optional: tweak env
cp .env .env.local
set -a && source .env && set +a   # or export vars your preferred way

python src/manage.py migrate
python src/manage.py runserver
```

Run commands from the **project root** (the directory that contains `src/`).

### Local development (Docker)

```bash
docker compose -f docker/docker-compose.yml up --build
docker compose -f docker/docker-compose.yml exec web python src/manage.py migrate
docker compose -f docker/docker-compose.yml exec web python src/manage.py createsuperuser
```

The `web` service mounts `src/` for hot reload.

### Tests

```bash
pytest
docker compose -f docker/docker-compose.yml exec web pytest
```

### Linting and types

```bash
ruff check .
ruff format .
mypy src
```

### Pre-commit

```bash
pre-commit install
pre-commit run --all-files
```

### Choosing settings

| Environment | `DJANGO_SETTINGS_MODULE` |
|-------------|---------------------------|
| Development | `config.settings.development` |
| Staging     | `config.settings.staging` |
| Production  | `config.settings.production` |
| Tests       | `config.settings.test` |

Secrets and hosts typically come from `.env` / the environment (`DJANGO_SECRET_KEY`,
`DJANGO_ALLOWED_HOSTS`, database variables, etc.).

### New Django app (companion template)

From the **project root** (not inside `src/`):

```bash
mkdir -p src/apps/billing
python src/manage.py startapp billing src/apps/billing \
  --template=path/to/django-app-template/app_template
```

Register `apps.billing` in `LOCAL_APPS` in `src/config/settings/base.py`.

See the companion **django-app-template** repository for app scaffolding and remote zip notes:
`https://github.com/your-org/django-app-template#readme`

---

## License

MIT — see [LICENSE](LICENSE).
