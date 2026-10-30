# Recipes

Outcome: four complete, working configurations for the ways
people actually adopt Watchtower. All are documented-only in
this guide's lab, assembled from parts the earlier chapters
verified or sourced.

## 1. The safe rollout (the one to actually follow)

**Week one — watch only.** Run Watchtower in monitor-only
mode, with a notification set up:

```yaml
services:
  watchtower:
    image: nickfedor/watchtower:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_MONITOR_ONLY=true
      - WATCHTOWER_NOTIFICATION_URL=smtp://user:pass@mail.example.com:587/?from=watchtower@example.com&to=you@example.com
```

For a week, a message tells you every image that *would* have
been updated. You learn your stack's update tempo and spot the
containers you would never want touched automatically.

**Week two — opt in the safe ones.** Switch to label mode and
label the stateless, easily-rebuilt containers — the web
frontend, the media server, the dashboard:

```yaml
    environment:
      - WATCHTOWER_LABEL_ENABLE=true
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_NOTIFICATION_URL=...
```

```yaml
  # in each chosen service's definition:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
```

**After that — widen deliberately.** Bring in the rest one
service at a time. Databases and anything stateful stay
labelled `enable=false` (chapter 5) or monitor-only forever,
on purpose, as a decision rather than an oversight.

## 2. The quiet-hours home server

Everything on the host updated nightly at 4 a.m., old images
cleaned up, a report only when something changed or failed.
Note the six-field cron — the leading field is seconds:

```yaml
    environment:
      - WATCHTOWER_SCHEDULE=0 0 4 * * *
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_NOTIFICATIONS_LEVEL=info
      - WATCHTOWER_NOTIFICATION_URL=gotify://gotify.example.com/yourtoken
```

On the fork, add a cooldown so brand-new images get a day to
be somebody else's problem first:

```yaml
      - WATCHTOWER_COOLDOWN_DELAY=24h
```

(the flag form is `--cooldown-delay 24h`; the variable name
was verified against the real program — transcript t04).

## 3. Backup before update, enforced

A small database container whose pre-update hook dumps its
data to a volume — and *refuses* the update when the dump
fails, by exiting 75 (chapter 8):

```yaml
  appdb:
    image: example/appdb:1
    volumes:
      - appdb-backups:/backups
    labels:
      - "com.centurylinklabs.watchtower.lifecycle.pre-update=/usr/local/bin/dump.sh"
      - "com.centurylinklabs.watchtower.lifecycle.pre-update-timeout=10"
```

```sh
#!/bin/sh
# /usr/local/bin/dump.sh, inside the container
set -e
appdb-dump > /backups/pre-update.sql || exit 75
```

with `WATCHTOWER_LIFECYCLE_HOOKS=true` on Watchtower itself.
The timeout label gives the dump ten minutes instead of the
default one.

## 4. Update when the pipeline says so

CI builds and pushes an image, then pokes the API; Watchtower
runs no schedule at all:

```yaml
    environment:
      - WATCHTOWER_HTTP_API_ENDPOINTS=update,health
      - WATCHTOWER_HTTP_API_TOKEN=<strong random token>
    ports:
      - "127.0.0.1:8080:8080"
```

```bash
curl -X POST -H "Authorization: <strong random token>" \
  http://localhost:8080/v1/update
```

(token sent in the form this guide verified — see chapter 11;
the API port is bound to localhost only, and the pipeline runs
on the same host). The deploy stays a deliberate act; only the
swap is automated.
