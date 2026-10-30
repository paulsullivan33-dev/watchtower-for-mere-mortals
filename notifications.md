# Notifications

Outcome: every Watchtower run tells you what it did — by
email, chat message, or webhook — instead of your finding out
from a user. The notification mechanism is documented
behaviour; the flag names are quoted from the real 1.23.2
help output.

## One URL per destination

Watchtower sends notifications through **shoutrrr**, a
library that turns a URL into a notification service. You
give Watchtower one or more URLs with `--notification-url`
(repeatable; environment form `WATCHTOWER_NOTIFICATION_URL`),
and the scheme of each URL chooses the service:

```
--notification-url "smtp://user:pass@mail.example.com:587/?from=watchtower@example.com&to=you@example.com"
--notification-url "slack://hook:TOKEN@CHANNEL"
--notification-url "discord://token@channel"
--notification-url "telegram://token@chatid"
--notification-url "gotify://gotify.example.com/token"
--notification-url "generic://webhook.example.com/path"
```

Email, Slack, Discord, Telegram, Gotify, Microsoft Teams and
generic webhooks are all in shoutrrr's repertoire, along with
dozens more; the shoutrrr documentation is the catalogue, and
each service's URL grammar lives there. Two practical notes:
the credentials are *in* these URLs, so give them to
Watchtower the way you would give it a password — environment
variables or a secrets file, not a command line that other
users on the host can read in the process list — and Watchtower
sends a short startup message when it launches, so you learn
within seconds whether the URL works, unless you silence it
with `--no-startup-message`.

## What gets sent

By default, a notification reports the run's log lines at
`info` level and above — in practice, the session summary and
any updates performed. Useful refinements, all in the real
help output:

- `--notification-report` — send the structured session report
  (scanned / updated / failed per container) as the message
  data instead of raw log lines.
- `--notification-template` / `--notification-template-file` —
  a Go `text/template` (shoutrrr's dialect) shaping the
  message, for fitting it to a chat channel's taste.
- `--notification-split-by-container` — one message per updated
  container instead of one grouped message.
- `--notifications-level` — the lowest log level worth
  sending; set it to `error` if you only want to hear about
  failures.
- `--notifications-delay` — hold notifications this many
  seconds before sending, so a flurry settles into one report.
- `--notification-title-tag` — a prefix on the message title,
  so one inbox can tell two Watchtower hosts apart.

## The deprecated flags you will meet in old posts

Older tutorials configure email with
`--notification-email-server`, `--notification-email-to` and
friends, Slack with `--notification-slack-hook-url`, and pick
services with `--notifications`. The 1.23.2 help output marks
every one of those **DEPRECATED**, each with the same
direction: use `--notification-url` with the matching shoutrrr
scheme instead. The fork even ships a migration helper — the
`notify-upgrade` subcommand, which converts a legacy
notification configuration into shoutrrr URLs. If you are
copying a configuration from a blog post, check its date; if
it uses the old flags, translate it rather than copying it.

## Why notifications are half the product

An updater you cannot hear from is an updater you have to
audit by hand — at which point you have rebuilt the chore
Watchtower exists to remove. Notifications are also what make
the safe rollout of chapter 12 work: a monitor-only Watchtower
with notifications is a *reporting service* that tells you,
for a week, everything it would have updated. Set up the
notification first; tune everything else after.
