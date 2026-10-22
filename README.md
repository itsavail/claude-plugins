# Availnet Claude plugins

Plugin marketplace for Availnet.

| Plugin | Contents |
| --- | --- |
| `availnet-agency` | Remote MCP server `https://agency.availnet.ai/mcp` (OAuth sign-in with your Availnet account) and the `availnet-agency` skill |

## Install in Claude Code

```
/plugin marketplace add itsavail/claude-plugins
/plugin install availnet-agency@availnet
```

Run `/mcp` and sign in to `plugin:availnet-agency:availnet-agency` with your Availnet account.

On claude.ai, add **Availnet Agency** from the Connectors Directory instead.

## Updating the skill

`SKILL.md` is generated from `apps/agency/lib/mcp/skill.ts` in the Availnet monorepo. Regenerate it there and bump `version` in `plugin.json`.
