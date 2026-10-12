# Optale Console skills

Agent Skills that teach an AI agent how to work in an Optale Console through the hosted Optale MCP server. They follow the open [Agent Skills format](https://agentskills.io/specification), so any client that reads `SKILL.md` folders can use them: Claude Code, Codex, Cursor, GitHub Copilot in VS Code, Gemini CLI, goose, OpenCode and others.

The MCP server gives an agent the tools. These skills give it the workflows: which tool answers which question, in what order, how to page to a complete answer, and how to recover from errors.

| Skill | Use it for |
| --- | --- |
| `optale-console` | Start here. What the Console is, the connection, session mode, discovering the tenant's types, reading responses and errors, and which skill to load next. |
| `optale-find-and-read` | Finding, listing, filtering, counting and opening Objects, Relationships and search results. |
| `optale-objects` | Creating, updating, linking, archiving and deleting Objects and Relationships without duplicates. |
| `optale-functions` | Running and repairing saved queries and saved actions. |
| `optale-wiki` | Reading, writing, patching, moving and restoring Wiki pages, and attaching pages to Objects. |
| `optale-files` | Storing, placing, attaching, viewing, reading and exporting files. |

## 1. Connect your agent to the Console

1. In the Console, open **Settings > External agents** and create a new connection.
2. Name it after the agent and machine, choose its **mode** and tick the **tools** it may use:
   - **Auto** runs the selected tools, reads and writes.
   - **Read Only** runs reads; writes are refused.
3. Copy the **MCP URL** and the **key**. The key is shown once.
4. Add the server to your client. The Console shows the exact snippet for several clients. For example, Claude Code:

   ```sh
   claude mcp add --scope user --transport http optale <MCP URL> \
     --header "Authorization: Bearer <key>"
   ```

   Codex, in `~/.codex/config.toml`, with the key in the `OPTALE_MCP_KEY` environment variable:

   ```toml
   [mcp_servers.optale]
   url = "<MCP URL>"
   bearer_token_env_var = "OPTALE_MCP_KEY"
   ```

   Any other client that supports remote MCP servers over HTTP works the same way: the MCP URL plus an `Authorization: Bearer <key>` header.

Every call runs as the person who created the key and sees only what they can see. Revoke the connection in the same place to cut access.

## 2. Install the skills

### Claude Code

```text
/plugin marketplace add Optale-no/console-skills
/plugin install optale@optale-console-skills
```

Or copy the folders in `skills/` into `~/.claude/skills/` (for you) or `.claude/skills/` (for one project).

### Codex, Cursor, GitHub Copilot in VS Code, Gemini CLI

These clients read skills from `~/.agents/skills/` (for you) or `.agents/skills/` in a project:

```sh
git clone https://github.com/Optale-no/console-skills.git
mkdir -p ~/.agents/skills
cp -R console-skills/skills/* ~/.agents/skills/
```

Each client also has its own folder if you prefer it: `~/.codex/skills/`, `~/.cursor/skills/`, `~/.copilot/skills/`, `~/.gemini/skills/`.

### Any other client

Copy the folders in `skills/` into the directory your client loads skills from. Keep each folder whole: a skill is its folder, with `SKILL.md` and any `references/`.

## 3. Use them

Ask your agent about your Console in plain words: "which customers have open deals", "add Acme as a company and link Jane to it", "append today's notes to the launch plan page". The agent loads `optale-console` first, discovers your Console's own types, and moves on to the skill the task needs.

Tool names in the skills are the bare MCP names, such as `object_query`. Clients add their own prefix, such as `mcp__optale__object_query` in Claude Code with a server named `optale`.

## Updates

This repository is published from the Optale Console source when a Console release reaches production, so the skills match the tools that release serves. Pull or reinstall to update.

## License

MIT. See [LICENSE](LICENSE).
