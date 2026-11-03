# Shared test vectors

One set of cases for Backchannel's text rules, run by every client:
the page (`contrib/backchannel/helper/public/app.js`), Android
(`net/mikesimone/backchannel/Markdown.kt`, `UnicodeStyles.kt`), the Mac
app (which uses the page), and iOS (`Backchannel/Features/`).

Status: **to build.** The cases come out of the existing tests first:
`contrib/backchannel/helper/test/backchannel_test.mjs` (F33 cases) and
Android's `UnicodeStylesTest.kt`. Where the page and Android disagree,
that's a bug to raise with Mike, not a case to pick a side on silently.

## Files

- `markdown-shortcuts.json`: typed or pasted text in, Signal body ranges out.
- `unicode-copy.json`: styled text in, Unicode clipboard text out, and back.
- `forward-mentions.json`: forward text with "@Name", the destination's
  members, expected mention ranges.

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
