# Private Registries

Outcome: Watchtower can pull your private images — from a
private repository on Docker Hub, a company's registry, or
your own — because it holds the same credentials your
`docker pull` uses. Documented behaviour throughout; the
registry TLS flags are quoted from the real help output.

## The problem

Everything so far assumed public images. The moment a
container runs a private image, Watchtower's check fails in a
way that looks, from its log, like the image vanished: it asks
the registry for the tag's digest and the registry, quite
rightly, demands to know who is asking. Watchtower needs
registry credentials of its own — it does not borrow the
credentials of the containers it watches.

## config.json: the file Docker already keeps

Docker stores registry logins in a file called **config.json**
(usually `~/.docker/config.json`), created when you run
`docker login`. Its heart is a set of `auths` entries — one
per registry, holding a base64-encoded username-and-password:

```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "XXXXXXX"
    },
    "registry.example.com": {
      "auth": "YYYYYYY"
    }
  }
}
```

Note the Docker Hub entry's key: for Hub the registry name is
the full `https://index.docker.io/v1/`, not `docker.io`.
Sharing your existing file with Watchtower is the standard
move — mount it into the Watchtower container at
`/config.json`:

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/.docker/config.json:/config.json \
  nickfedor/watchtower
```

You can also write the file by hand (the format above is the
whole format) if you'd rather give Watchtower a dedicated,
least-privilege registry account than your own — a registry
user that can only *read* images is all Watchtower needs, and
a read-only account in this file is a genuinely better idea
than your personal one. Either way, treat the file as a
password: the base64 is an encoding, not an encryption.

## Credential helpers

Where Docker is set up with a **credential helper** — the
programs that keep registry passwords in an operating-system
keychain or a cloud metadata service instead of in
config.json — the file holds a `credsStore` (or per-registry
`credHelpers`) entry naming the helper instead of an `auth`.
Watchtower supports this, with a catch worth understanding:
the helper is a program, and it has to exist *inside the
Watchtower container* for Watchtower to run it. The project's
documentation shows the pattern — build a small custom image
from the Watchtower image with your helper added — which is
the one place in this guide where extending the image is the
intended path.

## Self-hosted and plain-HTTP registries

Talking to a registry with a certificate your host does not
trust, or (on a closed home network) a plain-HTTP registry,
needs the matching switches. From the 1.23.2 help output:

- `--registry-tls-skip` — "Disable TLS verification for
  registry connections; allows HTTP or insecure TLS
  registries (use with caution)". The caution is the
  documentation's own: with verification off, whoever can
  intercept the connection can feed Watchtower an image of
  their choosing. Closed network, understood risk, or not
  at all.
- `--registry-tls-min-version` — the floor for TLS versions on
  registry connections, default TLS1.2.

Separately, if Watchtower itself must reach a *remote daemon*
over TLS (rather than a local socket), the daemon-side
counterparts are `--tlsverify` and `--cert-path` — chapter 2's
socket story with certificates; most readers will never need
them.
