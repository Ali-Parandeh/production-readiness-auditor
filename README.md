# Production Readiness Auditor

A Claude Code skill that runs a structured production-readiness audit on a FastAPI + LLM/Generative AI codebase. It is grounded in *[Building Generative AI Services with FastAPI* (Ali Parandeh, O'Reilly, 2025)](https://buildinggenai.com).

![https://buildinggenai.com](images\book.webp)

The skill runs **inside your repository** from a coding agent. It walks the code itself, grades seven areas one at a time as Pass / Partial / Fail with book-grounded feedback, and only asks you questions the code cannot answer. At the end it writes a timestamped audit report to the repo root.

## What it checks

Seven areas, each mapped to chapters of the book:

| Area                                           | Chapter  |
|------------------------------------------------|----------|
| 1. Modular project architecture (Onion design) | 2        |
| 2. FastAPI routing and API design              | 2, 4     |
| 3. LLM integration and real-time streaming     | 3, 5, 6  |
| 4. Async concurrency for GenAI                 | 5        |
| 5. LLM security and optimisation               | 8, 9, 10 |
| 6. Tests and behavioural testing               | 11       |
| 7. Docker and container security               | 9, 12    |

Feedback is grounded first in the reference files under `references/`, optionally enriched by the O'Reilly MCP (see below), and falls back to built-in checks where neither covers a point.

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) installed.
- A FastAPI + LLM/GenAI repository to audit.
- Optional: access to the O'Reilly MCP for live book-grounded recommendations. Without it, the skill uses `references/oreilly-fallback.md`.

## Installation

The skill is a directory containing `SKILL.md` and its `references/`. Install it by placing that directory under a Claude Code skills path.

**Personal (available in every project):**

```bash
git clone https://github.com/aliparandeh/production-readiness-auditor.git ~/.claude/skills/production-readiness-auditor
```

**Project-scoped (checked in with a specific repo):**

```bash
git clone https://github.com/aliparandeh/production-readiness-auditor.git .claude/skills/production-readiness-auditor
```

On Windows PowerShell, swap `~/.claude` for `$env:USERPROFILE\.claude`.

The skill directory must be named so it contains `SKILL.md` at its top level. Restart Claude Code (or start a new session) so it picks the skill up.

## Usage

From inside the repo you want to audit, ask Claude Code to run it, for example:

- "Is my codebase production-ready?"
- "Run a FastAPI production audit"
- "Review this GenAI service for deployment"

Or invoke it directly:

```
/production-readiness-auditor
```

It works through the seven areas in order, reports each before moving on, then gives a final overview with a summary table, the top three urgent fixes, and consolidated reading. A report file named `PRODUCTION_READINESS_AUDIT-{YYYY-MM-DD-HHMM}.md` is written to the repo root when it finishes.

## Setting up the O'Reilly MCP (optional)

Connecting the O'Reilly MCP lets the skill pull current editions, links, and (where your account supports it) book passages, instead of the static fallback list. Without it everything still works, just with the bundled recommendations.

The MCP is an HTTP server:

- **Endpoint:** `https://api.oreilly.com/api/content-discovery/v1/mcp/`
- **Docs:** https://learning.oreilly.com/apidocs/mcp/content/

Add it to Claude Code:

```bash
claude mcp add oreilly --transport http https://api.oreilly.com/api/content-discovery/v1/mcp/
```

Access requires an O'Reilly learning account/subscription, and the endpoint expects authentication. Check the docs above for the current auth method (for example, an `Authorization` header) and pass it when registering the server, for example:

```bash
claude mcp add oreilly --transport http https://api.oreilly.com/api/content-discovery/v1/mcp/ --header "Authorization: Bearer YOUR_TOKEN"
```

Keep the token out of version control. Do not paste it into `SKILL.md` or any tracked file. Once connected, verify with:

```bash
claude mcp list
```

The skill auto-detects the MCP. If tools like `search_oreilly_content` or `ask_oreilly_experts` are present it uses them; otherwise it degrades cleanly to `references/oreilly-fallback.md`.

## Repository layout

```
SKILL.md                     the skill definition and the seven-area audit loop
references/
  ch02-architecture.md       grounding for areas 1 and 2
  ch05-async.md              grounding for areas 3 and 4
  ch06-streaming.md          grounding for area 3
  ch08-auth.md               grounding for area 5
  ch09-security.md           grounding for area 5
  ch10-optimisation.md       grounding for area 5
  ch11-testing.md            grounding for area 6
  ch12-deployment.md         grounding for area 7
  oreilly-fallback.md        recommendations used when the MCP is not connected
```

## Licence

MIT. See [LICENSE](LICENSE).
