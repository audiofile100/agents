#### Custom Commands

A message that starts with `$idx` or `::<prefix>` is one of the commands below. Both use `Index.md` at the vault root (the folder containing this file): a table with columns `LINK`, `PREFIX`, and `DESCRIPTION`, created by the first `$idx`. Each prefix is unique and points to exactly one note.

**Conventions**
- `::<prefix>` (e.g. `::log`) is how a prefix is typed and stored in the `PREFIX` column; the double colon keeps chat inputs from opening an emoji picker. `<prefix>` (e.g. `log`) is the bare name used in entry headings. A prefix is not an Obsidian tag: "tag" here always means an Obsidian `#tag`.
- `<path>` is relative to the vault root, without `.md` (strip it if given). The file on disk is `<path>.md`.
- In table cells, escape every literal `|` as `\|`, including a wikilink's alias pipe (`[[path/note\|display]]`). Otherwise Obsidian splits the cell.

##### `$idx <path>[|<display>] ::<prefix> [description]`

Registers `<path>` under `::prefix`. The path runs up to the first `::`, so it may contain spaces. An optional `|<display>` suffix sets the link's display text. The description is optional free text.

**Examples**
- `$idx path/path/test ::test a short description of this note`
- `$idx path/path/something|log ::log`

**Behavior**
1. If `<path>` or `::prefix` is missing, show the usage line above and stop.
2. If `Index.md` doesn't exist, create it with: blank line ➜ hint line `` `$idx <path>[|<display>] ::<prefix> [description]` `` (in backticks, so Obsidian doesn't read `<path>` as HTML) ➜ blank line ➜ this table header. No other text.
	```
	| LINK | PREFIX | DESCRIPTION |
	| ---- | ------ | ----------- |
	```
3. If `::prefix` is already in the `PREFIX` column, warn the user, show the existing row, and stop.
4. Otherwise, append a row:
	- `LINK`: `[[<path>\|<display>]]`, using the given display text or else the last path segment (`path/path/something|log` ➜ `[[path/path/something\|log]]`, `path/path/test` ➜ `[[path/path/test\|test]]`).
	- `PREFIX`: the `::prefix`, with both colons.
	- `DESCRIPTION`: the description (escape any `|`), or blank.

##### `::<prefix> <prompt>`

Answers `<prompt>` and logs the prompt and response as a new entry in the prefix's note.

**Example:** `::log what's the weather pattern in Seattle? #weather [[Travel]]`

**Behavior**
1. Look up `::prefix` in the `PREFIX` column of `Index.md`. If the file or the prefix isn't there, warn the user and stop. Don't guess a target.
2. The target is `<path>.md`, where `<path>` is the `LINK` text inside `[[ ]]` before the `\|`. Create the file and any missing folders if needed.
3. Number the entry: the highest existing `` `<prefix>-NNN:` `` heading in the file + 1, zero-padded to 3 digits, starting at `001`.
4. Get the time from the system clock (e.g. `date "+%Y-%m-%d %H:%M"`). Don't estimate it.
5. Append the entry in the format below, after a blank line if the file isn't empty.
6. Show the response in chat too, and say where it was logged (e.g. `log-004` in `path/path/something.md`).

```
##### `<prefix>-NNN:`  <prompt>
#some-tag #another-tag
[[some wikilink]] [[another wikilink]]

<YYYY-MM-DD HH:mm>

**Response**

A concise response, using bullets with a **bold** lead-in where it helps readability.

**Sources**
Cite sources whenever possible; otherwise leave this line blank.

---
```

- **Heading:** `<prompt>` with its `#tags` and `[[wikilinks]]` removed and any leftover spaces tidied. Join a multi-line prompt into one line.
- **Tag and link lines:** the removed tags on one line, then the removed wikilinks on the next, space-separated and in prompt order, directly under the heading with no blank line. Leave out a line that would be empty.
- **`---`:** keep the blank line before it. Without it, Obsidian turns the line above into a heading.
