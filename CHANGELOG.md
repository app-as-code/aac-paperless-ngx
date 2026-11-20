# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed

- Upstream image tracks Paperless-ngx v3.4.0 as defined in `service.yml`.

## [0.0.1]

### Added

- Initial App-as-Code service definition for Paperless-ngx with PostgreSQL 17 and Redis 8.6.1 dependencies.
- Optional Paperless-AI sidecar (`admonstrator/paperless-ai-next`) with its own volume.
- Extended healthcheck `start_period` to cover one-time upgrade migrations on first start.
- README with configuration reference and project icon.
