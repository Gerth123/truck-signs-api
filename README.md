# Truck Signs API

A Django REST Framework backend for managing truck sign and vinyl products, containerized with Docker Compose and PostgreSQL.

## Table of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Steps](#steps)
- [Project Goal](#project-goal)
- [Usage](#usage)
  - [Configuration](#configuration)
  - [How to Build the Image](#how-to-build-the-image)
  - [Running a Single Container Manually](#running-a-single-container-manually)
- [Features](#features)
- [Environment Variables](#environment-variables)
- [CI/CD](#cicd)
- [Additional Files](#additional-files)

## Quickstart

### Prerequisites

- Docker and Docker Compose installed
- Git

### Steps

1. Clone the repository:
```bash
   git clone https://github.com/Gerth123/truck-signs-api.git
   cd truck-signs-api
```
2. Copy the example environment file and fill in your own values:
```bash
   cp example.env .env
```
3. Build the application image (see [How to Build the Image](#how-to-build-the-image) below).
4. Start the stack:
```bash
   docker compose up
```
5. Open the admin panel at `http://localhost:8020/admin/` and log in with the superuser credentials configured in your `.env`.

## Project Goal

This repository provides the backend API for managing truck sign and vinyl products. It is built with Django and Django REST Framework, uses PostgreSQL for persistent storage, and Cloudinary for media file storage. The project is containerized using Docker and orchestrated with Docker Compose for a reproducible, production-ready setup.

## Usage

### Configuration

The application is configured entirely through environment variables, loaded from a `.env` file at the repository root (see [Environment Variables](#environment-variables)). The `MODE` variable controls the environment:

- `MODE=dev` uses SQLite and enables Django's debug mode, intended for local development without containers.
- `MODE=prod` uses PostgreSQL and disables debug mode, intended for the Docker Compose setup.

Both the `backend` and `db` services in `docker-compose.yml` read their configuration from the same `.env` file via `env_file`. The `DB_HOST` must match the service name of the database in `docker-compose.yml` (`db`), not `localhost`.

### How to Build the Image

Build the backend image from the repository root:

```bash
docker build -t truck-signs-api:latest .
```

`docker-compose.yml` references this image by name rather than building it inline, so this step must be run before `docker compose up`.

### Running a Single Container Manually

To run the backend container by itself (for example, against an already running database), replace the placeholders below with your own values:

```bash
docker run -d \
  --name truck-signs-api \
  --network truck-signs-api_default \
  -p 8020:8000 \
  -e MODE=prod \
  -e SECRET_KEY=<your-secret-key> \
  -e DB_NAME=<your-db-name> \
  -e DB_USER=<your-db-user> \
  -e DB_PASSWORD=<your-db-password> \
  -e DB_HOST=db \
  -e DB_PORT=5432 \
  -e DJANGO_SUPERUSER_USERNAME=<your-admin-username> \
  -e DJANGO_SUPERUSER_EMAIL=<your-admin-email> \
  -e DJANGO_SUPERUSER_PASSWORD=<your-admin-password> \
  truck-signs-api:latest
```

## Features

- REST API for managing truck sign and vinyl products, built with Django REST Framework
- PostgreSQL persistence in production, with data surviving container restarts via a named Docker volume
- Media file storage via Cloudinary, configurable through environment variables
- Static file serving via WhiteNoise
- CORS configuration for frontend integration
- Idempotent superuser creation on container startup, safe to run on repeated deployments

## Environment Variables

| Variable | Description |
|---|---|
| `MODE` | `dev` (SQLite, debug on) or `prod` (PostgreSQL, debug off) |
| `DEBUG_ENABLED` | Overrides debug mode explicitly if needed |
| `SECRET_KEY` | Django secret key |
| `ALLOWED_HOSTS` | Comma-separated list of allowed hosts |
| `DB_NAME` | PostgreSQL database name (Django-side) |
| `DB_USER` | PostgreSQL user (Django-side) |
| `DB_PASSWORD` | PostgreSQL password (Django-side) |
| `DB_HOST` | Database host, must be `db` when using Docker Compose |
| `DB_PORT` | Database port, default `5432` |
| `POSTGRES_DB` | Database name (used by the official Postgres image on first init) |
| `POSTGRES_USER` | Database user (used by the official Postgres image on first init) |
| `POSTGRES_PASSWORD` | Database password (used by the official Postgres image on first init) |
| `DJANGO_SUPERUSER_USERNAME` | Username for the automatically created superuser |
| `DJANGO_SUPERUSER_EMAIL` | Email for the automatically created superuser |
| `DJANGO_SUPERUSER_PASSWORD` | Password for the automatically created superuser |
| `CLOUD_NAME` | Cloudinary cloud name |
| `CLOUD_API_KEY` | Cloudinary API key |
| `CLOUD_API_SECRET` | Cloudinary API secret |
| `CORS_ALLOWED_ORIGINS` | Comma-separated list of allowed CORS origins |
| `EMAIL_HOST_USER` | SMTP username for outgoing email |
| `EMAIL_HOST_PASSWORD` | SMTP password for outgoing email |

`POSTGRES_DB`, `POSTGRES_USER`, and `POSTGRES_PASSWORD` must match `DB_NAME`, `DB_USER`, and `DB_PASSWORD` respectively, since the official Postgres image uses the former to initialize the database while Django connects using the latter.

## CI/CD

The repository includes three GitHub Actions workflows:

- **Lint and Test Python Code Base**: runs `flake8`, `black`, and `isort` checks, then runs the Django test suite, on pushes and pull requests affecting Python files.
- **Build Application**: builds and pushes the Docker image to GitHub Container Registry on tag pushes.
- **Check open PR for feature branch**: verifies that an open pull request exists for the current branch.

## Additional Files

- `.gitattributes`: enforces LF line endings for `entrypoint.sh` to avoid shebang failures on Windows checkouts.
- `example.env`: template listing all environment variables required to run the project; copy to `.env` and fill in your own values.