# Linked Containers

Outcome: when one of your containers depends on another, an
update takes the group down and brings it back in an order that
works — dependents first on the way down, dependencies first on
the way up. This is the project's documented behaviour; nothing
here could be run in this guide's lab.

## The problem

Run a small blog: a **wordpress** container and a **mysql**
container it stores everything in. MySQL gets a new image. If
Watchtower simply swapped the mysql container while wordpress
kept running, wordpress would spend the swap talking to a
database that is stopping, gone, then starting — and, with old
Docker **links**, would keep connection details that point at
the old container entirely.

## What Watchtower does

Watchtower detects links between running containers and treats
the linked group as one unit. In the documented example: when
an update is detected for mysql, Watchtower first shuts down
the linked wordpress container, then mysql. On the way back it
starts mysql first, then wordpress, so the link still works
when wordpress comes up. The same ordering applies to the
recreate ceremony of chapter 2 — the whole group is cycled,
even the containers whose own images did not change, because
their partner's did.

## Saying it yourself: the depends-on label

Links are a legacy Docker feature; most stacks today are built
with Compose networks instead, where containers find each other
by name and there is no declared link to detect. For those, you
can state the dependency yourself, with a label on the
*dependent* container naming the containers it depends on,
comma-separated:

```
com.centurylinklabs.watchtower.depends-on=database,cache
```

Watchtower then orders that container's group the same way:
dependencies stopped last and started first.

One case is detected with no label at all: a container using
`network_mode: service:<container>` — sharing another
container's network — is treated as an implicit link, because
it cannot outlive the container whose network it borrows.

## With Compose

Compose files declare `depends_on` between services, and the
fork honours it when ordering updates: its help carries
`--use-compose-depends-on`, on by default, described as
"Include Docker Compose depends_on label when determining
container update order" (Compose records the declaration as a
label on each container, which is how Watchtower sees it). If
your Compose stack's start order matters — database, then
backend, then frontend — you have usually already declared it
in `depends_on`, and Watchtower will follow it for free.

## The honest caveat

Ordering solves *sequence*, not *readiness*. Watchtower starts
the database container and then starts the application — but
"container started" is not "database ready to accept
connections"; a database may need several seconds after start
before it answers. Applications that retry their database
connection (most do, or can be configured to) ride over this.
Applications that connect once at startup and give up can come
back from an update needing a manual restart. Chapter 13 lists
this among the limits; the practical fix lives in the
application's own configuration, not in Watchtower.
