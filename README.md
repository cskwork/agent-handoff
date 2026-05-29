# handoff

**The document your next agent opens before it works.**

Live page → **https://cskwork.github.io/agent-handoff/**

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill for long-running tasks. When a
session runs long, `handoff` compacts everything that matters into one structured markdown file —
objective, current state, what's done (with evidence), what's left, key decisions and why, files
touched, gotchas, the single exact next action, and how to verify — so a **fresh-context agent
resumes with zero ambiguity**.

Pipeline and document structure are inspired by [oh-my-pi](https://github.com/can1357/oh-my-pi)'s
`/handoff`. See [Credit](#credit).

## What it does

- **Captures exact state, not vibes** — real file paths, commands, test results, error text; the
  read/modified file sets are derived mechanically from tool history, never from recall.
- **Defends against cold-resume failures** — an explicit DO-NOT-REDO done list, a single exact next
  step, and any pending user question preserved verbatim.
- **Paste-ready and chainable** — writes `handoffs/handoff-<timestamp>.md` and emits a
  `<handoff-context>` block to paste into a fresh chat; each handoff links to its predecessor.

## How it works (pipeline)

`gather → structure → write → verify → resume`

1. **Gather** — scan tool history, derive read/modified sets, run read-only `git`/test probes.
2. **Structure** — fill the fixed template; Current State first, Done labeled DO NOT REDO.
3. **Write** — save `handoffs/handoff-<UTC>.md`, fill the Handoff Chain link.
4. **Verify** — run the quality checklist (no secrets, no placeholders, copy-paste verify commands,
   open question preserved).
5. **Resume** — print the path + the `<handoff-context>` resume block.

Full detail: [`ref/pipeline.md`](ref/pipeline.md).

## Install

```bash
git clone https://github.com/cskwork/agent-handoff.git
ln -s "$PWD/agent-handoff" ~/.claude/skills/handoff
```

Then in Claude Code, the skill auto-triggers near ~80% context, on milestones, or after 5+ edits —
or invoke it explicitly:

> create a handoff · 핸드오프 만들어 · summarize where we left off · next plan of action

Output lands in `handoffs/` in your working project. Other agents (Codex, Gemini) can use it too —
point them at `SKILL.md`.

## Layout

```
SKILL.md                      lean skill entry (frontmatter + pipeline summary)
ref/
  pipeline.md                 full pipeline + cold-resume failure-mode table
  templates/handoff*.md       general / coding / debugging templates
  checklists/quality.md       authoring self-verification gate
  checklists/resume.md        what the receiving agent does first
scripts/check-staleness.sh    validate a handoff before trusting it
examples/                     filled handoffs (coding + debugging)
docs/index.html               GitHub Pages landing page
```

## Credit

Pipeline and document structure inspired by [oh-my-pi](https://github.com/can1357/oh-my-pi)'s
`/handoff` (Goal / Constraints / Progress / Key Decisions / Critical Context / Next Steps), its
`createHandoffFileName` ISO timestamp naming, its mechanically-derived `<read-files>`/
`<modified-files>` tags, and its `<handoff-context>` re-injection wrapper. This skill adapts that
design to a file-writing skill: because a skill cannot fork a session, it **always writes the
markdown file** as the durable cross-agent transport.

## License

MIT — see [LICENSE](LICENSE).
