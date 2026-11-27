# postgres

A local PostgreSQL server with pgAdmin, run by Docker Compose.

## Setup

Clone the repository with git and go into the directory:

```sh
git clone https://github.com/rtomrud/postgres.git
cd postgres
```

Write each password to its own file in the project root. Git ignores both files.

```sh
echo "your-postgres-password" > postgres_password
echo "your-pgadmin-password" > pgadmin_password
```

Or generate random passwords:

```sh
openssl rand -hex 16 > postgres_password
openssl rand -hex 16 > pgadmin_password
```

pgAdmin runs as a user in group 0, so give that group read access to the files while keeping them hidden from other users on the host:

```sh
sudo chgrp 0 postgres_password pgadmin_password
sudo chmod 640 postgres_password pgadmin_password
```

Start the services:

```sh
docker compose up -d
```

## Usage

Open pgAdmin at <http://localhost:3000> and sign in as `postgres@example.com` with the pgAdmin password.

The `postgres` server appears under Servers and connects without asking for a password, since it's configured in the [`servers.json`](./servers.json) file.

Postgres listens only on the Compose network. To reach it from the host, uncomment the `ports` section of the `postgres` service in [`docker-compose.yml`](./docker-compose.yml), then connect:

```sh
psql -h localhost -U postgres
```

Or open a shell inside the container:

```sh
docker compose exec postgres psql -U postgres
```

## Schema

Postgres runs the `.sql` and `.sh` files in `initdb/` in name order, but only when it creates a new database. Put your schema in `initdb/schema.sql`.

To apply changes to it, delete the data volume and start again:

```sh
docker compose down -v
docker compose up -d
```

This deletes all data and loads your .sql files in the `initdb/` directory.

## Backup

Export the database to `initdb/`:

```sh
docker compose exec -T postgres pg_dump -U postgres --clean --if-exists > "initdb/zz_backup_$(date +%Y%m%d_%H%M%S).sql"
```

The `zz_` prefix makes Postgres run backups after `schema.sql`. The `--clean --if-exists` flags make each backup drop the tables that earlier files create before restoring its own, so the files don't conflict. Git ignores the backups.

To restore, keep only the backup you want in `initdb/`, then delete the data volume and start again:

```sh
docker compose down -v
docker compose up -d
```
