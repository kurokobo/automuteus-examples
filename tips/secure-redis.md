# Secure Redis

By default, the Redis server does not require authentication, so anyone with access to the docker network can read and write the information on Redis.

This introduces the way to secure your Redis by password authentication.

## Setup

### Prepare to setup

Ensure that you can get data from Redis without authentication.

```bash
$ docker-compose exec redis redis-cli keys '*'
1) "automuteus:games:total"
2) "automuteus:count:guilds"
3) "automuteus:users:total"
4) "automuteus:commit"
5) "automuteus:version"
```

Then stop your instance.

```
docker-compose down
```

### Change configuration

Add `REDIS_PASS` and set some value to your `.env` file.

```
...
POSTGRES_USER=postgres
POSTGRES_PASS=putsomesecretpasswordhere
REDIS_PASS=thisismysupersecurepassword  👈👈👈
...
```

Then, add `REDIS_PASS=${REDIS_PASS}` under `environment` for `automuteus` and `galactus` in your `docker-compose.yml`.

```yaml
...
  automuteus:
    ...
    environment:
      ...
      - REDIS_ADDR=${AUTOMUTEUS_REDIS_ADDR}
      - REDIS_PASS=${REDIS_PASS}            👈👈👈
      - GALACTUS_ADDR=${GALACTUS_ADDR}
      - POSTGRES_ADDR=${POSTGRES_ADDR}
      ...
  galactus:
    ...
    environment:
      ...
      - REDIS_ADDR=${GALACTUS_REDIS_ADDR}
      - REDIS_PASS=${REDIS_PASS}            👈👈👈
      - GALACTUS_PORT=${GALACTUS_PORT}
      ...
```

Finally, add `command` for `redis` in your `docker-compose.yml`.

Ideally, we should control the password in the configuration file `redis.conf` rather than passing it in plain text as an argument. However, this would be a bit exaggerated, so I have simplified it to this.

```yaml
...
  redis:
    image: redis:alpine
    restart: always
    command:               👈👈👈
      - redis-server       👈👈👈
      - --requirepass      👈👈👈
      - ${REDIS_PASS}      👈👈👈
    volumes:
      - "redis-data:/data"
...
```

### Check the results

Start your instance.

```
docker-compose up -d
```

Ensure that you can not get data from Redis without password.

```bash
$ docker-compose exec redis redis-cli keys '*'
(error) NOAUTH Authentication required.
```

```
$ docker-compose exec redis redis-cli -a thisismysupersecurepassword keys '*'
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
1) "automuteus:version"
2) "automuteus:games:total"
3) "automuteus:commit"
4) "automuteus:count:guilds"
5) "automuteus:users:total"
```
