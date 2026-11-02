<!-- claude-projects-attribution: {"github_login":"yxshee"} -->
_Requested by **Sir** · [project thread](https://claude.ai/code/project/chan_01FGhb5Fba7ZosnqFBVKeY7t?thread=cmsg_01FGhb5Fba7ZosnqFBVKeY7tB5HtdF6YvT4RNDg9dYg6Fi)_

`ics()` writes `SUMMARY`, `DESCRIPTION` and `LOCATION` without escaping them as RFC 5545 requires for TEXT values ([section 3.3.11](https://www.rfc-editor.org/rfc/rfc5545#section-3.3.11)). Calendar apps read the file with the RFC escaping rules, so:

```js
ics({
  title: "Standup\nDTSTART:20300101T000000Z",
  description: "Files are in C:\\new\\data; bring a laptop, charger",
  location: "Room 4, Floor 2",
  start: "2026-11-03T10:00:00Z",
  end: "2026-11-03T11:00:00Z",
});
```

produces

```
SUMMARY:Standup
DTSTART:20300101T000000Z
DESCRIPTION:Files are in C:\new\data; bring a laptop, charger
LOCATION:Room 4, Floor 2
```

Parsed with ical.js, the title is just `Standup`, the event gets a second `DTSTART`, and the description reads `Files are in C:` + line break + `ew\data; ...`, because `\n` is the escape for a line break.

The code already tried to escape commas and semicolons in the description and location (added in #179), but `.replace(/,/gm, "\,")` is a no-op: `"\,"` in a JS string is just `","`. Backslashes and the title were never escaped.

This adds an `escapeIcsText` helper that escapes `\`, `;`, `,` and line breaks (`\r\n`, `\r` and `\n`), and keeps the existing behavior of dropping whitespace at the start of a line. It's used for `SUMMARY`, `DESCRIPTION` and `LOCATION`. With it, ical.js reads the three values back exactly as they were passed in. The other services are unchanged, and the existing snapshots don't change because none of them contain these characters.

Added a test in `src/index.spec.ts` with a title that contains a line break, a description with a Windows path, a semicolon and a comma, and a location with a comma.
