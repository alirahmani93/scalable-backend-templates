# Scalable Backend Templates
Production-grade scalable backend templates for modern frameworks like Django, Gin, Fiber, and more — featuring clean architecture, modular structure, Docker, testing, linting, and deployment-ready setups.

# Django template kit

This **single repository** ships two reusable Django scaffolds:

| Package | Purpose |
|---------|---------|
| [**django-project-template**](django/django-project-template/) | Full project layout (`src/`, settings split by env, Docker, tooling). Use with `django-admin startproject --template=…`. |
| [**django-app-template**](django/django-app-template/) | Modular app layout (models, services, views, versioned URLs). Use with `python src/manage.py startapp … --template=…`. |

Stack targets: **Python 3.12**, **Django 5+**, **PostgreSQL**, **Docker**, **Ruff**, **MyPy**, **pytest**, **pre-commit**.

---

## Repository layout

```
.
├── README.md                      ← you are here
├── .gitignore
├── django-project-template/       # project template repo contents
│   ├── README.md                  # full docs: flags, remote zip, troubleshooting
│   ├── LICENSE
│   └── project_template/          # ← pass this path to --template
└── django-app-template/           # app template repo contents
    ├── README.md
    ├── LICENSE
    └── app_template/              # ← pass this path to --template
```

For **deep usage**, flags (`--extension`, `--name`), and edge cases, read:

- [django-project-template/README.md](django/django-project-template/README.md)
- [django-app-template/README.md](django/django-app-template/README.md)

---

## Quick start: new project

From the machine where you clone **this** repo:

```bash
pip install "django>=5.1"

django-admin startproject myproject \
  --template=django-project-template/project_template \
  --extension=py,toml,yml,yaml,md,in \
  --name=Dockerfile,.env,.gitignore,.dockerignore,.aiignore,.pre-commit-config.yaml

cd myproject
pip install -r requirements/dev.txt
python src/manage.py migrate
python src/manage.py runserver
```

Always point `--template` at **`project_template/`**, not at `django-project-template/` alone.

---

## Quick start: new app (inside a generated project)

Run from the **project root** (folder that contains `src/manage.py`). Create the destination directory first:

```bash
mkdir -p src/apps/billing
python src/manage.py startapp billing src/apps/billing \
  --template=/absolute/path/to/django-app-template/app_template
```

Then register `apps.billing` in `LOCAL_APPS` in your settings.

---

## Remote template from GitHub (zip)

You can pass a GitHub archive URL to `--template`, but Django treats the **root of the zip** as the template directory. This repo keeps the real templates under `project_template/` and `app_template/`, so **after cloning or downloading** you either:

1. Use a **local path** to `…/project_template` or `…/app_template` (recommended), or  
2. Download the zip, unzip, and use `--template=<extracted-dir>/django-template-main/project_template` (folder name follows `repo-branch`).

Example zip URL (adjust org and repo name):

```bash
curl -L -o tpl.zip https://github.com/your-org/django-template/archive/main.zip
unzip -q tpl.zip
django-admin startproject myproject \
  --template=django-template-main/django-project-template/project_template \
  --extension=py,toml,yml,yaml,md,in \
  --name=Dockerfile,.env,.gitignore,.dockerignore,.aiignore,.pre-commit-config.yaml
```

If your GitHub repo **only** contains `project_template/` at the root (no extra nesting), shorten the `--template=` path accordingly.

---

## Common mistakes

| Problem | Fix |
|---------|-----|
| `Destination directory … does not exist` | `mkdir -p src/apps/<name>` before `startapp`. |
| Double `src/src/…` | Run `startapp` from project root, not from inside `src/`. |
| Wrong template root | Use `…/project_template` or `…/app_template`, not the parent folder only. |
| Zip `--template` fails | Unzip and pass path to inner `project_template` / `app_template`. |

---

## License

Template packages include their own **MIT** `LICENSE` files under each subdirectory.
