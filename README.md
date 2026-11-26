# PublishBuddy for AI agents

Connect your AI agent to [PublishBuddy](https://publishbuddy.com), a social media management tool.

Your agent reads your accounts and your brand kit, and looks at how your posts are doing. It drafts posts and checks them against each network's rules. You approve, then it schedules them. Nothing is published without your OK.

## What's in here

No code. Settings files and one skill, all pointing at PublishBuddy's hosted MCP server:

```text
https://api.publishbuddy.com/mcp
```

- **The settings files** tell each agent where the server is. You sign in with your PublishBuddy account in the browser. No API key goes in any file.
- **The skill** teaches the agent how to work: look at your accounts, read your brand kit, draft, check, and ask you before anything is scheduled.

## What you need

- A PublishBuddy account on Pro or Business. The 14-day free trial works too, with no credit card required.
- At least one social account connected in PublishBuddy.

## Install

Each route below says whether we've tested it ourselves or it follows the vendor's docs.

### Claude Code

```bash
claude plugin marketplace add publishbuddy/agent-kit
claude plugin install publishbuddy@publishbuddy
```

Or, inside a session: `/plugin install publishbuddy --marketplace publishbuddy/agent-kit`. Then run `/mcp`, pick the PublishBuddy server and sign in.

- Status: install tested on 26 Nov 2026 with Claude Code 2.1.330, from a local copy of this repository.
- Vendor page: https://code.claude.com/docs/en/plugins/publish

### Claude and Claude Cowork

Open **Customize > Plugins**, choose **Add > Add marketplace** and enter `publishbuddy/agent-kit`. Add the PublishBuddy plugin, then open its **Connectors** tab and connect.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://claude.com/docs/plugins/overview

### Cursor

Open **Customize**, find PublishBuddy and select **Install**. Sign in to PublishBuddy when Cursor asks.

- Status: not live yet. PublishBuddy isn't in the Cursor Marketplace yet.
- Vendor pages: https://cursor.com/docs/plugins and https://cursor.com/docs/reference/plugins

To try it before the listing, copy this folder to `~/.cursor/plugins/local/publishbuddy` and reload the window. That route follows Cursor's docs, and we haven't tested it yet.

### Grok Bot

Grok Bot's **Connect apps** catalog carries Cursor Marketplace plugins, so PublishBuddy shows up there after the Cursor listing.

- Status: not live yet. It needs the Cursor Marketplace listing first.
- Vendor pages: https://docs.x.ai/grok-bot/computer-and-apps and https://github.com/cursor/plugins/blob/main/schemas/plugin.schema.json

### Grok Build

Grok Build reads Claude Code marketplaces and plugins as they are. Add this repository with `grok plugin marketplace add` (run it with `--help` for the argument form), then install PublishBuddy from the Marketplace tab that `/plugins` opens.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor pages: https://docs.x.ai/build/features/skills-plugins-marketplaces and https://docs.x.ai/build/cli/reference

### ChatGPT desktop app and Codex

```bash
codex plugin marketplace add publishbuddy/agent-kit
```

Restart the ChatGPT desktop app, open the Plugins Directory, choose the `publishbuddy` marketplace and install the plugin.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://developers.openai.com/plugins/build/plugins

### Perplexity Computer

Open **Plugins**, choose **Add custom > Import plugin from GitHub** and paste `https://github.com/publishbuddy/agent-kit`. Then connect PublishBuddy on the plugin's page.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://www.perplexity.ai/help-center/en/articles/2026100802-create-or-upload-a-custom-plugin

### Gemini CLI

```bash
gemini extensions install https://github.com/publishbuddy/agent-kit
```

Then run `/mcp auth publishbuddy` in Gemini CLI to sign in.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://geminicli.com/docs/extensions/reference/

### Kimi Code CLI

```text
/plugins install https://github.com/publishbuddy/agent-kit
/reload
```

When the server asks you to sign in, run `/mcp-config login` with the server name that `/mcp` shows.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://moonshotai.github.io/kimi-code/en/customization/plugins.html

### NanoClaw

Copy this repository into your NanoClaw `templates/` folder as `templates/publishbuddy`, then:

```bash
ncl groups create --template publishbuddy --name "PublishBuddy"
```

NanoClaw keeps credentials in its own vault. It asks for a PublishBuddy API key the first time the agent calls the server. Create one in PublishBuddy under **User Settings > Developers**.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://docs.nanoclaw.dev/templates/using-templates

### Only the skill

```bash
npx skills add publishbuddy/agent-kit
```

This adds the skill to the agents on your machine. The agent still needs the MCP server, which the next section covers.

- Status: follows the vendor's docs. Not tested by us yet.
- Vendor page: https://skills.sh/docs

### Any other agent that speaks MCP

Add a remote MCP server with the address above and sign in when the browser opens.

- Status: works today.
- Guide: https://docs.publishbuddy.com/mcp-server/connect

## You stay in charge

- Every post the agent creates is a draft.
- The agent shows you the text, the account and the time, then asks.
- It schedules or publishes only after your yes. Publishing right away and deleting always show a preview first.
- By default, a workspace keeps posts from agents as drafts until you confirm.
- The workspace owner can pause agents or limit them under **Team Members > Agent accounts**.

## What this plugin connects to

- **One host:** `api.publishbuddy.com`. The MCP server and the file upload address both live there.
- **Sign-in:** OAuth in your browser. You pick the workspace the agent works in, or tick the box that covers all your workspaces. No password or key is stored in this repository.
- **Files from your computer:** the agent asks the server for a short-lived upload address and sends the file there with `curl`. It sends only the files you name.
- **Nothing runs on install.** This repository has no scripts and no hooks.
- **Disconnect** at any time in PublishBuddy under **User Settings > Developers > Connected apps**.

## Permissions

A new connection can read, draft, schedule, add media and read analytics. Publishing right away, deleting, spending AI credits and saving preferences are extra permissions that you grant.

## AI credits

Images, video and caption variations made with PublishBuddy's Studio use AI credits from your plan. The agent shows the cost and asks first. Drafting, scheduling, uploads and analytics use none.

## Files

| File | Read by |
| --- | --- |
| `plugin.json`, `mcp.json` | Agent Plugins 1.0.0 clients: Cursor, ChatGPT and Codex, Perplexity Computer, NanoClaw |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code, Claude, Cowork, Grok Build |
| `.cursor-plugin/plugin.json` | Cursor, for the Marketplace listing: display name, logo, category |
| `gemini-extension.json` | Gemini CLI |
| `kimi.plugin.json` | Kimi Code CLI |
| `skills/publishbuddy/SKILL.md` | All of them |

## Maintaining this repository

**Logo files.** One square PNG icon, 512 by 512 pixels, sits in the assets folder. The Cursor and Claude manifests both point at it. No other logo file is required. Two optional extras: a vector (SVG) copy of the mark for sharper listings, and a 1280 by 640 pixel social preview image for the GitHub page.

**Version.** The version number is in five files: `plugin.json`, `.claude-plugin/plugin.json`, `.cursor-plugin/plugin.json`, `gemini-extension.json` and `kimi.plugin.json`. Change all five together. The description is the same sentence in those five files too.

**Checks before a push.**

```bash
claude plugin validate . --strict
npx --yes ajv-cli@5 validate --spec=draft2020 --strict=false -s plugin.schema.json -d plugin.json
npx --yes ajv-cli@5 validate --spec=draft2020 --strict=false -s mcp.schema.json -d mcp.json
```

The two schema files come from https://agent-plugins.org/schemas. Cursor publishes its manifest schema at https://github.com/cursor/plugins/tree/main/schemas.

**The skill follows the server.** Tool names, arguments and error codes in the skill come from the live MCP server. When the server's tools change, update the skill.

## Support

Email hello@publishbuddy.com. The docs are at https://docs.publishbuddy.com/mcp-server/connect.

## License

MIT. See the [LICENSE](LICENSE) file.

The PublishBuddy name and logo, including the icon in the assets folder, aren't covered by that license.
