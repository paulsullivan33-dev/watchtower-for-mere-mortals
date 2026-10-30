# The HTTP API and Metrics

Outcome: Watchtower answers questions over HTTP — "are you
alive?", "what would you update?", "update now" — and reports
counters a monitoring system can graph. Unusually for this
guide, most of this chapter was **run for real** in the lab:
the API needs no daemon to start, so it served genuine
requests here, with the daemon's absence showing up in its
answers exactly as designed.

## Turning it on

The fork generalises what the original offered as separate
switches into one: `--http-api-endpoints`, taking a
comma-separated list of endpoint groups — `health`, `update`,
`metrics`, `containers`, `check`, `history`, `images`,
`config`, `events`, `swagger` — or `all`. The port is set by
`--http-api-port` (default 8080; this lab used its assigned
18088). With endpoints enabled and no schedule flags,
Watchtower sits idle until asked; add
`--http-api-periodic-polls` if the usual interval or schedule
should keep running alongside the API. The lab's run logged:

```
level=info msg="HTTP API enabled and periodic updates disabled"
level=info msg="HTTP API server is enabled" host=0.0.0.0 notify=no port=18088 tls=false
```

(transcript t03). Note `tls=false` and host `0.0.0.0`: by
default the API listens on all interfaces **without**
encryption. The project's documentation is blunt — never
expose the HTTP API directly to the public internet — and the
options exist to bind it to one interface (`--http-api-host`),
wrap it in TLS (`--http-api-tls-cert` / `--http-api-tls-key`),
and rate-limit authentication attempts. On a home network,
bind it to the host's LAN address, or better, to localhost
and let your monitoring come to it.

## The token, and a verified quirk

Every data endpoint requires a token, set with
`--http-api-token`. The events endpoint (`/v1/events`, a
stream of update events) takes a **separate** token,
`--http-api-events-token`, deliberately — event streams are
often consumed by browsers, whose URLs end up in logs, so
their token is kept apart from the master one.

Now the quirk, verified three ways on the 1.23.2 binary
(transcript t03). The documentation recommends sending the
token as `Authorization: Bearer <token>`. In this lab that
form was **rejected** — `401`, body `missing or invalid API
Key` — as was the documented `access_token` cookie. The form
that worked was the token alone in the header, the way
Swagger UI sends it:

```bash
curl -H "Authorization: <your-token>" http://host:8080/v1/metrics
```

That is what this build, on this day, actually accepts; the
discrepancy is reported here rather than smoothed over, and
chapter 14 keeps it as a troubleshooting step. (A later
release may restore the documented behaviour — test your
build the same way rather than trusting either text.)

## The endpoints

- **Health probes** — `/livez`, `/readyz`, `/startupz`,
  unauthenticated, in the Kubernetes style: is the process
  alive, has it started, is it *ready*. Ready is the
  interesting one: with no daemon reachable behind it, the
  lab's Watchtower answered `/livez` with `OK`, `/startupz`
  with `OK`, and `/readyz` with **Service Unavailable** — a
  probe that distinguishes "running" from "able to work",
  which is precisely what a probe is for.
- **`/v1/update`** (POST) — trigger an update run now; the
  on-demand form of `--run-once`. Classic setups pair this
  with a deploy pipeline: the pipeline builds and pushes an
  image, then pokes this endpoint, and Watchtower performs
  the swap.
- **`/v1/check`** — report what *would* update, without
  updating: monitor-only as a question.
- **`/v1/metrics`** — Prometheus-format counters (next
  section).
- **`/v1/containers`, `/v1/images`, `/v1/history`,
  `/v1/config`, `/v1/events`** — the watched containers and
  their current digests, past session results, the effective
  configuration, and the live event stream.

## Metrics worth graphing

From the lab's real `/v1/metrics` response (transcript t03),
the counters that matter:

```
watchtower_containers_scanned     per-scan watched total
watchtower_containers_updated     containers updated
watchtower_containers_failed      containers that failed to update
watchtower_containers_skipped     skipped (labels, scopes, filters)
watchtower_scans_total            completed scans
watchtower_scans_failed_total     failed scans
watchtower_docker_images_bytes    disk used by images
```

`watchtower_containers_failed` above zero, or
`watchtower_scans_failed_total` climbing, is your early
warning that updates have quietly stopped working — the exact
failure mode an unattended updater must never be allowed to
have. Point Prometheus at the endpoint and alert on those
two; the rest is bookkeeping.
