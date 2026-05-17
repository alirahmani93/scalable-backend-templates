# django-app-template

A Django app template designed to be consumed via
`python manage.py startapp --template=...`.

It scaffolds a clean, modular app following service-layer / clean
architecture conventions: models, managers, services, serializers, views,
versioned URLs and tests are each their own package, ready to grow.

## Layout of this repository

```
django-app-template/
├── README.md             # you are here
├── LICENSE
└── app_template/         # <-- passed to --template=...
    ├── __init__.py-tpl
    ├── admin.py-tpl
    ├── apps.py-tpl
    ├── migrations/
    ├── managers/
    ├── models/
    ├── serializers/
    ├── services/
    ├── tests/
    ├── urls/
    └── views/
```

`apps.py-tpl` generates a fully-qualified `AppConfig` dynamically using
`{{ camel_case_app_name }}` and `{{ app_name }}`, so the resulting class is
named `BillingConfig` for an app called `billing`, etc.

---

## Usage

> Replace `your-org` and repo name (`django-template` vs `django-app-template`)
> with your GitHub org/user and repository. Replace `billing` with your app name.

This template is meant to be used **inside a project generated from
django-project-template**, where local apps live under `src/apps/`.

Always create the destination directory first; run `startapp` from the **project root**
(the folder that contains `src/manage.py`), not from inside `src/`.

### A. Local clone (recommended)

```bash
git clone https://github.com/your-org/django-app-template.git

cd /path/to/your-django-project    # project root
mkdir -p src/apps/billing
python src/manage.py startapp billing src/apps/billing \
    --template=/path/to/django-app-template/app_template
```

All files use `*.py-tpl`, which Django renames to `*.py` and renders — **no** `--extension` flag needed.

Register the app in `src/config/settings/base.py`:

```python
LOCAL_APPS = [
    "apps.billing",
]
```

Mount URLs if needed (`src/config/urls.py`):

```python
from django.urls import include, path

urlpatterns = [
    ...
    path("api/v1/billing/", include("apps.billing.urls.v1_urls")),
]
```

### B. Remote template from GitHub (zip)

Short URL (default branch):

```bash
mkdir -p src/apps/billing
python src/manage.py startapp billing src/apps/billing \
    --template=https://github.com/your-org/django-template/archive/main.zip
```

Long ref form:

```bash
python src/manage.py startapp billing src/apps/billing \
    --template=https://github.com/your-org/django-app-template/archive/refs/heads/main.zip
```

**Important:** With a zip URL, Django uses the **top-level folder inside the archive**
as the template root. Because this repo keeps the real template under `app_template/`,
that single `--template=<zip>` line only works if the zip root **is** `app_template`
(e.g. after restructuring a fork). Otherwise download and point at the inner folder:

```bash
curl -L -o app-tpl.zip https://github.com/your-org/django-app-template/archive/main.zip
unzip -q app-tpl.zip
python src/manage.py startapp billing src/apps/billing \
    --template=django-app-template-main/app_template
```

If your archive contains nested folders (for example a monorepo), adjust the path so it ends at the directory that directly contains `apps.py-tpl`.

---

## Template variables (`startapp`)

| Variable | Example (`billing`) |
|----------|---------------------|
| `{{ app_name }}` | `billing` |
| `{{ camel_case_app_name }}` | `Billing` |
| `{{ django_version }}` | Installed Django |
| `{{ docs_version }}` | Django docs version |

---

## Example generated app

```
src/apps/billing/
├── __init__.py
├── admin.py
├── apps.py                   # class BillingConfig(AppConfig): ...
├── migrations/__init__.py
├── managers/__init__.py
├── models/__init__.py
├── serializers/__init__.py
├── services/__init__.py
├── tests/__init__.py
├── urls/__init__.py
├── urls/v1_urls.py           # app_name = "billing_v1"
└── views/__init__.py
```

---

## License

MIT — see [LICENSE](LICENSE).
