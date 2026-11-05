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
| F42 to F45 (2026-11-03/04): full markdown, list hanging indents, emoticons, double-click to react, starred messages, your own photo for anyone, What's New | native to the page; checked against the Mac build on 2026-11-04 | port, if Mike wants v2 |
| What's New and the page's own settings | kept in the window's storage (`persist:backchannel`, per port); they reset if the app ever has to move off port 47830 | Signal Desktop's own |
| Signal links (`sgnl://`, `signalcaptcha://`) | left as upstream: the last app started opens them (2026-11-02). Android instead offers "Open by default" (bc5) | decide |
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
| P9b | Full markdown at send, typing helpers for lists, pasted markdown converts | No | port | Wire: must send exactly the characters and ranges in [test-vectors](test-vectors/README.md#full-markdown-wire-format) (page F42, Android `Markdown.kt` `blocks()`, bc8) |
| P9c | Hanging indent for list lines in bubbles | No | port | Display only: wrapped lines of a line starting with em spaces/tabs/spaces then a bullet, `*`, `-` or a number with `.`/`)` and a space line up under the text |
| P10 | Scheduled messages | check | check | If missing, they can only send while the app runs (iOS won't wake an app at an exact time); decide whether to ship |
| P11 | Contact photos first | check | check | A linked device gets contacts through Signal's contact sync; see whether photos arrive |
| P12 | Photo viewer steps only through the opened message's attachments | check | decide | Mike's 2026-10-13 call; matched on Android |
| P13 | Calls | Full calling | port | Hide call buttons; decline incoming offers; calls stay in official Signal |
| P14 | Notifications | Push (won't work for a fork) | port | Off by default; optional local alerts after background refresh. When on, show sender and text by default (Android bc4: "usable defaults") |

### From Backchannel Android bc4 to bc9 (Mike's change list, 2026-11-04)

Mike's rule (2026-11-04): "Where we can get parity, we should." "Wire" means
other people's apps see the result, so iOS must send exactly what Android
and the page send.

| # | Backchannel feature | Signal-iOS today | Status | How |
|---|---|---|---|---|
| P15 | Typed emoticons become emoji | No | port | Wire (emoji characters). Rules and emoticon list in [test-vectors](test-vectors/README.md); Android bc6, page F43 |
| P16 | Double-tap to react with your first reaction; again to remove | check | port | Wire (a normal reaction); Android bc6, page F43 (double-click) |
| P17 | Starred messages | check (Labs flag?) | check, then port | This device only: no sync message, not in backups. Android turned on Signal's Labs feature; Desktop has none, so the page keeps `stars.json` |
| P18 | Lock a chat behind Face ID / passcode | No | port | This device only; locked chats hide their last message and notification text |
| P19 | Link a person to a phone contact by hand | No | check | A linked device may not have Contacts access or numbers for hidden-number people; see what I0 gets |
| P20 | Your own photo for anyone | No | port | This device only; square, at most 640 px, beats their profile photo (Android bc6, page F44) |
| P21 | One default skin tone for every emoji | check | check | Wire (the emoji carries the tone) |
| P22 | What's New after an update | No | port | This device only; same entries as Android and the page |
| P23 | Immich picker | No | decide | Android bc9. On iOS, photos could come in through the Share sheet instead |
| P24 | Donate row (Mike's Ko-fi) | Signal's donations | decide | CLAUDE.md removes Signal's donations; whether to add Mike's Ko-fi is Mike's call |
| — | Passphrase, screenshot and keyboard defaults; push registration; edge-to-edge fixes; "Photo apps" button | — | n/a | Molly- or Android-specific |

## Mike's standing calls that apply

- Forward has no limits (P3).
- Real Signal formatting on send; Unicode only on copy (P9).
- Photo viewer steps only through the opened message's attachments (P12).
- Note to Self keeps Mike's photo: check on device in I1.
- Usable defaults, less paranoid than Molly (Android bc4, 2026-10-30).
- `*` is the bullet; a line starting with `-` stays a dash (2026-11-03).
- Pasted markdown, for example from an LLM, converts itself (2026-11-03).
- Where we can get parity with Android, we should (2026-11-04).

## Risk to check in I0

Android bc5 notes that linking Backchannel Android to Signal on another
phone, as a linked device, fails with "Network error" for now. iOS is
always a linked device, so I0 must confirm that a Signal-iOS fork links
before anything else is built on it.
