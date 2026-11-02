# Revised agent skills

Agent skills for [Revised](https://getrevised.com), a directory of expired and dropped domain names that still carry backlinks. Revised sells nothing: you register the name at any registrar.

## Install

```bash
npx skills add getrevised/skills
```

Global install, or a specific agent:

```bash
npx skills add getrevised/skills -g
npx skills add getrevised/skills -a claude-code
```

## Revised MCP server

The skill drives the Revised MCP server at `https://getrevised.com/api/mcp`. If it is not connected, the skill's first step adds it. To add it yourself:

```bash
# Claude Code (then run /mcp, pick revised, Authenticate)
claude mcp add --transport http revised https://getrevised.com/api/mcp

# Codex
codex mcp add revised --url https://getrevised.com/api/mcp && codex mcp login revised
```

Sign-in is OAuth in the browser; there is no key to paste. Cursor, VS Code, Claude.ai, ChatGPT and other clients: https://getrevised.com/docs/agent-setup.

## Skills

### `revised`

An end-to-end workflow for an agent helping someone use an expired domain properly:

1. Connect the Revised MCP server (or REST API).
2. Check the account and plan (`whoami`).
3. Read the user's own site to build a brief: topic, audience, language, geography.
4. Search and shortlist by category, TLD and metric floors.
5. Vet each candidate against the user's site: archive history, linking sites, red flags, trademark risk.
6. Reveal and hold, respecting plan and republication rules (only `open`-tier names may be repeated publicly).
7. Register at a registrar an agent can drive (Cloudflare Registrar, Vercel or Porkbun, through their MCP servers, CLI or API), only after the user confirms the name and price. Any other registrar works by hand.
8. Build a redirect map from the old site's real URLs: relevant 301s, 410 for the rest, never a blanket homepage redirect.
9. Keep and monitor the redirects.

It is built around Google's expired domain abuse policy. The skill steers agents to legitimate uses (topically related domains, relevant redirects, rebuilding useful content) and tells them to refuse plans that exist only to pass link equity, build private blog networks or impersonate a previous owner. It makes no ranking promises.

Layout:

```
skills/revised/
  SKILL.md                    workflow
  references/
    mcp-and-api.md            tools, fields, tiers, errors, limits
    vetting.md                candidate checks and reject rules
    registration.md           Cloudflare, Vercel, Porkbun steps and DNS
    redirects.md              CDX extraction, mapping, snippets
    policy.md                 expired domain abuse guardrail
```

## Requirements

A Revised account (https://getrevised.com). The Free plan can search and read open-tier names; paid plans show every name (https://getrevised.com/pricing). Connecting takes one command; see https://getrevised.com/docs/agent-setup.

## License

MIT. See [LICENSE](LICENSE).
