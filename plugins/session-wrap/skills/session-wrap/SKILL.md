---
name: session-wrap
description: Use when the user says "let's wrap this session up", "wrap up", "anything to save?", "save anything important from this session", "end of session", or similar. Reviews the conversation and saves anything worth remembering to the auto-memory system — new project context, corrections, infrastructure details, preferences, or in-progress work. DO NOT skip this skill when the user signals they're done for the day — it's the primary way knowledge is preserved across sessions.
---

# Session Wrap

Review the conversation and persist anything worth remembering to the auto-memory system. The goal is that the next session starts with full context — no re-explaining, no rediscovering.

## What to save

Work through each memory type and ask: did this session produce anything new?

**`project`** — New facts about ongoing work, decisions made, features built or specced, infra details learned, in-progress items not yet complete. Decay fast — include enough context that a future session can judge whether it's still relevant.

**`feedback`** — Corrections or preferences the user expressed. "Don't do X", "do Y instead", anything where the user had to redirect you. Save these aggressively — repeated corrections are frustrating.

**`user`** — New understanding of the user's role, expertise, or working style. Only if genuinely new — don't duplicate what's already there.

**`reference`** — Locations of things: URLs, hostnames, file paths to important configs, external systems. Especially credentials patterns, server addresses, service names.

## Memory Tier Routing

Most memories go in the flat top-level `memory/` directory — that's the default.

Use tiers when TTL matters:
- **`working/`** — things that only matter for 1-2 sessions. In-progress task state, temporary blockers, "I'm halfway through X." Archived after 48h.
- **`weekly/`** — rolling patterns. "This week we've been focused on X." "Recurring issue with Y." Archived after 14 days.
- **`monthly/`** — durable insights that aren't project memories. "After extensive testing, approach X works better than Y for this codebase." Never archived.

The `last-handoff.md` file always goes in `working/` (step 5).

## What NOT to save

- Things already in the memory files (check before writing)
- Code patterns derivable from reading the codebase
- Ephemeral task state that won't matter next session
- Anything in CLAUDE.md already

## Process

1. **Scan the conversation** for each memory type above. Note candidates.

2. **Read existing memory files** relevant to what you found — check for overlap before writing anything new. Also check for **staleness**: if the conversation contradicted or superseded an existing memory, update or remove it. Don't let outdated memories accumulate.

3. **Check for infra changes.** If any of these were modified during the session, save a project memory noting what changed and why:
   - Agent definitions (`~/.claude/agents/`)
   - Hooks (`~/.claude/hooks/`)
   - Skills (`~/.claude/skills/`)
   - Settings (`~/.claude/settings.json`)
   - CLAUDE.md files
   These changes affect all future sessions and are easy to forget.

4. **Write or update memory files** in the project's memory directory (`~/.claude/projects/<project>/memory/`). Use the frontmatter format:
   ```
   ---
   name: <name>
   description: <one-line — used to decide relevance in future sessions>
   type: <user|feedback|project|reference>
   ---

   <content>
   ```
   For `feedback` and `project` types, include a **Why:** line and a **How to apply:** line.

5. **Update `MEMORY.md`** index with any new files. Keep entries brief — just the filename and a one-liner.

6. **Check project docs.** If the session produced knowledge that belongs in project documentation (not memory), update those docs. Examples:
   - A design spec was discussed and decisions were made → update the spec
   - A strategy doc exists and the session changed the strategy → update the strategy doc
   - Infrastructure or architecture changed → update relevant docs under `docs/`
   - A README or runbook became stale due to this session's work → update it
   
   Look for `docs/` directories, `strategy.md`, `*.spec.md`, and similar files in the project root. Only update docs that already exist and are clearly stale — don't create new docs unless the session produced a substantial deliverable with no home.

7. **Write `last-handoff.md`** in the project's memory directory. This file is read by the session-start-daily hook to inject context into the next session. Write it to `working/last-handoff.md` (it's ephemeral — gets archived after 48h). Structure:
   1. **Notable progress** — what was accomplished this session (bullet points)
   2. **Blockers, issues, open questions** — anything unresolved or stuck
   3. **Next steps** — up to 5 concrete next actions, in priority order
   Keep it under 20 lines. No frontmatter needed — just plain text.

8. **Commit and push any updated docs** — if the current working directory is a git repo, stage and commit any modified or new files under `docs/` (specs, plans, etc.) that were produced or updated this session (including step 6), then push to origin. Use a commit message like `docs: session wrap <YYYY-MM-DD>`. Skip if no docs changed.

9. **Report back** with a compact summary: what was saved and why, what docs were updated, and whether a docs commit was pushed. If nothing new was worth saving, say so. Don't ask for permission before writing — just do it and report what you did.

## Memory file location

The memory directory is always a sibling to `MEMORY.md`. Look for it at `~/.claude/projects/<hash>/memory/` — the exact path is visible in your system context.

## Tone

Be selective. One well-written memory is worth ten vague ones. If you're unsure whether something is worth saving, ask yourself: would the next session genuinely benefit from knowing this, or would it just add noise?

<!-- created_from: 5bf8bf7 -->
