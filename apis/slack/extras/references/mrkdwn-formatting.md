# Slack mrkdwn Formatting

Slack's `mrkdwn` is **not** CommonMark. Wrong markdown silently renders as literal characters. Always validate before sending.

Applies to: `text` field of `chat.postMessage`, `chat.postEphemeral`, `chat.scheduleMessage`, `chat.update`, `chat.meMessage`, and Block Kit `section.text` / `context.elements` with `type: "mrkdwn"`.

## Inline syntax

| Style       | Slack mrkdwn         | NOT (CommonMark)        |
|-------------|----------------------|-------------------------|
| Bold        | `*bold*`             | `**bold**`              |
| Italic      | `_italic_`           | `*italic*` / `_italic_` |
| Strike      | `~strike~`           | `~~strike~~`            |
| Inline code | `` `code` ``         | same                    |
| Code block  | ```` ```code``` ```` | same (no lang hint)     |
| Block quote | `> line` per line    | same                    |

Notes:
- `**x**` renders literally as `**x**`. Use single `*`.
- Code block has NO language tag — ```` ```python ```` shows `python` as first line.
- No headings. `# H1` renders literal. Use `*bold*` on its own line.
- No tables. Use code block or Block Kit fields.
- No nested formatting inside code spans.

## Lists

- Bullet: prefix line with `• ` or `- ` (renders as bullet). No auto-indent — use spaces manually.
- Numbered: type `1. `, `2. ` literally. Slack does not renumber.
- For reliable lists prefer Block Kit `rich_text_list` element.

## Links

```
<https://example.com>                  → bare URL
<https://example.com|click here>       → linked text
<mailto:a@b.com|email me>              → mailto
```

Do NOT use `[text](url)` — renders literal.

## Mentions

| Target              | Syntax                          |
|---------------------|---------------------------------|
| User                | `<@U024BE7LH>`                  |
| Channel             | `<#C024BE7LR>` or `<#C024BE7LR\|name>` |
| User group          | `<!subteam^SAZ94GDB8\|@team>`   |
| Here (online users) | `<!here>`                       |
| Channel (all)       | `<!channel>`                    |
| Everyone (workspace)| `<!everyone>`                   |
| Date                | `<!date^1392734382^{date_short}\|Feb 18, 2014>` |

Mentions require IDs, not display names. Get IDs from `users.lookupByEmail`, `users.list`, `conversations.list`.

## Escaping

Slack auto-escapes `&`, `<`, `>` in user-supplied text **only if you escape yourself before sending**:

| Char | Replace with |
|------|--------------|
| `&`  | `&amp;`      |
| `<`  | `&lt;`       |
| `>`  | `&gt;`       |

Do not escape inside `<...>` link/mention syntax — only the visible text.

To prevent Slack from parsing `*_~` etc., wrap in inline code or set `mrkdwn: false`.

## Disabling mrkdwn

- `chat.postMessage` body: `mrkdwn=false` → `text` sent verbatim, no formatting.
- In Block Kit `section.text`: use `type: "plain_text"` instead of `"mrkdwn"`.

## Block Kit vs mrkdwn

| Use mrkdwn in `text`  | Use Block Kit `blocks` |
|-----------------------|------------------------|
| Plain notification    | Buttons, selects, inputs |
| Single-paragraph msg  | Multi-section layout     |
| Fallback for old clients | Fields, dividers, images |

When sending `blocks`, still include `text` — it's the notification preview and a11y fallback.

Block text fields accept mrkdwn when `"type": "mrkdwn"`. `plain_text` blocks render literal — no formatting.

## Common pitfalls

1. **`**bold**` not bold** — use `*bold*`.
2. **`[link](url)` literal** — use `<url|link>`.
3. **`@username` not a mention** — must be `<@Uxxx>` with ID.
4. **`#channel-name` not a link** — must be `<#Cxxx>` or `<#Cxxx|name>`.
5. **Code block language tag visible** — Slack ignores it; strip before sending.
6. **Newlines lost** — set `text` with literal `\n` (JSON `"line1\nline2"`); HTTP form bodies need URL-encoded `%0A`.
7. **Mentions inside code spans don't notify** — pull mention out of backticks.
8. **`mrkdwn` field, not `markdown`** — common typo.

## Quick reference: postMessage payload

```json
{
  "channel": "C024BE7LR",
  "text": "Build *failed* on <https://ci/123|job 123>. <@U024BE7LH> please check.",
  "mrkdwn": true,
  "thread_ts": "1234567890.123456",
  "blocks": [
    {
      "type": "section",
      "text": {"type": "mrkdwn", "text": "*Deploy* failed: `migration 0042`"}
    }
  ]
}
```

## References

- Slack docs: https://api.slack.com/reference/surfaces/formatting
- Block Kit: https://api.slack.com/block-kit
- Date formatting tokens: https://api.slack.com/reference/surfaces/formatting#date-formatting
