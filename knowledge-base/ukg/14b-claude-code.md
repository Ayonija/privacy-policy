# 14b — Claude Code as a Force-Multiplier (skills, subagents, worktrees, hooks, MCP)

> Short sheet. Purpose: when the panel asks "how did you drive 2× commit velocity?" or "how do engineers actually use AI day-to-day?", you can speak concretely about agentic coding tooling — not vaguely about "AI helps." This *is* your velocity anchor's mechanism.

---

## What Claude Code is
**Plain English:** Claude Code is an agentic command-line coding tool — an LLM that lives in your terminal/IDE, reads and edits your repo, runs commands, and works in a loop toward a task. It's the concrete tool behind "AI-assisted development workflows."

---

## The features to name (each with a one-line definition + why it matters)

**Skills** — reusable, named capabilities/instructions you define once (a markdown file with steps + allowed tools) and invoke with `/name`. *Why it matters:* you encode a team's "how we do X" (scaffold a service, run the release) into a repeatable command — turning tribal knowledge into tooling. **This is exactly your "templates teams adopted by default" story.**

**Subagents** — spawning a separate agent with its own fresh context to handle a sub-task (research, a broad search, an isolated refactor), returning just the result. *Why it matters:* keeps the main context clean and lets you parallelize/decompose — orchestrator-worker pattern (ties to 14.4 agentic design). Each spawn starts cold, so you use them for genuinely separable work, not everything.

**Worktrees** — running an agent in its own **git worktree**: an isolated working copy of the repo on a separate branch, so the agent's changes don't collide with your main checkout. *Why it matters:* safe parallel/experimental work; you can run a risky change in isolation and discard if unchanged. (Git worktree = multiple working directories backed by one repo, each on its own branch.)

**Hooks** — shell commands the harness runs automatically on events (e.g., after every file edit run the linter; before commit run tests). *Why it matters:* enforces standards deterministically — the harness executes them, not the model, so "always format on save" actually happens. This is how you make quality *default* (your "raised engineering standards by default" line).

**MCP servers in Claude Code** — connect external tools/data (your enterprise APIs, Jira, a DB) via MCP so the agent can use them. *Why it matters:* this is literally what your MCP server plugged into — assistants reaching internal systems safely.

**Slash commands / CLAUDE.md** — `CLAUDE.md` is a per-repo memory/instructions file loaded into context; custom slash commands codify common workflows. *Why it matters:* onboard a repo's conventions to the agent automatically — your "15-minute onboarding" mechanism.

---

🎤 **Say it like this (the velocity story, mechanized):** *"The 2× velocity wasn't 'people typed less' — it was encoding our team's workflows into tooling so the good path was the default path. Concretely: Skills turned our repeated procedures — scaffold a service, wire CI, run a release — into one-command operations, so tribal knowledge became reusable. Hooks enforced standards deterministically: lint and tests ran automatically on edits, so quality wasn't a code-review afterthought. A CLAUDE.md per repo taught the assistant our conventions, which is what cut onboarding. And our MCP server let the assistant safely reach internal systems. For bigger tasks I'd decompose with subagents and run risky changes in isolated git worktrees. The principle is the same one behind my CLI platform: make the right thing the easy thing, and adoption happens without a mandate."*

**Follow-up ladder:**
- *Q: How do you stop AI-assisted dev from lowering quality?* Hooks (lint/test/SAST run automatically), require review, evals for any AI feature, and treat generated code as a draft a human owns — not auto-merge.
- *Q: Subagent vs just one long session?* Subagents give a fresh isolated context for separable work (research, broad search) and keep the main thread focused; overusing them re-derives context and costs more — use for genuinely independent sub-tasks.
- *Q: How is this different from autocomplete (Copilot)?* Autocomplete suggests the next lines in your editor; an agentic tool plans, edits multiple files, runs commands, and iterates against test results in a loop. Different altitude: line-completion vs task-completion.

⚠️ **Pitfalls:** Don't claim AI wrote everything — claim you *engineered the workflows and guardrails* so a team could safely move faster. The seniority signal is **leverage + governance**, not raw speed.

---

*Next topic, or drill deeper on this one?*
