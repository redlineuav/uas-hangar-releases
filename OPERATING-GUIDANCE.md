# Operating guidance

How to run UAS Hangar so it protects your flight records and does not become a liability on a tablet you
later lose, retire or hand to somebody else.

**This is guidance, not requirements.** Where the app can enforce something it does. What follows is the
part it cannot.

---

## 1. What the app does, and what it will never do

| It does | It never does |
|---|---|
| Reads flight logs your ground station wrote | Writes to, moves, or deletes the ground station's own files |
| Copies them onto the tablet, on a timer | Deletes anything from the ground station's own folder |
| Sends them to cloud storage **you** own | Sends anything anywhere else |
| Reads back three small files — the shared aircraft list, the withdrawal list and the operator brand | Downloads flights back onto the tablet |

**Your ground station remains the system of record.** The app is a copier and a courier. If it ever
disagrees with the ground station, the ground station is right.

---

## 2. Set up harvesting first. Archiving can wait.

**Harvesting needs no cloud account at all.** Give the app the source folder and the file-access
permission, and it starts copying flight logs onto the tablet immediately.

That order is not a convenience. **Ground stations rotate their own logs away** — older flights are
deleted to make room, on a schedule you do not control. A tablet that is harvesting is a tablet not losing
flights. A tablet waiting for somebody to find a cloud client ID is not.

⇒ **Set up harvesting on every tablet on day one.** Do the archive when you have time, from
**Menu → Advanced → Set up the archive**. Nothing is redone, and flights kept while there was no archive
are sent as soon as one exists.

A tablet in that state reports **Harvest only — ready**. That is a working tablet, not a broken one.

---

## 3. ⭐ Choosing the cloud account — the most important decision here

**Use an account created for the tablets. Not a person's account.**

Create a user — `tabletapp`, or whatever suits your naming — and grant it access to **the archive folder
and nothing else**. Every tablet signs in as that user.

**What this buys you, concretely:**

| | A person's account | A scoped tablet account |
|---|---|---|
| Tablet is lost or stolen | Exposes everything that person can reach | Exposes one folder |
| That person leaves | Every tablet stops working at once | Nothing changes |
| Auditing who did what | Mixed with their normal work | Tablet activity is separable |

**It also removes a nuisance.** Sign-in events are rare in normal operation, so the small inconvenience of
a separate account costs almost nothing and contains the blast radius when something goes wrong.

### Per provider

**Microsoft / OneDrive / SharePoint.** Create the user in your tenant and share only the archive folder or
library with it. The app needs its own application registration; the identifiers are configuration, not
secrets, but they are yours and should not be shared outside your organisation.

**Google Drive.** The app asks for full Drive access because it reads the archive as well as writing to it.
A dedicated account matters more here for that reason: the scope is broad, so what the account can reach
should be narrow.

**WebDAV / Nextcloud / Synology.** **Always use a server-issued app password, never the account password.**
WebDAV authenticates on every request, so the tablet must keep the credential. An app password can be
revoked from the server without touching the account. The app refuses plain `http` for this reason.

---

## 4. What a tablet holds, and what signing out actually clears

**Be precise about this, because the app cannot be more secure than the tablet it runs on.**

A tablet that has been set up holds:

- **The credential** for the cloud account — a token, or for WebDAV a password.
- **The deployment configuration** — application identifiers, server addresses, the destination folder.
  These are configuration rather than secrets, but they point at *your* organisation.
- **Flight logs** in its staging folder, until they are archived.

### Sign out of this cloud

Clears the credential from the tablet. On **Google Drive** it also **revokes** the grant at Google, so it
stops working everywhere.

🔴 **On Microsoft and WebDAV it clears the tablet only.** It does not revoke anything at the provider.

🔴 **And the browser stays signed in.** Sign-in for Microsoft and Google goes through the tablet's browser,
which keeps its own session. After signing out of the app, signing back in **may not ask for a password**.
That is the tablet's browser, not the app.

### So when a tablet is lost, stolen, or leaves your control

1. **Revoke at the provider**, not on the tablet. Change the account password, or revoke the app password
   at your WebDAV server. This is the only step that actually stops access.
2. Remove the tablet's device access in your identity provider if you have that capability.
3. **Do not rely on having signed out.** You may not have the tablet to sign out from.

### When a tablet moves to a different organisation

Use **Menu → Advanced → Forget this cloud setup**. That clears the configuration as well as the
credential, so the tablet is not carrying your identifiers into somebody else's hands. Clear the browser's
data as well.

---

## 5. Running several tablets

**One archive, many tablets.** They share it safely and none of them can overwrite another's work.

- **Usually one tablet, one aircraft.** A ground controller is normally tied to an airframe, so for
  telemetry logs — which carry no airframe serial of their own — the tablet supplies the identity.
- ✅ **But a tablet can serve several aircraft, and it files them all correctly.** The aircraft you set on
  a tablet is a **fallback, not a filter**: the app records which aircraft a flight belongs to **when it
  picks the flight up**, so a tablet reading more than one source archives each flight under its own
  aircraft. Observed on 2026-08-23 — one tablet archived 43 flights across 4 aircraft, each in the right
  place. **A shared or bench tablet is a supported way to work**, not something to avoid.
- ⚠️ **Changing a tablet's aircraft does not re-label flights already picked up**, and it should not. They
  keep the aircraft they were harvested as.
- **The aircraft register lives in the archive**, not on the tablets. Add an aircraft on one tablet and the
  others pick it up. Two tablets editing different aircraft both keep their edits.
- **Every archived flight records which tablet filed it**, so "where did this come from" is answerable
  later without guesswork.

### Replacing a tablet

Each tablet keeps a copy of its pick-up record in the archive. On the replacement, use
**Advanced → Restore the pick-up record from the archive** and choose the record belonging to the tablet
being replaced.

🔴 **Choose carefully.** A restored record tells the app those flights are already handled. Restore the
wrong tablet's and its flights will never be picked up on this one, **and nothing will warn you** — the
failure is an absence. The chooser shows which tablet each record came from and when it was written.

---

## 6. Updates

The app checks for a newer version when it starts, **and once a day after that** — a tablet that lives on a charger and never restarts still finds out. It tells you about a version once rather than on every check. **Later** keeps quiet until the next time
you open the app.

**Android needs permission to install, granted once per tablet.** It is called **Install unknown apps** —
some devices say *Install from unknown sources*. Search Settings for "unknown apps" rather than following
a menu path, which varies by manufacturer.

Android may also show its own confirmation, and a Play Protect scan may run. Both are the operating
system's, not this app's.

**Updates are safe to take.** Every release is signed with the same key and Android refuses any update
signed with a different one. Nothing archived is affected, and nothing on the tablet is cleared.

---

## 7. Knowing it is actually working

**Run `verify`.** Menu → Verify. It reads the archive and compares what is there against what this tablet
believes it sent.

🔴 **Nothing else is evidence.** The counts on the home screen come from the tablet's own record of what it
*believes* it did. `verify` is the only thing that reads the destination.

**On the home screen:**

| | Means |
|---|---|
| **Ready** | Both halves working |
| **Harvest only — ready** | Harvesting fine, no archive set up. Flights are accumulating on the tablet |
| **Harvesting needs attention** / **stopped** | Flights may not be being collected. Deal with this first |
| **Archiving needs attention** / **stopped** | Flights are being collected but not sent |

**"Flights waiting" is normal.** It counts flights harvested but not yet archived. It should fall to zero
when the tablet has a network. **Worry when it climbs steadily and never falls** — that means the archive
half is not working, and the home screen will say which part.

**Check `verify` after any change** to the destination, the provider, or the aircraft register.

---

*Published by Redline Aerials, LLC. Behaviour described here is what the app does today; where something
is not yet proven on hardware, the release notes say so.*
