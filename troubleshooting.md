# Troubleshooting

Outcome: the failures you are most likely to meet, each with
its tell-tale log line and its fix. The first three come from
this guide's own lab, word for word.

## "failed to connect to the docker API at unix:///var/run/docker.sock"

The most common Watchtower failure there is. From the lab's
real run (transcript t02):

```
level=error msg="Update execution failed" error="failed to list containers: failed to list containers: failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory"
```

The `level=warn` line a moment earlier — "Ping failed during
initialization. Will negotiate on first API call" — is not the
failure; it is Watchtower deciding to carry on and find out
later. The error at the end is the report. Causes, in order of
likelihood: the socket volume was not mounted into the
Watchtower container (`-v /var/run/docker.sock:/var/run/docker.sock`
missing); Docker is not running on the host; or the daemon
listens somewhere else — on rootless Docker the socket lives
under the user's runtime directory, and `--host` (or
`DOCKER_HOST`) must point at it.

A companion trap: that failed `--run-once` run **exited with
code 0** (verified in the lab). Scripts and systemd units that
only check the exit status will report success. Grep the log
for `Update execution failed`, or use `--porcelain v1` output.

## The API says "missing or invalid API Key" — with the right token

Verified on the 1.23.2 binary (transcript t03): sending the
token in the documentation's recommended form,
`Authorization: Bearer <token>`, was answered `401` and this
message. Sending the same token bare —
`Authorization: <token>`, the form Swagger UI uses — worked
immediately, as did the unauthenticated probes `/livez` and
`/readyz`. If your monitoring gets 401s from a healthy
Watchtower, change the header form before you change the
token. Remember also that `/v1/events` uses its own token
(`--http-api-events-token`), not the main one.

## "What version is this?" — there is no --version flag

`watchtower --version` answers `Error: unknown flag: --version`
plus the whole help text (verified). The version is printed at
every startup: look for the log line
`Watchtower 1.23.2 using Docker API v1.56`.

## Nothing ever updates, and nothing looks wrong

Work through the filters; one of them is silently doing its
job:

- **Digest already equal.** The registry's digest for the tag
  matches the running image — there genuinely is no update.
  `--debug` shows the comparison.
- **Monitor-only is on** — globally, or via the
  `monitor-only` label on that container (chapter 5). The log
  still reports the update; it just never applies it.
- **Opt-in mode without the label.** `--label-enable` is set
  and the container lacks `enable=true`: it is invisible by
  design.
- **A scope mismatch** — the container's scope label names a
  different instance (chapter 5).
- **A pinned digest.** A container started from
  `image@sha256:…` is nailed to that image; there is no tag
  for a registry to move.
- **Cooldown (fork).** `--cooldown-delay` is holding back an
  image younger than the delay — working as intended.

## An update broke the application

Watchtower has no rollback button (chapter 13). Recover by
hand: redeploy the previous image. If `--cleanup` was off,
the old image is still on the host — recreate the service
from it. If cleanup removed it, pull it again **by digest**
(`image@sha256:…`); registries keep old images addressable by
digest even after the tag has moved, and Watchtower's own log
of the update records both digests. Then label the container
`enable=false` while you decide what happened.

## Two Watchtowers, twice the updates

Containers being recreated twice per cycle, or updates
"un-happening", means two instances are watching the same
daemon — typically an old containrrr container never removed
after the fork was installed. Give each its own `--scope`,
or (usually right) delete the one you no longer mean to run.

## Notifications never arrive

Watchtower sends a startup message unless
`--no-startup-message` is set, which makes this testable in
seconds: restart Watchtower and watch the destination. No
message means the URL is wrong — shoutrrr URLs are grammars,
not guesses, and each service's exact form is in the shoutrrr
documentation — or the credentials inside it are. The
destination received the startup message but not update
reports? Then `--notifications-level` is filtering them:
reports ride at `info`, and a level of `error` sends only
failures.
