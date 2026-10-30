# Lifecycle Hooks

Outcome: your containers can run their own commands at the
four turning points of an update — most importantly a backup
just before the old container stops — and you know exactly
what happens when such a command fails or runs long.
Behaviour here is the project's documented behaviour, from the
classic documentation and the fork's own docs; it could not be
run in this guide's lab.

## The four hooks

A **lifecycle hook** is a command Watchtower runs *inside one
of your containers* at a set point in the update cycle:

| Hook | When it runs |
|---|---|
| **pre-check** | for each watched container, before every update cycle |
| **pre-update** | in a container about to be updated, just before it is stopped |
| **post-update** | in the new container, just after it starts |
| **post-check** | for each watched container, after the cycle completes |

The classic uses write themselves: pre-update dumps a small
database to a volume before the swap; post-update runs a
migration or clears a cache; post-check sends a heartbeat
somewhere that expects one.

Hooks are off until switched on — `--enable-lifecycle-hooks`
on Watchtower (or `WATCHTOWER_LIFECYCLE_HOOKS=true`). That
switch is the *global* part of the feature: one setting on
Watchtower arms hooks for the whole host. The commands
themselves are *per container*, given as labels on each
container that has them:

```
com.centurylinklabs.watchtower.lifecycle.pre-check
com.centurylinklabs.watchtower.lifecycle.pre-update
com.centurylinklabs.watchtower.lifecycle.post-update
com.centurylinklabs.watchtower.lifecycle.post-check
```

```yaml
    labels:
      - "com.centurylinklabs.watchtower.lifecycle.pre-update=/usr/local/bin/dump-data.sh"
```

Commands run through the container's own `sh`, via Docker's
exec facility — so the container must actually contain `sh`
and whatever tools the command uses (a backup script needing
`jq` needs `jq` in that image). Minimal images built `FROM
scratch` have no shell at all and cannot run hooks. And a
container that is not running cannot run hooks either: the
documentation notes the update simply proceeds without them.

The hook command is told which container it is running in
through an environment variable, `WT_CONTAINER`, holding a
small JSON object — the container's name, ID, image name,
stop signal and Watchtower labels — for scripts that need to
know their own context.

## Timeouts

Every hook gets **60 seconds** by default; a command still
running at the limit is cut off and the update loop carries
on. Each hook's limit can be overridden with a label — for
pre-update, `com.centurylinklabs.watchtower.lifecycle.pre-update-timeout`
— whose value is a whole number of **minutes**; `0` disables
the timeout entirely. A long database dump belongs under a
raised pre-update timeout, not in a race against the default
minute. Anything that is not a whole number of minutes is
ignored with a warning, and the default applies.

## Failure, and the one magic exit code

A hook that fails — exits with any code other than 0 — is
logged and the update **proceeds anyway**. Hooks advise;
they do not veto. With one deliberate exception: if a
**pre-update** hook exits with code **75** (`EX_TEMPFAIL`,
the classic Unix "temporary failure, try later"), Watchtower
**skips that container's update** for this cycle. That is
the designed escape hatch: a backup script that could not
take its backup exits 75, and the container is left running
on its old image until the next cycle rather than updated
unprotected.

## Who the hooks run as

By default a hook runs as the container's own configured
user. Watchtower-wide defaults can be set with
`--lifecycle-uid` / `--lifecycle-gid`, and individual
containers override them with
`com.centurylinklabs.watchtower.lifecycle.uid` /
`.gid` labels — the fork's documented precedence is:
container label beats global flag. Set these when a backup
script must write to a volume owned by a particular user.
