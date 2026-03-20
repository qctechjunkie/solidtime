# Solidtime

Self-hosted time tracking with Docker Compose and PostgreSQL.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/) with the Compose plugin
- A reverse proxy (Nginx, Traefik, Caddy) for production deployments

## Installation

### 1. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and set all required values. Pay special attention to:

- `DOCKER_DATA_PATH` — absolute path on the host where all data will be stored
- `APP_URL` — your public-facing domain (e.g. `https://solidtime.example.com`)
- `DB_PASSWORD` — generate with `openssl rand -hex 32`

### 2. Generate required secrets

These must be generated before the first start. All commands require only `openssl`, available on any Linux system:

```bash
# Application encryption key — copy output to APP_KEY
echo "base64:$(openssl rand -base64 32)"

# OAuth2 keypair — copy each file's output to PASSPORT_PRIVATE_KEY / PASSPORT_PUBLIC_KEY
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out passport_private.pem 2>/dev/null
openssl rsa -in passport_private.pem -pubout -out passport_public.pem 2>/dev/null
cat passport_private.pem
cat passport_public.pem
rm passport_private.pem passport_public.pem
```

**In `.env`**, PEM values must be wrapped in double quotes:

```dotenv
PASSPORT_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----
MIIEvQ...
-----END PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
MIIBIj...
-----END PUBLIC KEY-----"
```

**In Komodo** (or any env var UI), paste the raw PEM output directly — no quoting needed.

### 3. Prepare data directories (Linux only)

The app containers run as UID/GID 1000. The `logs` and `app-storage` directories
must be owned by that user before starting:

```bash
mkdir -p ${DOCKER_DATA_PATH}/{postgres,logs,app-storage}
chown -R 1000:1000 ${DOCKER_DATA_PATH}/{logs,app-storage}
```

### 4. Start Solidtime

```bash
docker compose up -d
```

Solidtime will be available on the port defined by `PORT` in your `.env` (default: `8000`).

## Upgrading

```bash
docker compose pull
docker compose up -d
```

## Database access from the host

To connect to the PostgreSQL database using a local tool (e.g. pgAdmin, DBeaver),
uncomment the `ports` block in the `database` service in `compose.yaml` and set
`FORWARD_DB_PORT` in your `.env`:

```dotenv
FORWARD_DB_PORT=5432
```

> [!CAUTION]
> Only expose the database port on trusted networks. Never open it to the internet.

## Further reading

- [Self-Hosting Guide](https://docs.solidtime.io/self-hosting/guides/docker)
- [Configuration Reference](https://docs.solidtime.io/self-hosting/configuration)
