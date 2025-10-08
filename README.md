# 🚀 n8n — Local Docker Setup

Production-grade, convention-compliant local setup for n8n using Docker Compose.
Follow this README.md to run n8n locally with persistent data, secure defaults, and optional PostgreSQL support for scale.

---

## 📌 Prerequisites

Before starting, ensure you have the following installed:

- [Docker](https://www.docker.com/get-started) (latest version)
- [Docker Compose](https://docs.docker.com/compose/install/)
- A terminal or command-line environment

## 📁 Repository layout

```
n8n-docker/
├── .env
├── docker-compose.yml
└── data/                 # persistent storage (add to .gitignore)

```

## ⚙️ Configure environment

Create a `.env` file in the project root. Do not commit this file to version control.

```bash
# n8n environment configuration
GENERIC_TIMEZONE=Asia/Manila
N8N_PORT=5678
N8N_PROTOCOL=http

# Generate a secure key: openssl rand -base64 32
N8N_ENCRYPTION_KEY=yourStrongEncryptionKeyHere

# Basic Auth (recommended for local usage)
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=superSecurePassword123

# Host / webhook config
WEBHOOK_URL=http://localhost:5678/
N8N_HOST=n8n.local

# Persistent path inside the container
DATA_FOLDER=/home/node/.n8n

# Execution mode (main | queue)
EXECUTIONS_PROCESS=main

```

Generate a secure encryption key

```bash
openssl rand -base64 32
```

Paste the result into `N8N_ENCRYPTION_KEY`.

---

## 🐳 docker-compose.yml

Create ```docker-compose.yml``` in the project root:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "${N8N_PORT}:5678"
    environment:
      GENERIC_TIMEZONE: ${GENERIC_TIMEZONE}
      N8N_PROTOCOL: ${N8N_PROTOCOL}
      N8N_ENCRYPTION_KEY: ${N8N_ENCRYPTION_KEY}
      N8N_BASIC_AUTH_ACTIVE: ${N8N_BASIC_AUTH_ACTIVE}
      N8N_BASIC_AUTH_USER: ${N8N_BASIC_AUTH_USER}
      N8N_BASIC_AUTH_PASSWORD: ${N8N_BASIC_AUTH_PASSWORD}
      WEBHOOK_URL: ${WEBHOOK_URL}
      N8N_HOST: ${N8N_HOST}
      EXECUTIONS_PROCESS: ${EXECUTIONS_PROCESS}
    volumes:
      - ./data:${DATA_FOLDER}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5678/"]
      interval: 30s
      timeout: 10s
      retries: 3
```
```
🔒 Tip: For predictable behavior in production, pin the image to a specific version (for example docker.n8n.io/n8nio/n8n:1.###.#) rather than :latest.
```

## 🧩 Optional — PostgreSQL (recommended for multi-user / production-like setups)

Add the `postgres` service and set DB environment variables for `n8n`:

```yaml
services:
  postgres:
    image: postgres:15
    container_name: n8n-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: strong_pg_password
      POSTGRES_DB: n8n
    volumes:
      - ./data/postgres:/var/lib/postgresql/data

  n8n:
    # ...
    depends_on:
      - postgres
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: n8n
      DB_POSTGRESDB_PASSWORD: strong_pg_password
```

## 🚀 Common commands

Start in detached mode:

```docker compose up -d```

Stop and remove containers:

```docker compose down```

View container logs (follow):

```docker compose logs -f n8n```


Pull latest images (careful with ```:latest``` in production):

```docker compose pull```

## 🌐 Access the n8n editor

Open your browser:

```http://localhost:5678```


Log in using Basic Auth if enabled (```N8N_BASIC_AUTH_USER``` / ```N8N_BASIC_AUTH_PASSWORD```).

For exposing webhooks externally during development, use a tunnel (e.g., ```ngrok```) or configure a reverse proxy with HTTPS (see Traefik section).

## 💾 Persistence & Git

- Persistent data is stored in ```./data```. This folder keeps workflows, credentials, and execution data.

- Add ```data/``` to your ```.gitignore```:

```bash
/data/
.env
```

## 🔐 Security notes

- Keep ```.env``` out of version control.

- Use a strong ```N8N_ENCRYPTION_KEY```.

- Use HTTPS and a reverse proxy (Traefik / Nginx) when exposing n8n publicly.

- Rotate credentials and keys periodically.

## ⚠️ Troubleshooting

- Container not starting: run docker compose logs n8n and check for errors (port conflicts, invalid env vars).

- Healthcheck failing: ensure curl is present in the container or adjust the healthcheck to an appropriate endpoint.

- Webhook receiving errors: confirm WEBHOOK_URL is correct and reachable by the service that will call the webhook.

## 🔁 Upgrading

Pull newer images: ```docker compose pull```

Restart: ```docker compose up -d```

For DB-backed setups, apply any DB migration steps required by the n8n release notes.

## 🔧 Example .gitignore

```gitignore
# Local n8n data and environment
/data/
/.env
```