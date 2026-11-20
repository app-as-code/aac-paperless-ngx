<p align="center">
  <img src="assets/icon.svg" alt="Paperless-ngx logo" width="128">
</p>

<h1 align="center">aac-paperless-ngx</h1>

<p align="center">Self-hosted deployment definition for Paperless-ngx, a document management system with OCR.</p>

<p align="center">
  <a href="https://github.com/paperless-ngx/paperless-ngx"><img alt="Upstream" src="https://img.shields.io/badge/upstream-paperless--ngx%2Fpaperless--ngx-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-ghcr.io%2Fpaperless--ngx%2Fpaperless--ngx%3A3.4.0-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/paperless-ngx/paperless-ngx/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-GPL--3.0-green"></a>
</p>

## About

Paperless-ngx turns scanned and digital documents into a searchable archive. Incoming files
are processed with OCR, so their text can be searched, and they are enriched with tags,
correspondents, document types and dates. Documents can be added by uploading them, by
dropping them into a watched folder, or through email import, and they are kept in a
single web interface.

Key features (as provided by upstream):

- OCR of scanned documents with full-text search
- Tags, correspondents, document types and custom fields for organization
- Automatic matching and classification rules for incoming documents
- Watched consume folder and email import
- Multi-user support with permissions
- REST API and web interface

## Upstream

- Project: <https://github.com/paperless-ngx/paperless-ngx>
- Documentation: <https://docs.paperless-ngx.com>
- Container image: `ghcr.io/paperless-ngx/paperless-ngx:3.4.0`
- Database: `postgres:17` (deployed as `aac-paperless-db`)
- Cache / queue: `redis:8.6.1` (deployed as `aac-paperless-redis`)
- Optional sidecar: `admonstrator/paperless-ai-next:v2026.10.03` (deployed as `aac-paperless-ai`), an AI classifier interface for Paperless-ngx
- License: GPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external network used for routing must exist first:

```bash
docker network create services-secured

# Provide the values listed under "Configuration" in .env / stack.env
docker compose up -d
```

The first start can take several minutes, because Paperless-ngx runs one-time upgrade
migrations before the web server becomes available. The healthcheck therefore allows a
`start_period` of 300 seconds.

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the `.env` and `stack.env` files yourself (see below). Do not commit them.

## Configuration

Environment variables of the `aac-paperless` service (defaults from `service.yml`;
`CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `PAPERLESS_URL` | `CHANGE_ME` | Public URL of the Paperless-ngx instance |
| `PAPERLESS_SECRET_KEY` | `CHANGE_ME` (secret) | Django secret key |
| `PAPERLESS_PASSPHRASE` | `CHANGE_ME` (secret) | Passphrase used for document encryption |
| `PAPERLESS_ADMIN_USER` | `admin` (secret) | Initial administrator user name |
| `PAPERLESS_ADMIN_PASSWORD` | `CHANGE_ME` (secret) | Initial administrator password |
| `PAPERLESS_DBPASS` | `CHANGE_ME` (secret) | Database password for the Paperless-ngx user |
| `PAPERLESS_DBENGINE` | `postgresql` | Database engine |
| `PAPERLESS_DBHOST` | `aac-paperless-db` | Database host (dependency container) |
| `PAPERLESS_DBNAME` | `paperless` | Database name |
| `PAPERLESS_DBUSER` | `paperless` | Database user |
| `PAPERLESS_DBPORT` | `5432` | Database port |
| `PAPERLESS_REDIS` | `redis://aac-paperless-redis:6379` | Redis connection URL |
| `PAPERLESS_APPS` | `CHANGE_ME` | Optional list of apps to enable |
| `PAPERLESS_OCR_LANGUAGE` | `deu` | OCR language(s) |
| `PAPERLESS_TIME_ZONE` | `Europe/Berlin` | Time zone used by Paperless-ngx |
| `PAPERLESS_CONSUMER_POLLING` | `10` | Polling interval (seconds) of the consume folder |
| `PAPERLESS_CONSUMER_DELETE_DUPLICATES` | `true` | Delete duplicate files found in the consume folder |
| `PAPERLESS_CONSUMER_SUBDIRS_AS_TAGS` | `true` | Use consume subfolder names as tags |
| `PAPERLESS_PORT` | `8000` | Port the web server listens on inside the container |
| `PAPERLESS_AI_TOKEN` | `CHANGE_ME` (secret) | Token used by the Paperless-AI sidecar |
| `PAPERLESS_TOKEN` | `CHANGE_ME` (secret) | API token used by the Paperless-AI sidecar |
| `PAPERLESS_SOCIALACCOUNT_PROVIDERS` | `CHANGE_ME` (secret) | Social login provider configuration |
| `HOMEPAGE_WIDGET_KEY` | `CHANGE_ME` (secret) | API key for the Homepage dashboard widget |
| `POSTGRES_USER` / `POSTGRES_DB` | `paperless` | Database user and name of the `postgres` dependency |
| `POSTGRES_PASSWORD` | `CHANGE_ME` (secret) | Password of the `postgres` dependency |
| `PUID` / `PGID` | `1000` / `2000` | User and group ID the containers run as |
| `TZ` | `Europe/Berlin` | Time zone |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `8000` | TCP | Web interface (`web`), published as `8190` by default |

Volumes:

| Mount | Description |
|---|---|
| `data` -> `/usr/src/paperless/data` | Paperless-ngx application data |
| `paperless_media` -> `/usr/src/paperless/media` | Archived and original document files (bind mount) |
| `paperless_import` -> `/usr/src/paperless/consume` | Consume folder for incoming documents (bind mount) |
| `paperless_export` -> `/usr/src/paperless/export` | Export folder (bind mount) |
| `paperless_saved` -> `/usr/src/paperless/saved` | Saved files (bind mount) |
| `redis` -> `/data` | Redis data (redis dependency) |
| `db` -> `/var/lib/postgresql/data` | PostgreSQL data (database dependency) |
| `ai_data` -> `/app/data` | Data of the Paperless-AI sidecar |

The bind-mount source paths are set per deployment in `service.yml`.

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-paperless-ngx>.

## Credits and license

Paperless-ngx is developed by the Paperless-ngx project and licensed under GPL-3.0. This
repository only contains the deployment definition for it and is not a fork of the
application. The logo is provided by the Paperless-ngx project; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
