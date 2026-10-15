# Backchannel for Android: feature map

Which Backchannel features the Android app needs, and how each one lands.
Source paths are in the Molly tree (`mollyim/mollyim-android` main,
checked 2026-10-15 at `892d626`); they move, so re-check when porting.

Most of Backchannel's work was getting a browser page up to Signal
Desktop. On Android that part comes for free: the app *is* Signal's
client. What has to be ported is the "Beyond Signal Desktop" list, the
things Backchannel does that Signal doesn't.

Ports live in `mikesimone/backchannel-android` main (hashes below).

Status: **done** (ported), **native** (Signal/Molly already does it), **port** (we build it),
**n/a** (browser-only, nothing to do), **decide** (Mike's call).

## Beyond Signal: the port list, in build order

| # | Backchannel feature | Android today | Status | How |
|---|---------------------|---------------|--------|-----|
| P1 | Name, icon, tagline, "not made by Signal" | Molly branding | done (`7ba38a5`) | M1 in [README.md](README.md): build variables, launcher icon, About screen. |
| P2 | Notifications with no open port | Molly: Google push, else kept-open connection | native | Molly's default. Google push when Google Play is present, kept-open connection otherwise. |
| P3 | Forward to any number of chats | Molly sets `RemoteConfig.shareSelectionLimit = NO_LIMITS` | native | Verify on device. Separate cap: at most 32 selected messages per forward (`MenuState.MAX_FORWARDABLE_COUNT`); leave it. |
| P4 | One poll to many chats | Forward is hidden for polls (`MenuState`: `!hasPoll`) | done (`5aa2eca`) | Allow forward on a poll; on send, create a fresh poll per destination with the same question/options via `ConversationRepository.sendPoll`, as the page does. |
| P5 | Reply privately to someone in a group | Not in Android | done (`853e031`), bubble link still to do | Long-press menu item on others' group messages: open the 1:1 chat with that message quoted. Store the group↔1:1 link locally (Backchannel's `privateReplies.mjs` keeps it server-side) so both bubbles show the link. |
| P6 | Sticker replies | Built but behind `SignalStore.labs().stickerReplies`, `falseForExternalUsers()` (`keyvalue/LabsValues.kt`) | done (`a498420`) | Turn the flag on for our build (`// BACKCHANNEL:` override), no new UI. |
| P7 | Bold italic in one step | Format menu in `ComposeText.java` has bold and italic separately | done (`a498420`) | Add a "Bold italic" item applying both body-range styles. |
| P8 | Edit and @mention when forwarding | Forward sheet takes a text add-on, no mentions | done (`1653a9f`): typed "@Name" becomes a real mention; picker UI still to do | Use the conversation's mention picker in the forward sheet; real mention ranges where the person is in the group, plain "@Name" elsewhere (same rule as F32). |
| P9 | Markdown shortcuts + SMS-ready copies | Not in Android | done (`fcba70a`) | TextWatcher on `ComposeText`: \*bold\*, \_italic\_, \_\*bold italic\*\_, \~strike\~, \`mono\` turn into styles as typed and on paste; Backspace right after undoes. Copy puts Unicode-styled text on the clipboard; pasting Unicode-styled text turns it back into styles. Port the rules and tests from the page (`helper/public/app.js`, F33). |
| P10 | Scheduled messages | Native on Android | native | Signal Android already has them (`ScheduleMessageDialogCallback`). Nothing to do. |
| P11 | Starred messages | Behind `labs().starredMessages`, external users off | decide | Signal ships it hidden. Backchannel Desktop doesn't have it either. Default: turn it on with P6, since it's one flag. |

## Mike's standing calls that apply

- **Photo viewer steps only through the opened message's attachments**
  (2026-10-13). Signal Android's viewer pages through the whole chat's
  media. Default: match Mike's call in Backchannel for Android too (port
  item P12, after P9); say so if it should stay as Signal does it.
  Done in fork `bdd6975`.
- **Forward has no limits.** Covered by P3.
- **Note to Self keeps Mike's photo.** Check on device in M1; port if
  Android shows the Note to Self icon instead.
- **Real Signal formatting on send; Unicode only on copy.** P9 follows it.

## Desktop parity items

Everything on [PARITY.md](https://github.com/mikesimone/Signal-Desktop/blob/main/contrib/backchannel/helper/PARITY.md)
that Backchannel built to catch up with Signal Desktop (chats, reactions,
edits, polls, stickers, voice notes, groups, settings, profile, search,
All media, typing, disappearing messages, link previews) is **native** on
Android. Several of PARITY.md's open gaps (safety numbers, view-once,
edit history, message details, stories, chat colors, GIF search, contact
cards) are native here too.

## Browser-only, nothing to port (n/a)

- The server, the page, Rambox/Ferdium tabs, the secret page address.
- The local external-client API and the "Allow Backchannel" prompt.
- `screen.sh`, VNC, QR linking from a container.
- Notifications from the tab (P2 covers notifications on Android).
- Scheduled sends from the server with the PC off: Android's scheduled
  messages send from the phone, so they need the phone on.
