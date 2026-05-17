# syntax=docker/dockerfile:1.7
# ---------------------------------------------------------------------------
# Dockerfile for the {{ project_name }} project.
# Multi-stage build:
#   - builder: installs build dependencies and compiles wheels
#   - runtime: minimal image running as a non-root user
# ---------------------------------------------------------------------------

ARG PYTHON_VERSION=3.12

# ---------- builder ---------------------------------------------------------
FROM python:${PYTHON_VERSION}-slim AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /build

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        build-essential \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements/ /build/requirements/
RUN pip wheel --wheel-dir /wheels -r requirements/dev.txt


# ---------- runtime ---------------------------------------------------------
FROM python:${PYTHON_VERSION}-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    DJANGO_SETTINGS_MODULE=config.settings.development \
    PYTHONPATH=/app/src \
    PORT=8000

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        libpq5 \
        curl \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd --system --gid 1000 app \
    && useradd  --system --uid 1000 --gid app --create-home --home-dir /home/app app

WORKDIR /app

COPY --from=builder /wheels /wheels
COPY requirements/ /app/requirements/
RUN pip install --no-index --find-links=/wheels -r requirements/dev.txt \
    && rm -rf /wheels

COPY --chown=app:app . /app

USER app

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
    CMD curl --fail --silent http://127.0.0.1:${PORT}/healthz/ || exit 1

CMD ["python", "src/manage.py", "runserver", "0.0.0.0:8000"]
