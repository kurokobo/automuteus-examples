# AutoMuteUs on Raspberry Pi

Some helpful information for getting AutoMuteUs to work on Raspberry Pi.

## TL;DR

* With Ubuntu, there are no special considerations.
* With Raspberry Pi OS (former Raspbian), we will need to manually update `libseccomp2` and `libseccomp-dev`.
  * `curl -O http://ftp.jp.debian.org/debian/pool/main/libs/libseccomp/libseccomp2_2.5.1-1_armhf.deb`
  * `curl -O http://ftp.jp.debian.org/debian/pool/main/libs/libseccomp/libseccomp-dev_2.5.1-1_armhf.deb`
  * `sudo apt install ./libseccomp-dev_2.5.1-1_armhf.deb ./libseccomp2_2.5.1-1_armhf.deb`

## On Raspberry Pi OS (Raspbian)

On Raspbian 10, `redis:6-alpine` and `postgres:12-alpine` used in AutoMuteUs don't work correctry. 

<details>
<summary>Crash logs for Redis on startup</summary>

```
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

```
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

### Technical Details

These problems are caused by the update of the base image for the PostgreSQL and Redis from `alpine:3.12` to `alpine:3.13`.

* https://github.com/docker-library/postgres/commit/a6b426236d827763cf3f5b17e489b62289b75ff1
* https://github.com/docker-library/redis/commit/04e150e0c6fb21e1eb0a79dde9998c37903358d3

The technical details for these problems can be found in the release notes of [Alpine](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0#musl_1.2) and related [musl](https://musl.libc.org/time64.html). In short, the system call for time on 32-bit systems has changed and it is no longer well handled by the old `libseccomp` on the container host side. Since the latest PostgreSQL and Redis used this system call indirectly, it no longer work well.

### Workaround

If we want to use the latest Redis or PostgreSQL images on Raspberry Pi OS, the problem will be solved by updating `libseccomp` for `armhf` platform. Note that the latest `libseccomp` in the APT repository (`2.3.3-4`) is a bit outdated to solve these problems, so we will need to download the newer one and install it manually.

```
curl -O http://ftp.jp.debian.org/debian/pool/main/libs/libseccomp/libseccomp2_2.5.1-1_armhf.deb
curl -O http://ftp.jp.debian.org/debian/pool/main/libs/libseccomp/libseccomp-dev_2.5.1-1_armhf.deb
sudo apt install ./libseccomp-dev_2.5.1-1_armhf.deb ./libseccomp2_2.5.1-1_armhf.deb
```

## Additional information to use v7 on Raspberry Pi (Experimental)

In the current v7 implementation, AutoMuteUs may not start properly on Raspberry Pi. This is due to the high load on limited hardware resources during startup causing some of the internal processing to time out.

The timeout is hardcoded and cannot be extended without modifying the code.

As a workaround to avoid high load at startup, start Redis and PostgreSQL first, and wait a little while before starting the rest of the services.

```
docker-compose up -d redis postgres
sleep 30
docker-compose up -d automuteus galactus
sleep 30
docker-compose up -d wingman
```

For technical information, see [the issue on Galactus](https://github.com/automuteus/galactus/issues/12).
