# Changelog — 2026-05-30

Initial build of the `handoff` skill and the `cskwork/agent-handoff` public repo.

## What

Created a Claude Code skill that, for long-running tasks, compacts session state into a structured
markdown file (`handoffs/handoff-<UTC>.md`) so a fresh-context agent can resume. Built from a deep
research pass over [oh-my-pi](https://github.com/can1357/oh-my-pi)'s handoff/compaction internals
plus cold-resume best practices.

## Key decisions and why

- **Always write a markdown file** (vs. oh-my-pi's manual `/handoff`, which re-injects context into
  a forked in-process session without a file). A skill has no session-forking primitive, so the file
  is the only durable cross-agent transport. We *also* emit oh-my-pi's `<handoff-context>` wrapper so
  the doc can be pasted into a fresh chat.
- **Reuse oh-my-pi's load-bearing primitives verbatim**, verified against source:
  - `<handoff-context>…</handoff-context>` + "The above is a handoff document… continue the work
    seamlessly." (`agent-session.ts:571`).
  - filename = `handoff-${new Date().toISOString().replace(/[:.]/g,"-")}.md` (`agent-session.ts:575`).
  - mechanically-derived `<read-files>`/`<modified-files>` from tool history, not recall
    (`compaction/prompts/file-operations.md`).
  - "Output ONLY the handoff document, no preamble" discipline (`handoff-document.md`).
  - auto-continue "honor latest intent" note on automatic triggers only (`system/auto-continue.md`).
- **Template extends oh-my-pi's section set** (Goal / Constraints / Progress / Key Decisions /
  Critical Context / Next Steps) with resume-hardening sections: Current State first, Metadata
  (git anchor for staleness), Immediate Next Step (exactly one), Gotchas, Verification, Open
  Questions (verbatim), Handoff Chain.
- **Progressive disclosure**: `SKILL.md` stays lean (two-field frontmatter, pipeline summary);
  full pipeline, templates, checklists live under `ref/` and are read on demand. Per Anthropic
  skill-authoring guidance.
- **Skill name kept as `handoff`** per the explicit request (gerund form `creating-handoffs` would
  be the convention; noted but not used).
- **Omitted `scripts/refresh.sh`** from the proposed structure — there is no upstream rule to vendor,
  so a no-op script would be dead code. Kept the substantive `scripts/check-staleness.sh` only.
- **Landing page** is a single self-contained `docs/index.html` (inline CSS/JS, EN/한국어 toggle,
  reveal-on-scroll), served via GitHub Pages from `/docs` on `main` — mirroring the
  `cskwork/ten-rules-skill` convention.

## Update — nested skill folder + leaner description

- **Moved the skill into a `handoff/` subfolder** (SKILL.md + ref/ + examples/ + scripts/). Repo
  stays `agent-handoff` (descriptive), but the skill directory name now equals the skill name
  `handoff` — so the install symlink target's basename matches without renaming the repo. `docs/`,
  `README.md`, `LICENSE`, `log/` remain at repo root (Pages still serves `/docs`).
- Repointed global links to the nested folder: `~/.claude/skills/handoff` and
  `~/.codex/skills/handoff` → `agent-handoff/handoff`.
- **Shortened the SKILL.md `description`** from ~1.1k to ~480 chars: kept what + when + the core
  English/Korean trigger phrases for discovery, dropped the exhaustive synonym list.
- Updated README install/layout and `docs/index.html` install block + examples link to the
  `handoff/` paths.

## Research artifact

Synthesis spec produced by the `handoff-skill-research` workflow (oh-my-pi clone at
`/tmp/oh-my-pi-research`); 4 agents, findings cross-checked against source files before building.
