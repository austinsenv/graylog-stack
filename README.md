# Graylog Stack

A Docker Compose deployment of Graylog with a 3-node OpenSearch data cluster and MongoDB.

## Prerequisites

- Docker and Docker Compose installed on the host
- Linux host (required for the `vm.max_map_count` setting below)

## Setup

### 1. Configure the environment file

Copy the example env file and fill in the required values:

```sh
cp .env.example .env
```

Edit `.env` and set the following variables:

**`GRAYLOG_PASSWORD_SECRET`** — a random secret used to pepper stored passwords. Must be at least 64 characters. Generate one with:
```sh
pwgen -N 1 -s 96
```

**`GRAYLOG_ROOT_PASSWORD_SHA2`** — SHA-256 hash of the admin password you want to use to log into Graylog. Generate one with:
```sh
echo -n 'yourpassword' | shasum -a 256
```

**`GRAYLOG_HTTP_EXTERNAL_URI`** — the externally reachable URL of the Graylog web interface. Set this to the host's IP or hostname so that redirects and assets work from other machines:
```
GRAYLOG_HTTP_EXTERNAL_URI="http://192.168.1.100:9000/"
```

> **Important:** `GRAYLOG_PASSWORD_SECRET` must be identical across all Graylog nodes. Changing it after initial setup invalidates all user sessions and encrypted values in the database.

### 2. Set vm.max_map_count

The Graylog data nodes require the `vm.max_map_count` kernel parameter to be at least `262144`. Apply it immediately with:

```sh
sudo sysctl -w vm.max_map_count=262144
```

To make the change permanent across reboots, add the following line to `/etc/sysctl.conf`:

```
vm.max_map_count=262144
```

Then apply it without rebooting:

```sh
sudo sysctl -p
```

### 3. Start the stack

```sh
docker compose up -d
```

The services start in dependency order: MongoDB first, then the three data nodes, then Graylog. Full startup can take a few minutes while health checks pass.

### 4. Access Graylog

Once all containers are healthy, the Graylog web interface is available at the `GRAYLOG_HTTP_EXTERNAL_URI` you configured (default: `http://localhost:9000/`).

Log in with username `admin` and the password you hashed in step 1.

## Troubleshooting

**Missing env variables warning on startup:**
```
WARN[0000] The "GRAYLOG_PASSWORD_SECRET" variable is not set.
```
This means `.env` is missing or incomplete. Re-check step 1.

**`mongodb-container is unhealthy` / datanodes fail to start:**
The data nodes depend on MongoDB being healthy. Check MongoDB logs:
```sh
docker compose logs mongodb
```

**General container logs:**
```sh
docker compose logs -f
docker compose logs <service-name>
```

**Stop the stack:**
```sh
docker compose down
```

**Stop and remove all volumes (full reset):**
```sh
docker compose down -v
```
