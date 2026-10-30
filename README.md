# Zindex

Immutable agent diagrams. Zindex saves the diagrams your agent makes as numbered revisions that never change. Later, from another conversation or another tool, the agent can read the same diagram, change it and render it, and open any revision by number or by date. Your agent says what's in a diagram; Zindex lays it out and draws it with code, not a model.

This repository is the Zindex plugin. It works in Claude Code, Claude, VS Code, Cursor, Antigravity CLI and other agents that load plugins.

## What it installs

- **The Zindex skill**, in `skills/zindex/`. It tells the agent how to find, read, change and render a diagram, and how to write one. It's the same text zindex.ai serves at <https://zindex.ai/skill/SKILL.md>.
- **The Zindex MCP server's address**, `https://api.zindex.ai/mcp`. The server has nine `diagram_*` tools.

Nothing runs on your computer. The MCP server is remote, and the plugin only tells your client where it is. The skill is Markdown text. There are no scripts, hooks or commands. Your diagrams live in your Zindex account, not on your disk or in this repository.

## Signing in

Without an account, the agent can check a diagram (`diagram_validate`) and preview one without saving it (`diagram_render`). When a tool needs your account, your client sends you to a Zindex page where you sign in or sign up, choose whether the agent may change diagrams or only read them (and, if you belong to a team workspace, which workspace it works in), and choose **Allow**. A new password account verifies its email on that page first; GitHub and Google sign-ups are verified at once. Cursor and Antigravity CLI send you there when they connect instead. Each client's steps are below.

The plugin always connects with that sign-in. It carries no API key. To connect with an API key instead, skip the plugin and add the server yourself, as the [connect guide](https://zindex.ai/docs/tools/) shows.

## Install

### Claude Code

In a session:

```text
/plugin marketplace add zindexai/zindex
/plugin install zindex@zindex
```

Or from your shell:

```bash
claude plugin marketplace add zindexai/zindex
claude plugin install zindex@zindex
```

Then run `/mcp`, select the plugin's `zindex` server and authenticate.

### Claude (claude.ai and the desktop app)

1. Open **Customize > Plugins**, select **Add > Add marketplace** and enter `zindexai/zindex`.
2. Add the Zindex plugin.
3. On the plugin's **Connectors** tab, add Zindex and connect it. On Team and Enterprise plans, an Owner adds the connector first.

A plugin you add on claude.ai is also available in Claude Code when you sign in with the same account.

### VS Code

With GitHub Copilot, run **Chat: Install Plugin From Source** from the Command Palette and enter `https://github.com/zindexai/zindex`. Sign in when VS Code asks.

### Cursor

Clone this repository into Cursor's local plugin folder, then run **Developer: Reload Window**:

```bash
git clone https://github.com/zindexai/zindex ~/.cursor/plugins/local/zindex
```

Cursor asks you to sign in when it connects. Run `git pull` in that folder to update. On Enterprise plans, an admin decides whether local plugins load. To add only the MCP server, use the one-click link in the [connect guide](https://zindex.ai/docs/tools/).

### Antigravity CLI

```bash
agy plugin install https://github.com/zindexai/zindex
```

Then start Antigravity CLI and sign in when it asks; `/mcp` lists the server. Run the install again to update.

### Gemini CLI

Since June 18, 2026, Google doesn't serve Gemini CLI to individual Google accounts and points them to Antigravity CLI (above). Gemini CLI still runs with a Gemini API key or a Gemini Code Assist license:

```bash
gemini extensions install https://github.com/zindexai/zindex
```

Then start Gemini CLI and run `/mcp auth zindex` to sign in. `gemini extensions update zindex` installs a newer version.

### ChatGPT

On the web, open Plugins ([chatgpt.com/plugins](https://chatgpt.com/plugins)), choose **Add custom MCP server**, enter `https://api.zindex.ai/mcp` with OAuth sign-in, and create it as a plugin. ChatGPT opens the Zindex page the first time a tool needs your account.

### Other clients

Clients that support [Agent Plugins 1.0](https://agent-plugins.org/specification), such as GitHub Copilot and Codex, read `plugin.json`, `mcp.json` and `skills/` from this repository. Any other MCP client can connect to `https://api.zindex.ai/mcp` directly; the [connect guide](https://zindex.ai/docs/tools/) has the settings.

## The tools

| Tool | What it does |
|---|---|
| `diagram_find` | Lists your diagrams |
| `diagram_get` | Reads a diagram as an outline, scene JSON or Mermaid; any revision by number or date |
| `diagram_validate` | Checks a diagram without saving it |
| `diagram_create` | Saves a new diagram from a scene, or from Mermaid `erDiagram` or `flowchart` text |
| `diagram_edit` | Changes a diagram; every change is a new revision |
| `diagram_render` | Returns SVG and PNG links |
| `diagram_history` | Lists revisions, or compares two |
| `diagram_delete` | Deletes a diagram, with a 24-hour undo |
| `diagram_share` | Publishes a diagram, makes share links (Pro or Team), reports a problem or sends feedback |

`diagram_delete` asks you to confirm. `diagram_share` asks before it publishes a diagram, makes a share link, reports a problem or sends feedback. The [MCP tools reference](https://zindex.ai/docs/tools/) has every input.

## What the tools send

Each tool call sends its arguments to `api.zindex.ai` over HTTPS: a diagram as scene JSON or Mermaid text, an edit, a one-line message, a search. Zindex keeps your diagrams and every revision in their workspace for as long as it exists, unless you delete them; closing your account, or a free account unused for 12 months, deletes its personal workspace. The [privacy policy](https://zindex.ai/privacy/) covers the rest.

## Files

| File | Read by |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code and Claude |
| `plugin.json`, `mcp.json` | Clients that support Agent Plugins 1.0 |
| `plugin.json`, `mcp_config.json` | Antigravity CLI |
| `gemini-extension.json` | Gemini CLI |
| `skills/zindex/` | All of them |

Each tag, such as `v6.0.0`, names the Zindex release these files shipped with.

## Links

- Docs: <https://zindex.ai/docs/>
- Connect guide: <https://zindex.ai/docs/tools/>
- The Diagram Scene Protocol, the format Zindex stores diagrams in: <https://github.com/zindexai/diagram-scene-protocol>
- Privacy: <https://zindex.ai/privacy/>
- Terms: <https://zindex.ai/terms/>

## Reporting a problem

Open an issue on [this repository](https://github.com/zindexai/zindex/issues) for a problem with the skill or the install steps. For a problem with the service itself, such as an error from a tool or a drawing that looks wrong, write to support@zindex.ai. Report a security problem privately, never in an issue: <https://zindex.ai/security/>.

## License

The skill and the manifests are licensed under the [Apache License 2.0](LICENSE). See [NOTICE](NOTICE). The Zindex service is closed source.

"Zindex" is a trademark. The license covers the files in this repository, not the Zindex name or logo. Section 6 of the license says the same.
