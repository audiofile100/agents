#### Custom Commands

A message that starts with `$idx` or `::<prefix>` is one of the commands below. They all use `Index.md` at the vault root (the folder containing this file): a table with columns `LINK`, `PREFIX`, and `DESCRIPTION`, created by the first `$idx`. Each prefix is unique and points to exactly one note.

**Conventions**
- `::<prefix>` (e.g. `::log`) is how a prefix is typed and stored in the `PREFIX` column; the double colon keeps chat inputs from opening an emoji picker. `<prefix>` (e.g. `log`) is the bare name used in entry headings. A prefix is one word with no `:` in it.
- `::<prefix>:<mode>` (e.g. `::log:note`) runs a mode of the logging command. The mode isn't part of the prefix: look up `::<prefix>` alone. If the mode isn't one defined below, warn the user and stop.
- `<path>` is relative to the vault root, without `.md` (strip it if given). The file on disk is `<path>.md`.
- A tag is an Obsidian `#tag`, never a prefix: `#` plus a word, at the start of a line or after a space, and not all digits (`#gpu`, but not `#42`, `C#`, or `# Heading`). A wikilink is `[[...]]`, but not an embed (`![[...]]`).
- In table cells, escape every literal `|` as `\|`, including a wikilink's alias pipe (`[[path/note\|display]]`). Otherwise Obsidian splits the cell.

##### `$idx <path>[|<display>] ::<prefix> [description]`

Registers `<path>` under `::<prefix>`. The path runs up to the first `::`, so it may contain spaces. An optional `|<display>` suffix sets the link's display text. The prefix is the word right after `::`; any text after it is the description.

**Examples**
- `$idx path/path/test ::test a short description of this note`
- `$idx path/path/something|log ::log`

**Behavior**
1. If `<path>` or `::<prefix>` is missing, or the prefix contains a `:`, show the usage line above and stop.
2. If `Index.md` doesn't exist, create it with: blank line ➜ hint line `` `$idx <path>[|<display>] ::<prefix> [description]` `` (in backticks, so Obsidian doesn't read `<path>` as HTML) ➜ blank line ➜ this table header. No other text.
	```
	| LINK | PREFIX | DESCRIPTION |
	| ---- | ------ | ----------- |
	```
3. If `::<prefix>` is already in the `PREFIX` column, warn the user, show the existing row, and stop.
4. Otherwise, append this row and show it in chat:
	- `LINK`: `[[<path>\|<display>]]`, using the given display text or else the last path segment (`path/path/something|log` ➜ `[[path/path/something\|log]]`, `path/path/test` ➜ `[[path/path/test\|test]]`).
	- `PREFIX`: the `::<prefix>`, with both colons.
	- `DESCRIPTION`: the description (escape any `|`), or blank.

##### `::<prefix> <prompt>`

Answers `<prompt>` and logs the prompt and response as a new entry in the prefix's note.

**Example:** `::log what's the weather pattern in Seattle? #weather [[Travel]]`

**Behavior**
1. If `<prompt>` is empty, show the usage line above and stop.
2. Look up `::<prefix>` in the `PREFIX` column of `Index.md`. If the file or the prefix isn't there, warn the user and stop. Don't guess a target.
3. The target is `<path>.md`, where `<path>` is the `LINK` text inside `[[ ]]` before the `\|`. Create the file and any missing folders if needed.
4. Number the entry: the highest existing `` `<prefix>-NNN:` `` heading in the file + 1, zero-padded to 3 digits, starting at `001`. All modes share this numbering.
5. Get the time from the system clock (e.g. `date "+%Y-%m-%d %H:%M"`). Don't estimate it.
6. Append the entry in the format below, after a blank line if the file isn't empty.
7. Show the response in chat too, and say where it was logged (e.g. `log-004` in `path/path/something.md`).

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

- **Heading:** `<prompt>` with its tags and wikilinks removed and any leftover spaces tidied. Join a multi-line prompt into one line.
- **Tag and link lines:** the removed tags on one line, then the removed wikilinks on the next, each listed once, space-separated, in the order they appear. Put them directly under the heading with no blank line, and leave out a line that would be empty.
- **`---`:** keep the blank line before it. Without it, Obsidian turns the line above into a heading.

##### `::<prefix>:summarize <text>`

Summarizes `<text>` (usually pasted) and logs the summary as a new entry in the prefix's note, under a title you write.

**Example:** `::log:summarize <pasted article> #weather [[Travel]]`

**Behavior:** the same as `::<prefix> <prompt>`, with `<text>` in place of `<prompt>`, except:
- **Heading:** a short title you write that captures what the text is about (a few words, like a note title).
- **Response:** a concise summary of `<text>`. Don't copy the original text into the entry.
- **Sources:** the text's origin (URL, author, publication) if it names one; otherwise blank.

##### `::<prefix>:note <text>`

Logs `<text>`, the user's own note, verbatim as a new entry in the prefix's note, under a title you write. Doesn't answer or comment on it.

**Example:** `::log:note Applications involving deep learning are demanding on current hardware capabilities #gpu`

**Behavior:** the same as `::<prefix> <prompt>`, with `<text>` in place of `<prompt>`, except:
- **Heading:** a very short title you write that captures the note (a few words, like a note title).
- **Body:** `<text>` exactly as written, minus its tags and wikilinks (tidy only the spaces they leave). Keep its line breaks and formatting; don't reword it, fix typos, or add to it.
- **Format:** a `**Note**` label in place of `**Response**`, and no `**Sources**` section:
	```
	##### `<prefix>-NNN:`  <title>
	#some-tag #another-tag
	[[some wikilink]] [[another wikilink]]

	<YYYY-MM-DD HH:mm>

	**Note**

	The note, verbatim.

	---
	```
- **Chat:** confirm the title and where it was logged; there's no response to show.
