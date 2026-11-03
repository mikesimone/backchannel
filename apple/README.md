# Backchannel for Apple (plan)

_Signal for people who actually have friends._

Status: **planning, not built yet.** Started 2026-10-30 (Brian Reed with
Mike). This folder holds the plan, the feature map and the first-run
setup design. The code lives in two places:

- **Mac:** [mikesimone/Signal-Desktop](https://github.com/mikesimone/Signal-Desktop),
  `contrib/backchannel/mac/` plus a `mac.yml` workflow.
- **iPhone and iPad:** `mikesimone/backchannel-ios`, a fork of
  [signalapp/Signal-iOS](https://github.com/signalapp/Signal-iOS).

Backchannel is not made, endorsed or supported by Signal Messenger.
"Signal" is their trademark and is used here only to say what this works
with.

## Decisions

| # | Decision | Why |
|---|----------|-----|
| B1 | **Self-contained apps; nothing to host.** | The user's rule (2026-10-30). Rules out a native client of the Docker server. |
| B2 | **Mac: a Mac build of the Signal-Desktop fork**, Apple Silicon only (`arm64`). | The fork already carries the bridge, the helper and every Backchannel feature, and Mike already merges each Signal Desktop release. A Mac build is packaging, not porting. |
| B3 | **Mac v1 runs the helper inside the app and shows the Backchannel page in its own window,** on loopback only. Mac v2 (later) moves the extras into Signal Desktop's own React UI. | Every web feature works on day one with no new UI. |
| B4 | **iPhone/iPad: fork Signal-iOS, Molly-style,** marked edits and a `Backchannel/` folder. | No Molly for iOS exists; a native client on libsignal would take years. |
| B5 | **iOS is always a linked device; official Signal stays the main device** (2026-11-01). | Signal's server sends push only to Signal's own app ID ([BUILDING.md](https://github.com/signalapp/Signal-iOS/blob/main/BUILDING.md), "Known issues"), so a fork gets no alerts. Official Signal keeps alerts and calls. Signal iOS 8.22 (2026-08-05) supports linked iPhones. |
| B6 | **Recommend Signal, never require it at launch.** | App Store guideline 4.2.3(i) bars apps that need another app installed; it also lets someone whose Signal is on an Android phone link normally. |
| B7 | **Hide calling, don't remove RingRTC.** | Calls ring once, in Signal; a small diff keeps upstream merges cheap. |
| B8 | **Distribution outside the App Stores.** Mac: notarized DMG on GitHub Releases + Homebrew cask. iOS: build-it-yourself, AltStore source, EU marketplace only if there's demand. | Both forks are AGPL-3.0 code whose copyright is mostly Signal's, and guideline 5.2.2 wants Signal's permission. |
| B9 | **Same marking rules as Android:** `// BACKCHANNEL:` on every edit to upstream code; new code in its own folder; a scheduled job merges each upstream release, opening an issue on conflict. | Keeps merges cheap; Desktop builds stop working ~90 days after they're built. |
| B10 | **Shared test vectors** for the markdown-shortcut and Unicode-copy rules in `apple/test-vectors/`, used by the page, Android, Mac and iOS. | The same rules exist in `app.js` and `Markdown.kt` already. |

## Milestones

Mac first, then iOS. Detailed prompts are in each repo's `KICKOFF.md`.

| # | Milestone | Repo | Done when |
|---|-----------|------|-----------|
| M0 | Build the Desktop fork for macOS arm64, unmodified | Signal-Desktop | It runs on an Apple Silicon Mac and links to a phone |
| M1 | Rebrand: name, icon, app ID, data folder, updater off | Signal-Desktop | Installs and runs beside official Signal Desktop |
| M2 | Bridge on at build time; helper in a `utilityProcess`; page in its own window | Signal-Desktop | Every web Backchannel feature works with no Docker and no browser |
| M3 | Sign, notarize, publish on each `backchannel-v*` tag; Homebrew cask | Signal-Desktop | A tag produces a notarized DMG on GitHub Releases |
| I0 | Fork Signal-iOS and build it unmodified | backchannel-ios | Runs on a real iPhone and links to a primary |
| I1 | Rebrand, new bundle IDs, linked-only, no push, payments, donations or calls | backchannel-ios | Installs beside official Signal; links; sends and receives while open |
| I2 | First-run setup on the same iPhone ([FIRST-RUN.md](FIRST-RUN.md)) | backchannel-ios | A new user links with Signal on the same iPhone and a Mac, an iPad or any browser, first try |
| I3 | Upstream sync workflow | backchannel-ios | Two Signal-iOS releases merge with no hand edits outside `Backchannel/` |
| I4 | Feature ports ([FEATURE-MAP.md](FEATURE-MAP.md)) | backchannel-ios | Each feature has a test and passes the shared vectors |
| I5 | Companion polish and distribution | backchannel-ios | Alerts off by default, background refresh, "Link again" screen, AltStore source |
