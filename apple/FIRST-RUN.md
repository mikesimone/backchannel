# Backchannel for iPhone and iPad: first-run setup

Backchannel is a linked device. Official Signal on your phone stays the
main device and scans Backchannel's link code once. When Signal is on the
same iPhone, that code has to be shown on a second screen, because a phone
can't scan itself and Signal won't read the code any other way.

## Facts that shape the design

Checked against Signal-iOS main at `b6beaf0` (2026-10-29).

| Fact | Where |
|---|---|
| Signal links only by camera scan. Opening a `sgnl://linkdevice` URL from another app shows a warning and sends you to Linked devices. | `Signal/URLs/UrlOpener.swift`, case `.linkDevice` |
| Typing or pasting the link URL exists only in Signal's test builds. | `LinkDeviceViewController.swift`, `#if TESTABLE_BUILD` |
| The scanner supports a "from photo" button, but the link screen leaves it off. | `SignalUI/ViewControllers/ScanQRCodeViewController.swift`, `showUploadPhotoButton` |
| The new device's code rotates every 45 s, five times, then shows a refresh button. Signal's server closes each provisioning socket after 90 s. | `Signal/Provisioning/ProvisioningSocketManager.swift`, `rotate()` |
| The code only works while the new device holds its connection open, so Backchannel must still be running when Signal scans. | Same file |
| An iPhone can be a linked device since Signal iOS 8.22 (2026-08-05). Up to 5 linked devices; history and up to 45 days of media can come across; a companion unused for 45 days is unlinked. | [iphone-ticker.de](https://www.iphone-ticker.de/ein-konto-zwei-iphones-signal-erweitert-die-geraeteunterstuetzung-282806/), [Signal support](https://support.signal.org/hc/en-us/articles/360007320451) |

So: a saved picture of the code is dead within 90 seconds, and a screen
mirror shows Signal's camera once you switch apps. What works is sending
the live code to another screen from inside Backchannel, then scanning it
from Signal while Backchannel's short background window (about 30 s, to be
measured) keeps the connection open.

## The flow

1. **Welcome.** "Backchannel works alongside Signal. Your Signal account
   stays on your phone." Choice: *My Signal is on this iPhone* or *on
   another phone*. If official Signal isn't detected, say it's recommended
   and link to it, but carry on (guideline 4.2.3(i)).
2. **Another phone.** Show the code full screen with the usual rotation.
   "On that phone: Signal > your photo > Linked devices > Link new device."
3. **This iPhone, step 1 (in Signal).** "Open Signal, tap your photo,
   Linked devices, Link new device, confirm with Face ID, and leave the
   camera open. Then come back here." An *Open Signal* button.
4. **Step 2 (in Backchannel): choose a second screen.**
   - **AirDrop to a Mac or iPad.** It opens in Preview or Photos by itself.
   - **Share sheet.** Messages to yourself on a Mac, or anything else.
   - **Any browser on this Wi-Fi.** Backchannel shows
     `http://<iphone>.local:<port>/` and a 4-digit code. The page shows the
     code and updates itself while Backchannel is running.
5. **Step 3 (back in Signal).** As you leave, Backchannel asks iOS for
   background time and holds one code still (no rotation, within the 90 s
   limit), with a countdown on screen before you switch. Point Signal at
   the other screen.
6. **Finish (back in Backchannel).** Show Signal's history choice and the
   sync progress. If time ran out, say so plainly and offer a fresh code;
   never leave it half-linked.

## Rules for the local web page

- Served only during setup, on a random high port, bound to the Wi-Fi
  interface; closed when linking succeeds, fails or times out.
- First request must carry the 4-digit code; wrong codes count toward a
  small limit, then the page closes.
- The page shows only the QR image and the countdown. No other data.
- Needs `NSLocalNetworkUsageDescription`; ask at the moment the user picks
  this option, not at launch.

## Not doing

- Keeping Backchannel alive with a silent audio session or
  picture-in-picture: an abuse of background modes.
- Anything that gets around Signal's anti-phishing guard (the URL warning,
  the photo button being off).
- A dedicated AirPlay second-screen view as the main path: it may stop
  when Backchannel goes to the background. Test it in I2; keep it only if
  it holds.

## Fallbacks if the window is too tight

- Link while another Apple device shows the code: the Mac or iPad does the
  showing, which is the AirDrop path.
- Ask Signal for a same-device link option.

## Tests for I2

| Setup | Second screen | Pass |
|---|---|---|
| Signal on the same iPhone | Mac via AirDrop | Links first try |
| Signal on the same iPhone | iPad via AirDrop | Links first try |
| Signal on the same iPhone | Windows laptop browser | Links first try |
| Signal on an Android phone | none | Links first try |
| Signal on the same iPhone | slow user, > 90 s | Clear "try again", fresh code, no broken state |
| iPad as the Backchannel device | iPhone scans directly | Links first try |

Measure and record how much background time iOS actually gives, on the
oldest supported iPhone.
