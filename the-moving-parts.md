# The Moving Parts

Outcome: after this chapter you can name every piece Watchtower
touches during an update, so that when chapter 4 walks through
one, nothing in it is a surprise.

## The Docker socket

Docker itself is run by a background program, the **daemon**.
Everything that manages containers — the `docker` command, and
Watchtower — talks to the daemon through a **socket**, a file
(usually `/var/run/docker.sock`) that programs send requests
through. Watchtower's one essential requirement is access to
that socket. Its own help text, captured in the lab, gives the
default it connects to:

```
-H, --host string   daemon socket to connect to
                    (default "unix:///var/run/docker.sock")
```

Whoever can write to that socket can start any container with
any settings — including one that mounts the host's whole disk.
Keep that fact in mind; chapter 3 returns to it.

## The registry and the digest

Images live in **registries**: Docker Hub is the public one, and
companies and projects run private ones too. An image is fetched
by a name and a **tag** — `nickfedor/watchtower:latest`,
`postgres:16` — and behind each tag sits a **digest**, a long
hexadecimal fingerprint of the image's exact contents, starting
`sha256:`. Tags move (that is their purpose: `latest` points at
whatever is newest); digests do not. Watchtower's check is
simply: *ask the registry for the digest behind the tag this
container was started from, and compare it with the digest of
the image the container is actually running.* Equal — nothing
to do. Different — time to update.

## The recreate

An update is a small ceremony, performed in this order:

1. **Pull** the new image.
2. **Stop** the old container, politely first (it sends the
   container's stop signal and waits — 30 seconds by default)
   and forcefully after the wait.
3. **Create** a new container from the new image, copying the
   old container's configuration: its name, ports, volume
   mounts, environment, labels, restart policy, command.
4. **Start** the new container.
5. Optionally, with `--cleanup`, **delete the old image**, which
   nothing is using any more.

Your data survives because it was never inside the container to
begin with: it lives in **volumes** and bind mounts, which the
new container mounts exactly as the old one did. (One flag
breaks that rule — `--remove-volumes` deletes the volumes too.
Chapter 6 names it as the dangerous one.)

## Labels: notes stuck on containers

Much of Watchtower's fine control is expressed as **labels** —
key-and-value notes attached to a container when it is created.
The label names still carry the original project's name, and
both versions honour them:

- `com.centurylinklabs.watchtower.enable` — include or exclude
  this container (chapter 5)
- `com.centurylinklabs.watchtower.monitor-only` — report updates
  for this container but never apply them (chapter 5)
- `com.centurylinklabs.watchtower.depends-on` — containers this
  one depends on (chapter 7)
- `com.centurylinklabs.watchtower.lifecycle.*` — commands to run
  inside the container around its update (chapter 8)

## Watchtower itself

Watchtower normally runs as a container too — one container
whose job is managing all the others, including, if you let it,
itself: when its own image updates, it recreates itself the
same way. It can also run as an ordinary program (that is how
this guide's lab ran it), and it can be driven three ways: on a
schedule of its own, once from the command line with
`--run-once`, or on demand through its HTTP API (chapter 11).

## The scope of one instance

One Watchtower watches one Docker daemon. If you run several
instances against the same daemon — say, one that may update
everything and one that may only watch — you separate them with
a **scope**: a name given to the instance (`--scope`), matched
by a `com.centurylinklabs.watchtower.scope` label on the
containers it is responsible for. Most readers will run exactly
one instance and can forget scopes exist.
