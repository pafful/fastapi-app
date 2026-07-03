# FROM python:3.10

# ENV PYTHONUNBUFFERED=1

# # Install uv
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#installing-uv
# COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

# # Compile bytecode
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#compiling-bytecode
# ENV UV_COMPILE_BYTECODE=1

# # uv Cache
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#caching
# ENV UV_LINK_MODE=copy

# WORKDIR /app/

# # Place executables in the environment at the front of the path
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#using-the-environment
# ENV PATH="/app/.venv/bin:$PATH"

# # Install dependencies
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#intermediate-layers
# RUN --mount=type=cache,target=/root/.cache/uv \
#     --mount=type=bind,source=uv.lock,target=uv.lock \
#     --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
#     uv sync --frozen --no-install-workspace --package app

# COPY ./backend/scripts /app/backend/scripts

# COPY ./backend/pyproject.toml ./backend/alembic.ini /app/backend/

# COPY ./backend/app /app/backend/app

# # Sync the project
# # Ref: https://docs.astral.sh/uv/guides/integration/docker/#intermediate-layers
# RUN --mount=type=cache,target=/root/.cache/uv \
#     --mount=type=bind,source=uv.lock,target=uv.lock \
#     --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
#     uv sync --frozen --package app

# WORKDIR /app/backend/

# # CMD ["fastapi", "run", "--workers", "4", "app/main.py"]
# ENV PYTHONPATH=/app/backend
# CMD ["uvicorn", "backend.app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]


###############################################SECOND TEST V3###########################################

FROM python:3.14

ENV PYTHONUNBUFFERED=1

# Install uv
COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

ENV UV_COMPILE_BYTECODE=1
ENV UV_LINK_MODE=copy

WORKDIR /app

# IMPORTANT: fix imports
ENV PYTHONPATH=/app

# Add venv to PATH
ENV PATH="/app/.venv/bin:$PATH"

# Install dependencies first (cached layer)
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-workspace --package app

# Copy backend code
COPY ./backend/scripts /app/backend/scripts
COPY ./backend/pyproject.toml ./backend/alembic.ini /app/backend/
COPY ./backend/app /app/backend/app

# Sync again (final install with app code)
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --package app

WORKDIR /app
ENV PYTHONPATH=/app

# IMPORTANT FIX: use 1 worker first (stability)
CMD ["uvicorn", "backend.app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]

##############IMAGE V4: FINAL WORKING VERSION WITH UV##################
# FROM python:3.10

# ENV PYTHONUNBUFFERED=1

# COPY --from=ghcr.io/astral-sh/uv:0.9.26 /uv /uvx /bin/

# ENV UV_COMPILE_BYTECODE=1
# ENV UV_LINK_MODE=copy

# WORKDIR /app

# # IMPORTANT: makes "app" importable
# ENV PYTHONPATH=/app/backend

# ENV PATH="/app/.venv/bin:$PATH"

# # --------------------
# # Copy dependency files FIRST (better caching)
# # --------------------
# COPY uv.lock pyproject.toml /app/

# COPY backend/pyproject.toml backend/alembic.ini /app/backend/

# # --------------------
# # Copy application code
# # --------------------
# COPY backend/app /app/backend/app
# COPY backend/scripts /app/backend/scripts

# # --------------------
# # Install dependencies
# # --------------------
# RUN --mount=type=cache,target=/root/.cache/uv \
#     uv sync --frozen --no-install-workspace --package app

# # --------------------
# # Runtime
# # --------------------
# CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1"]

