# Schedules and Restarts

Outcome: Watchtower checks when you want it to, restarts things
in the order and at the pace you chose, and cleans up after
itself — without the one dangerous flag sneaking in. Flags are
quoted from the real 1.23.2 help output; behaviour is the
project's documented behaviour.

## When to check: interval or schedule, not both

Two ways to set the rhythm, and the documentation is explicit
that you choose one — defining both is an error.

- **`--interval` (`-i`)**: seconds between checks. The default
  is **86400 — once a day**. `--interval 3600` checks hourly.
- **`--schedule` (`-s`)**: a cron expression, for checks tied
  to the clock rather than to when Watchtower happened to
  start: `--schedule "0 0 4 * * *"` checks at 4 a.m. daily.
  Watchtower's cron expressions have **six fields** — a leading
  seconds field the classic five-field cron does not have — so
  the expression above reads "second 0, minute 0, hour 4". For
  most home setups, "in the quiet hours" is the right instinct:
  an update that goes wrong at 4 a.m. is found in the morning;
  one that goes wrong at 7 p.m. interrupts the evening.

Add `--update-on-start` if you also want a check the moment
Watchtower itself starts, on top of the rhythm.

## How restarts happen

When several containers need updating, the default is to stop
and recreate them in dependency order (chapter 7) but roughly
together. **`--rolling-restart`** changes that to strictly one
at a time: finish one container's whole cycle before starting
the next. For a set of containers that serve the same site
behind a load balancer, rolling restarts are what keep the
site up *through* its updates; for a home media server they
simply make the log easier to follow.

Two more controls shape each individual restart:

- **`--stop-timeout` (`-t`)**, default 30s: how long the old
  container gets to shut down gracefully after its stop signal
  before being killed. Databases and anything that flushes state
  on exit may deserve more.
- **`--no-restart`**: pull new images and recreate containers
  *without starting them* — a specialist setting, mostly for
  hosts whose containers are started by something else.

## Tidying up: --cleanup, and the dangerous one

**`--cleanup` (`-c`)** deletes each old image once its
replacement is running. Without it, old images accumulate —
harmless but hungry: a stack updated weekly keeps a month of
predecessor images within a month. With it, disk stays flat,
at one real cost: the old image is gone, so rolling back means
pulling it again (chapter 13). Most readers should turn cleanup
on.

**`--remove-volumes`** is the dangerous flag, and this guide's
advice is simply: do not use it. It deletes the anonymous
volumes attached to a container before recreating it. Volumes
are where container data lives; for any container whose data
you cannot trivially rebuild, this flag converts an update
into a loss. It exists for stateless workers with junk volumes,
and if that is not a precise description of your container,
leave it off.

## Stopped containers

By default Watchtower concerns itself with running containers.
**`--include-stopped` (`-S`)** also watches containers that
exist but are stopped (created or exited); add
**`--revive-stopped`** and a stopped container that gets
updated is also started afterwards. Without revive, an updated
stopped container stays stopped — recreated on the new image,
ready for whoever starts it.

## Two fork additions worth knowing

The maintained fork adds guards the original never had, both
visible in the real help output:

- **`--cooldown-delay`** (for example `24h`, `3d`, `1w`): an
  image must be at least this old before Watchtower will update
  to it. A simple, effective defence against being the first
  person bitten by a bad release — let the rest of the world
  test it for a day.
- **`--disk-space-warn` / `--disk-space-max`**: warn, or refuse
  to start an update session at all, when Docker's image usage
  passes a size you set. An update that cannot pull because the
  disk filled up fails mid-ceremony; these turn that into a
  planned refusal instead.
