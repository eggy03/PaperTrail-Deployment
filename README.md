# PaperTrail-Deployment
Docker-based deployment configuration for the PaperTrail Discord bot and its supporting services.

This repository provides a simple Compose setup for running PaperTrail locally or on a VPS with minimal configuration.

> [!NOTE]
> The provided Docker Compose configuration is designed for single-node deployments.
> It runs one instance of each service and is not intended for horizontally scaled environments.
> This setup is suitable for local hosting or small VPS deployments serving fewer than ~2500 Discord servers.

# Getting Started

## Step 1: Create an application in the Developer Portal

Log on to the [Discord Developer Portal](https://discord.com/developers/applications) and create an application.

The application can have any name, avatar, banner and description but the following scopes, permissions and intents are
needed
for it to work properly:

**Installation Contexts**

1) Guild Install

**Scopes**

1) applications.commands
2) bot

**Permissions**

1) Manage Server
2) Read Message History
3) Send Messages
4) Send Messages In Threads
5) View Audit Log
6) View Channels

**Privileged Gateway Intents**

1) Presence Intent
2) Server Members Intent
3) Message Content Intent

Don't forget to copy the bot `TOKEN` as it will be required in the next step.

If you have never created a bot before, follow this [visual guide](https://jda.wiki/using-jda/getting-started/#creating-a-discord-bot)

## Step 2: Set up the required environment variables

You will need the following environment variables :

| Variable      | Description                                               |
|---------------|-----------------------------------------------------------|
| `TOKEN`       | Discord application bot token (from the Developer Portal) |
| `DB_USERNAME` | Username for the PostgreSQL database                      |
| `DB_PASSWORD` | A strong password for the database user                   |


## Step 3: Deploying the services

### 3.1: Clone this repository

```shell
git clone https://github.com/eggy03/PaperTrail-Deployment.git
cd PaperTrail-Deployment
```

### 3.2: Create a `.env` file having the required variables in the repository root

Example `.env`

```dotenv
TOKEN=my-token
DB_USERNAME=admin
DB_PASSWORD=admin@123
```

> [!CAUTION]
> Never commit your .env file to version control.

### 3.3: Start the services

```shell
#without metrics
docker compose -f compose-base.yml up -d
```
OR

```shell
#with metrics
docker compose -f compose-base.yml -f compose-insight.yml up -d
```

## Step 4: Test your deployment

If the deployment was successful:

- Invite the bot to a Discord server.
- Confirm the bot appears online.
- Run the `/setup` slash command.

This command will guide you through the initial configuration for your server.

# Compose Variants

## 1: Compose Base

`compose-base.yml` contains the core services required to run PaperTrail.

Services included:

| Service                 | Description           | Internal Endpoint                       |
|-------------------------|-----------------------|-----------------------------------------|
| PostgreSQL v18          | Core database service | `postgresql://postgres:5432/papertrail` |
| Valkey v9               | Core cache service    | `redis://valkey:6379`                   |
| PaperTrail API (latest) | Core API service      | `http://papertrail-api:8080`            |
| PaperTrail Bot (latest) | Core Bot service      | N/A                                     |

These services communicate over the **internal Docker network** and are **not exposed to the host machine via ports**.

## 2: Compose Insight

`compose-insight.yml` extends `compose-base.yml` with optional services
that provide **observability and debugging tools** for containers running in the PaperTrail network.

Services included:

| Service       | Description                                | URL                   |
|---------------|--------------------------------------------|-----------------------|
| PgAdmin       | Web UI for viewing and managing PostgreSQL | http://127.0.0.1:5050 |
| Redis Insight | UI for inspecting Valkey/Redis data        | http://127.0.0.1:5540 |
| Dozzle        | UI for viewing container logs and activity | http://127.0.0.1:9090 |

These services **publish ports to the host machine**, making them accessible through the URLs above.

> [!NOTE]
> `compose-insight.yml` requires `compose-base.yml` and cannot be run independently.

# Configuring Memory Limits

By default, each service has a memory limit of 512 MB.

You can override this limit by modifying the `mem_limit` parameter in the Docker Compose files.

Example:
```yaml
mem_limit: 1g
```

# License

[MIT](/LICENSE)