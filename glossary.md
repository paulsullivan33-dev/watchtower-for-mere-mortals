# Glossary

**cleanup** — Watchtower's habit, when `--cleanup` is set, of
deleting an old image once its replacement container is
running.

**config.json** — the file where Docker keeps registry
logins; shared with Watchtower so it can pull private images.

**container** — a running instance of an image, with its own
settings, storage and network identity; disposable by design.

**cooldown delay** — a fork feature (`--cooldown-delay`): an
image must be at least this old before Watchtower updates to
it.

**credential helper** — a program that keeps registry
passwords in a keychain or cloud service instead of in
config.json.

**daemon** — Docker's background program; everything that
manages containers talks to it through its socket.

**digest** — the `sha256:` fingerprint of an image's exact
contents. Tags move; digests do not. Watchtower detects
updates by comparing them.

**EX_TEMPFAIL (75)** — the exit code a pre-update lifecycle
hook returns to say "not this time": Watchtower skips that
container's update for the cycle.

**image** — the packaged snapshot a container is started
from: program, libraries and default settings.

**label** — a key-and-value note attached to a container at
creation. Watchtower's per-container settings — enable,
monitor-only, depends-on, lifecycle hooks — are labels.

**lifecycle hook** — a command Watchtower runs inside one of
your containers at a set point (pre-check, pre-update,
post-update, post-check) of an update cycle.

**monitor only** — Watchtower mode, global or per container,
that reports available updates but never applies them.

**porcelain** — `--porcelain v1`: session results written to
standard output in a stable machine-readable form, for
scripts.

**recreate** — how Watchtower updates: stop the old container,
create a new one from the new image with the same settings,
start it. Nothing is ever patched in place.

**registry** — the service images are pulled from; Docker Hub
is the public one.

**rolling restart** — updating containers strictly one at a
time, each cycle finished before the next begins.

**scope** — a name given to a Watchtower instance and matched
by a label on containers, so several instances can share one
daemon without overlap.

**shoutrrr** — the library Watchtower uses for notifications;
each destination is a URL whose scheme picks the service.

**socket** — `/var/run/docker.sock`: the file through which
programs give the Docker daemon its orders. Access to it is
control of the host's containers.

**tag** — the human name for an image version — `latest`,
`16`, `1.23.2`. Tags are pointers and can be moved to a new
image at any time.
