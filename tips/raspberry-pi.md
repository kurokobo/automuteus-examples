<!-- markdownlint-disable MD033 -->
<!-- omit in toc -->
# AutoMuteUs on Raspberry Pi

Some helpful information for getting AutoMuteUs to work on Raspberry Pi.

<!-- omit in toc -->
## Table of Contents

- [Summary](#summary)
- [Procedure](#procedure)
  - [Update `libseccomp2` and `libseccomp-dev`](#update-libseccomp2-and-libseccomp-dev)
  - [Build `automuteus` container image and run AutoMuteUs](#build-automuteus-container-image-and-run-automuteus)
- [Appendix: Explanation of the issue with `libseccomp` on Buster on ARM platform](#appendix-explanation-of-the-issue-with-libseccomp-on-buster-on-arm-platform)
- [Appendix: To use v7 on Raspberry Pi (Experimental)](#appendix-to-use-v7-on-raspberry-pi-experimental)

## Summary

Follow the instructions for each OS.

- On **Raspberry Pi OS (Buster)**
  1. [Update `libseccomp2` and `libseccomp-dev`](#update-libseccomp2-and-libseccomp-dev)
  2. [Build `automuteus` container image and run AutoMuteUs](#build-automuteus-container-image-and-run-automuteus)
- On **Raspberry Pi OS (Bullseye)**
  - [Build `automuteus` container image and run AutoMuteUs](#build-automuteus-container-image-and-run-automuteus)
- On **Ubuntu**
  - [Build `automuteus` container image and run AutoMuteUs](#build-automuteus-container-image-and-run-automuteus)

## Procedure

### Update `libseccomp2` and `libseccomp-dev`

This procedure is required on Buster-based Raspberry Pi OS only. See [the appendix](#appendix-explanation-of-the-issue-with-libseccomp-on-buster-on-arm-platform) for more details.

```bash
# Add Buster-based backport repository
echo "deb <http://deb.debian.org/debian> buster-backports main contrib non-free" | sudo tee -a /etc/apt/sources.list.d/backports.list

# Import public keys for new backport repository
sudo curl -o /etc/apt/trusted.gpg.d/archive-key-10.asc <https://ftp-master.debian.org/keys/archive-key-10.asc>

# Update libseccomp2 and libseccomp-dev
sudo apt update
sudo apt install -t buster-backports -y libseccomp2 libseccomp-dev
```

### Build `automuteus` container image and run AutoMuteUs

Prepare required files.

```bash
cd ~
git clone https://github.com/automuteus/automuteus.git
git clone https://github.com/automuteus/deploy.git
```

Then provide your specific Environment Variables in the `.env` file under `deploy` directory, as relevant to your configuration. See [the Environment Variables reference on the `README.md` on `deploy` repository](https://github.com/automuteus/deploy#environment-variables) for details, as well as the `sample.env` file under `deploy` directory provided.

Then modify your `docker-compose.yml` under `deploy` directory by commenting out the line `image: automuteus/automuteus:${AUTOMUTEUS_TAG:?err}` and uncommenting the `build: ../automuteus`.

```yaml
version: "3"

services:
  automuteus:
    # Either:
    # - Use a prebuilt image
    #image: automuteus/automuteus:${AUTOMUTEUS_TAG:?err}     👈👈👈
    # - Use an old prebuilt image (prior to 6.16.1)
    #image: denverquane/amongusdiscord:${AUTOMUTEUS_TAG:?err}
    # - Build image from local source
    build: ../automuteus     👈👈👈
    # - Build image from github directly
    #build: https://github.com/automuteus/automuteus.git
    ...
```

Next, pull related images and build your `automuteus` container.

```bash
cd ~/deploy
docker-compose pull
docker-compose build
```

Finally, start it up.

```bash
docker-compose up -d
```

If your Redis or PostgreSQL are stucked for some reason, reset AutoMuteUs by deleting related volumes. Note that the command `docker-compose down --volumes` in this procedure will remove all volumes including database used by AutoMuteUs.

```bash
docker-compose down --volumes
docker-compose up -d
```

## Appendix: Explanation of the issue with `libseccomp` on Buster on ARM platform

On the Buster-based Raspberry Pi OS, `redis:6-alpine` and `postgres:12-alpine` used in AutoMuteUs don't work correctry.

<details>
<summary>Crash logs for Redis on startup</summary>

```bash
redis_1  | 1:C 29 May 2071 13:52:08.000 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 29 May 2071 13:52:08.000 # Redis version=6.2.1, bits=32, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 29 May 2071 13:52:08.000 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  | Assertion failed: rc == 0 (monotonic.c: monotonicInit_posix: 149)
redis_1  |
redis_1  |
redis_1  | === REDIS BUG REPORT START: Cut & paste starting from here ===
redis_1  | 1:M 29 May 2071 13:31:52.000 # Redis 6.2.1 crashed by signal: 6, si_code: -6
redis_1  | 1:M 29 May 2071 13:31:52.000 # Killed by PID: 1, UID: 999
redis_1  |
redis_1  | ------ INFO OUTPUT ------
redis_1  | 1:M 29 May 2071 12:47:52.000 # Redis 6.2.1 crashed by signal: 11, si_code: 1
redis_1  | 1:M 29 May 2071 12:47:52.000 # Accessing address: 0x14
redis_1  | 1:M 29 May 2071 12:47:52.000 # Killed by PID: 20, UID: 0
redis_1  |
redis_1  | ------ INFO OUTPUT ------
```

</details>

<details>
<summary>Crash logs for PostgreSQL on startup</summary>

```bash
postgres_1  | The files belonging to this database system will be owned by user "postgres".
postgres_1  | This user must also own the server process.
postgres_1  |
postgres_1  | The database cluster will be initialized with locale "en_US.utf8".
postgres_1  | The default database encoding has accordingly been set to "UTF8".
postgres_1  | The default text search configuration will be set to "english".
postgres_1  |
postgres_1  | Data page checksums are disabled.
postgres_1  |
postgres_1  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
postgres_1  | creating subdirectories ... ok
postgres_1  | selecting dynamic shared memory implementation ... posix
postgres_1  | selecting default max_connections ... 100
postgres_1  | selecting default shared_buffers ... 128MB
postgres_1  | selecting default time zone ... GMT
postgres_1  | creating configuration files ... ok
postgres_1  | running bootstrap script ... ok
postgres_1  | performing post-bootstrap initialization ... sh: locale: not found
postgres_1  | 1970-05-05 07:14:24.010 GMT [30] WARNING:  no usable system locales were found
postgres_1  | ok
postgres_1  | syncing data to disk ... initdb: warning: enabling "trust" authentication for local connections
postgres_1  | You can change this by editing pg_hba.conf or using the option -A, or
postgres_1  | --auth-local and --auth-host, the next time you run initdb.
postgres_1  | ok
postgres_1  |
postgres_1  |
postgres_1  | Success. You can now start the database server using:
postgres_1  |
postgres_1  |     pg_ctl -D /var/lib/postgresql/data -l logfile start
postgres_1  |
postgres_1  | waiting for server to start....1970-04-25 04:53:36.009 GMT [35] LOG:  starting PostgreSQL 12.6 on arm-unknown-linux-musleabihf, compiled by gcc (Alpine 10.2.1_pre1) 10.2.1 20201203, 32-bit
postgres_1  | 1970-04-25 04:53:36.009 GMT [35] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres_1  | .........1970-04-25 04:53:36.009 GMT [35] LOG:  startup process (PID 36) was terminated by signal 11: Segmentation fault
postgres_1  | 1970-04-25 04:53:36.009 GMT [35] LOG:  aborting startup due to startup process failure
postgres_1  | 1970-04-25 04:53:36.009 GMT [35] LOG:  database system is shut down
postgres_1  | pg_ctl: could not start server
postgres_1  | Examine the log output.
postgres_1  |  stopped waiting
```

</details>

Additionaly, `docker-compose build` will also be failed.

<details>
<summary>Build failed due to temporary error</summary>

```bash
$ docker-compose build
redis uses an image, skipping
galactus uses an image, skipping
postgres uses an image, skipping
Building automuteus
Sending build context to Docker daemon  32.17MB
Step 1/19 : FROM golang:1.15-alpine AS builder
 ---> 458bd357894f
Step 2/19 : RUN apk add --no-cache git
 ---> Running in b048eada6b3a
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/main/armv7/APKINDEX.tar.gz
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.14/main: temporary error (try again later)
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/community/armv7/APKINDEX.tar.gz
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.14/community: temporary error (try again later)
ERROR: unable to select packages:
  git (no such package):
    required by: world[git]
The command '/bin/sh -c apk add --no-cache git' returned a non-zero code: 1
ERROR: Service 'automuteus' failed to build : Build failed
```

</details>

The technical details for these problems can be found in the release notes of [Alpine](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0#musl_1.2) and related [musl](https://musl.libc.org/time64.html). In short, the system call for time on 32-bit systems has changed and it is no longer well handled by the old `libseccomp` on the container host side. Since the latest PostgreSQL and Redis used this system call indirectly, it no longer work well.

The latest `libseccomp` in the repository for Buster-based Raspberry Pi OS is a bit outdated (`2.3.3-4`) to solve these problems, so we will need update related packages by using backport repository as described above.

## Appendix: To use v7 on Raspberry Pi (Experimental)

In the current v7 implementation, AutoMuteUs may not start properly on Raspberry Pi. This is due to the high load on limited hardware resources during startup causing some of the internal processing to time out.

The timeout is hardcoded and cannot be extended without modifying the code.

As a workaround to avoid high load at startup, start Redis and PostgreSQL first, and wait a little while before starting the rest of the services.

```bash
docker-compose up -d redis postgres
sleep 30
docker-compose up -d automuteus galactus
sleep 30
docker-compose up -d wingman
```

For technical information, see [the issue on Galactus](https://github.com/automuteus/galactus/issues/12).
