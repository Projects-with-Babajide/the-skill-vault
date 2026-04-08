# The Skill Vault

Reusable Claude Code skills for engineering workflows, app exploration, LinkedIn automation, and productivity. These are invoked as slash commands (e.g. `/ship-chore`, `/linkedin-feed`) inside Claude Code.

Built and maintained by [Disposable by Default](https://disposablebydefault.ai).

Skills are symlinked from `~/.claude/skills/` into this repo — edits here are live immediately.

---

## LinkedIn

| Skill | What it does |
|---|---|
| `linkedin-feed` | Read and summarize the LinkedIn home feed |
| `linkedin-messages` | List, read, and send LinkedIn messages |
| `linkedin-post` | Create a post or comment on an existing one |
| `linkedin-search` | Search LinkedIn for posts or people on a topic |

> Requires [`link-pulse`](https://www.npmjs.com/package/link-pulse) installed globally.

## Engineering Workflow

| Skill | What it does |
|---|---|
| `review-ticket` | Validate a ticket against project specs before work begins |
| `implement-ticket` | Implement a reviewed ticket end-to-end |
| `ship-and-watch` | Commit, push, open a PR, and monitor CI until it passes |
| `ship-chore` | Ship an ad-hoc change without a pre-existing ticket |
| `test-and-fix` | Run tests, diagnose failures, and iterate until they pass |
| `ticket-from-report` | Turn a skill-generated report into Linear tickets |

> Engineering skills require `.claude/skill-config.md` in the project root.

## App Exploration

| Skill | What it does |
|---|---|
| `explore-app` | Crawl a web app via Playwright, extract design system, map features, produce ticket or user story plan |
| `detail-exploration` | Flesh out approved items from an `/explore-app` run into detailed tickets or user stories |

> Requires [Playwright MCP server](https://www.npmjs.com/package/@anthropic-ai/mcp-server-playwright). Optionally reads `.claude/skill-config.md` for project-specific settings.

## Productivity

| Skill | What it does |
|---|---|
| `reddit-read` | Read subreddits, threads, and search Reddit posts |

---

## Usage

1. Clone this repo
2. Symlink the skills you want into `~/.claude/skills/`
3. Open Claude Code and use them as slash commands

```bash
# Example: symlink the linkedin-search skill
ln -s /path/to/the-skill-vault/linkedin-search ~/.claude/skills/linkedin-search
```

## License

MIT — use these skills however you want. If you build something cool with them, [let us know](https://disposablebydefault.ai).
