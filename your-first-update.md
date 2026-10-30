# Your First Update

Outcome: you have watched one complete update cycle — or one
complete failure — and can read Watchtower's log well enough to
tell the two apart. The successful cycle is described from the
project's documentation (this guide's lab has no daemon to run
one against); the failure is quoted from the lab's real run.

## Run once, by hand, first

Before trusting Watchtower to a schedule, run it once in the
open. `--run-once` (short form `-R`) performs a single check of
everything it watches and then exits:

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  nickfedor/watchtower --run-once
```

In that one pass, for each watched container, Watchtower:

1. asks the registry for the digest behind the container's
   image tag, and compares it with the running image's digest;
2. where they differ, pulls the new image;
3. stops the old container (its stop signal, then a wait of
   `--stop-timeout`, 30 seconds by default, then force);
4. creates and starts the replacement with the old container's
   settings, in dependency order if containers are linked
   (chapter 7);
5. if `--cleanup` was given, deletes the now-unused old image.

The log narrates each step at the default `info` level and ends
with a session summary — how many containers were scanned, how
many updated, how many failed. `--debug` shows the working in
full, including every digest compared; it is voluminous, and
worth one read on a quiet afternoon. Two flags reshape a manual
run usefully: `--monitor-only` turns the whole run into a
report — checks everything, updates nothing (chapter 5 makes
this the heart of a safe rollout) — and naming containers on
the command line limits the run to just those.

## What failure looks like (verified)

Failure is the part this lab *can* show you, because its machine
has no Docker daemon at all. Running the real 1.23.2 program
with `--run-once` there produced this log, unedited
(transcript t02):

```
time=2026-10-30T22:24:32Z level=warn msg="Ping failed during initialization. Will negotiate on first API call" error="failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory"
time=2026-10-30T22:24:33Z level=info msg="Watchtower 1.23.2 using Docker API v1.56"
time=2026-10-30T22:24:33Z level=info msg="Using no notifications"
time=2026-10-30T22:24:33Z level=info msg="Running a one time update"
time=2026-10-30T22:24:33Z level=error msg="Update execution failed" error="failed to list containers: failed to list containers: failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory"
```

Read it the way you would read it on your own host at 2 a.m.:
the first line is a *warning*, not the failure — Watchtower
could not ping the daemon at startup and decided to carry on
and negotiate later. The failure arrives at the end,
`Update execution failed`, and its error names the real
problem precisely: it cannot connect to the Docker API at the
socket path. On a real host this exact message means the socket
mount is missing or wrong, or Docker itself is down — the
single most common Watchtower problem there is, which is why
this guide shows it in full.

One more lab observation, filed under things worth knowing
before you script around Watchtower: that failed run **exited
with code 0**. The failure is in the log, not the exit status.
If you wrap `--run-once` in a script or a systemd unit, check
the output, not just `$?` — or use `--porcelain v1`, which
writes session results to standard output in a stable,
machine-readable form made for exactly that.
