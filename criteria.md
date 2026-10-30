# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   Example services are a generic blog, a media server, and a
   small database tool.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 13 names what the tool
   cannot do: it follows tags blindly across major versions,
   tests nothing, rolls nothing back, recreates rather than
   migrates, starts containers without knowing they are ready,
   and should never be handed databases or pinned services
   without the safeguards the chapter names.
8. **The ethics rule is in the text, early.** Watchtower holds
   root-equivalent power over its host through the Docker
   socket; chapter 1 raises the choice of a maintained source
   for such a program, and chapter 3 states the bargain
   plainly — full control of Docker, granted on purpose, to
   software you have reason to trust — before any procedure
   depends on it.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a shared Linux sandbox with **no Docker
daemon and no container runtime** — the one thing Watchtower
exists to talk to. The practical core of the tool (checking a
live registry against running containers, pulling, recreating)
could therefore not be run, and this guide is one of the
series' documented-only cases: that ceiling is stated in
chapter 1, in the README, and again here, rather than dressed
up. What the lab ran instead was the genuine program, as far as
it can go without a daemon — which turned out to be further
than expected.

Software identity, verified 2026-10-30:

- The original project, containrrr/watchtower, is **archived**
  on GitHub; its README opens with "This project is no longer
  maintained" and points to discussion 2135; its last release
  is v1.7.1 of 2023-11-11.
- The maintained fork, nicholas-fedor/watchtower (Docker Hub
  image nickfedor/watchtower), had its latest release **1.23.2**
  published 2026-10-22 and its repository updated within the
  lab week. This guide teaches the fork.

Run for real, with transcripts as evidence (all in
`watchtower-lab/transcripts/`):

- The genuine **watchtower 1.23.2** release binary
  (`watchtower_linux_amd64_1.23.2.tar.gz` from the fork's
  GitHub releases), its SHA-256 checked against the project's
  published `checksums.txt`: **OK** —
  `e8429673b67a61ff00cd6891c781b744a39f479ada9d8bfddc89c324d11c7ae8` (t00).
- The binary's complete `--help` output, 126 lines; every flag
  this guide names is taken from it, including the fork's
  additions (`--http-api-endpoints`, `--cooldown-delay`,
  `--disk-space-max`, the `notify-upgrade` subcommand) and the
  DEPRECATED markings on the legacy notification flags (t01).
- A `--run-once` execution with no daemon reachable: the full
  log quoted in chapter 4 — the startup ping warning, the
  version line, and the closing `Update execution failed`
  error naming the missing socket — and the observation that
  the process **exited 0** despite the failure (t02).
- `watchtower --version`: there is no such flag; the error and
  the resulting help dump were captured (noted in chapters 3
  and 14).
- Two environment variable names proven behaviourally: feeding
  the binary invalid values through `WATCHTOWER_POLL_INTERVAL`
  and `WATCHTOWER_COOLDOWN_DELAY` produced fatal configuration
  errors naming the matching settings, so the variables in
  chapter 3 and recipe 2 are read by the real program (t04).
- The HTTP API, run on the lab's assigned port 18088 with
  endpoints enabled and a token set: startup log lines;
  `/livez` → `OK`; `/startupz` → `OK`; `/readyz` → `Service
  Unavailable` (the probe honestly reporting the absent
  daemon); `/v1/metrics` attempted three ways — the documented
  `Authorization: Bearer <token>` form returned **401
  "missing or invalid API Key"**, the cookie form returned
  401, and the bare `Authorization: <token>` form returned the
  real 170-line Prometheus output, from which the
  `watchtower_*` metric names in chapter 11 are taken (t03).

Documented only — described from the project's documentation,
not run, and the text does not claim otherwise:

- Everything past a successful daemon connection: digest
  checks against a live registry, pulls, the stop/recreate/
  start ceremony, cleanup, self-update. Sources: the fork's
  quickstart and the classic documentation at containrrr.dev
  (full list in `watchtower-lab/sources.txt`).
- Container selection semantics (enable label, label-enable,
  monitor-only, scopes), from containrrr.dev's
  container-selection page.
- Linked-container ordering and the depends-on label, from
  containrrr.dev's linked-containers page, plus the fork's
  `--use-compose-depends-on` flag from its help output.
- Lifecycle hooks — labels, the 60-second default timeout,
  per-hook timeout labels in whole minutes, exit code 75
  skipping an update, hook UID/GID precedence, the
  `WT_CONTAINER` variable — from containrrr.dev's
  lifecycle-hooks page and the fork's own lifecycle-hooks
  documentation.
- Private registries: the config.json format, sharing a
  `docker login` file, credential helpers — from
  containrrr.dev's private-registries page. The registry TLS
  flags are from the real help output (t01).
- Notification behaviour beyond flag names: shoutrrr URL
  grammars belong to the shoutrrr project and are pointed to,
  not reproduced.
- The compose and `docker run` snippets are the fork
  quickstart's own forms, re-typeset; they were not executed.

Fixtures are this guide's own and deliberately generic: a
blog with a database, a media server, a dashboard — no names,
organisations or real events.
