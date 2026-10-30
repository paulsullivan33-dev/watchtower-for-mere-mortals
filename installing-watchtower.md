# Installing Watchtower

Outcome: Watchtower running on your Docker host, as a container,
with its powers understood and its version pinned. The
`docker run` and Compose forms below are the fork's own
quickstart, documented-only in this guide's lab (which has no
daemon); the program itself was installed and run here, and that
part is verified — see the end of the chapter.

## The one-line version

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower
```

With no arguments at all, Watchtower checks every container on
the host once a day and updates whatever has a newer image. The
rest of this guide is about taming that sentence — what to
watch, when, and with what safeguards. Most readers will end up
with a Compose file instead, because that is where the rest of
their stack already lives:

```yaml
services:
  watchtower:
    image: nickfedor/watchtower:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## The socket mount, honestly

That one volume line is the whole security story. Mounting
`/var/run/docker.sock` into Watchtower gives it full control of
Docker on your host — it could start a container that mounts
your entire filesystem. This is not a flaw in Watchtower; the
job cannot be done with less. But it means:

- Run Watchtower only on hosts where that bargain makes sense —
  typically your own home server or a small fleet you administer.
- Treat the Watchtower image as trusted software, on the same
  footing as Docker itself. This is a second reason to use the
  maintained fork: an unmaintained container with root-equivalent
  power is a poor thing to leave running for years.
- If you expose its HTTP API (chapter 11), protect it. The API
  can trigger updates; it is guarded by a token, and the
  project's own documentation says plainly: never expose it
  directly to the public internet.

## Pin the version

`nickfedor/watchtower:latest` means Watchtower will also update
*itself* whenever the fork publishes — usually what you want,
and chapter 13 discusses the self-update honestly. If you would
rather move deliberately, pin a version you have read the
release notes for:

```yaml
    image: nickfedor/watchtower:1.23.2
```

Pinning Watchtower's image does not stop it updating itself if
it watches its own container and the tag it watches is a moving
one; a pinned version tag (`1.23.2`) never moves, so a pinned
Watchtower stays put until you change the number.

## Flags or environment variables

Every setting in this guide can be given two ways: as a command
line flag to the Watchtower program, or as an environment
variable — usually `WATCHTOWER_` followed by the flag's name in
capitals with dashes turned into underscores, so `--cleanup`
becomes `WATCHTOWER_CLEANUP=true`. One famous exception:
`--interval`'s variable is `WATCHTOWER_POLL_INTERVAL`. Use
whichever form your setup already favours; in Compose,
environment variables keep the file readable:

```yaml
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=86400
```

One warning the lab can give from the real program: there is no
`--version` flag. Asking for one gets you an error and the full
help text. The version is printed in the first log line at every
startup — `Watchtower 1.23.2 using Docker API v1.56`, as the
lab's own transcripts show — so check the logs, not a flag.

## How the lab installed it (verified)

The lab has no Docker, so it ran Watchtower the other way: as an
ordinary program, from the fork's GitHub releases. That route is
worth knowing for testing on any machine:

```bash
curl -sLO https://github.com/nicholas-fedor/watchtower/releases/download/v1.23.2/watchtower_linux_amd64_1.23.2.tar.gz
curl -sLO https://github.com/nicholas-fedor/watchtower/releases/download/v1.23.2/checksums.txt
sha256sum -c <(grep linux_amd64 checksums.txt)   # tarball only
tar xzf watchtower_linux_amd64_1.23.2.tar.gz
./watchtower --help
```

The checksum matched the published one
(`e8429673…c7ae8`, verified; transcript t00 in the lab folder),
and `--help` printed the full 126-line flag list that this
guide's flag claims are taken from (transcript t01). Running it
on a machine *with* Docker this way works too — it talks to the
same socket — but the container form is the normal home for it.
