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

**After presenting the retro, save learnings to memory.**

Use judgment to scope each learning to the right place:

| Scope | Where to save | When to use this scope |
|-------|--------------|----------------------|
| **Project** | Project memory dir | Learning is specific to this codebase, its tools, its quirks, or its conventions. E.g., "This project's ESLint config silently ignores `src/generated/` — always lint those files manually" or "The staging API requires a VPN connection that isn't documented anywhere" |
| **Global** | User-level memory dir | Learning applies across projects and is about general workflow, tool usage, or development patterns. E.g., "When starting a dev server, always check if the port is already in use first — killed 15 minutes last time" or "Playwright's headed mode leaves zombie Chrome processes; always run cleanup after" |
| **Feedback** | Feedback memory | Learning is about how Claude should behave — corrections the user made, approaches the user confirmed as good, or interaction patterns to repeat/avoid. E.g., "User prefers seeing the failing test output before discussing fixes" or "Don't refactor surrounding code when fixing a bug — user had to ask me to stop twice" |

**What makes a good learning to save:**
- Mistakes that burned real time or effort — the kind you'd kick yourself for repeating. Include enough context that a future session can recognize the same situation forming.
- Approaches that worked surprisingly well or were non-obvious — things a future session wouldn't naturally try without being told. "Used browser devtools network tab to debug the API issue instead of adding logging" is worth saving if it was a breakthrough.
- Non-obvious discoveries about the codebase, tools, or environment that aren't documented anywhere and would take time to rediscover. "The `build` script silently swallows errors when run with `--quiet` flag" saves future debugging time.
- Corrections the user made — these are the highest-signal learnings because the user explicitly told you something was wrong. Always save these.

**What NOT to save:**
- Anything already captured in code, comments, git history, or existing documentation. If someone could find it by reading the codebase or running `git log`, it doesn't need to be in memory.
- Standard practices or common knowledge. "Write tests before committing" or "Read error messages carefully" aren't learnings — they're basics.
- Details that only matter for this exact task and won't recur. "We added a `getUserById` function to `api.ts`" is ephemeral — it's in the code now.
- Raw activity logs. Don't save "we did X then Y then Z." Save the *insight* that came from doing X, Y, Z.

**Before saving — deduplicate against existing memory:**
1. Read `MEMORY.md` and scan existing memory files for overlap with each proposed learning.
2. If an existing memory already covers the same topic, **update or append to that file** instead of creating a new one. For example, if there's already a `feedback_skill_writing.md` and you have a new insight about skill writing, add to it rather than creating `feedback_skill_writing_2.md`.
3. Only create a new memory file if the learning is genuinely new territory not covered by any existing entry.
4. Show the user each proposed entry — the content, the type (project/global/feedback), whether it's a **new file** or an **edit to an existing file**, and the filename. Let them approve, edit, or skip each one. Don't batch-save without review.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Nothing to clean up" | Check anyway. Stray processes are silent resource drains. |
| "The retro is obvious" | Non-obvious patterns hide in plain sight. Review the full conversation. |
| "This mistake was too small to save" | Small repeated mistakes compound. Save it. |
| "This learning is too specific" | Specific learnings are the most actionable. Save it. |
| "I'll skip the positive learnings" | Reinforcing good patterns is as valuable as avoiding bad ones. |
