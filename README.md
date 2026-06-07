# obsidian-vault-tools

A portable [Claude Code](https://code.claude.com) plugin that gives any workspace read/write
access to a local [Obsidian](https://obsidian.md) vault, via the
[Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) plugin and the
[`mcp-obsidian`](https://github.com/MarkusPfundstein/mcp-obsidian) MCP server.

## What's inside

- **`obsidian-vault-tools:vault-reader`** — read-only subagent (list / search / fetch notes).
  No write tools at all.
- **`obsidian-vault-tools:vault-writer`** — write subagent (append + patch). **Cannot delete.**
  Operates autonomously (does not ask before writing). Both agents take the target folder (the
  "scope") from the caller's task prompt — they have no hardcoded project folder.
- **`obsidian` MCP server** — declared in [`.mcp.json`](.mcp.json), run via `uvx mcp-obsidian`.
  The API key is read from the `OBSIDIAN_API_KEY` environment variable — **no secret is stored in
  this repo.**

## Prerequisites (per machine)

1. **Obsidian** running, with the **Local REST API** community plugin enabled.
2. **[uv](https://docs.astral.sh/uv/)** installed (provides `uvx`).
3. The REST API key exported as `OBSIDIAN_API_KEY` (see setup).

## Install

```text
/plugin marketplace add ataersoy/obsidian-vault-tools
/plugin install obsidian-vault-tools@obsidian-vault-tools
```

…or add to `~/.claude/settings.json` (matches the manual-config style):

```json
{
  "extraKnownMarketplaces": {
    "obsidian-vault-tools": {
      "source": { "source": "github", "repo": "ataersoy/obsidian-vault-tools" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": { "obsidian-vault-tools@obsidian-vault-tools": true }
}
```

## Required user setup (not shipped by the plugin)

Plugins cannot ship a secret or a permission allowlist, so add both to your
**`~/.claude/settings.json`** once:

```json
{
  "env": {
    "OBSIDIAN_API_KEY": "<your-obsidian-local-rest-api-key>"
  },
  "permissions": {
    "allow": [
      "mcp__obsidian__obsidian_list_files_in_vault",
      "mcp__obsidian__obsidian_list_files_in_dir",
      "mcp__obsidian__obsidian_get_file_contents",
      "mcp__obsidian__obsidian_batch_get_file_contents",
      "mcp__obsidian__obsidian_simple_search",
      "mcp__obsidian__obsidian_complex_search",
      "mcp__obsidian__obsidian_get_periodic_note",
      "mcp__obsidian__obsidian_get_recent_periodic_notes",
      "mcp__obsidian__obsidian_append_content",
      "mcp__obsidian__obsidian_patch_content"
    ]
  }
}
```

Notes:
- `OBSIDIAN_API_KEY` can also be exported from your shell profile (`~/.zshrc`) instead of
  `settings.json` — either works for the `${OBSIDIAN_API_KEY}` reference in `.mcp.json`.
- `delete_file` is intentionally left out of the allowlist, so any deletion still prompts.

## Usage

Tell Claude which vault folder to work in (the scope) when delegating:

> "Use `obsidian-vault-tools:vault-reader` to pull my notes from `Project_X` into this draft."
>
> "Use `obsidian-vault-tools:vault-writer` to append these findings to `Project_X/notes.md`."

If you don't name a folder, the reader searches the whole vault and the writer asks for the
target rather than guessing.
