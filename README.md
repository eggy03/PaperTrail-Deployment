# PaperTrail-Deployment
Docker-based deployment configuration for the PaperTrail Discord bot and its supporting services.

This repository provides a simple Compose setup for running PaperTrail locally or on a VPS with minimal configuration.

> [!NOTE]
> The provided Docker Compose configuration is designed for single-node deployments.
> It runs one instance of each service and is not intended for horizontally scaled environments.
> This setup is suitable for local hosting or small VPS deployments serving fewer than ~2500 Discord servers.

# Latest Releases

[![Latest API Release](https://img.shields.io/github/v/release/eggy03/PaperTrail-API-Quarkus?sort=date&display_name=tag&style=for-the-badge&label=PaperTrail%20API)](https://github.com/eggy03/PaperTrail-API-Quarkus/pkgs/container/papertrail-api)
[![Latest Bot Release](https://img.shields.io/github/v/release/eggy03/PaperTrailBot?sort=date&display_name=tag&style=for-the-badge&label=PaperTrail%20Bot)](https://github.com/eggy03/PaperTrailBot/pkgs/container/papertrail-bot)

# Creating The Bot

## Step 1: Create an application in the Developer Portal

Log on to the [Discord Developer Portal](https://discord.com/developers/applications) and create an application.

The application can have any name, avatar, banner, or description.
However, the following scopes, permissions, and intents are required:

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

| Variable         | Description                                               |
|------------------|-----------------------------------------------------------|
| `TOKEN`          | Discord application bot token (from the Developer Portal) |
| `DB_NAME`        | A name for your database                                  |
| `DB_USERNAME`    | A username for your database                              |
| `DB_PASSWORD`    | A strong password for the database user                   |
| `CACHE_PASSWORD` | A strong password for your valkey cache                   |

# Installing PaperTrail

## Step 1: Clone this repository

```shell
git clone https://github.com/eggy03/PaperTrail-Deployment.git
cd PaperTrail-Deployment
```

## Step 2: Create your environment file in the repository root

```shell
cp .env.example .env
```
Use your preferred editor to set required values in your `.env` file

> [!CAUTION]
> Never commit your .env file to version control.
> If your `TOKEN` is ever exposed, immediately regenerate it in the Discord Developer Portal.

## Step 3: Deploy the services

```shell
# base deployment
docker compose -f compose-base.yml up -d

# deployment with insights
docker compose -f compose-base.yml -f compose-insight.yml up -d
```

## Step 4: Verify your deployment

To check if everything is running properly

```bash
# base deployment
docker compose -f compose-base.yml ps

# deployment with insights
docker compose -f compose-base.yml -f compose-insight.yml ps

# check logs
docker compose -f compose-base.yml -f compose-insight.yml logs -f
```

If the deployment was successful:

- Invite the bot to a Discord server.
- Confirm the bot appears online.
- Run the `/setup` slash command.

This command will guide you through the initial configuration for your server.

# Stopping and Starting PaperTrail

To stop all the services

```bash
docker compose -f compose-base.yml -f compose-insight.yml down
```

To start them again
```bash
docker compose -f compose-base.yml -f compose-insight.yml up -d
```

# Updating PaperTrail

```bash
# stop containers
docker compose -f compose-base.yml -f compose-insight.yml down

# update images
docker compose -f compose-base.yml -f compose-insight.yml pull

# restart containers
docker compose -f compose-base.yml -f compose-insight.yml up -d

# remove dangling images
docker image prune

# check the updated images
docker compose -f compose-base.yml -f compose-insight.yml images
```

# Uninstalling PaperTrail

```bash
# Remove containers, networks and volumes, keeping the images intact
docker compose -f compose-base.yml -f compose-insight.yml down -v

# Removing everything
docker compose -f compose-base.yml -f compose-insight.yml down --rmi all -v
```

# Compose Variants

## 1: Compose Base

`compose-base.yml` contains the core services required to run PaperTrail.

Services included:

| Service                 | Description           | Internal Endpoint                                                  |
|-------------------------|-----------------------|--------------------------------------------------------------------|
| PostgreSQL v18          | Core database service | `postgresql://<DB_USERNAME>:<DB_PASSWORD>@database:5432/<DB_NAME>` |
| Valkey v9               | Core cache service    | `redis://default:<CACHE_PASSWORD>@cache:6379`                      |
| PaperTrail API (latest) | Core API service      | `http://papertrail-api:8080`                                       |
| PaperTrail Bot (latest) | Core Bot service      | N/A                                                                |

These services communicate over the **internal Docker network** and are **not exposed to the host machine via ports**.

## 2: Compose Insight

`compose-insight.yml` extends `compose-base.yml` with optional services
that provide **observability and debugging tools** for containers running in the PaperTrail network.

`compose-insight.yml` requires `compose-base.yml` and cannot be run independently.

Services included:

| Service       | Description                                    | URL                   |
|---------------|------------------------------------------------|-----------------------|
| PgAdmin       | Web UI for viewing and managing PostgreSQL     | http://127.0.0.1:5050 |
| Redis Insight | Web UI for inspecting Valkey/Redis data        | http://127.0.0.1:5540 |
| Dozzle        | Web UI for viewing container logs and activity | http://127.0.0.1:9090 |

These services **publish ports to the host machine**, making them accessible through the URLs above.

### Accessing PgAdmin

You will need an email and a password to log in to PgAdmin.
If you are logging in for the first time, the default e-mail and password is:

Email: `admin@example.com`

Password: `admin`

It is highly recommended that you change your Email/Username and password upon login, via the admin panel.
The default Email and Password can be updated in `compose-insight.yml`.

- In the `Dashboard` tab, click on `Add New Server`.
- In the `General` tab, provide a server name and then go to the `Connections` tab.
- For `Host name/address`, write "database"
- For `Port`, write "5432"
- For `Maintainence DB`, substitute the value you used for `DB_NAME`
- For `Username`, substitute the value you used for `DB_USERNAME`
- For `Password`, substitute the value you used for `DB_PASSWORD`

### Accessing Redis Insight

If you are logging in for the first time, you will have the option to `Add Redis Database`

Use the following Connection URL `redis://default:<CACHE_PASSWORD>@cache:6379`

Substitute <CACHE_PASSWORD> with the password set by you in the environment variables.

# License

[MIT](/LICENSE)
