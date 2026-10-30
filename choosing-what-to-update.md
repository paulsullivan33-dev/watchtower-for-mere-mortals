# Choosing What to Update

Outcome: Watchtower updates exactly the containers you intend —
all of them, only the labelled ones, or some of them in
report-only mode — and you know which knob does which. The
selection rules below are the project's documented behaviour,
shared by the original and the fork; the flags are quoted from
the real 1.23.2 help output.

## The default: everything

With no selection settings at all, Watchtower watches **every
container on the daemon**. For a home server running five
containers you own and understand, that is a reasonable place
to start. For anything else, narrow it. There are three
strengths of narrowing, and they combine.

## Exclude: the enable label set to false

To keep one container entirely out of Watchtower's sight — not
watched, not reported, not touched — label it false when you
create it:

```yaml
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
```

This is the setting for the container you update by hand on
purpose: the database you upgrade on a planned afternoon, the
thing pinned to a version for a reason.

## Opt-in: --label-enable

The opposite posture: Watchtower ignores everything *except*
containers labelled true. Turn it on with `--label-enable`
(`-e`, or `WATCHTOWER_LABEL_ENABLE=true`), then label the
chosen few:

```yaml
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
```

Opt-in is the safer default for a host you do not fully own —
a work machine, a shared server, anything where a surprise
update to somebody else's container would be a bad day.
Chapter 12's rollout recipe starts here.

## Monitor only: report, never touch

Between "update it" and "ignore it" sits the mode this guide
likes most. **Monitor only** containers are checked like any
other, and Watchtower reports — in its log, and through
notifications if you set them up — that a new image exists. It
just never applies the update.

- For everything: `--monitor-only` (`-m`,
  `WATCHTOWER_MONITOR_ONLY=true`).
- For one container: the label
  `com.centurylinklabs.watchtower.monitor-only=true`.

Run a new Watchtower in monitor-only for a week and you learn,
with zero risk, exactly what it *would* do to your stack and
how often. That week is step one of the rollout in chapter 12.

## When flags and labels disagree

By default the command line wins in one specific way: if you
pass container names or `--disable-containers` on Watchtower's
command line, those arguments filter the list *before* labels
are considered. `--label-take-precedence` reverses that, making
a container's own labels the final word over the arguments.
The documented selection rule, worth internalising, is that a
container is watched only if **all** configured criteria agree:
named on the list (if a list is given), not disabled by name or
by label, and — in opt-in mode — enabled by label. The fork
adds two list-style refinements visible in its help:
`--enable-containers-by-label` / `--disable-containers-by-label`
(restrict or exclude by arbitrary key=value labels) and
`--skip-image-names` / `--monitor-image-names` (filter by image
rather than by container).

## Scopes: several Watchtowers, one daemon

If you truly need two instances with different rules on one
host — one updating the media stack nightly, one only reporting
on the rest — give each a `--scope` name and label its
containers `com.centurylinklabs.watchtower.scope=<name>`. An
unscoped instance watches everything; a scoped one watches only
its own. Two unscoped instances on one daemon will fight over
the same containers, so scope is the answer *before* you run a
second instance, not after.
