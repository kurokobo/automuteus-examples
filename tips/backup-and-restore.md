<!-- omit in toc -->
# Backup and Restore

This is how to backup and restore your AutoMuteUs.

<!-- omit in toc -->
## Table of Contents

- [💎 Data and files to consider backing up and restoring](#-data-and-files-to-consider-backing-up-and-restoring)
- [💾 Backing up your `.env` and `docker-compose.yml`](#-backing-up-your-env-and-docker-composeyml)
- [💾 Backing up your Redis](#-backing-up-your-redis)
  - [Online backup (without downtime)](#online-backup-without-downtime)
  - [Offline backup (with downtime)](#offline-backup-with-downtime)
- [💾 Backing up your PostgreSQL](#-backing-up-your-postgresql)
  - [Online backup (without downtime)](#online-backup-without-downtime-1)
  - [Offline backup (with downtime)](#offline-backup-with-downtime-1)
- [🩺 Restoring your `.env` and `docker-compose.yml`](#-restoring-your-env-and-docker-composeyml)
- [🩺 Restoring your Redis](#-restoring-your-redis)
- [🩺 Restoring your PostgreSQL](#-restoring-your-postgresql)
- [❓ Troubleshoot](#-troubleshoot)
  - [Got `the input device is not a TTY` when invoke `docker compose exec` command](#got-the-input-device-is-not-a-tty-when-invoke-docker-compose-exec-command)
  - [Recreate entire environment](#recreate-entire-environment)

## 💎 Data and files to consider backing up and restoring

- **`.env` file**
  - Super important file that includes Discord token.
- **`docker-compose.yml`**
  - If you have made any changes to this file, you shoud keep this file.
- **Redis**
  - The Redis includes the following data that should be saved:
    - Some counters for total users, total games, total guilds, etc.
    - Settings per guild (any values that changed by `.au settings`)
- **PostgreSQL**
  - The PostgreSQL stores the history of the past games. It is mainly used for leaderboards (`.au stats`).

## 💾 Backing up your `.env` and `docker-compose.yml`

Just copy your file and keep it in safe place.

## 💾 Backing up your Redis

### Online backup (without downtime)

During Redis running, by default, a dump file is created periodically depending on the amount of changes of the keys and values. The date and time of the latest dump file creation in Unix time can be checked with the `LASTSAVE` command.

```bash
$ docker compose exec redis redis-cli lastsave
(integer) 1614434015

$ date -ud @1614434015
Sat 27 Feb 2021 01:53:35 PM UTC
```

If necessary, you can start saving the dump file again by `BGSAVE` command.

```bash
docker compose exec redis redis-cli bgsave
```

Ensure that the saved time has been updated.

```bash
$ docker compose exec redis redis-cli lastsave
(integer) 1614434965

$ date -ud @1614434965
Sat 27 Feb 2021 02:09:25 PM UTC
```

Once the dump file is created, copy the file from container.

```bash
mkdir -p ./backup/redis
docker cp $(docker compose ps -q redis):/data/dump.rdb ./backup/redis
```

Now you have `dump.rdb` in the `./backup/redis` directory. Keep this file in safe place.

### Offline backup (with downtime)

Redis creates a dump file when shut it down itself, so there is no need to explicitly invoke additional command to create a dump file. Just stop your instance.

```bash
docker compose down
```

Then run one-shot worker container and invoke `cp` inside the container to copy the dump file to mounted host-side directory by `-v` option.

```bash
mkdir -p ./backup/redis
docker compose run --rm -v $PWD/backup/redis:/tmp redis cp /data/dump.rdb /tmp
```

Keep the backup file `./backup/redis/dump.rdb` in safe place.

## 💾 Backing up your PostgreSQL

### Online backup (without downtime)

Invoke `pg_dump` by appropriate user and save standard output as a backup file.

```bash
mkdir -p ./backup/psql
docker compose exec postgres sh -c 'pg_dump -c --if-exists -U $POSTGRES_USER' > ./backup/psql/dump.sql
```

Keep the backup file `./backup/psql/dump.sql` in safe place.

### Offline backup (with downtime)

Stop your instance, and then start only postgres.

```bash
docker compose down
docker compose up -d postgres
```

Then, perform the same way as the online backup

```bash
mkdir -p ./backup/psql
docker compose exec postgres sh -c 'pg_dump -c --if-exists -U $POSTGRES_USER' > ./backup/psql/dump.sql
docker compose down
```

Keep the backup file `./backup/psql/dump.sql` in safe place.

Alternatively the file-level backup can be also executed instead of `pg_dump`, but this is not recommended.

```bash
docker compose run --rm -v $PWD/backup/psql:/tmp postgres tar c -zvf /tmp/backup.tar.gz /var/lib/postgresql/data
```

## 🩺 Restoring your `.env` and `docker-compose.yml`

Stop your instance, and then simply replace your files.

```bash
docker compose down
```

## 🩺 Restoring your Redis

Stop your instance, and copy your dump file into the volume.

```bash
docker compose down
docker compose run --rm -v $PWD/backup/redis:/tmp redis cp /tmp/dump.rdb /data/dump.rdb
```

## 🩺 Restoring your PostgreSQL

Stop your instance, and then start only postgres.

```bash
docker compose down
docker compose up -d postgres
```

Then, perform `psql` using backup file to restore.

```bash
docker compose exec -T postgres sh -c 'psql -U $POSTGRES_USER' < ./backup/psql/dump.sql
docker compose down
```

If your backup file is created at file-level, you should invoke file-level restore instead of `psql`.

```bash
docker compose run --rm -v $PWD/backup/psql:/tmp postgres tar x -zvf /tmp/backup.tar.gz
```

## ❓ Troubleshoot

### Got `the input device is not a TTY` when invoke `docker compose exec` command

This is a problem that can occur in some environments. To solve this, add `-T` just after `exec`.

```bash
# Original command
docker compose exec redis redis-cli lastsave

# Add "-T" after "exec"
docker compose exec -T redis redis-cli lastsave
```

### Recreate entire environment

By invoking `docker compose down` with `-v` (or `--volumes`) option, all containers, networks and volumes related to your `docker-compose.yml` file will be deleted.

```bash
docker compose down -v
```
