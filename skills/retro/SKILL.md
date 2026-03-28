---
name: retro
description: Use when a conversation's main task is complete and successful — triggers resource cleanup (stray processes, temp files) and a structured retrospective that saves learnings to memory
---

# Retro

Post-session cleanup and retrospective. Run after successfully completing your main task.

## Phase 1: Resource Cleanup

Clean up resources that were used during the session and are no longer needed.

**How to find what needs cleanup:**

1. **Review the conversation history** — identify every process you started, every server you spun up, every temp file you created. This is your primary source of truth.
2. **Verify what's still running** — for each process you started, check if it's still alive by PID, port, or command pattern. Use `ps aux | grep <pattern>` and `lsof -i :<port>` for any ports you used.
3. **Cast a wider net** — check for processes that may have been spawned indirectly (e.g., Chrome instances opened by Playwright, child processes forked by a dev server, file watchers started by a build tool). These won't always be obvious from the conversation history alone.

This could be dev servers (Node, Python, Go, Ruby, etc.), browser processes spawned for testing, background watchers, build processes, database containers — anything. Derive from what was actually done, not a hardcoded list.

**Temp files to check for:**
- Build artifacts or bundles generated in `/tmp` or the project directory during development
- Test fixtures, screenshots, or recordings downloaded or generated for verification
- Output files that were created to inspect results but serve no ongoing purpose
- Any files you explicitly created as scratch/temporary during the session

**Rules:**
- ALWAYS present a summary of what you found before killing or deleting anything. Format it clearly:
  - Processes: PID, full command, port (if applicable)
  - Files: full path, size
- Ask for confirmation before acting. If something looks ambiguous (e.g., a Chrome process that might be the user's personal browser), call it out specifically.
- Do NOT touch the user's editor, shell, music player, or anything unrelated to the session.
- When in doubt, leave it alone and flag it for the user to decide.

### Phase 2: Retrospective

Go through the full conversation from start to finish. Identify patterns, not just individual events.

**Present the retro in this format:**

```
## Retro

### Context
[What was the goal of this session? What problem were we trying to solve or what feature were we building? How did the approach evolve — did we pivot, hit dead ends, change strategy? Did we ultimately achieve the goal? Give enough narrative that someone reading this cold understands the arc of the session, not just the outcome.]

### What went well
- [Be specific: name the decision, approach, or tool that worked. "Used X approach for Y problem which avoided Z pitfall" is good. "Things went smoothly" is not.]

### What didn't go well
- [Be specific: what went wrong, why, and how much time/effort it cost. "Spent 20 minutes debugging X because we assumed Y, when the real issue was Z" is good. "Had some issues" is not.]

### Learnings
- [Actionable insights. Each learning should be something that would change behavior in a future session. "When working with X, always check Y first because Z" is good. "X can be tricky" is not.]
```

### Phase 3: Aggressive Memory Extraction

This is the most important phase. The retro is only useful if learnings actually get saved to memory. **Default to saving, not skipping.** When in doubt, save it — a slightly redundant memory is far less costly than a lost insight that leads to repeating a mistake.

**HARD RULE: Mine every one of these signal types from the conversation. If any occurred, they MUST produce at least one memory entry:**

| Signal | What to extract | Why it matters |
|--------|----------------|----------------|
| **User corrections** | Any time the user said "no", "don't", "stop", "not like that", or redirected your approach. Save the correction as a feedback memory with the anti-pattern and the correct behavior. | Highest-signal learning. The user explicitly told you something was wrong. Missing this means they'll have to correct you again. |
| **User confirmations of non-obvious choices** | Any time the user approved an unusual approach, said "yes exactly", "perfect", or accepted something without pushback that could have gone either way. | Positive reinforcement is invisible — if you only save corrections, you'll drift away from validated approaches. |
| **Pivots and dead ends** | Any time the approach changed mid-session — a strategy that didn't work, a tool that failed, a wrong assumption that had to be corrected. | Future sessions will try the same wrong approach unless warned. Save what failed and why. |
| **Surprising discoveries** | Anything about the codebase, tools, APIs, or environment that was non-obvious and took effort to find. | If it took you 10 minutes to discover, save the 10 seconds it'll take to read the memory. |
| **New project context** | New information about the project's goals, architecture decisions, team dynamics, deadlines, or constraints that aren't in the code. | Project context decays fastest — if you don't capture it now, it's gone. |
| **Tool/workflow insights** | A tool, command, or workflow pattern that was particularly effective or ineffective for this type of task. | Builds institutional knowledge about how to work efficiently. |

**Minimum extraction target: If the session involved any real work (not just a quick question), you should produce at least 3 memory entries.** If you're finding fewer than 3, you're not looking hard enough. Re-read the conversation and check each signal type above.

**Scope each learning to the right place:**

| Scope | Where to save | When to use this scope |
|-------|--------------|----------------------|
| **Project** | Project memory dir | Learning is specific to this codebase, its tools, its quirks, or its conventions. E.g., "This project's ESLint config silently ignores `src/generated/` — always lint those files manually" or "The staging API requires a VPN connection that isn't documented anywhere" |
| **Global** | User-level memory dir | Learning applies across projects and is about general workflow, tool usage, or development patterns. E.g., "When starting a dev server, always check if the port is already in use first — killed 15 minutes last time" or "Playwright's headed mode leaves zombie Chrome processes; always run cleanup after" |
| **Feedback** | Feedback memory | Learning is about how Claude should behave — corrections the user made, approaches the user confirmed as good, or interaction patterns to repeat/avoid. E.g., "User prefers seeing the failing test output before discussing fixes" or "Don't refactor surrounding code when fixing a bug — user had to ask me to stop twice" |

**What NOT to save (narrow list — when in doubt, save it):**
- Raw activity logs ("we did X then Y then Z") — save the *insight*, not the play-by-play
- Things literally already in existing memory files with the same content
- Standard practices any developer would know without being told

That's it. Everything else is fair game. Err on the side of saving.

**Why you can be aggressive:** Memory uses an index file (`MEMORY.md`) + separate detail files. Only the index loads into context — the detail files are only read when relevant. This means more memories do NOT bloat the context window. There is almost no cost to saving an extra learning, but there IS a cost to losing one. So save aggressively.

**Before saving — deduplicate against existing memory:**
1. Read `MEMORY.md` and scan existing memory files for overlap with each proposed learning.
2. If an existing memory already covers the same topic, **update or append to that file** instead of creating a new one.
3. Only create a new memory file if the learning is genuinely new territory.

**Then save. Present the full list of proposed entries to the user (content, type, filename, new vs. edit), then save them all immediately.** The user can tell you to revert any they disagree with — but the default is save, not ask-then-save. This prevents approval fatigue from killing good learnings.

**Memory quality rules:**
- Frame behavior-changing memories as hard rules with anti-pattern examples, not soft guidelines. "ALWAYS do X when Y" beats "Consider doing X."
- Include enough **Why** context that a future session can judge edge cases, not just blindly follow.
- Be specific. "The staging deploy script requires `--env=staging` even though the README says it auto-detects" beats "staging deploys can be tricky."

## Red Flags

| Thought | Reality |
|---------|---------|
| "Nothing to clean up" | Check anyway. Stray processes are silent resource drains. |
| "The retro is obvious" | Non-obvious patterns hide in plain sight. Review the full conversation. |
| "This mistake was too small to save" | Small repeated mistakes compound. Save it. |
| "This learning is too specific" | Specific learnings are the most actionable. Save it. |
| "I'll skip the positive learnings" | Reinforcing good patterns is as valuable as avoiding bad ones. |
| "Only 1-2 things to save" | You're not looking hard enough. Re-read the conversation against every signal type. |
| "The user might not want this saved" | Save first, revert if asked. Lost learnings are worse than extra memories. |
| "This is already kind of covered by an existing memory" | "Kind of" means it's NOT covered. Update the existing memory with the new nuance. |
