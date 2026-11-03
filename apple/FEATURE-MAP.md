# Backchannel for Apple: feature map

Which Backchannel features each Apple app needs and how each one lands.
Same status words as [android/FEATURE-MAP.md](../android/FEATURE-MAP.md):
**done**, **native** (Signal already does it), **port** (we build it),
**n/a**, **decide** (Mike's call), plus **check** (find out first).

Signal-iOS paths are against `signalapp/Signal-iOS` main at `b6beaf0`
(2026-10-29); they move, so re-check when porting.

## Mac (Signal-Desktop fork)

Mac v1 shows the existing Backchannel page in its own window, so every
feature the page has works as soon as M2 ships. Nothing below needs
porting for v1.

| Feature | Mac v1 | Mac v2 (later, in Signal Desktop's own UI) |
|---|---|---|
| Everything in the page's PARITY.md and "Beyond Signal Desktop" list | native to the page | port, one feature at a time, if Mike wants v2 |
| Scheduled messages | helper sends them while the app runs | same; they don't send while the Mac is asleep or the app is quit |
| Notifications | page notifications through Electron | Signal Desktop's own |
| Rambox/Ferdium tabs, secret URL, reverse proxy, VNC, `screen.sh` | n/a | n/a |

## iPhone and iPad (Signal-iOS fork)

| # | Backchannel feature | Signal-iOS today | Status | How |
|---|---|---|---|---|
| P1 | Name, icon, tagline, "not made by Signal" | Signal branding | port | `SIGNAL_BUNDLEID_PREFIX` + Backchannel/Branding; strings overlay like Android's `rebrand_strings.py` |
| P2 | Linked device only, official Signal recommended | Registration or link | port | Hide registration; first-run flow in [FIRST-RUN.md](FIRST-RUN.md) |
| P3 | Forward to any number of chats | Limited | check, then port | Find the picker's limit in the forward flow; lift it |
| P4 | One poll to many chats | Polls not forwardable | port | New poll per chat, same question, options and multi-vote setting (Android `PollForward.kt`) |
| P5 | Reply privately in a group | No | port | Menu item on others' group messages; quote saved as the 1:1 chat's draft (Android `PrivateReply.kt`) |
| P6 | Sticker replies | check | check | If it's behind a flag, turn it on (Android flipped `labs().stickerReplies`) |
| P7 | Bold italic in one step | Separate styles | port | One format-menu item applying both (Android `BoldItalic.kt`) |
| P8 | Edit and @mention when forwarding | check | port | Typed "@Name" becomes a real mention where the person is in the group, plain text elsewhere (Android `ForwardMentions.kt`) |
| P9 | Markdown shortcuts + Unicode copies | No | port | Port from the shared vectors; same rules as the page (F33) and Android `Markdown.kt` / `UnicodeStyles.kt` |
| P10 | Scheduled messages | check | check | If missing, they can only send while the app runs (iOS won't wake an app at an exact time); decide whether to ship |
| P11 | Contact photos first | check | check | A linked device gets contacts through Signal's contact sync; see whether photos arrive |
| P12 | Photo viewer steps only through the opened message's attachments | check | decide | Mike's 2026-10-13 call; matched on Android |
| P13 | Calls | Full calling | port | Hide call buttons; decline incoming offers; calls stay in official Signal |
| P14 | Notifications | Push (won't work for a fork) | port | Off by default; optional local alerts after background refresh |

## Mike's standing calls that apply

- Forward has no limits (P3).
- Real Signal formatting on send; Unicode only on copy (P9).
- Photo viewer steps only through the opened message's attachments (P12).
- Note to Self keeps Mike's photo: check on device in I1.
