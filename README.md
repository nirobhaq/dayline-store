<p align="center">
  <img src="icon.png" width="128" alt="Dayline icon">
</p>

<h1 align="center">Dayline</h1>

<p align="center">
  A personal daily dashboard for iPhone and Mac.<br>
  To-dos, bookmarks and reminders — synced in real time to a server you run yourself.
</p>

<p align="center">
  <a href="#install-with-sidestore">Install</a> ·
  <a href="#what-dayline-does">Features</a> ·
  <a href="#saving-links-from-safari">Share extension</a> ·
  <a href="#changelog">Changelog</a> ·
  <a href="#faq">FAQ</a>
</p>

---

This repository is the **SideStore / AltStore source** for the Dayline iOS app. It hosts
the app manifest (`store.json`), the icon, and each release's IPA. The app's source code
is private; this page exists so the app can be installed and kept up to date on a
sideloaded iPhone without a cable or an Xcode session.

## Install with SideStore

1. Open **SideStore** (or AltStore) on your iPhone.
2. Go to **Sources → +** and add this URL:

   ```
   https://raw.githubusercontent.com/nirobhaq/dayline-store/main/store.json
   ```

3. Open the **Dayline** entry and tap **Install**.
4. When SideStore asks how to handle app extensions, choose
   **Keep App Extensions (Use Main Profile)** — this is what lets the Safari
   share extension work (see [below](#saving-links-from-safari)).
5. Open Dayline once and sign in to your sync server. Everything else follows from sync.

New versions show up in SideStore's **Updates** tab automatically. SideStore also
renews the app's 7-day signature in the background, so there is no weekly re-install.

**Requirements:** iOS 17 or later, SideStore or AltStore, and a Dayline sync server
(a small Cloudflare Worker + D1 database — one deployment per person).

## What Dayline does

Dayline is one screen for the day, kept identical on every device you own.

**To-Do**
- Tasks live in sections you name; a **Pinned** section keeps the standing items on top.
- Quick-add is focused the moment the app opens — type, return, done.
- Move a task between sections by dragging it or from its menu; order syncs everywhere.
- **Tags** with colours, shared with the Mac app in both directions.
- **Reminders** fire as notifications with *Mark Complete* and *Dismiss* actions.
- Section headers show progress; which sections are collapsed is remembered per device.

**Bookmarks**
- A library of links and notes with rich previews (site cards, images, social embeds).
- Each link can carry a **comment** and any number of tags. Search across everything.
- Save from anywhere on iPhone with the **Share** sheet — Safari, YouTube, X, Reddit, Mail…

**Sync**
- Every change goes to your own server and fans out live over a WebSocket, so a task
  ticked on the phone is ticked on the Mac before you look up.
- Works offline: changes queue locally and reconcile when you're back online
  (last-writer-wins per item, tombstones for deletes).
- Nothing is stored in iCloud or with a third party. The server is yours.

**Mac companion** (distributed separately)
- Full dashboard with the same tasks, bookmarks and reminders.
- Optional **menu bar To-Do panel** with per-section progress rings.

## Saving links from Safari

Dayline ships a share extension: tap **Share → Dayline** on any page, optionally add a
name, a comment and tags, and tap **Save**. The link appears in Bookmarks on every device.

Because a free Apple ID cannot grant an *App Group* to a sideloaded app, the extension
does **not** rely on one. It talks to your sync server directly using the credentials the
main app stores in the keychain — which is why installing with **Keep App Extensions
(Use Main Profile)** matters: that option gives the extension the same keychain as the app.

If a save ever fails, the extension falls back to opening Dayline with the link, and only
then shows an error explaining what went wrong. **Gear menu → Share extension status…**
inside the app shows the diagnostics if you need them.

## Changelog

### 1.8.9 · 2026-09-09 — Share extension: instant, never doubles, takes comments
- **Saves are instant.** The extension used to ask the sync server for *everything since
  the beginning of time* along with its one new link, and then download that whole change
  log before it could say "Saved". It now sends the app's current sync position instead,
  so the server answers with a handful of rows. What took several seconds is now a single
  short request.
- **No more duplicates.** The Save button locks the moment it's tapped (spinner +
  "Saving…", inputs frozen) and each share builds exactly one sync record, so tapping twice
  — or retrying — can no longer create two bookmarks.
- **Comment box.** Add a note to the link right from the share sheet; it lands in the
  bookmark's comment field, the same one the app edits.
- **Lighter, cleaner sheet.** The page behind the card is dimmed instead of blacked out,
  and the Save button is now the large full-width primary action.
- **Faster to open.** The App Group check no longer re-reads the app binary on every launch;
  the verdict is cached per signed build and the file is memory-mapped, not copied.
- The app now keeps its sync position in the keychain for the extension, refreshed whenever
  the app goes to the background or finishes syncing.

### 1.8.8 · 2026-09-08 — Share extension works under SideStore
- Captures upload **directly to the sync server** when no App Group is available — the same
  wire record the app itself emits, so the Mac and every other device receive it through
  normal sync. Confirmed working on a free-Apple-ID SideStore install.
- Three-tier save: App Group queue → direct upload → `dayline://` deep link, with a plain
  in-extension error if none of them can run.
- The app mirrors its sync server address into the keychain for the extension.

### 1.8.7 · 2026-09-08 — App Group discovery from the signed binary
- The extension reads the App Group from the entitlements embedded in the signed
  executable (own and host app) and from team-ID rename patterns, each verified with the
  OS. *Share extension status* lists every candidate with ✓/✗.
- Superseded by 1.8.8: diagnostics showed SideStore grants **no** App Group at all on a free
  Apple ID, which is why the direct-upload path exists.

### 1.8.6 · 2026-09-08 — OS-verified App Group probing + diagnostics
- App Group candidates are probed from both the extension's and the app's signed profiles
  and confirmed with the OS before use.
- New **Gear menu → Share extension status…** panel showing which container each side
  resolved and when the extension last ran.

### 1.8.5 · 2026-09-08 — Runtime App Group resolution
- The App Group ID is resolved at runtime from the embedded provisioning profile instead of
  being hard-coded, so re-signers that rename groups don't break Safari captures on
  Xcode-installed builds.

### 1.8.4 · 2026-09-08 — First SideStore release
- **To-Do first:** the To-Do tab opens by default with quick-add focused in the Pinned section.
- Move tasks between sections by drag or from the task menu; ordering syncs.
- Task **tags** with a two-way registry sync to the Mac, and **reminders** with
  notification actions.
- Section header bands with progress; collapsed/expanded state persists.
- Sync status moved into the gear menu; keyboard-aware bottom bar.
- Sync engine tuned for low CPU and memory (payload memoisation, watchdog sleeps until its
  deadline).

## FAQ

**Is this an App Store app?** No. It's a personal app installed through SideStore. The
IPAs here are unsigned; SideStore signs them with your own Apple ID on the device.

**Does it need the Mac app?** No. The iPhone app is complete on its own; the Mac app is a
companion that shares the same sync server.

**Where is my data?** On your device and on your own sync server. There are no analytics,
no third-party services and no iCloud storage.

**The share sheet says "Open Dayline and sign in once."** The extension needs the app's
sync credentials from the keychain. Open the app, make sure it's signed in and synced, then
share again. If it persists, reinstall from SideStore choosing *Keep App Extensions (Use
Main Profile)*.

**How are releases built?** Each release is a Release-configuration `xcodebuild` with code
signing disabled, zipped as `Payload/Dayline.app`. `store.json` lists the exact size and
SHA-256 of every IPA so SideStore can verify what it downloads.
