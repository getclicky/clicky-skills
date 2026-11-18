# clicky-skills

[Agent Skills](https://agentskills.io) for working with [Clicky](https://clicky.com) web analytics.
Each skill is a folder with a `SKILL.md`: instructions an AI agent (Claude, Claude Code, Cursor,
Zed, ...) loads when the task matches. Read-only: nothing here can change your Clicky account.

## Skills

| Skill | What it does |
|---|---|
| [clicky-analytics](clicky-analytics/) | Read your traffic: visitors, pages, referrers, goals, ecommerce, uptime. Report types, dates, filtering, segmentation, etiquette. |
| [clicky-install-tracking](clicky-install-tracking/) | Install, fix, or verify the Clicky tracking code: the one-line script, tag managers, CSP, SPA setups, ad blockers, API verification. |

## Install

**Claude Code** — drop a folder into your skills directory:

```sh
git clone https://github.com/getclicky/clicky-skills && cp -r clicky-skills/clicky-analytics ~/.claude/skills/
```

**Zed / agents using the shared convention** — into `.agents/skills/` (project) or `~/.agents/skills/` (global):

```sh
git clone https://github.com/getclicky/clicky-skills && cp -r clicky-skills/* .agents/skills/
```

**Via the skills CLI** ([npx skills](https://skills.sh)):

```sh
npx skills add getclicky/clicky-skills
```

**No install path?** The files are plain markdown — point your agent at
`https://clicky.com/skills/<name>/SKILL.md` and tell it to follow them.

## Using them

The analytics skill assumes either the [Clicky MCP server](https://clicky.com/mcp) is connected
(bearer token = your [userkey](https://clicky.com/user/edit)) or your agent can make plain HTTP
calls to the [API](https://clicky.com/help/api). Install tracking works over the API alone.

Discovery: `https://clicky.com/.well-known/agent-skills/index.json` (machine-readable index), or
`skills/list` over MCP.

## License

MIT (see LICENSE).
