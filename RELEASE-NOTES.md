# Release notes

Newest first. Published by **Redline Aerials, LLC**.

---

## 1.0.2

**An aircraft that is not on your fleet list no longer archives.**

### What changed

A flight is sent to the archive only if its airframe serial is **on the fleet list**. A serial the
list does not carry is **not sent**, and the activity log names it:

```
NOT ON FLEET SN12345678: <flight> not sent -- add the serial to the fleet list to archive it
```

The pass summary counts them separately — *"3 sent, 12 already there, 0 failed, 41 not on the fleet
list"* — so they can never be mistaken for flights that were already up there.

🔴 **Nothing is deleted, and this is reversible by one edit.** The flight stays on the tablet exactly
as it was. **Add the serial to the fleet list and it archives on the next pass**, with no other
action and nothing to recover. A refusal here costs you a pass; it never costs you a flight.

### Why

A tablet coming back online sent **hundreds of flights** for a serial that had never been on the
fleet list, into `UNKNOWN-AIRCRAFT`, in one pass. Some of them had been deliberately removed from
the archive and came straight back. Nothing in the app had ever asked whether an airframe was one of
yours.

### What is deliberately NOT blocked

Two cases look similar and are not, and both still archive:

- **A tablet nobody has set up yet**, which reports no serial at all. Those flights are real and
  still reach the archive, under `UNKNOWN-AIRCRAFT`, exactly as before. There is no serial there to
  be missing from a list.
- **An aircraft on the list with no manufacturer or model filled in.** It is on the list, so it
  archives — it just files under `UNKNOWN-AIRCRAFT` until you complete its entry.

⚠️ **And if the fleet list cannot be read at all, nothing is blocked.** A tablet whose list has not
synced yet keeps archiving everything. Refusing on a missing list would turn one unread file into a
tablet that quietly stops sending.

### One thing to watch

A blocked flight stays staged on the tablet and **is never cleaned up**, because *Free up space*
only removes flights the archive has confirmed. A tablet holding a foreign aircraft's history will
keep holding it until the serial is added to the list or the flights are removed by hand.

---

## 1.0.1

**Fixes for 1.0.0, and the two features its published build never carried.** Most of what follows
corrects code that the 1.0.0 notes *described*; the published 1.0.0 artifact did not contain it.

### An aircraft's in-service date is checked before it is trusted

The date has to be written `YYYY-MM-DD`. Anything else is **ignored**, and the activity log says which
aircraft and what was typed.

🔴 **This matters more than it sounds.** A date like `9/1/2026` used to stop that aircraft archiving
anything at all — not some flights, all of them, for as long as the date stayed in the fleet list. The
only sign was a line in a log that rolls over.

**Two other corrections to the same rule.** It now works for an aircraft filed under `UNKNOWN-AIRCRAFT`
— one whose manufacturer or model is blank — where before it silently did nothing if that aircraft had a
nickname. And it can no longer reach aircraft it was never set on: a date set on an aircraft whose name
matched a manufacturer or model folder used to apply to everything filed underneath it.

### What the log will and will not tell you

**At most five bad in-service dates are named per pass**, then a line saying how many more there were.
The log holds 500 lines and a fleet with several bad dates would otherwise flush everything else out of it.

### Two aircraft that share an archive folder name lose the rule, rather than share it

If two aircraft would file into the **same folder** in the archive, **neither aircraft's date is applied
to that folder** and the activity log says so. Picking one silently would apply one airframe's history to
the other's flights.

**Two ways that happens, and they do not behave the same.** Two aircraft sharing a **nickname** file into
one folder on a tablet using the simple layout, and that folder loses the rule **whatever dates the two
carry — including none at all, and including the same date on both.** Two different names that merely
clean down to the same folder name lose it only when their dates **differ**; the same date on both is not
a clash and works normally.

🔴 **The nickname case was still wrong until this build, and it withheld flights.** An aircraft whose date
field is **blank**, or holds something the app refuses — `08/25/2025`, written the way most people write a
date — used not to count as a claim on the shared folder at all. The other aircraft's date then governed
that folder on its own and **held back the undated aircraft's flights**, against the rule that a blank
field can never discard one. All the operator saw was a log line saying the refused date had been ignored.

⚠️ **What is lost is the folder, not the aircraft.** Where either aircraft files somewhere that did not
clash, its own in-service date still applies there as normal. The 1.0.0 notes said *nothing of either is
withdrawn by date*, which read as a blanket promise and was broader than the code.

### `Verify archive` reports a lost flight again

A flight missing from **both** the tablet and the archive is reported **MISSING**. In 1.0.0 an
in-service date could make it count as withdrawn instead, so the one check that says *this flight exists
nowhere* could be silenced by a fleet setting. Only what you wrote in `withdrawn.json` silences it now.

### A `withdrawn.json` that cannot be read no longer disables your in-service dates

Those dates come from the fleet list, not from that file, so a network blip reading one has no business
switching off the other. It used to, for that pass only — which meant pre-service flights went up again,
unpredictably.

### The *Free up space* card says what it actually does

**It waits seven days** after a flight is staged before removing anything, and that was not written down
anywhere outside the card itself — so an operator who turned it on and saw nothing removed had no
explanation. **What goes:** the converted flight, its receipt, and this tablet's staged copy of the ground
station's record. **What never goes:** the ground station's own folder.

It said it never deletes *"the original record the ground station wrote."* It does remove **this tablet's
staged copy** of that record, once the archive is confirmed to hold it. **The ground station's own folder
is never touched** — that part was always true, and the card now says which is which.

---

## 1.0.0

⚠️ **PUBLISHED 2026-09-15, AND THE BUILD THAT SHIPPED DOES NOT CONTAIN TWO OF THE FEATURES DESCRIBED
BELOW.** The release carried a build made before the *Free up space* control and the per-aircraft
in-service date existed. **1.0.1 contains everything in this section**, so take the update. The same thing
happened to 0.9.10 and the note there says so too.

**The first release that is not a beta.**

Everything below is new since 0.9.11. Coming from further back, read the sections beneath this one too.

### Flights you delete from the archive now stay deleted

This app lists the archive on every pass and treats **the archive, not its own memory, as the truth.** A
flight that is on the tablet but not in the archive reads as missing, so it goes up again. That rule is
why an archive which genuinely loses a file gets it back — and it is also why deleting a flight you did
not want never worked. The app could not tell *you deleted this on purpose* from *the archive lost this*.

**Now you tell it, and there are two ways.**

**An aircraft's in-service date does it on its own.** Set the day the aircraft entered *your* service on
its entry in the fleet list, and flights dated before that day are never archived — for that aircraft
only. Each one carries its own date, because they do not all arrive on the same day. Nothing else to
write.

**For anything else, write `withdrawn.json`** in the archive's `Admin Data` folder, beside `fleet.json`:

```json
{
  "before": "2024-01-01",
  "names": ["AB123456_2024-07-20_19-52-11.tlog"],
  "folders": ["Old Hire Aircraft - 1234567890"]
}
```

All three keys are optional; use the ones you need. Dates are read from the flight's own name — the day
you flew, not the day the file landed on the tablet. A flight **on** the `before` date is kept.

**The list lives in the archive, so every tablet honours it.** Write it once. It survives a reinstall, a
factory reset and a new tablet, because it is not on a tablet.

**Withdrawing deletes nothing** — not the copy on the tablet, not the record the ground station wrote. It
stops the upload. Delete the archived copy yourself; withdrawing is what keeps it from coming back.

**`Verify archive` counts them as withdrawn** rather than reporting them missing, so the check can still
come back clean.

⚠️ **If the list cannot be read, nothing is withdrawn that pass** and the app carries on archiving
normally. A withdrawn flight uploaded once more can be deleted again; a flight that never reaches the
archive at all may be the only copy. The activity log says when the file is unreadable.

### A tablet that never restarts now checks for updates on its own

The update check ran when the app started. A tablet that lives on a charger and is never restarted
therefore never checked. **It now checks once a day as well.**

**And it tells you about a version once, not every day.** The prompt used to reappear on every check
until you acted on it. It now appears for a version you have not been told about, and clears itself when
you take the update.

### Converted DJI flights draw their track in log-analysis tools

Converted flights were written with the same MAVLink sequence number on every frame. Readers use that
number to spot dropped frames, so a file that never increments it reads as one frame repeated — and the
track did not draw. **The numbers now increment properly.**

⚠️ **This applies to flights converted from now on.** Flights already in the archive are not revisited.

### Nothing empty reaches the archive

An empty file made it all the way into the archive, and the archive check passed it. Three things
changed: **an empty file is not staged, an empty file is not uploaded, and the archive check now reports
an empty object as a problem** instead of counting it as correctly archived.

A pass interrupted part-way can also leave a partial file behind. **Those are cleared at the start of the
next pass** — only the ones that are genuinely empty. Anything with bytes in it is reported to you rather
than removed.

⚠️ **If the archive check starts reporting an empty object, it was already there.** The check got better,
not the archive worse.

### Free up space on this tablet

A new control on the **Advanced** screen. It removes the app's own converted copies of flights the
archive already holds — and **it asks the archive first, every time**, so the last copy of a flight is
never the one it deletes.

**It never touches the ground station's own folder.** That file is the aircraft's, not the app's.

⚠️ **Corrected in 1.0.1 — this said *"never touches the record the ground station wrote"*, and that was wrong.** The app keeps its own staged copy of that record beside the flight, and **that copy is removed once the archive is confirmed to hold it.** The ground station's own folder is untouched, which is the part that was always true.

**Off unless you switch it on**, and the screen says plainly what it will and will not delete.

### Still not fixed

**Sequence numbers are not applied to flights already in the archive** — those appear as new flights are
collected.

⚠️ **Original records ARE retroactive, and this section said otherwise.** A flight archived by an earlier
build that is **still on the tablet** gets its original record sent up on the next pass that meets it. Only
flights whose staged copy is already gone stay as they are.

---

## 0.9.11 Beta

**This is a beta, and it replaces 0.9.10** — everything in the 0.9.10 section below is in this release
too. Read both.

🔴 **If you already have 0.9.10, take this one.** The 0.9.10 release was published with a build that did
**not** include the landscape keyboard fix, even though its notes described it. That fix is in this
release, and it is the only difference between the build you have and this one.

### The archive check now tells you whether your original records are there

0.9.10 started keeping the original record beside each converted flight. This adds the one thing missing
from that: **the archive check counts them.**

```
VERIFY OK
352 of 352 archived correctly
348 of 352 converted flights have their original record
```

**It is a count, not a complaint.** Every flight archived before 0.9.10 has no original record and never
will, so a shortfall is not reported as a problem and does not change the verdict — it is simply stated,
the same way the check already tells you how many flights went to a different cloud. A tablet whose
source needs no conversion never sees the line at all.

The number comes from **the archive itself**, not from what this tablet believes it sent.

---

## 0.9.10 Beta

⚠️ **PUBLISHED 2026-08-27, AND THE BUILD THAT SHIPPED DID NOT CONTAIN THE KEYBOARD FIX BELOW.** The
release carried an earlier build than the notes described. **0.9.11 contains everything in this section,
built correctly.** If you are on 0.9.10, update.

**This is a beta.** Ten fixes, and two of them are the reason to take this release promptly: **the
keyboard covering the account field during setup**, and **two date fields that would not accept typing.**

### The keyboard no longer covers the account field in landscape

On some tablets the on-screen keyboard sat over the account line during first-time setup, and the screen
could not be scrolled far enough to bring the line back into view. It is the first screen of first run, on
a UI that is landscape-first, so there was no comfortable way around it.

The cause is a platform change: from Android 15 the system stops shrinking an app's window to make room
for the keyboard and expects the app to do it. This app was not doing it. It does now, and it works out
how much room to make rather than assuming — so tablets on older Android, which never had the problem
because the system still shrinks the window for them, are unaffected.

⚠️ **This one has not yet been watched on a tablet.** The reasoning is sound and the change is small and
self-limiting, but if the setup screen behaves oddly around the keyboard, that is the thing to report.

### You can type a date again — and one of those dates decides what the archive takes

**The two date fields would not accept typing.** An aircraft's in-service date and *Ignore flights before*
both refused keyboard input on every tablet tried, across four Android versions. Both accept a typed date
now, and both check it before it is stored.

**The archive cutoff asks about the FLIGHT's date, not the file's timestamp.** *Ignore flights before* was
comparing your date against when the file happened to land on the tablet's storage — which is not when the
flight happened, and on a tablet that has ever had files bulk-copied or restored onto it, is nowhere near
it. On one tablet every file carried the same recent timestamp for flights spanning six years, so no
cutoff could have worked at all. It now reads the date out of the record itself.

⚠️ **If you set a cutoff on an earlier version, check what the archive holds.** Flights outside your
cutoff may already have been archived under the old comparison. Nothing was lost — but flights you meant
to exclude may be there.

### The archive now keeps the original record beside the converted flight

For a source the app has to convert, the archive used to hold only the converted flight and its
provenance file — and that provenance file tells you to cite **the original record** if the flight is ever
questioned. The original was not in the archive.

**It is now filed beside the flight**, under the flight's own name without the `.tlog`. Storage cost
measured on a full tablet before this was built: about **1.2×**, not the several-fold figure it looks
like it should be.

⚠️ **This applies to flights collected from now on.** Records the app has already dealt with are not
revisited, so the originals appear as new flights are collected.

### You can tell it you have seen a refusal

Some records can never be converted — an encrypted format, a structurally broken file. Those were
reported forever, and the caution they raised could never clear, which is the fastest way to teach
somebody to ignore a caution. **Accept a refusal and it stops counting against you.** It is still listed,
still on the record, and still re-processable if a future version learns that format — accepting changes
what you are told, never what the app will do.

### It tells you it is working

A long collection pass looked identical to a frozen app. There is now a **spinner** on the home screen
whenever a pass is running, with a line beside it — and if the work has been quiet for a while it says
so, in as many words, without claiming anything is stuck. A large flight on a slow connection looks
exactly like that and is fine.

### Your name and logo live in the archive

Set them on one tablet and the next tablet **finds them** rather than having to be told. They are read on
every pass and written only when you press publish.

### Smaller things

**A swipe no longer throws away what you typed into the add-an-aircraft dialog.**

**"Open settings" on the app-hibernation warning opens the hibernation screen**, not battery optimisation.
Two different settings, one button, and it was going to the wrong one.

**Notifications really do say when they are switched off now.** 0.9.9's notes promised this and the
message could not actually be reached. It can.

### Still not fixed

Nothing known from this round is left open.

---

## 0.9.9 Beta

**This is a beta.** It is the first release since 0.9.5, and it carries everything from the four builds
in between — those were internal test builds and were never published, so their sections below are
folded into this one.

### One pass at a time, and the guard now sits with the thing it protects

**Two taps no longer start two archive passes.** Pressing *Archive now* or *Check now* twice, or having
a scheduled pass fire while you are pressing a button, could previously let a second pass run part of
its work underneath the first — including the step that writes the shared aircraft list. Both buttons
and the timer now go through one path, and the second one is refused and says so.

**The same is true of the aircraft list on its own.** Editing your fleet, or tapping *Read the fleet
list* on the home screen, could run at the same moment as a scheduled pass doing the same thing. On a
shared cloud folder that meant two devices' worth of edits being merged and written at once. A sync
that arrives while one is already running now waits its turn rather than overlapping, and tells you
that is what happened instead of reporting an empty result.

**Restoring a pick-up record checks that the backup is really this tablet's.** The warning that says a
backup came from a *different* tablet was being decided by a comparison that did not match the check
the app documents — so in some cases it could stay silent when it should have warned. Restoring another
tablet's record marks flights as already collected when they never were, and nothing reports it
afterwards, so this one is worth the care. If the app cannot tell, it now warns.

### Staged flights

**A flight can no longer be renamed over another flight.** There was a brief window between the app
checking a name was free and using it. If something took the name inside that window, the new copy
replaced the old one silently. The check now happens immediately before the file is put in place.

### Smaller things

**Notifications say when they are switched off.** If notification permission was refused, an update
notice simply never appeared. The app now records that notifications are off, so a tablet that seems to
be missing update prompts explains itself.

**Icons tint correctly on every theme.** Three icons used an older tinting attribute that newer Android
versions may ignore.

---

## Included in 0.9.9 — internal build RD-0012, never published on its own

### The archive check tells you the truth in four more places

**A file that is not yours no longer counts as your flight.** If you share a cloud folder with another
tablet, and one of your uploads failed, the check could find that tablet's flight under the same name and
report yours as safely archived. It now says where the file it found actually is, and does not count it.

**A failed pass says what went wrong, not just how many.** "1 failed" with the reason buried in a log on
another screen is not a report. The reason now comes back with the count.

**A WiFi network that wants you to sign in through a browser is named as such.** Hotel, airport and guest
networks answer every request with their own login page. The app used to show you the error from the part
of it that was trying to read that page as data. It now tells you to open a browser.

**Renaming an aircraft no longer hides its folder from the archive check.** After a rename the check
stopped looking at that aircraft entirely — so anything left behind in its folder went unreported. And if
another tablet had already applied the same rename, the app could create an empty folder and report files
that never moved. Both are fixed.

### Two things that protect a shared archive

**A typed destination path is now matched by what it says, not by its exact characters.** A path pasted
out of an email can carry invisible characters that look like an ordinary space. That used to create a
second archive folder that looked identical to the real one, and everything after it went into the wrong
place.

**A tablet keeps one identity.** On a rooted tablet, a superuser prompt that was declined once could make
the app record that tablet under a second name — splitting its own records in two.

### Housekeeping

Unused code and unused resources removed, and a checker now runs on every build to keep them from
accumulating. No change to what the app does.

---

## Included in 0.9.9 — internal build RD-0011, never published on its own

### Setting up a tablet that already has a lot of flights on it

**Setup now asks a date: "Ignore flights before".** Some ground stations never delete anything, so a
tablet can be carrying years of flights — including flights from before you owned the aircraft. Enter the
date you started flying it and the app leaves everything older where it is.

**Leaving it blank is allowed, and the app tells you what it means**: every flight at the source gets
picked up, however old. One tablet in service holds over three thousand records going back to 2019.

**It is asked again whenever you change the ground station**, because a date that made sense for one is
usually wrong for another.

**It does not apply to Auterion Mission Control**, which keeps only its newest ten telemetry logs — there
is no backlog for a date to hold back. The app says so on the screen rather than leaving you to wonder.

*The date must be written `YYYY-MM-DD`. Anything else is refused rather than guessed at — `01/08/2025`
means two different days in two different countries, and a date read the wrong way would quietly skip
flights.*

---

## Included in 0.9.9 — internal build RD-0010, never published on its own

**Everything 0.9.9 was going to bring, plus four things found by running the app on a tablet for a full
session rather than reading the code.** 0.9.9 was built and never released.

This is the first release checked end to end on an unrooted tablet — 211 flights harvested, converted and
archived in one sitting, with no failures.

### Checking the archive

**`Verify` now tells you when a flight has been archived in two places.** Before this it looked for each
flight, found one copy, and called it correct — so an archive holding the same flight twice was reported
as clean. Two copies of a flight with nothing to say which is the real one is exactly the thing `verify`
exists to catch, and now it says so.

**Renaming aircraft folders no longer offers a change it cannot make.** When flights are archived before
the app knows which aircraft they came from, they go into an `UNKNOWN-AIRCRAFT` folder. Adding that
aircraft afterwards used to offer to rename that folder — but a rename can only change a folder's name,
not where it sits, so the flights stayed in the unknown area under a new name and were then archived a
second time. The app now explains why it will not do that, instead of doing it.

*If you have already applied one of those renames, the flights are safe — there are simply two copies.
`Verify` will now point them out.*

### Flight records

**Provenance receipts name the file your ground station actually wrote.** Each converted flight is
archived with a small receipt recording where it came from and that it is a converted copy rather than
the original. The receipt was recording an internal working filename that exists nowhere else, while
telling you to refer to the original. It now names the original.

### Setting up

**The two "Custom folder" options say what they are.** The list of ground stations ended with two entries
both called *Custom folder*, and picking the wrong one silently found nothing. They now read **Custom
folder — MAVLink tlogs (\*.tlog)** and **Custom folder — DJI records (\*.txt)**.

---

## Included in 0.9.9 — internal build RD-0009, never published on its own

**Eleven fixes, five of them serious.** All were found by an independent review of the app rather than by
anything going wrong in the field.

### Flights could be lost or damaged

**A flight interrupted while being copied off the ground station could be filed under its real name as
though it were complete**, and marked as dealt with — so it was never picked up again. The app now checks
the whole file arrived before it commits to it.

**Two copies of the harvest could run at once** — the timer and the button, or two taps — and between them
publish a half-written flight. Only one runs now.

**A flight could be archived twice** when the aircraft was added to the register after its first flights
had already been filed.

**One momentary cloud error during a check could cause a flight to be archived a second time**, and the
duplicate was invisible.

### Records and reporting

**A transient read error could wipe a month of the archive's record of which tablet filed what.** An
unreadable file is no longer treated as an empty one.

**A pass could report "0 sent" after genuinely archiving flights**, if a fault occurred on one particular
path.

**The app now refuses to mark a flight archived when the cloud reports a different size than was sent.**

**Local copies could be deleted on the strength of a confirmation from an archive you had already switched
away from.** They stay.

**On a WebDAV or Nextcloud archive, a server could redirect a request to another address and your password
would have followed it.** Those redirects are refused.

---

## Included in 0.9.9 — build 0.9.8, prepared but never published

**Everything 0.9.7 was going to bring, plus a day of fixes found by running it on a real tablet.** 0.9.7 was
built and never released; its notes are below and still describe most of what you get here.

### Signing in and out of Google

**Signing out now really signs you out.** Before this, signing out cleared the app's access and left the
tablet's browser still logged in to Google — so the next person could sign back in without typing anything.
A Google sign-out page now opens briefly and closes itself, and the next sign-in asks for the password.

*⚠️ If you use the same tablet for anything personal, note that this signs that tablet's browser out of
Google, not only out of UAS Hangar.*

**Changing your cloud provider signs you out of the old one properly too**, the same way.

**The Google redirect address is filled in for you.** It is always the same value, it never varied between
installations, and typing it by hand cost an afternoon to a missing colon. If it is ever wrong, the app now
says which part is missing instead of handing you a Google error page that names neither.

### Archiving

**Large flights no longer get stuck.** If an upload was interrupted partway — the app closed, the tablet
rebooted, the signal dropped — it was possible for a flight to become permanently unarchivable, with
`verify` reporting it missing and the cloud refusing to accept it. Flights now upload in a single request,
and the app cleans up after an interruption instead of leaving something behind.

**Harvesting could fail on a brand-new or freshly-reset tablet** with a permission error on every flight,
while everything else looked healthy. Fixed, and a tablet already in that state repairs itself on its next
harvest.

**Resetting a tablet to clean now actually clears it.** Deleting the app's working folder could bring the
old pick-up history straight back, so a "clean" tablet would skip flights it had never actually collected.

### Screens

**The flight counters take one line instead of two**, which gives the home screen back some room.

**You can clear the aircraft a tablet is set to.** Previously, once set it could only be changed, never
unset, and the only way out was to forget the whole cloud setup.

**One tablet can serve several aircraft.** This already worked and was never written down: the aircraft you
set on a tablet is a fallback, not a filter — the app records which aircraft a flight belongs to when it
picks it up, so a tablet reading more than one source files each flight in the right place.

### Notifications

**The "App updates" channel now appears in Android's settings** before anything has been posted, so you can
find and check it rather than wondering whether notifications are on.

---

## 0.9.7 — never published

> **This version was built, tested on a tablet, and not released.** It is kept here because the rest of
> what it changed is real and carries forward into the version that does ship. **The Google sign-in
> paragraphs below were rewritten on 2026-08-23**: the panel they described was removed before anybody
> outside Redline could receive it.

### Changed

**Signing in to Google asks for your password again.** Sign-in opens one page - not the full browser, and
not a tab behind the ones you already had open - and the blank email that used to open when you tried to
pick an account does not happen.

**Signing out of Google now signs the tablet's browser out of Google too.** That is deliberate. Before
this, signing out cleared the app's access and left the browser still logged in, so the next person to
pick up the tablet could sign back in without typing anything. Now they cannot.

*⚠️ If you use the same tablet for anything personal, be aware that pressing Sign out logs you out of
Google in that tablet's browser, not only out of UAS Hangar.*

**All three providers now ask for your account in the app.** Microsoft, Google and WebDAV each have a box
for the account you archive as. Fill it in and the next thing you see is your own password or two-factor
prompt, rather than a list of accounts to pick from first. It is optional on all three, and once you have
signed in successfully the app remembers the address.

**Setting up Google needs the client id and the redirect URI** from your Google Cloud console, entered on
the Google step. Both are required.

**The home screen is rebuilt around the two things the app actually does.** Harvesting and archiving now
have a card each, side by side, each with its own status and its own explanation of what is wrong. Above
them, one line says how the app as a whole is doing.

**Every status is one of three words: Ready, Needs Attention, or Not Ready** - on each half and on the app
as a whole - with the colour saying how serious it is.

- **Green** is working.
- **Amber** means it still works, but worse - a metered connection that costs data, or flights filing under
  an unnamed aircraft because none has been chosen.
- **Red** means that part cannot do its job at all.

**An archive that is only half set up is red.** Being signed in with no folder chosen used to show amber,
which read as a small thing to get to later. Nothing can be filed until both are done, so it is red - and
the line underneath still says which of the two is missing.

**A tablet that is still harvesting is never reported as Not Ready.** The overall status is worked out from
both halves together rather than from the worse of them, so a tablet with no archive set up reads *Needs
Attention* while it is still collecting flights normally - because it is.

**Harvest is on the left and archive is on the right, everywhere.** That is the order flights travel. It
used to be the other way round on the status mark at the top of the screen, which meant the picture and the
words disagreed.

**The flight counts are split to match**, one column under each half: what is on the tablet, and what is
still waiting to be sent.

**Check now and Menu sit side by side across the bottom** rather than stacked in the corner.

### Fixed

**Signing out now means the next person needs the password.** Signing out of Microsoft used to clear the
app's own record and leave the browser still signed in, so signing back in was a single **Continue** button
with no password and no two-factor prompt. It now asks every time. WebDAV always did.

*One exception, and it is worth knowing rather than discovering:* **Google sign-in on a tablet with Google
Play services cannot ask for a password**, because the Google account belongs to the tablet rather than to
this app — signing out of UAS Hangar cannot sign the tablet out of Google. **On those tablets the screen
lock is what protects the archive.** Use a tablet account that is not a person's own, and lock the tablet.

**Changing your cloud provider goes to the archive setup**, not back to the beginning of everything.

**Every link on the home screen goes where it says it does.** *Sign in* used to open a message telling you
which screen to go to; it now signs you in. *Choose folder* used to drop you at the very beginning of
setup, so a tablet that only needed a folder had to retrace the flight-log path and the file permission to
reach it; it now opens the folder step. *Set up* goes to the account step.

**Changing your cloud provider signs you out of the old one.** It did so from Advanced, but not from the
setup screens - and the setup screens are the ordinary way to get there. A tablet could end up holding a
live sign-in for a provider it had stopped using, and the only way to clear it was to go and sign out by
hand.

**Run setup again is back**, in Advanced. Setting up one half at a time is still in Menu; this is the whole
run from the beginning, which is what you want on a tablet being set up for the first time.

---

## Included in 0.9.9 — build 0.9.6, never published

### Fixed

**The keyboard no longer closes itself while you are scrolling.** 0.9.3 made a touch anywhere off a text
field put the keyboard away. That also caught the start of a **scroll**, so the keyboard collapsed and the
page jumped the moment you tried to move down a setup screen. The guess is gone.

**Every text field now carries a button that puts the keyboard away.** It sits at the right-hand end of
the field you are typing in and appears only while that field is in use. It is a control on the screen
rather than a key on the keyboard, so it is there whichever keyboard the tablet uses — some draw a tick
for *Done* and no hide key at all, and a tick does not read as *put this away*.

The **Done** key on the keyboard still works where your keyboard provides one.

---

## 0.9.5

### Fixed

**Signing in to Google opens a single sign-in page.** It used to hand the sign-in to whatever browser the
tablet uses, which meant it arrived behind whatever tabs were already open, in a window whose address bar
slides away as the page moves. On a Samsung tablet that shifted the page while you were reaching for it,
and the tap landed on the account's email address instead of the account — which opens a blank email
rather than signing you in.

The sign-in now opens in its own page with a fixed bar that does not move, no tabs, and it returns to the
app by itself when you are done.

**If a blank email still opens, you have tapped the address line.** Tap the round initial, or the name
above the address, and the sign-in will continue.

**Advanced says whether the tablet is signed in.** The cloud card named the provider and nothing else, so
a tablet that had just been signed out looked exactly like one that was still signed in. It now reads one
of three things: not set up on this tablet, not signed in, or signed in as the account it is using.

**Each button's answer appears beside the button.** Results used to print at the very bottom of the page,
which on a tablet is a long way from the button you pressed and often off the screen entirely — so a
button that had worked looked like one that had done nothing. Every card now shows its own result.

**The keyboard closes in dialogs too.** 0.9.4 said the keyboard gets out of the way, and it did — on full
screens only. Adding an aircraft and naming a new folder are dialogs, and those are the two places the app
actually asks you to type. Both now have a **Done** key on every field.

**A harvest-only tablet no longer shows a full green mark for a job that has not run yet.** *Harvest only
— ready* is a correct state and it is now marked as one.

**Forgetting a cloud setup leaves the tablet harvest-only**, which is what you just chose, instead of
reporting that archiving has stopped.

### Moved

**Setting up a tablet is an ordinary thing to do, so it lives in Menu.** The two setup doors now sit beside
**Run setup again** rather than inside Advanced.

---

## 0.9.4

### New

**Set up harvesting first; add the archive when you are ready.** First run now asks for the flight-log
folder and the file permission, then stops and offers you the choice. Harvesting works with **no cloud
account at all** - and it is the half worth doing first, because ground stations delete their own older
logs on a schedule you do not control.

A tablet in that state reports **Harvest only - ready**, which is a working tablet, not a half-configured
one. Set up the archive whenever it suits, from **Menu > Advanced**. Nothing is redone, and flights kept
while there was no archive are sent as soon as one exists.

**The home screen reports the two halves separately.** The status mark is split - one half for harvesting,
one for archiving - and the checks below it are grouped under each, so you can see which half is fine
without reading every line. The wording always says which half it means.

**Sign out.** **Menu > Advanced > Sign out of this cloud** clears the tablet's credential, and changing
cloud provider now signs you out of the one you are leaving.

- On **Google Drive** this revokes access at Google, so it stops working everywhere.
- On **Microsoft and WebDAV** it clears the credential **from that tablet only**.
- **The tablet's browser stays signed in either way**, so signing back in may not ask for a password. The
  app now says so rather than leaving you to discover it.

**Forget this cloud setup.** For a tablet moving to a different organisation: clears the account settings
as well as signing out, so it does not carry your identifiers with it.

**The running version is on the home screen**, at the bottom. It changes on its own now that the app
updates itself, so it should not need looking for.

**The keyboard gets out of the way.** Tap anywhere off a text field to close it, and every single-line
field has a **Done** key.

### See also

**[Operating guidance](OPERATING-GUIDANCE.md)** - new. How to choose a cloud account, what a tablet holds,
what to do when one is lost or handed on, and how to tell the app is actually working.

---

## 0.9.3

**The first release published since 0.8.0.** If you are upgrading from 0.8.0, read
[Upgrading from 0.8.0](#upgrading-from-080) below — this release carries everything from 0.9.0, 0.9.1 and
0.9.2 as well, and those are described in their own sections.

### New

**The app updates itself.** On launch, with a connection, it checks whether a newer release exists and
offers it. **Update now** downloads, verifies the installer against a published checksum, and hands it to
Android. **Later** keeps quiet until the next time you start the app.

- Nothing is downloaded until you ask. The version check is a few hundred bytes; the installer is about
  19 MB, and spending a field connection on it uninvited is not the app's decision.
- A failed check is silent. No signal is the normal state of a tablet at a site, and an error message
  every time you opened the app outdoors would be noise.
- **See [Installing an update](#installing-an-update) — Android requires a one-time permission.**

**Choose the destination folder by browsing it.** Setup used to ask you to type a path and tell you
afterwards whether it existed. It now shows what is actually in your account, one level at a time, and
lets you create a folder where you need one. Typing a path still works and is quicker when you already
know it.

**Sign out.** Earlier versions had no way to sign out of anything — not when changing provider, not when
retiring a tablet. Now:

- Changing cloud provider signs out of the one you are leaving.
- **Advanced → Sign out of this cloud** does it without changing provider, for a tablet being retired or
  handed to somebody else.
- **On Google Drive this revokes access at Google**, so the credential stops working everywhere. On
  Microsoft and WebDAV it clears the credential **from that tablet only** — the app says which it is
  doing. If a tablet has been lost rather than retired, change the account password or revoke the app
  password at the server as well.

**The keyboard gets out of the way.** On a landscape tablet the on-screen keyboard covers about half the
screen, which left very little room to scroll. Tap anywhere off a text field and it closes, and every
single-line field now has a **Done** key that closes it.

### Changed

**Administrative files are grouped.** The fleet list, the tablet journals and the pick-up records now sit
together in an **`Admin Data`** folder beside `Aircraft Logs`, rather than being scattered among your
flights. `Aircraft Logs` now contains flight records and nothing else.

- **An existing fleet list is moved for you** on the first archive pass. The copy in its old location is
  left where it is — this app never deletes anything it did not create — so tablets still running an
  older version keep working.

### Fixed

**Verification no longer reports other aircraft's flights as unexpected.** In an archive shared by several
tablets, `verify` was listing every flight filed by a *different* tablet as unaccounted for. Nothing was
misfiled and the counts were always right, but on a large shared archive it produced pages of noise. It
now judges only the folders the tablet in front of you actually files into.

---

## 0.9.2

A field-testing release. Both faults below only appeared when a tablet was pointed at a **new** cloud
provider, so an archive that had been in use for a while never showed them.

### Fixed

**The pick-up record was not copied to a newly chosen archive.** The app tracked *what* the record
contained but not *which archive* it had been written to. Switch provider and the record was unchanged, so
the app concluded there was nothing to do — against an archive that had never received a copy.

**The fleet list was not written by a manual archive.** *Advanced → Archive now* did less than the
scheduled pass: it uploaded flights but skipped the fleet list. A brand-new archive would receive flights
and no `fleet.json`.

---

## 0.9.1

### Fixed

**A manual archive did less than the scheduled one.** *Advanced → Archive now* uploaded flights but did
not copy the pick-up record to the archive, which only happened when the periodic pass ran.

---

## 0.9.0

The largest release since 0.8.0. Not published on its own; everything here is included in 0.9.3.

### Fixed — the most important change in this release

**Reinstalling the app no longer causes flights to be copied twice.**

The app keeps a record of which flight logs it has already picked up. That record used to live in the
app's private storage, while the flights themselves live in a folder on the tablet that survives
uninstalling. **Reinstalling therefore destroyed the record and left the files** — so every flight looked
new, was copied a second time, and appeared alongside the original with `(1)` in its name.

The record now lives **beside the flights it describes**, so it has exactly the same durability they do.
An existing record is carried over automatically the first time the new version runs.

A second safeguard was added underneath it: the app now refuses to write a file where one of that name
already exists, rather than quietly writing a renamed copy.

### New

**Your pick-up record is backed up to the archive.** Each tablet keeps a copy in `Admin Data`, so a tablet
that is replaced does not force its successor to re-examine everything from the beginning. Restore one
from **Advanced → Restore the pick-up record from the archive**.

> ⚠️ **Only restore a tablet's own backup, or the backup of the tablet it replaced.** A restored record
> tells the app a flight has already been handled. Restore the wrong tablet's and its flights will never
> be picked up on this one, with nothing to warn you. The chooser shows which tablet each backup came
> from and when it was written.

**Operator logo handling is clearer.** Any image size is accepted; large ones are resized automatically
and the app now tells you what it kept and what it resized from. A logo smaller than the header is stored
as given and flagged, rather than being enlarged into something blurry.

- **Shape matters more than size.** The header is short and wide, so a wordmark sits well and a tall logo
  renders small.

---

## Upgrading from 0.8.0

**Install it over the top. Do not uninstall first.** Everything is preserved — your cloud sign-in, your
destination, your fleet, and the record of what has already been archived.

**What happens on the first run:**

1. The pick-up record moves out of the app's private storage into the staging folder. Automatic, once, and
   nothing is lost.
2. On the first archive pass, the fleet list moves into `Admin Data`. The old copy is left in place, so
   tablets still on an older version keep working from it.
3. Nothing already archived is moved, renamed or deleted. Flights filed under the older layout stay where
   they are and continue to verify correctly.

**Nothing needs reconfiguring**, and no re-authentication is required.

---

## Installing an update

These builds are distributed directly rather than through an app store, so Android requires two things.

**1. Permission to install, granted once.** The permission is called **Install unknown apps** — some
devices call it *Install from unknown sources* — under **Special app access** in Settings. The exact path
varies by manufacturer and Android version, so search Settings for "unknown apps" rather than following a
fixed menu path.

Grant it to whichever app is doing the installing: your browser or file manager for a first install, and
**UAS Hangar** itself for updates after that. The update prompt offers to take you there if it is missing.

**2. Android may ask you to confirm.** Depending on the device and how the app was first installed,
Android may show its own confirmation on top of the app's. That dialog is the operating system's, not
this app's, and it cannot be skipped.

**Every release is signed with the same key, and Android refuses an update signed with a different one.**
That check is the operating system's and is what makes installing over the network safe. Each release's
manifest also carries the installer's SHA-256, which the app verifies before handing the file to Android —
that catches a download truncated by a poor connection, which is the likelier failure in the field.

---

## Known limitations

**Ground stations.** Auterion Mission Control and DJI Pilot / GO are proven on hardware. QGroundControl
and Mission Planner presets are included but have not been exercised against a real installation.

**Android storage.** A ground station that keeps its logs inside its own private storage can only be read
on a **rooted** tablet. On an unrooted tablet the app can only read logs the ground station writes to
shared storage. This is a limit of the Android platform rather than of this application, and no
application can work around it.

**Unrooted tablets are less tested than rooted ones.** The non-privileged file access mode is complete and
under test, but Redline's own fleet is entirely rooted, so it has had far less time on real hardware.

**DJI flight record versions.** Records at format version 13 and above are encrypted with keys held by
DJI's own service and are **refused by name** rather than decoded incorrectly. Older records — including
every Matrice 600 record this decoder was validated against — are unaffected.

**Google Drive.** The app asks for full Drive access because it reads the archive as well as writing to it
— another tablet's fleet list, a folder you created, an archive an earlier app filled. Google classes that
as a restricted scope, which has its own verification requirements for wide distribution. Setting up your
own Google credentials also requires enabling a setting in the Google Cloud console that is switched off
by default for new clients.

**WebDAV stores a password.** WebDAV authenticates on every request, so the tablet has to keep the
credential. **Use an app password issued by your server, never your account password** — an app password
can be revoked from the server without touching the account. Plain `http` is refused.

---

*Questions, or a build that does not behave as described here: contact Redline Aerials.*
