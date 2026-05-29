---
name: handoff
description: Writes a structured markdown handoff file so a fresh-context agent can resume a long-running task — objective, progress, key decisions, the exact next step, and how to verify. Use near the context limit, at a milestone or session end, or when asked to create a handoff / save state / "핸드오프 만들어".
---

# handoff

Write the document your next agent opens before it works. A handoff compacts everything that
matters about a long-running task into one structured markdown file, so an agent with an empty
context window resumes exactly where you left off — without re-reading the conversation.

This skill **always writes a markdown file** as the durable cross-agent transport, and **also**
emits a paste-ready resume block you can drop into a fresh chat.

## When to use

Trigger on any of: context window ~80%+ full · a milestone or work session ending · after 5+ file
edits · the user asks for a handoff / state save / "next plan of action".

Manual trigger (user present, steering): just write the file.
Automatic/threshold trigger (context near full): also append the auto-continue note (step 7).

## Pipeline (summary)

**gather → structure → write → verify → resume**. Full detail in
[ref/pipeline.md](ref/pipeline.md). Do NOT continue the task; produce only the artifact.

0. **Preflight.** If essentially nothing happened, say "Nothing to hand off yet" and stop — never fabricate.
1. **Gather mechanically.** Derive read/modified sets from tool history; capture commands, test output, and error text verbatim; confirm git branch/HEAD/dirty via read-only probes. Never trust recall.
2. **Preserve the open thread.** Capture any unanswered user question VERBATIM (the most-failed item) → Open Questions, and Current State if it blocks the next step.
3. **Structure.** Fill `ref/templates/handoff.md` in order: Current State first; Done labeled "DO NOT REDO" + quantified; ONE next step; decisions with rationale; concrete file:line / commands / real errors.
4. **Self-verify.** Run `ref/checklists/quality.md`; fix any secret, placeholder, vagueness, missing verify command, or dropped open question before writing.
5. **Write.** Save `handoffs/handoff-<UTC>.md` in the working project (ISO 8601, `:`/`.` → `-`); chain and merge a prior handoff if one exists.
6. **Emit + report.** Print the path + a one-line summary, then the doc wrapped in `<handoff-context>…</handoff-context>` (exact wrapper in [ref/pipeline.md](ref/pipeline.md)) as the paste-ready resume block.
7. **(Auto trigger only)** Add the auto-continue note: honor the user's LATEST intent; say so if nothing remains.

## Reference files

- [ref/pipeline.md](ref/pipeline.md) — the full step-by-step pipeline and cold-resume failure modes
- [ref/templates/handoff.md](ref/templates/handoff.md) — the document template (general)
- [ref/templates/handoff-coding.md](ref/templates/handoff-coding.md) — feature-build variant
- [ref/templates/handoff-debugging.md](ref/templates/handoff-debugging.md) — bug/debug variant
- [ref/checklists/quality.md](ref/checklists/quality.md) — pre-write self-verification gate
- [ref/checklists/resume.md](ref/checklists/resume.md) — what the RESUMING agent does first
- [examples/](examples/) — filled handoffs you can imitate
- [scripts/check-staleness.sh](scripts/check-staleness.sh) — validate a handoff before trusting it

## Hard rules

- Capture exact technical state, not abstractions — paths, symbols, commands, real errors.
- Never include secrets, tokens, or keys.
- Output ONLY the handoff artifact and where it lives; do not answer the conversation's questions.
- The file is the source of truth; the `<handoff-context>` block is its transport.
