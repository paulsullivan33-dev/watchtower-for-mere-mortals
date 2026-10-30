# Limits

Outcome: you know what Watchtower will not do, and which
containers should never be handed to it. This chapter is the
guide's honest centre: read it before chapter 12's rollout,
not after your first bad morning.

## It updates to the tag, blindly

Watchtower follows the tag your container was started from.
A container on `postgres:16` gets newer `postgres:16` images —
patch releases, as intended. A container on `postgres:latest`
gets *whatever latest means this week*, including the day
latest becomes version 17 with a data format your volume
cannot go back from. Watchtower has no notion of "major
version" and no semver judgement. **The tag is your policy.**
Pin tags to the major version you mean (`:16`, `:1.2`) and
let Watchtower handle movement *within* it.

## When NOT to auto-update

- **Databases and stateful services** — Postgres, MySQL,
  MariaDB, anything whose on-disk format changes between
  versions and whose downgrade path is "restore from backup".
  Even within a major version, take these deliberately, with a
  backup first — or use recipe 3's hook to take the backup
  automatically and veto the update when it fails. Monitor-only
  (chapter 5) exists for exactly these containers.
- **Anything pinned for a reason** — a version held back for a
  bug, a licence server, a tool that only works against one
  API version. Label it `enable=false` and write the reason in
  a comment next to the label; future-you will ask.
- **Containers whose image tag is `latest` and whose author
  ships breaking changes** — the combination, not either alone,
  is the trap.
- **Things that must not restart unscheduled** — a container
  is stopped and recreated on every update. If a restart at
  4 a.m. pages somebody or drops a long-running job, that
  container belongs on a schedule you control, or in
  monitor-only.

## No testing, no rollback

Watchtower does not run your test suite, check your health
endpoint, or ask the new container whether it feels well. It
starts the new container; if the process runs, the update is
a success as far as Watchtower is concerned — even if the
application inside is failing on a bad new configuration.
And there is **no automatic rollback**: if the new version is
broken, the fix is you, redeploying the old image (easy if
you still have it — one more reason `--cleanup` is a choice,
chapter 6 — or pullable again by digest). The mitigations are
the ones this guide keeps recommending: monitor-only weeks,
cooldown delays on the fork, notifications that arrive
*before* your users do, and tags pinned to majors.

## It recreates; it does not migrate

The replacement container is built from the old one's
configuration. Changes you made by hand inside the running
container — the package you apt-installed during debugging,
the config you edited in place — **vanish** at the next
update, because they were never in the image or its declared
configuration. This is the container discipline working as
designed, and it surprises everyone exactly once.

## Readiness is not start

Chapter 7's caveat, repeated because it bites: Watchtower
starts containers in the right order, but a started database
may not be a *ready* database. Applications must tolerate a
few seconds of connection refusals after an update, or be
restarted by hand.

## One daemon, one trust domain

An instance watches one Docker daemon. A fleet means an
instance per host (or remote daemons over TLS, with the trust
that implies). And the standing limit from chapter 3:
Watchtower's power over the host is total, by design. It is
system software, not an app — choose its source (the
maintained fork), pin it when in doubt, and let it update
itself only when that trust is deliberate.
