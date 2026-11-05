# Shared test vectors

One set of cases for Backchannel's text rules, run by every client:
the page (`contrib/backchannel/helper/public/app.js`), Android
(`net/mikesimone/backchannel/Markdown.kt`, `UnicodeStyles.kt`), the Mac
app (which uses the page), and iOS (`Backchannel/Features/`).

Status: **to build.** The cases come out of the existing tests first:
`contrib/backchannel/helper/test/backchannel_test.mjs` (F33, F42 and F43
cases) and Android's `UnicodeStylesTest.kt` and `Markdown.kt` tests. Where the page and Android disagree,
that's a bug to raise with Mike, not a case to pick a side on silently.

## Files

- `markdown-shortcuts.json`: typed or pasted text in, Signal body ranges out.
- `unicode-copy.json`: styled text in, Unicode clipboard text out, and back.
- `forward-mentions.json`: forward text with "@Name", the destination's
  members, expected mention ranges.
- `markdown-blocks.json`: full markdown (bullets, nesting, task boxes,
  numbered lists, headings, quotes, rules, code fences, `**bold**`, links)
  in, the characters and ranges sent out. See the wire format below.
- `emoticons.json`: typed text and the point where it changes, emoji out,
  plus the cases that must stay alone (`:Pizza`).

## Format

```json
{
  "version": 1,
  "cases": [
    {
      "id": "bold-simple",
      "input": "say *hello* now",
      "expectText": "say hello now",
      "expectRanges": [{ "start": 4, "length": 5, "style": "BOLD" }],
      "note": "closing mark applies the style"
    }
  ]
}
```

- `start` and `length` count UTF-16 code units, as Signal's body ranges do
  on every platform.
- `style` is one of `BOLD`, `ITALIC`, `STRIKETHROUGH`, `MONOSPACE`,
  `SPOILER`.
- Each `id` is unique and never reused; a changed rule gets a new case.
- The example above shows the shape only; real cases are copied from the
  existing tests.

## Full markdown wire format

From Mike's "Backchannel vs. Signal: every difference" (2026-11-04),
matching page decision F42 and Android bc8. Signal carries only five body
range styles (bold, italic, strikethrough, monospace, spoiler), so
everything else goes as plain Unicode characters plus those styles, and any
Signal app shows it. iOS must send exactly these, and recognize them on
receive and when a message is edited.

| Typed or pasted | Sent as | Style range |
|---|---|---|
| `* item` | `• item` | none |
| Nested `* item` (each tab, two spaces or U+2003 is one level) | one U+2003 per level, then `◦` at level 1, `▪` at level 2, `•` again at level 3 | none |
| `* [ ] task` / `* [x] task` | `• ☐ task` / `• ☑ task` | none |
| `1. item` or `1) item` (up to 3 digits), indented | U+2003 per level, the number as typed, one space | none |
| `# Title` to `###### Title` | `Title` | bold over the title |
| `> quote` | `▍ quote` (U+258D and a space) | italic over the quote text |
| `---`, `***` or `___` alone on a line | `──────────` (ten U+2500) | none |
| A ```` ``` ```` fenced block | the content, fence lines removed | monospace over the content |
| `**bold**` or `__bold__` | `bold` | bold (not inside monospace) |
| `[text](https://…)` | `text (https://…)` | none |
| Other leading spaces or tabs | U+2003 per level | none |
| `*bold*`, `_italic_`, `~strike~`, `` `mono` `` (F33, Android bc2) | the marks removed | bold, italic, strikethrough, monospace |
| A line starting with `-` | left as is (a dash is not a bullet) | none |

Two smaller wire points: the bold italic button sends overlapping bold and
italic ranges, and emoticons go out as plain emoji characters.

Display only (not on the wire): a line that starts with any run of U+2003,
tabs or spaces, then a bullet, `*`, `-`, or a number with `.` or `)`, then
a space, wraps with a hanging indent under its text.

### Emoticons

Change the moment the last character goes in: `:)` `:-)` `=)` `:(` `:-(`
`;)` `;-)` `:'(` `:|` `:-|` `>:(` `O:)` `:-/` `:-*`. Change only when a
space or new line follows: `:D` `:-D` `:P` `:p` `;P` `:O` `:o` `XD` `xD`
`<3` `</3` `:*`. An emoticon counts only on its own, after a space, a new
line or the start of the text. Backspace right after an instant change puts
the characters back. The emoji each one becomes is in the page's `app.js`
and Android's emoticon table; copy both into `emoticons.json`, and treat any
difference as a bug for Mike.
