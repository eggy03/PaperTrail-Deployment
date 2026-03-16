# Table of Contents

- [Overview](#overview)
- [Latest Releases](#latest-releases)
- [Creating the Bot](#creating-the-bot)
- [Installing PaperTrail](#installing-papertrail)
- [Updating PaperTrail](#updating-papertrail)
- [Uninstalling PaperTrail](#uninstalling-papertrail)
- [Compose Variants](#compose-variants)

# Overview
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
Use your preferred editor to edit the required values in your `.env` file.
If you have replicated the provided `.env.example` using the above command,
you can keep the default values for all the variables except `TOKEN` since it has no value set.

> [!CAUTION]
> Never commit your .env file to version control.
> If your `TOKEN` is ever exposed, immediately regenerate it in the Discord Developer Portal.

## Step 3: Deploy the services

```shell
docker compose up -d
```

## Step 4: Verify your deployment

To check if everything is running properly

```bash
# base deployment
docker compose ps

# check logs
docker compose logs -f
```

If the deployment was successful:

- Invite the bot to a Discord server.
- Confirm the bot appears online.
- Run the `/setup` slash command.

This command will guide you through the initial configuration for your server.

# Stopping, Starting and Restarting PaperTrail

```bash
docker compose stop
```
```bash
docker compose start
```
```bash
docker compose restart
```

# Updating PaperTrail

```bash
# stop containers
docker compose down

# update images
docker compose pull

# restart containers
docker compose up -d
```
# Uninstalling PaperTrail

```bash
# Remove containers, networks and volumes, except images
docker compose down -v

# Removing everything
docker compose down --rmi all -v
```

# Compose Variants

## 1: Compose Base

`compose.yml` contains the core services required to run PaperTrail.

Services included:

| Service                 | Description           |
|-------------------------|-----------------------|
| PostgreSQL v18          | Core database service |
| Valkey v9               | Core cache service    |
| PaperTrail API (latest) | Core API service      |
| PaperTrail Bot (latest) | Core Bot service      |

All the above services communicate over the **internal Docker network**
and are **NOT exposed to the host machine via ports**.

## 2: Compose Insight

`compose-insight.yml` extends `compose.yml` with optional services
that provide **observability and debugging tools** for containers running in the PaperTrail network.

`compose-insight.yml` requires `compose.yml` and cannot be run independently.

Services included:

| Service       | Description                                    | URL                   |
|---------------|------------------------------------------------|-----------------------|
| PgAdmin       | Web UI for viewing and managing PostgreSQL     | http://127.0.0.1:5050 |
| Redis Insight | Web UI for inspecting Valkey/Redis data        | http://127.0.0.1:5540 |

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
