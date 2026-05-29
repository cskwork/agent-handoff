---
name: handoff
description: Compacts a long-running session into a structured, paste-ready handoff document and writes it to a markdown file so a fresh-context agent can resume the task with zero ambiguity — capturing the objective, current state, what is done (with evidence), what is left, key decisions and their rationale, files touched, gotchas, the single exact next action, and how to verify. Use when the context window is filling up (~80%+), a milestone or work session is ending, after 5+ file edits, or when the user says "create a handoff", "save state", "write a handoff doc", "summarize what we did / where we left off", "next plan of action", "hand off to a new agent", "resume from", "continue where we left off" — or in Korean "핸드오프 만들어", "지금까지 한 거 정리해", "다음 작업 계획 정리", "새 에이전트한테 넘겨", "이어서 작업하게 정리해줘", "상태 저장해".
---

# handoff

Write the document your next agent opens before it works. A handoff compacts everything that
matters about a long-running task into one structured markdown file, so an agent with an empty
context window resumes exactly where you left off — without re-reading the conversation.

Inspired by [oh-my-pi](https://github.com/can1357/oh-my-pi)'s `/handoff` pipeline. The key
adaptation: oh-my-pi can fork an in-process session, so its manual `/handoff` re-injects context
without a file. A skill has no session-forking primitive, so this skill **always writes a markdown
file** as the durable cross-agent transport, and **also** emits a paste-ready resume block.

## When to use

Trigger on any of: context window ~80%+ full · a milestone or work session ending · after 5+ file
edits · the user asks for a handoff / state save / "next plan of action" (see description for the
full English + Korean trigger phrases).

Manual trigger (user present, steering): just write the file.
Automatic/threshold trigger (context near full): also append the auto-continue note (step 7).

## Pipeline (summary)

Mirror oh-my-pi: **gather → structure → write → verify → resume**. Full detail in
[ref/pipeline.md](ref/pipeline.md). Do NOT continue the task; produce only the artifact.

0. **Preflight guard.** If essentially nothing has happened (trivial conversation, <2 real
   exchanges), say "Nothing to hand off yet" and stop. Never fabricate progress.
1. **Gather state mechanically, not from memory.** Derive the read-set and modified-set from your
   own tool history; collect commands run, test/build/lint output and error text verbatim. Confirm
   live state with read-only probes: `git rev-parse --abbrev-ref HEAD`, `git rev-parse --short HEAD`,
   `git status --porcelain`. Never trust recall for paths, symbol names, or numbers.
2. **Preserve the open thread.** Capture any unanswered user question or pending request VERBATIM —
   this is the most resumption-critical and most-failed item. Put it in Open Questions (and in
   Current State if it blocks the next step).
3. **Structure into the fixed template.** Fill [ref/templates/handoff.md](ref/templates/handoff.md)
   in order. Lead with Current State. Label finished work "DO NOT REDO" with quantified progress
   (e.g. "30/90 tests passing"). Reduce next steps to ONE exact first action. Pair every decision
   with its rationale. Use real file:line, symbol names, exact commands, real error text.
4. **Self-verify.** Run [ref/checklists/quality.md](ref/checklists/quality.md). Reject the draft if
   it leaks secrets, has unfilled `{{...}}`/`[TODO]`, is vague instead of concrete, lacks
   copy-paste verification commands, or drops a preserved open question. Fix, then continue.
5. **Write the file.** Save to `handoffs/handoff-<UTC-timestamp>.md` in the working project (ISO
   8601 with `:` and `.` replaced by `-`, e.g. `handoffs/handoff-2026-05-30T12-00-00-000Z.md`).
   Create `handoffs/` if missing. If a prior handoff exists, fill the Handoff Chain link and merge:
   keep all prior info, promote In Progress → Done, refresh Next Steps and stale questions.
6. **Emit the resume primitive + report.** Print the absolute saved path and a one-line summary,
   then the ready-to-paste block — the document wrapped in `<handoff-context>` … `</handoff-context>`
   followed by: `The above is a handoff document from a previous session. Use this context to
   continue the work seamlessly.`
7. **(Automatic trigger only)** Append the auto-continue note: the resuming agent must honor the
   user's LATEST intent over older recorded plans, and say so briefly if nothing remains rather
   than inventing busywork.

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
