# agents

A collection of agent instruction files.

| File | Purpose |
| --- | --- |
| `AGENTS.md` | Obsidian vault commands: `$idx` registers a note under a `::prefix`, `::prefix <prompt>` logs a prompt and its answer to that note, `::prefix:summarize <text>` logs a titled summary of pasted text there, and `::prefix:note <text>` logs your own note there verbatim under a short title. |

## Recommended Obsidian settings for `AGENTS.md`

In the vault's **Settings → Files & links**:

| Setting | Value |
| --- | --- |
| Automatically update internal links | On |
| New link format | Absolute path in vault |

**Why:** the links in `Index.md` are read as paths from the vault root, so `[[sample]]` means `sample.md` at the root. With the default "Shortest path when possible", moving a note can shorten its `Index.md` link to just the note name. The next `::prefix` command would then create a new, empty note at the root instead of writing to the one you moved.

These settings are stored in the vault's `.obsidian/` folder, so set them on each device that opens the vault unless you sync that folder.

## Moving and renaming notes

- Move and rename inside Obsidian (drag in the sidebar, or use "Move file to…") so it can update links. Moves made in Finder or the terminal aren't tracked.
- If you remove a note, also remove its row from `Index.md`. Otherwise the next `::prefix` command recreates the note.
