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

## Skills

### `revised`

An end-to-end workflow for an agent helping someone use an expired domain properly:

1. Check the account and plan (`whoami`).
2. Connect the Revised MCP server (or REST API).
3. Read the user's own site to build a brief: topic, audience, language, geography.
4. Search and shortlist by category, TLD and metric floors.
5. Vet each candidate against the user's site: archive history, linking sites, red flags, trademark risk.
6. Reveal and hold, respecting plan and republication rules (only `open`-tier names may be repeated publicly).
7. Hand off registration to the human at a registrar of their choice.
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
    registration.md           registrar handoff and DNS
    redirects.md              CDX extraction, mapping, snippets
    policy.md                 expired domain abuse guardrail
```

## Requirements

A Revised account (https://getrevised.com). The Free plan can search and read open-tier names; Starter ($29/month) shows every name. Connecting takes one command; see https://getrevised.com/docs/agent-setup.

## License

MIT. See [LICENSE](LICENSE).
