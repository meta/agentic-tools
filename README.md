# Agentic Tools

**Meta Agentic Tools** is Meta's marketplace of agent plugins and skills for developers building apps and integrations on the Meta Social Technologies Developer Platform (developers.facebook.com), also known as Meta for Developers: Social Technologies. Manage the Meta apps you build there: settings, webhooks, compliance, app review, and API health, via the [Meta Social Technologies MCP](https://developers.facebook.com/documentation/mcp/devtools-mcp).

> [!NOTE]
> **Meta Social Technologies MCP was formerly known as Meta Developer Tools MCP.** The server endpoint, the `devtools` plugin id, and the `devtools_` tool name prefix are unchanged, so existing installs and client configurations keep working.

This repo is structured as a **multi-plugin marketplace**: each product group ships as a self-contained plugin under `plugins/`, so it can host additional developer plugins alongside the first one, the **Meta Social Technologies Developer** plugin (`devtools`). The skills follow the open [Agent Skills standard](https://agentskills.io) (`SKILL.md`), so the same skills work in **Claude Code, Cursor, and Codex**.

> Developer Platform workflows call **Meta Social Technologies MCP** (`https://mcp.facebook.com/devtools`). `whatsapp-business-integration` spans it and **WhatsApp Business Tools MCP**; `whatsapp-business-marketing-messages-max-price-setup` uses only the WhatsApp server.
>
> The `devtools` plugin bundles both servers: Meta Social Technologies MCP (`devtools`) and WhatsApp Business Tools MCP (`whatsapp_business_tools`, `https://mcp.facebook.com/whatsapp_business_tools`). You do not need to connect both: connect one and the skills that use it work right away; connect the other when a skill you run needs it. Skills-only installs bundle neither; add them as shown under [Install via Vercel](#install-via-vercel-skillssh), following each skill's declared dependencies.

## Installation

### Claude Code

```bash
claude plugin marketplace add facebook/agentic-tools
claude plugin install devtools@facebook
```

Then connect the MCP servers you need (their config is bundled with the plugin). You do not need both; connect the other one later when a skill needs it:

```
/mcp → connect to devtools → authenticate                  # Developer Platform skills
/mcp → connect to whatsapp_business_tools → authenticate   # WhatsApp skills
```

### Cursor

Install the `devtools` plugin from the Cursor marketplace, then authenticate the MCP server:

1. Open the Cursor marketplace and install the `devtools` plugin (published from this repo). The plugin bundles the skills and the **Meta Social Technologies MCP** and **WhatsApp Business Tools MCP** configs.
2. Authenticate the remote server your skills use — **Meta Social Technologies MCP** (`devtools`) or **WhatsApp Business Tools MCP** (`whatsapp_business_tools`) — in **Settings → MCP**, then complete the OAuth flow. You do not need both; connect the other one when a skill needs it.

To try it before it's published, copy or symlink `plugins/devtools` to `~/.cursor/plugins/local/devtools`, then run **Developer: Reload Window**.

### Codex

Install the skills with the built-in installer, pointing it at this repo:

```text
$skill-installer
```

Then add the **Meta Social Technologies MCP** server to `~/.codex/config.toml`:

```toml
[mcp_servers.devtools]
url = "https://mcp.facebook.com/devtools"

[mcp_servers.whatsapp_business_tools]
url = "https://mcp.facebook.com/whatsapp_business_tools"
```

Codex will run its OAuth flow on first use. Restart Codex after editing the config.

## WhatsApp Max Price

`whatsapp-business-marketing-messages-max-price-setup` is grouped under `plugins/devtools/skills/marketing-messages/`. The leaf name and command identity are unchanged. Claude and Cursor explicitly scan both `./skills/` and `./skills/marketing-messages/` so existing flat skills remain available; Codex retains its recursive `./skills/` root. The grouping folder has no `SKILL.md`, inherited instructions or automatic namespace. Install the existing `devtools` plugin, or install just the skill from `meta/agentic-tools` with your client's skill installer. The repository organization does not rename the existing `facebook` marketplace ID or `devtools@facebook` plugin identifier.

Plugin installs bundle `whatsapp_business_tools` (`https://mcp.facebook.com/whatsapp_business_tools`); skills-only installs must add it. Complete OAuth in your client. The skill declares this dependency in `metadata.mcp`; installation does not authenticate it or grant Max Price eligibility. You can connect MCP first and then invoke the installed skill. Some clients require a reload. Without the skill, the existing template tools retain their guidance and confirmation gates.

The skill uses the Holiday 2026 guide for ladder-up/leap strategies, newly opted-in versus dormant cohort advice, and country-specific A/B test planning. Recommendations are relative to current published rates, not automatic country overrides or resolved monetary bids. Live rates, reach estimates and monetary normalization are not supplied by the current account/template tools; missing inputs require a strategy explanation or explicit-value fallback. Exact-preview approval remains mandatory. The skill does not schedule price changes or send messages. For new account/phone setup, use `whatsapp-business-integration` separately.

## Install via Vercel (skills.sh)

The skills are also listed on Vercel's skills directory, [skills.sh](https://skills.sh/facebook/agentic-tools), and install with the `skills` CLI into any agent it supports:

```bash
# A single skill
npx skills add facebook/agentic-tools --skill <name>

# All skills
npx skills add facebook/agentic-tools --skill '*'
```

> [!WARNING]
> This installs the skill text only. It does NOT install or authenticate the MCP server — you must add it separately, or the skills cannot run.

Add the servers each installed skill requires. `whatsapp-business-integration` uses both servers below; `whatsapp-business-marketing-messages-max-price-setup` needs only `whatsapp_business_tools`. The example uses Claude Code's `http` transport name; for Cursor, omit `type` and keep each URL.

```json
{
  "mcpServers": {
    "devtools": {
      "type": "http",
      "url": "https://mcp.facebook.com/devtools"
    },
    "whatsapp_business_tools": {
      "type": "http",
      "url": "https://mcp.facebook.com/whatsapp_business_tools"
    }
  }
}
```

Then complete the OAuth flow for each server in your own agent (for example `/mcp` in Claude Code), and restart or reload the agent if it does not pick up the new server.

The skills live under `plugins/devtools/skills/` rather than a top-level `skills/` folder. If `npx skills add facebook/agentic-tools -l` reports no skills, name one with `--skill <name>` or pass `--full-depth`.

## Skills

| Skill | Description |
|-------|-------------|
| `/app-health-check` | Full app audit — settings, security, compliance, and app review in one report |
| `/webhook-setup` | End-to-end webhook subscription wizard with test verification |
| `/app-review-prep` | App Review readiness check — requirements, privileges, and submission history |
| `/debug-webhooks` | Troubleshoot webhook delivery issues — inspect, test, and fix subscriptions |
| `/compliance-check` | Compliance audit — violations, required actions, and remediation guidance |
| `/api-health` | API health monitoring — rate limits, call volume, and a shared Graph API version-deprecation check; not an audit of actual calls |
| `/api-integration` | Start a new API integration — fetches setup guides, auth, permissions, and code examples |
| `/search-docs` | Search Meta developer documentation for API guides and references |
| `/search-code-examples` | Find code examples for a Meta API task, quoted verbatim from the docs |
| `/debug-errors` | Explain a Meta API error code and how to fix it, from the official error-code docs |
| `/debug-access-token` | Diagnose an access token issue (expired, wrong scopes, wrong app, invalid) without the token ever entering the agent context |
| `/whatsapp-business-integration` | Set up WhatsApp Business messaging end to end, from the app to the first sent message |
| `/app-setup` | Set up an app for a product integration — create or pick the app, connect a business portfolio, and add use cases |
| `/whatsapp-business-marketing-messages-max-price-setup` | Explain holiday/cohort choices and configure explicitly approved Max Price settings for Direct sends; WhatsApp MCP only |

## Architecture

```
agentic-tools/
├── server.json                  # Public MCP registry listing (registry.modelcontextprotocol.io)
├── .github/
│   └── workflows/
│       └── publish-mcp-registry.yml  # Publishes server.json to the registry via GitHub OIDC
├── .claude-plugin/
│   └── marketplace.json         # Claude Code / Codex marketplace manifest
├── .cursor-plugin/
│   └── marketplace.json         # Cursor marketplace manifest (lists plugins by source)
├── .cursor/
│   └── mcp.json                 # Meta Social Technologies MCP config for manual Cursor setup
└── plugins/
    └── devtools/                # The Meta Social Technologies Developer plugin
        ├── .claude-plugin/
        │   └── plugin.json       # Claude Code plugin manifest
        ├── .cursor-plugin/
        │   └── plugin.json       # Cursor plugin manifest
        ├── .codex-plugin/
        │   └── plugin.json       # Codex plugin manifest
        ├── .mcp.json             # MCP config for Claude Code / Codex (auto-bundled)
        ├── mcp.json              # MCP config for Cursor (auto-discovered)
        ├── skills/               # Skill definitions (SKILL.md per workflow)
        │   ├── app-health-check/
        │   ├── webhook-setup/
        │   ├── app-review-prep/
        │   ├── debug-webhooks/
        │   ├── compliance-check/
        │   ├── api-health/
        │   ├── api-integration/
        │   ├── search-docs/
        │   ├── search-code-examples/
        │   ├── debug-errors/
        │   ├── debug-access-token/
        │   ├── whatsapp-business-integration/
        │   ├── app-setup/
        │   └── marketing-messages/
        │       └── whatsapp-business-marketing-messages-max-price-setup/
        │           ├── SKILL.md
        │           └── references/
        │               ├── scenarios.md
        │               └── holiday-2026.md
        └── hooks/
            ├── session-init.sh       # Session-start hook
            └── session-context.json  # Context injected at session start
```

Both MCP servers are remote — they are not bundled as processes; users authenticate them in their agent.

## License

MIT
