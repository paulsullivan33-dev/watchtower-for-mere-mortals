# Watchtower for Mere Mortals

Every container you run was started from an image, and the
people who publish that image keep publishing newer ones —
security fixes, bug fixes, small improvements. Your container
notices none of it. It runs the image it was born with until
you remember to update it by hand, one `pull`, one restart at
a time. **Watchtower** is the program that does the
remembering: it watches your containers, compares each one's
image against the registry's current version, and when they
differ it swaps the container for a fresh one built from the
new image — same ports, same volumes, same settings — and
tells you it happened.

This guide first settles a question most Watchtower writing
skips: **which Watchtower?** The original project, containrrr's
Watchtower, is archived and unmaintained — its last release
was in November 2023, and its README says so in its first
lines. The living tool is the maintained fork,
**nicholas-fedor/watchtower** (on Docker Hub:
**nickfedor/watchtower**), same commands and labels, current
releases. This guide teaches the fork, and chapter 1 explains
how to tell the two apart and why it matters for a program you
are about to hand control of your Docker host.

Then the guide does something it is careful to be honest
about. Watchtower's whole job is talking to a Docker daemon,
and the lab that wrote this guide had none — so the updates
themselves are taught from the project's own documentation
and labelled documented-only wherever they appear. What the
lab *did* run is the real thing as far as it goes: the genuine
**1.23.2** binary from the fork's GitHub releases, its
published checksum verified before the program was ever run,
its complete `--help` output (every flag this guide teaches
is taken from it), a run with no daemon to talk to — whose
failure messages are quoted word for word, because "cannot
reach Docker" is the first failure every reader meets — and
its HTTP API, which served real requests in the lab: health
probes that honestly reported the missing daemon, and a
metrics endpoint that caught the project's own documentation
in a small lie about how its token is sent. `criteria.md`
keeps the full ledger.

## What this guide covers

- [**What Watchtower is**](what-watchtower-is.md) — the mental
  model, and the original-versus-fork question settled first
- [**The moving parts**](the-moving-parts.md) — the Docker
  socket, digests, the recreate ceremony, and labels
- [**Installing Watchtower**](installing-watchtower.md) — the
  container form, the socket mount's security bargain, pinning
  a version, flags versus environment variables
- [**Your first update**](your-first-update.md) — one cycle by
  hand with `--run-once`, and the verbatim log of a failed one
- [**Choosing what to update**](choosing-what-to-update.md) —
  update everything, exclude by label, opt-in mode, and
  monitor-only
- [**Schedules and restarts**](schedules-and-restarts.md) —
  interval versus cron, rolling restarts, `--cleanup`, and
  the dangerous `--remove-volumes`
- [**Linked containers**](linked-containers.md) — dependency
  order on the way down and up, the depends-on label, Compose
- [**Lifecycle hooks**](lifecycle-hooks.md) — commands inside
  your containers around an update, timeouts, and the exit
  code that vetoes an update
- [**Notifications**](notifications.md) — shoutrrr URLs for
  email, chat and webhooks, and the deprecated flags to stop
  copying from old posts
- [**Private registries**](private-registries.md) —
  config.json, credential helpers, and self-hosted registries
- [**The HTTP API and metrics**](http-api-and-metrics.md) —
  triggering updates over HTTP, health probes, Prometheus
  counters, and the token quirk the lab verified
- [**Recipes**](recipes.md) — the safe rollout, a quiet-hours
  home server, backup-before-update enforced by a hook, and
  update-on-deploy
- [**Limits**](limits.md) — blind tag-following, no testing,
  no rollback, and the containers never to hand over
- [**Troubleshooting**](troubleshooting.md) — the socket
  error, the 401 with the right token, the exit code that
  lies, and the missing `--version`
- [**Glossary**](glossary.md) — the terms, collected in one
  place

## What it doesn't cover

Running Docker itself — installing it, building images,
writing Dockerfiles. That is the Docker guide's ground (below),
and this one assumes a host where containers already run.
Kubernetes and other orchestrators are out of scope in the
other direction: they have their own update machinery, and
Watchtower's home is the single Docker host. Other updaters
are named where they genuinely differ, not surveyed.

## How to use this guide

Read chapter 1 before installing anything — the fork question
decides what you install — and chapter 2 before running it,
because the socket mount in chapter 3 is a security decision
you should make with your eyes open. Chapters 4 to 6 get a
tame Watchtower running: one manual cycle, a deliberate choice
of what it may touch, a schedule. Chapters 7 to 11 are the
working depth, there when your stack needs them. Chapter 12
is a plan, not just examples — follow its rollout in order.
Chapters 13 and 14 are the ones to come back to at 2 a.m. The
lab's transcripts live in `watchtower-lab/transcripts/` and
its source list in `watchtower-lab/sources.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
- [Docker Compose for Mere Mortals](https://paulsullivan33-dev.github.io/docker-compose-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/watchtower-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/watchtower-for-mere-mortals/blob/main/LICENSE-CODE).
