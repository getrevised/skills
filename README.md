# Revised agent skills

Agent skills for [Revised](https://getrevised.com), domain intelligence for the AI age. Look up any domain (backlinks and linking sites, traffic rank, Agent Citability, registration and expiry, archive and spam history, an estimated value), search the hundreds of thousands of names Revised tracks, browse a curated directory of expired names that still carry link history, and watch your own domains for changes. Revised does not sell domain names: you register a name at any registrar.

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

Sign-in is OAuth in the browser; there is no key to paste. Cursor, VS Code, Claude.ai, ChatGPT and other clients: https://getrevised.com/docs/agent-setup. Scripts and CI can use the REST API with a key instead: https://getrevised.com/docs/api/introduction.

## Skills

### `revised`

Teaches an agent which Revised tool answers which question, and walks it through using an expired domain properly:

1. Connect the Revised MCP server (or REST API).
2. Check the account and plan (`whoami`).
3. Read the user's own site to build a brief: topic, audience, language, geography.
4. Search the curated directory by category, TLD and metric floors, and on paid plans the wider pool by raw metrics.
5. Vet each candidate against the user's site: archive and spam history, linking sites, red flags, trademark risk, value.
6. Hold while the user decides, respecting plan and republication rules (only `open`-tier names may be repeated publicly).
7. Register at a registrar an agent can drive (Cloudflare Registrar, Vercel or Porkbun, through their MCP servers, CLI or API), only after the user confirms the name, term, price and auto-renew. Any other registrar works by hand.
8. Build a redirect map from the old site's real URLs: relevant 301s, 410 for the rest, never a blanket homepage redirect.
9. Keep and monitor the redirects, and add the name to the user's watched domains.

It also covers looking up a single name (with a live refresh for missing sections), monitoring a portfolio of the user's own domains, and paid deep checks, which the agent prices first and runs only with the user's yes.

It is built around Google's expired domain abuse policy. The skill steers agents to legitimate uses (topically related domains, relevant redirects, rebuilding useful content) and tells them to refuse plans that exist only to pass link equity, build private blog networks or impersonate a previous owner. It makes no ranking promises.

Layout:

```
skills/revised/
  SKILL.md                    tool choice and workflows
  references/
    mcp-and-api.md            tools, REST routes, fields, plans, errors, limits
    vetting.md                candidate checks and reject rules
    registration.md           Cloudflare, Vercel, Porkbun steps and DNS
    redirects.md              CDX extraction, mapping, snippets
    policy.md                 expired domain abuse guardrail
```

## Requirements

A Revised account (https://getrevised.com). Every plan can look up names, estimate values, check spam history, search the directory and watch its own domains; the Free plan shows names only for open-tier listings. Paid plans show every name, search the whole tracked pool and hold listings (https://getrevised.com/pricing). Connecting takes one command; see https://getrevised.com/docs/agent-setup.

## License

MIT. See [LICENSE](LICENSE).
