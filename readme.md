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

# OAuth2 keypair — outputs single-line values safe for .env files and any env UI
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out /tmp/passport_private.pem 2>/dev/null
openssl rsa -in /tmp/passport_private.pem -pubout -out /tmp/passport_public.pem 2>/dev/null
echo "PASSPORT_PRIVATE_KEY:"; awk '{printf "%s\\n", $0}' /tmp/passport_private.pem; echo ""
echo "PASSPORT_PUBLIC_KEY:";  awk '{printf "%s\\n", $0}' /tmp/passport_public.pem;  echo ""
rm /tmp/passport_private.pem /tmp/passport_public.pem
```

The `awk` command collapses the PEM key into a single line with literal `\n` escape sequences. Paste each output value directly into your `.env` file or env var UI — no quoting or reformatting needed.

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

### 5. Run database migrations (first run only)

The app does not run migrations automatically. After the containers are healthy, initialize the database schema:

```bash
docker exec solidtime-app php artisan migrate --force
```

In Komodo, use the **Exec** terminal on the `solidtime-app` container and run the same command.

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
