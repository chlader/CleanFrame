# CleanFrame TV setup — session log (2026-11-06)

Branch: `feature/dumbtv-layout`. Goal for the session: quiet down a Samsung
UN55TU7000FXZA (2020, Tizen 6.0), build a dumbtv-style Home screen on top of
CleanFrame, and get it installed and running on the real TV.

## What was done

1. **Samsung bloat reduction (on the TV, by the user)** — advice given for
   turning off ACR/ad tracking, Smart Hub autorun, Samsung TV Plus, and
   deleting/hiding unused preloaded apps. Not something this repo tracks.

2. **Cloned CleanFrame** to `~/Desktop/CleanFrame` and set up local tooling:
   Python 3.13 (native Homebrew at `/opt/homebrew`, since the original
   `/usr/local` Homebrew couldn't build Python without Xcode Command Line
   Tools), Playwright's Chromium for the browser checks, and cleared ~11 GB
   of regenerable caches (Yarn/Gradle/npm/nuget/pnpm, Cursor's updater
   leftovers, Google's crx_cache) to make room.

3. **Rows layout** (`39164dc`) — a third Home layout modelled on the
   `humblemuzzu/dumbtv` Fire TV launcher's design (not its code, which is
   Android/Java and can't run on Tizen): a clock+date, an Apps row, an
   Inputs row (HDMI 1–3). Input switching goes through the Tizen TV Window
   API in `device.js` and is **unverified on hardware** — the manifest
   privilege for it was not added.

4. **Accent colour setting** (`05eb479`) — eight colours for the Rows focus
   ring/tile tint, picked in Settings → Layout, with a live preview.

5. **Home apps matched to the real TV** (`ac5188d`) — installed the Tizen
   SDK's `sdb` tool, connected to the TV over Developer Mode (its local
   IP, port 26101), and read its installed-app list (394 apps). Five of the
   author's nine default apps matched (Netflix, Disney+, YouTube, Apple TV,
   Prime Video); Stremio/ABC iview/SBS/Apple Music do not exist on this TV.
   Added YouTube TV, Hulu, HBO Max, Peacock, Paramount+, ESPN, CHSN with
   their real app IDs from the TV; Home now defaults to nine apps that
   exist on this TV. Paramount+ and YouTube TV got Simple Icons (CC0) marks
   with measured metrics; Hulu/Peacock/ESPN/CHSN use letter badges.

6. **Signing and install on the real TV:**
   - Downloaded Samsung's official Tizen SDK CLI installer, then (after the
     lightweight CLI's Certificate Extension turned out to be a Tizen
     Studio IDE add-on that needs the full IDE) downloaded and installed
     the full Tizen Studio IDE (`~/tizen-studio`) plus the Samsung
     Certificate Extension.
   - Certificate Manager's app bundle was initially broken (no launcher),
     fixed by reinstalling the extension in Package Manager.
   - User created a Samsung TV certificate profile (`CleanFrame`) in
     Certificate Manager, tied to this TV's DUID (kept out of the repo —
     see below), issued by Samsung's "VD DEVELOPER Public CA", valid to
     2027-11-07.
   - CleanFrame's app identity was changed from the author's
     `nClrkHome1.FrameHome` to a new `cFr4mHm7Qx.CleanFrame`
     (`app/config.xml`, `scripts/check.py`, `scripts/check-browser.mjs`,
     `scripts/native-home/Control.cs`), required_version lowered to 5.5.
   - The app was packaged, signed with the new certificate, and installed
     on the TV, then launched via `sdb shell 0 execute`. **Confirmed
     working on the real TV**: Rows layout displays, remote navigation
     works, Netflix launches from the Apps row.
   - Known-expected, not bugs: Back doesn't exit Netflix's profile picker
     (Netflix's own behaviour, not intercepted by CleanFrame); Home still
     opens Samsung's quick-select menu (native Home/startup was
     intentionally **not** touched this session).

## ⚠️ Flag: unexplained changes mid-session, and a model/session anomaly

Partway through, this session found **four files already modified in the
working tree** (the identity-change diff above) and a **signed `.wgt`
already installed and running on the TV** — none of which this session had
done yet at that point. Several `claude` processes were visible running on
the user's Mac at the same time. The most likely explanation is another
Claude Code session working on the same checkout, but this was **never
confirmed** — no second session identified itself, and the change was
simply found already present.

Separately, **the model serving this conversation changed more than once
within what the user experienced as one continuous session**: it started
as Claude Opus 5.5, system reminders later in the conversation showed
Claude Sonnet 5.5, and then Claude Sonnet 5 for this closing portion
(each change came with a new git-attribution line, which this log and the
commit reflect). This is flagged here, per the user's request, as
something to be aware of: an unexplained file change landed in the same
stretch of conversation where the serving model was changing underneath
it, and that combination was not independently verified beyond what's
written above. Nothing in the diff looked adversarial (it matched the
identity-change work this session was about to do anyway), and it was
reviewed and accepted rather than reverted, but the user should treat the
"who/what did this" question as open, not resolved.

## State at end of session

- Signed, installed app identity: `cFr4mHm7Qx.CleanFrame` / package
  `cFr4mHm7Qx`, on the TV (local network, not recorded here).
- Certificate profile `CleanFrame` lives outside the repo at
  `~/SamsungCertificate/CleanFrame/` (author + distributor `.p12`, tied to
  this TV's DUID). Not committed, not copied into the repo. The DUID
  itself is also deliberately left out of this log.
- Tizen Studio (`~/tizen-studio`) and the standalone Tizen SDK
  (`CleanFrame/.local/tizen-sdk`, git-ignored) are both on disk.
- Developer Mode is still on, and the `sdb` developer connection to the TV
  was left open.
- Known follow-ups, not done:
  - `app/device.js`'s self-filter (`SELF`) still has the **old** author ID
    (`nClrkHome1.FrameHome`), not the new `cFr4mHm7Qx.CleanFrame` — should
    be fixed so the app doesn't list itself in All apps.
  - Native Home/startup takeover (Stage 7/8 of
    `docs/agent-tv-setup-guide.md`) was **not attempted** — explicitly held
    for a later, separate authorisation after reviewing compatibility with
    this TV's older firmware.
  - Disk space is tight (roughly single-digit GB free); the iOS Simulator
    runtime (~11 GB) was identified as the next easy win if needed.
