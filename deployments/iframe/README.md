# Iframe Deployment

This directory contains a standalone deployment configuration for the Iframe service.

## Services

- **iframe**: Iframe service for secure key operations

## Quick Start

1. **Copy the environment file:**
   ```bash
   cp .env.example .env
   ```

2. **Update `.env` with your configuration** (if needed)

3. **Deploy:**
   ```bash
   make run-iframe
   ```

   Or manually:
   ```bash
   docker compose --project-directory . -f deployments/iframe/docker-compose.yml up -d
   ```

## Configuration

See `.env.example` for all available environment variables. Copy it to `.env` and customize as needed. Key variables:

- `IFRAME_CONTAINER_PORT`: Container port (default: `5173`)
- `IFRAME_HOST_PORT`: Host port (default: `7050`)

## Ports

- Iframe: `7050` (configurable via `IFRAME_HOST_PORT`)


## Dependencies

The iframe service depends on `auth_service`, `hot_storage`, and `cold_storage` being available on the network. When running standalone, ensure these services are also running and accessible.

## Stopping

```bash
make stop-iframe
```

Or:
```bash
docker compose --project-directory . -f deployments/iframe/docker-compose.yml down
```

