# What Watchtower Is

You run some programs as **Docker containers** — a web service, a
media server, a small database tool, a home dashboard. Each one
was started from an **image**: a packaged snapshot of the program
and everything it needs. The people who make the program publish
new images as they fix bugs and add features. Your running
container, though, keeps using the image it was started from,
forever, until somebody notices and updates it by hand.

**Watchtower is the somebody who notices.** It is a small program
that watches your running containers, checks the registry each
image came from to see whether a newer version of that image has
been published, and — when one has — does the update for you:
it pulls the new image, stops the old container, starts a fresh
container from the new image with the same settings the old one
had, and tidies up. You go from "I should really update those
containers one day" to "they were updated last night, and here
is the message saying so."

Three ideas carry the whole tool, and the rest of this guide is
detail hung on them:

- **A container is disposable; its settings are not.** Watchtower
  never patches a running container. It replaces it, recreating
  the replacement from the new image with the same ports,
  volumes, environment variables and command the original had.
- **The registry is the source of truth.** Watchtower compares a
  short fingerprint (a **digest**) of the image your container is
  running against the digest the registry currently serves for
  that image's tag. Different digest, new version — that is the
  whole detection method.
- **Watchtower holds the keys to the whole house.** It works by
  talking to Docker itself through Docker's socket, so it can do
  anything Docker can do. That power is the point, and it is
  also the risk; chapter 3 treats it honestly.

## Which Watchtower? Read this before installing anything

There are two projects with this name, and choosing wrong means
installing software nobody maintains.

The original Watchtower, by the containrrr team, was for years
the standard tool for this job. It is finished: its GitHub
repository is archived, its README opens with the words "This
project is no longer maintained", and its last release, v1.7.1,
dates from November 2023. It still works, and a great deal of
what you will find written about Watchtower — including the
clear documentation at containrrr.dev — describes it. But it
receives no fixes.

The living project is the fork maintained by Nicholas Fedor:
**nicholas-fedor/watchtower** on GitHub, published on Docker Hub
as **nickfedor/watchtower**. It is a continuation of the same
tool — same commands, same flags, same labels on your containers
— kept current. At the time this guide was written its latest
release was **1.23.2** (October 2026), and the repository had
been updated the same week. **This guide teaches the fork.**
Every command and flag in it was checked against the fork's real
1.23.2 program. Where the classic documentation is quoted for
behaviour both versions share, the text says so.

## What this guide's lab could and could not do

This guide was written in a lab with **no Docker daemon** —
Watchtower's whole job is talking to one, so the updates
themselves could not be run here, and the guide never pretends
otherwise. What the lab did run for real: the genuine 1.23.2
program, downloaded from the fork's GitHub releases with its
published checksum verified; its complete `--help` output, from
which every flag in this guide is taken; a run with no daemon
reachable, whose failure messages are quoted word for word; and
its built-in HTTP API, which answered real requests on this
machine — including a readiness probe that honestly reported
"Service Unavailable" because there was no daemon behind it.
Behaviour past that point — the pull, the swap, the restart —
is taught from the project's own documentation and labelled as
documented-only, here and in `criteria.md`, wherever it appears.
