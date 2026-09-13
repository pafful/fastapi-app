# Full Stack FastAPI Application

A production-oriented full-stack application built with FastAPI, React, PostgreSQL, Docker Compose, Traefik, and GitHub Actions. This repository provides a modern web application template with authentication, user management, item CRUD, email workflows, local containerized development, and deployment automation for staging and production environments.

## Executive Summary

This project combines a Python API backend and a TypeScript frontend into a single application stack that is easy to develop locally and deploy in containerized environments. The backend is powered by FastAPI and SQLModel, while the frontend is a Vite + React + TypeScript application with a generated API client. PostgreSQL handles persistence, Docker Compose manages local orchestration, and Traefik exposes the services through DNS-based routing.

The repository also contains deployment automation for AWS ECR and Docker image publishing, plus Kubernetes/OpenShift manifests for runtime deployment. This makes it suitable for both internal platform workflows and public GitHub-based release management.

## Problem Statement

Teams often need a starter application that includes:

- Secure authentication and user lifecycle workflows
- API-first backend development with OpenAPI and Swagger
- Modern frontend UX with component-based architecture
- Database persistence and migrations
- Local development parity with production deployment patterns
- Containerized delivery and environment-based configuration
- CI/CD automation for testing, security, and deployment

This repository addresses those needs by providing a complete stack with sensible defaults, but still keeps the configuration flexible enough for custom deployments.

## Architecture Diagram

```mermaid
flowchart LR
    User[Browser / Admin User] --> FE[React Frontend\nVite + TypeScript]
    FE --> API[FastAPI Backend\nREST API / JWT Auth]
    API --> DB[(PostgreSQL)]
    API --> SMTP[SMTP / Mailcatcher]
    API --> Sentry[Sentry (optional)]

    subgraph LocalOrProd[Container Platform]
        Traefik[Traefik Reverse Proxy]
        FE
        API
        DB
    end

    User --> Traefik
    Traefik --> FE
    Traefik --> API

    GH[GitHub Actions] --> ECR[AWS ECR]
    ECR --> K8S[Kubernetes / OpenShift]
    K8S --> FE
    K8S --> API
```

## Folder Structure

```text
.
├── .env                              # Base environment variables for local/dev runtime
├── .github/
│   └── workflows/                   # CI/CD automation for tests, build, deploy
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── api/
│   │   │   ├── deps.py
│   │   │   ├── main.py
│   │   │   └── routes/
│   │   │       ├── items.py
│   │   │       ├── login.py
│   │   │       ├── private.py
│   │   │       ├── users.py
│   │   │       └── utils.py
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── db.py
│   │   │   └── security.py
│   │   ├── crud.py
│   │   ├── email-templates/
│   │   ├── initial_data.py
│   │   ├── main.py
│   │   ├── models.py
│   │   ├── tests_pre_start.py
│   │   └── utils.py
│   ├── alembic/
│   ├── tests/
│   ├── Dockerfile
│   ├── pyproject.toml
│   ├── README.md
│   └── scripts/
├── frontend/
│   ├── src/
│   ├── public/
│   ├── tests/
│   ├── Dockerfile
│   ├── package.json
│   ├── vite.config.ts
│   └── README.md
├── openshift/
│   ├── base/
│   │   ├── backend.yaml
│   │   ├── frontend.yaml
│   │   ├── kustomization.yaml
│   │   ├── namespace.yaml
│   │   ├── postgres.yaml
│   │   └── route.yaml
│   └── tekton/
│       └── rbac.yaml
├── compose.yml
├── compose.override.yml
├── compose.traefik.yml
├── deployment.md
├── development.md
├── README.md
├── release-notes.md
├── scripts/
├── hooks/
├── LICENSE
├── package.json
├── pyproject.toml
└── sonar-project.properties
```

## Technologies

### Backend

- Python 3.10+
- FastAPI
- SQLModel
- Pydantic v2
- PostgreSQL
- Alembic for migrations
- JWT-based authentication
- SMTP email integration
- Sentry support

### Frontend

- React 19
- Vite
- TypeScript
- TanStack Query
- TanStack Router
- Tailwind CSS
- shadcn-inspired UI patterns
- Playwright for end-to-end testing

### DevOps & Infrastructure

- Docker
- Docker Compose
- Traefik
- AWS ECR
- GitHub Actions
- Kubernetes / OpenShift manifests
- Adminer for database administration
- MailCatcher for local email testing

## Local Development

### Prerequisites

Ensure these tools are installed:

- Docker
- Docker Compose
- Python 3.10+
- Node.js / Bun (depending on your preferred frontend workflow)

### Starting the stack

From the repository root:

```bash
docker compose watch
```

This starts the application stack with the default local development setup.

### Access URLs

- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- Adminer: http://localhost:8080
- Traefik UI: http://localhost:8090
- MailCatcher: http://localhost:1080

### Running backend locally

```bash
cd backend
fastapi dev app/main.py
```

### Running frontend locally

```bash
cd frontend
bun run dev
```

### Testing backend

```bash
cd backend
uv run bash scripts/tests-start.sh
```

### Working with database migrations

The project includes Alembic configuration for migration management.

## Docker

The project uses Docker to package both application services and supporting infrastructure.

### Backend image

The backend Dockerfile builds the FastAPI service with the Python environment and application code.

### Frontend image

The frontend Dockerfile builds the Vite application and exposes the final static site via Nginx.

### Production deployment behavior

When deployed with Docker Compose, the stack includes:

- PostgreSQL database
- Backend API service
- Frontend web app
- Traefik routing layer
- MailCatcher in local development
- Adminer for database inspection

## Docker Compose

The repository includes several Compose definitions:

- `compose.yml` — base application stack
- `compose.override.yml` — development overrides
- `compose.traefik.yml` — standalone Traefik reverse proxy setup for public deployments

### Example local stack startup

```bash
docker compose up -d db mailcatcher
docker compose up -d --build
```

### Observed service topology in this repo

The Compose file configures:

- `db` using PostgreSQL image
- `backend` and `prestart` tasks
- `frontend` app behind Traefik
- `adminer` for DB access
- `traefik-public` network for public routing

## AWS ECR

The GitHub workflow at `.github/workflows/build.yml` demonstrates automated publishing to Amazon ECR.

### Workflow behavior

The pipeline:

1. Checks out source code
2. Resolves an image version from Git tag or backend package version
3. Authenticates to AWS using GitHub OIDC or stored AWS secrets
4. Logs into Amazon ECR
5. Builds backend and frontend images using Docker Buildx
6. Pushes images to ECR with both versioned and commit SHA tags

### Example ECR tags used by the repo

- `fullstack-fastapi-backend:<version>`
- `fullstack-fastapi-frontend:<version>`

### Required GitHub variables and secrets

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `ECR_REGISTRY`

## Kubernetes Deployment

The repository contains Kubernetes/OpenShift manifests under `openshift/base/` and pipeline-level access configuration under `openshift/tekton/`.

### Included manifests

OpenShift base manifests:

- `backend.yaml`
- `frontend.yaml`
- `postgres.yaml`
- `namespace.yaml`
- `route.yaml`
- `kustomization.yaml`

Tekton resources:

- `tekton/rbac.yaml`

### Deployment model

These manifests define:

- Namespace-scoped workloads
- Backend Deployment with env vars and health checks
- Frontend Deployment
- PostgreSQL service and secret-based configuration
- Exposed routes and service networking

### Example deployment notes

The OpenShift manifests use container images such as:

```text
image-registry.openshift-image-registry.svc:5000/fastapi-demo/fastapi-backend:initial
image-registry.openshift-image-registry.svc:5000/fastapi-demo/fastapi-frontend:initial
```

This pattern is appropriate for private registries or OpenShift-integrated image pipelines.

## CI/CD with GitHub Actions

The repository includes a set of workflows in `.github/workflows/`.

### Included workflows

- `build.yml` — build and push Docker images to ECR
- `test-backend.yml` — run backend tests and coverage checks
- `deploy-production.yml` — deploy with Docker Compose to a self-hosted runner
- `test-docker-compose.yml` — compose-level validation
- `security.yml` — security scanning
- `playwright.yml` — frontend/browser validation
- `pre-commit.yml` and `guard-dependencies.yml` — repository quality controls

### CI pattern

This stack follows a standard DevOps flow:

1. Push or pull request triggers automation
2. Install dependencies and environment
3. Validate code quality and tests
4. Build container images
5. Publish images to registry
6. Deploy through self-hosted or environment-specific runners

## Configuration

Configuration is managed through environment variables and `.env` files.

The core application reads settings from `backend/app/core/config.py`, which loads values from the repository root `.env` file.

This allows a clean separation between:

- local defaults
- deployment-specific secrets
- environment-specific runtime configuration

## Environment Variables

The root `.env` file contains the main runtime configuration. The most relevant values include:

```env
DOMAIN=localhost
FRONTEND_HOST=http://localhost:5173
ENVIRONMENT=local
PROJECT_NAME="Full Stack FastAPI Project"
STACK_NAME=full-stack-fastapi-project

BACKEND_CORS_ORIGINS="http://localhost,http://localhost:5173,..."
SECRET_KEY=...
FIRST_SUPERUSER=pauli@admin.com
FIRST_SUPERUSER_PASSWORD=...

SMTP_HOST=
SMTP_USER=
SMTP_PASSWORD=
EMAILS_FROM_EMAIL=info@example.com
SMTP_TLS=True
SMTP_SSL=False
SMTP_PORT=587

POSTGRES_SERVER=db
POSTGRES_PORT=5432
POSTGRES_DB=app
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres123

SENTRY_DSN=
DOCKER_IMAGE_BACKEND=backend
DOCKER_IMAGE_FRONTEND=frontend
```

### Security guidance

- Never commit production secrets to Git
- Use GitHub Actions secrets or a secrets manager for deployment environments
- Rotate `SECRET_KEY`, `FIRST_SUPERUSER_PASSWORD`, and database credentials regularly
- Ensure `ENVIRONMENT` is set correctly for staging and production

## API Endpoints

The API is mounted under `/api/v1` in the application startup configuration.

### Authentication and user lifecycle

- `POST /api/v1/login/access-token`
- `POST /api/v1/login/test-token`
- `POST /api/v1/password-recovery/{email}`
- `POST /api/v1/reset-password/`
- `POST /api/v1/password-recovery-html-content/{email}` *(admin-only)*

### User management

- `GET /api/v1/users/` *(admin-only)*
- `POST /api/v1/users/` *(admin-only)*
- `GET /api/v1/users/me`
- `PATCH /api/v1/users/me`
- `PATCH /api/v1/users/me/password`
- `DELETE /api/v1/users/me`
- `POST /api/v1/users/signup`
- `GET /api/v1/users/{user_id}`
- `PATCH /api/v1/users/{user_id}`
- `DELETE /api/v1/users/{user_id}`

### Item management

- `GET /api/v1/items/`
- `POST /api/v1/items/`
- `GET /api/v1/items/{id}`
- `PUT /api/v1/items/{id}`
- `DELETE /api/v1/items/{id}`

### Utility endpoints

- `POST /api/v1/utils/test-email/` *(admin-only)*
- `GET /api/v1/utils/health-check/`

### Local-only private route

- `POST /api/v1/private/users/` *(only enabled when `ENVIRONMENT=local`)*

## Security Considerations

This repository includes a secure-by-default foundation, but production deployment requires hardening.

### Current security controls

- JWT authentication
- Password hashing with strong password utilities
- CORS configuration via environment variable
- Environment-based secret management
- Non-default secret enforcement in settings validation
- Optional Sentry integration
- Admin-only actions for elevated functions

### Recommended production hardening

- Use real secrets in GitHub Actions environment variables or Azure/AWS/KMS-backed secret stores
- Restrict CORS origins to approved domains
- Enable TLS termination at the edge (Traefik/Ingress)
- Use managed PostgreSQL or private networking
- Limit admin roles and enforce least privilege
- Add rate limiting, WAF policies, and threat monitoring for internet-facing endpoints
- Review and rotate SMTP credentials and secret keys regularly

## Monitoring

### Runtime health checks

The application exposes a health endpoint:

```bash
GET /api/v1/utils/health-check/
```

This is used in container health checks and readiness checks.

### Observability features

- Sentry DSN support via `SENTRY_DSN`
- Docker health checks on database and backend services
- Traefik dashboard for routing inspection
- GitHub Actions test and security reporting
- Container logs via `docker compose logs`

### Recommended enhancements

- Add Prometheus/Grafana metrics
- Add structured JSON logs
- Integrate centralized log aggregation
- Add request tracing and latency dashboards
- Monitor DB connections, CPU, and memory usage

## Troubleshooting

### Database not ready

If the backend fails to start, ensure PostgreSQL is healthy and the `.env` values match:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`

### Permission or auth issues

Verify:

- `SECRET_KEY` is set to a secure value
- `FIRST_SUPERUSER_PASSWORD` is not default placeholder material
- The frontend is pointing at the correct backend origin

### CORS errors

Check `BACKEND_CORS_ORIGINS` and ensure the frontend host and API host are included in the allowlist.

### Email flows fail locally

MailCatcher is available at http://localhost:1080. If email doesn’t arrive, confirm SMTP values and ensure the local container stack is running.

### Deployment issues

Common issues include:

- missing environment variables
- invalid DNS and wildcard cert configuration
- Traefik network not created
- GitHub self-hosted runner permissions
- registry authentication issues for AWS ECR

## Future Roadmap

The repository is a strong starting point, and the next roadmap steps could include:

1. Standardized production infrastructure as code using Terraform or Bicep
2. Kubernetes deployment for EKS/AKS with Helm charts
3. Better observability: Prometheus, Grafana, Loki, OpenTelemetry
4. Role-based access refinement and audit logging
5. Automated database backup, restore, and point-in-time recovery
6. Enhanced frontend UX and modular page-level architecture
7. Expanded API versioning and contract tests
8. Larger security posture improvements including WAF, network isolation, and secrets rotation automation

## Conclusion

This repository is a practical, production-conscious full-stack template for building and shipping web applications with Python, React, and PostgreSQL. It gives developers a solid local environment, production-inspired deployment patterns, and automation that reduces the friction of shipping secure, containerized software.

For platform and DevOps-oriented deployment work, the project already includes strong Docker, Traefik, and GitHub Actions foundations, with Kubernetes/OpenShift manifests available for cluster-based rollout.

---

For additional operational guidance, see:

- [development.md](./development.md)
- [deployment.md](./deployment.md)
- [backend/README.md](./backend/README.md)
- [frontend/README.md](./frontend/README.md)
- [release-notes.md](./release-notes.md)

## Development

General development docs: [development.md](./development.md).

This includes using Docker Compose, custom local domains, `.env` configurations, etc.

## Release Notes

Check the file [release-notes.md](./release-notes.md).

## License

The Full Stack FastAPI Template is licensed under the terms of the MIT license.
