# {{ project_name }}

Production-grade Django {{ django_version }} project generated from
[`django-project-template`](https://github.com/your-org/django-project-template).

For **creating** projects from GitHub (remote zip), template usage, and troubleshooting,
see the template repo README:

**https://github.com/your-org/django-project-template#readme**

Hint — remote archive (extract first if your repo keeps files under `project_template/`):

```bash
curl -L -o tpl.zip https://github.com/your-org/django-template/archive/main.zip
unzip -q tpl.zip
django-admin startproject {{ project_name }} \
  --template=django-template-main/project_template \
  --extension=py,toml,yml,yaml,md,in \
  --name=Dockerfile,.env,.gitignore,.dockerignore,.aiignore,.pre-commit-config.yaml
```

---

## Stack

- Python 3.12
- Django {{ django_version }}
- PostgreSQL 16
- Docker / Docker Compose
- Ruff, MyPy, pytest, django-stubs, pre-commit

## Project layout

```
{{ project_name }}/
├── docker/
├── requirements/
├── src/
│   ├── manage.py
│   ├── apps/
│   └── config/
│       └── settings/
├── pyproject.toml
├── .pre-commit-config.yaml
├── .env
└── README.md
```

## Commands

**Host**

```bash
pip install -r requirements/dev.txt
python src/manage.py migrate
python src/manage.py runserver
```

**Docker**

```bash
docker compose -f docker/docker-compose.yml up --build
```

**Tests / lint**

```bash
pytest
ruff check . && ruff format .
mypy src
```

## New app

```bash
mkdir -p src/apps/myapp
python src/manage.py startapp myapp src/apps/myapp \
  --template=/path/to/django-app-template/app_template
```

Add `apps.myapp` to `LOCAL_APPS` in `src/config/settings/base.py`.

## Settings modules

| Environment | `DJANGO_SETTINGS_MODULE` |
|-------------|---------------------------|
| Development | `config.settings.development` |
| Staging     | `config.settings.staging` |
| Production  | `config.settings.production` |
| Tests       | `config.settings.test` |
