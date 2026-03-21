# retro — Session Cleanup & Retrospective for Claude Code

A Claude Code plugin that runs a post-session retrospective: cleans up stray processes and temp files, then reviews what went well, what didn't, and saves actionable learnings to memory.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) v1.0.33 or later

## Installation

**From the official marketplace:**
```
/plugin install retro
```

**From the self-hosted marketplace:**
```
/plugin marketplace add rlraymondluo/retro-skill-for-claude-code
/plugin install retro@rlraymondluo
```

Restart Claude Code after installing.

## Usage

After completing your main task in a session, run:

```
/retro:retro
```

The skill will ask for confirmation before killing any processes or deleting any files.

## What it does

### Phase 1: Resource Cleanup
Reviews your conversation history to find processes and temp files that were created during the session and are no longer needed — dev servers, browser instances, build artifacts, test fixtures, etc. Shows you everything it found and asks for confirmation before cleaning up.

### Phase 2: Retrospective
Walks through the full conversation and produces a structured retro:

- **Context** — what you were trying to solve and how the approach evolved
- **What went well** — specific decisions, tools, or approaches that worked
- **What didn't go well** — mistakes, dead ends, and their cost
- **Learnings** — actionable insights saved to Claude Code's memory system

Learnings are scoped appropriately (project-specific vs global vs feedback) and deduplicated against existing memories before saving. You review and approve each one.

## Example output

```
## Phase 1: Resource Cleanup

Found 2 stray processes:
- PID 45231: node ./node_modules/.bin/next dev (port 3000)
- PID 45298: /Applications/Google Chrome.app --headless (spawned by Playwright)

No temp files found.

Kill these processes? [y/n]

---

## Retro

### Context
We were adding OAuth2 login to the dashboard app. Started with the Google
provider, hit a redirect URI mismatch that cost ~15 minutes to debug (turned
out the dev server was on port 3001, not 3000). Pivoted to using the OAuth
library's built-in dev mode. Successfully got login working end-to-end.

### What went well
- Using the OAuth library's dev mode shortcut saved significant time vs
  manually configuring redirect URIs for local development
- Caught the CSRF token issue early by testing the full flow before writing
  more code

### What didn't go well
- Spent 15 minutes on the redirect URI mismatch because we assumed the dev
  server was on port 3000 without checking — a quick `lsof -i :3000` would
  have caught it immediately

### Learnings
- When debugging OAuth redirect issues, always verify the actual port the
  dev server is running on first — don't assume the default

Proposed memory entries:
1. [feedback] [EDIT feedback_dev_workflow.md] Append: "When starting OAuth
   or redirect-based flows, verify the dev server's actual port before
   configuring callback URLs"
   → Approve / Edit / Skip?
```

## Scope & Safety

- **Process cleanup**: Only targets processes that were started during the current Claude Code session (identified from conversation history). Your personal apps, editors, and background services are never touched.
- **File cleanup**: Only flags temp files that were explicitly created during the session (build artifacts, test fixtures, scratch files). Does not scan or delete files outside the working directory.
- **Confirmation required**: The skill always presents what it found and waits for your approval before killing processes or deleting files. Ambiguous items are flagged for you to decide.
- **Memory saving**: Proposed learnings are shown to you for review before saving. Each entry can be approved, edited, or skipped.

## License

MIT
