# tapri.dashovia.cafe

The website for **Tapri**, and the Sparkle update feed it serves.

Both live in the same deploy, so publishing the site publishes the updates —
one thing to push, one place for it to live.

```
index.html                the landing page
updates/
  appcast.xml             the feed Sparkle polls, signed
  Tapri-0.1.1.dmg         current release
  Tapri-0.1.0.dmg         previous release, kept so deltas work
  Tapri2-1.delta          12 KB patch from 0.1.0 to 0.1.1
render.yaml               the whole Render configuration
```

## Deploying

Render → Blueprint → point it at this repository. `render.yaml` carries
everything. Or by hand: **Static Site**, empty build command, publish
directory `.`.

The custom domain must be served over **HTTPS**. Sparkle refuses a plain-HTTP
feed, and rightly so — that feed decides what code runs on someone else's Mac.

## Publishing a new version

The disk image and the feed are built in the app repository, because the feed
has to be signed with an EdDSA key that must never leave the release machine's
Keychain:

```bash
./scripts/release.sh      # build universal, sign, notarise, staple
./scripts/appcast.sh      # sign the build, regenerate the feed
```

Then copy `site/index.html` and `site/updates/` here and push. Render deploys
on push.

**Keep the previous `.dmg`.** Sparkle uses it to generate a binary delta — the
0.1.0 → 0.1.1 patch is 12 KB against a 5.9 MB full download.

## Why the cache headers are what they are

- `appcast.xml` — **5 minutes**. Sparkle polls daily; a long cache would hide a
  release for hours after you ship it.
- `.dmg` and `.delta` — **one year, immutable**. The filename carries the
  version, so a new release is always a new URL and nothing is ever overwritten.

## Verification

Every download is checked against `SUPublicEDKey` in the app bundle before
anything installs. A tampered feed, or a tampered disk image, fails that check
and is refused. The private key is not in this repository and never will be.
