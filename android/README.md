# Backchannel for Android (plan)

_Signal for people who actually have friends._

Status: **planning, not built yet.** Started 2026-10-15. This folder holds
the plan and the feature map; the app's code will live in its own fork,
`mikesimone/backchannel-android`.

Backchannel for Android is a full APK: a fork of Signal's Android app, done
the way Molly does it, with Backchannel's features, its own name and icon,
and no Google push. It talks to Signal's servers directly, like Signal does.
Six is not involved and needs no open port.

## Decisions

| # | Decision | Why |
|---|----------|-----|
| A1 | **Fork Molly (`mollyim/mollyim-android`), not Signal-Android directly.** | Molly is Signal-Android plus exactly the plumbing we'd otherwise write: no Google Play Services (`core-gms/` FOSS replacements), notifications over a kept-open WebSocket or UnifiedPush, a de-branding pipeline (`mollyify` strings, `CI_APP_TITLE` / `CI_PACKAGE_ID` / `CI_APP_FILENAME` build overrides), reproducible Docker builds, and a sync workflow that tracks Signal's releases. It's AGPL-3.0, so our fork is too. Molly already removes Signal's 5-chat forward limit (`RemoteConfig.shareSelectionLimit = NO_LIMITS`). |
| A2 | **Notifications: kept-open WebSocket.** UnifiedPush stays available but off. | The WebSocket needs no server at all: the phone holds a connection to Signal. UnifiedPush needs a MollySocket server plus a distributor the phone can reach, which means either a public service or opening Six, and Mike rejected opening Six (2026-10-14). Molly also doesn't support UnifiedPush on linked devices. The cost is battery: the app must be exempt from battery optimization. |
| A3 | **Name "Backchannel", package `net.mikesimone.backchannel`.** | Own name, no Signal trademark, installs beside Signal and Molly without clashing. Set through Molly's existing `CI_APP_TITLE`, `CI_APP_FILENAME`, `CI_PACKAGE_ID` build variables plus our own launcher icon. |
| A4 | **Build on GitHub Actions in the fork**, using Molly's `release.yml` and Docker build unchanged. | Molly's build wants ~12 GB of Gradle heap (`org.gradle.jvmargs=-Xmx12g`); WIN11 is too small and Six's headroom is unknown. Public-repo runners are free. Fallback: `reproducible-builds/docker compose up --build` on Six. |
| A5 | **Signed with Mike's own key**, held only as Actions secrets (`SECRET_KEYSTORE`, `SECRET_KEYSTORE_ALIAS`, `SECRET_KEYSTORE_PASSWORD`). Never committed. | Android only installs updates signed by the same key, so the key must be stable and backed up. Mike generates it on his own machine (command below). |
| A6 | **Ship through GitHub Releases, installed and updated with Obtainium.** | No store account, no server. Molly's in-app updater (`MANAGE_MOLLY_UPDATES`) is pointed at our releases or turned off. |
| A7 | **Stay current automatically.** A scheduled workflow merges each new Molly release tag into our `main`, builds, and drafts a release. A merge conflict opens an issue instead. | Same standing rule as Desktop: keep every feature Signal adds. Molly follows Signal every two weeks. |
| A8 | **Our changes stay small and marked.** Every edit to Molly/Signal code carries a `// BACKCHANNEL:` comment, and new code goes under `app/src/main/java/net/mikesimone/backchannel/`. | Keeps upstream merges cheap, the same way Molly marks its own edits `// MOLLY:`. |

### Open: main app or linked device

Molly can run either as the phone's main Signal app (replacing Signal,
moved over with a Signal backup) or as a linked device beside Signal.
Default until Mike says otherwise: **linked device first** for testing,
since it can't disturb his Signal account. Switching to main later is a
backup restore.

## Steps

1. **Fork.** Mike runs (GitHub CLI, any machine):

   ```sh
   gh repo fork mollyim/mollyim-android --fork-name backchannel-android --clone=false
   ```

   Then adds `mikesimone/backchannel-android` to this project's
   repositories so the cloud sessions can push to it.

2. **Signing key.** Mike, on his own machine (PowerShell or bash, JDK's
   `keytool`):

   ```sh
   keytool -genkeypair -v -keystore backchannel.jks -alias backchannel -keyalg RSA -keysize 4096 -validity 36500
   ```

   Back up `backchannel.jks` and its password somewhere permanent. Then set
   the three secrets on the fork (`base64 -w0 backchannel.jks` for
   `SECRET_KEYSTORE`).

3. **Rebrand build (milestone M1).** In the fork: repo variables
   `CI_APP_TITLE=Backchannel`, `CI_APP_FILENAME=Backchannel`,
   `CI_PACKAGE_ID=net.mikesimone.backchannel`; Backchannel launcher icon
   (adaptive icon, same mark as the web page); About screen and README say
   it's a Molly fork and not made by Signal; turn the in-app updater toward
   our releases. Tag `bc-v<molly version>-1`, Actions builds a signed APK.
   **Done when:** the APK installs on Mike's phone next to Signal, links as
   a device, and gets messages with the app closed.

4. **Sync workflow (M2).** Daily job: fetch Molly tags, merge the newest
   release into `main`, push the tag, which triggers the release build.

5. **Feature ports (M3+).** In the order in [FEATURE-MAP.md](FEATURE-MAP.md).

6. **Release in this repo's README.** Once M1 ships, the Backchannel README
   gets an Android section linking the fork's releases and Obtainium.

## What doesn't carry over

The browser-only parts of Backchannel (the server, the page, the local
API, Rambox tabs) have no Android counterpart: the app *is* the client.
See [FEATURE-MAP.md](FEATURE-MAP.md) for the full list.
