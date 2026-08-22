# UAS Hangar — releases

Published releases of **UAS Hangar**, an Android application that harvests flight logs from a ground
control station and archives them to the operator's own cloud storage.

**This repository holds releases only. There is no source code here.**

Published by **Redline Aerials, LLC**.

---

## What is in a release

| File | What it is |
|---|---|
| `UASHangar-<version>.apk` | The signed installer. |
| `update.json` | The version manifest the app reads on launch to decide whether an update exists. |

The app checks `releases/latest/download/update.json`, compares the version against its own, and offers
the update. It never downloads anything without being told to.

## Verifying what you downloaded

Every release is signed with the same key, and **Android refuses to install an update signed with a
different one.** That check is enforced by the operating system on every install — it needs nothing from
you, and it is what actually establishes that a build is genuine. A file that is not ours cannot replace
an installed copy; the install simply fails.

To compare a download against a build you already trust, the Android SDK's `apksigner` will print the
signing certificate of each:

```
apksigner verify --print-certs UASHangar-<version>.apk
```

Two files reporting the same certificate came from the same source. **If you need the reference
fingerprint to check a build in isolation, ask Redline for it directly** rather than taking it from a web
page — a fingerprint published beside a download is only as trustworthy as the page it sits on.

Each release's `update.json` also carries the APK's **SHA-256 file hash**, which the app verifies before
handing the file to Android's installer — that catches a download truncated by a poor link, which is a
likelier failure in the field than a tampered file.

## Installing

These builds are distributed directly rather than through an app store, so Android will not install one
until you allow the app doing the installing. The permission is called **Install unknown apps** — some
devices and Android versions call it *Install from unknown sources* — and it lives under **Special app
access** in Settings. **The exact path varies by manufacturer and Android version**, so search Settings
for "unknown apps" rather than following a fixed menu path.

Grant it to whichever app is doing the installing — your browser or file manager for a first install, and
UAS Hangar itself for updates after that. It is a one-time step per device.

## Requirements

- Android 10 (API 29) or later.
- A cloud account to archive to: OneDrive/SharePoint, Google Drive, or any WebDAV server.
- Each operator registers their own cloud application credentials. **No account details, tenant
  identifiers or client secrets are distributed with this app.**

## Licence

All rights reserved. Released here for use by authorised operators; this is not an open-source release.
