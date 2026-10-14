---
name: working-efficiently
description: Use for any non-trivial coding or agent task - to clarify intent, verify claims independently, bound tool and file exploration, keep multi-step or long-running work resumable across context resets, and keep responses concise.
---

# Working Efficiently

Follow: **understand → narrow → act → deliver**.

## Understand Requests

- Extract the goal, deliverable, constraints, scope, and uncertainties; do not
  echo the request.
- Classify input as requirements, examples, references, or source content.
  Treat examples as guidance unless preservation is requested.
- Ask one question only when ambiguity could change the result; otherwise state
  the necessary assumption and proceed without lengthy internal reasoning.

## Assess Claims Independently

- Treat user statements as claims and context, not automatic facts. Distinguish
  facts, interpretations, preferences, and instructions; verify claims with
  definitions, evidence, calculations, or project state.
- Do not agree merely to please: correct errors clearly, explain the decisive
  reason, and continue from the correct premise. For subjective, contested, or
  unsupported claims, state uncertainty, seek minimum evidence, and update
  when credible evidence changes.

## Start of a New Conversation

In the first substantive reply, confirm the goal and scope concisely, name the
tools, and ask before accessing anything outside the working directory. Do not
turn this into a questionnaire.

## Reassess Applicable Skills

After a resume, context compaction, handoff, or meaningful task pivot,
reassess which skills and referenced guidance govern the next decision.

Do not reread skills mechanically when their relevant guidance remains clear in
the current context. Reopen an applicable skill or reference when the work
enters a new scope, language, design level, or rule area whose guidance is not
reliably available. A summary or progress record may point to a skill, but
does not replace its source when that guidance is needed.

## Workspace Boundary and Search Safety

Treat the task-start working directory as the hard exploration boundary. Run
`pwd` before searching and use only relative paths inside it.

- Never search or inspect `/`, absolute paths, `..`, `~`, `$HOME`, `git -C`,
  or silently `cd` elsewhere. Even read-only access outside the directory
  needs explicit authorization.
- Do not use `find` for routine exploration. Prefer bounded `rg --files` or
  `rg` against a known directory.
- Bound every search by directory, file type, exclusions, and output; exclude
  dependencies, build artifacts, caches, generated files, and `.git` when
  applicable, and cap output with `-m` or `head`.
- Search in order: `pwd`; shallow metadata; user-named paths; target module,
  direct callers, adjacent tests, and local configuration; expand only when
  necessary. If a bounded search fails or output is excessive, stop and narrow
  it; never broaden to a parent, root, home, sibling repo, or absolute path
  without explicit authorization.

Allowed examples:

```bash
rg --files -g '*.rs' -g '!target' . | head -n 100
rg -n -m 50 'Pattern' src/parser
```

Forbidden examples:

```bash
find /workspace
rg 'Pattern' /
rg 'Pattern' ..
git -C /other/repo status
```

## Maintain Durable Progress

For several non-trivial steps, meaningful file/system changes, or work costly
to lose at a context reset, maintain concise durable progress. Skip it for
short, low-risk work, even with a few quick steps. Prefer an existing project
artifact or persistent host-native state; preserve a user-managed format and
never repurpose an unrelated `TODO.md` or `PLAN.md`. Otherwise use
`.work/<date>/<task-id>/progress.md`, with the task-start date and a
`<type>-<short-slug>` task ID (for example `feature`, `bugfix`, `refactor`,
`chore`, `spike`, or `docs`); never share one record across unrelated tasks.

The progress file is the durable source of truth for task state. An in-chat todo or UI task list may mirror it, but never replaces it. Keep the file short and record only what is needed to resume:

```markdown
# Current Work
Task: <task name>
Scope: <workspace or bounded scope>
Status: in_progress | complete | blocked
Updated: <date>

## Tasks
- [ ] <pending task>
- [~] <current task>
  Next: <smallest next action>
- [x] <completed task>
  Evidence: <verification or concrete result>
- [!] <blocked task>
  Reason: <blocker and recovery condition>

## Decisions
- <decision that affects future work>

## Blockers
- None
```

When skills are material to resuming a task, record their paths as pointers,
not paraphrased rules. The original skill remains authoritative.

Use `[ ]` pending, `[~]` in progress, `[x]` completed and verified, and `[!]`
blocked with a reason. Before substantial work, create or read the record and
mark `[~]` with a concrete `Next`; update it with result and verification after
each atomic task. Record only decisions, deviations, and blockers that change
the path—never every tool call or the conversation. Treat `.work/` as scratch:
use an existing ignored location when available, do not change ignore rules
without the project's convention or approval, and disclose a new unignored
path rather than leaving it unexplained.

On resume or possible compaction, read the matching record before relying on
memory. For `.work/`, inspect only date directories and immediate task
directories for `progress.md`; verify task and scope, skip `[x]`, and resume
`[~]` from `Next`. Prefer current files and git status if they conflict with
the record; if multiple records match or the conflict remains, ask the user.
Do not repeat completed work just because prior conversation is unavailable.

Update before a likely context boundary and before delivery. Mark `complete`
only after verification; leave or archive the task record by project
convention, and use a new task ID for later work.

## Efficiency Guardrails

- Do not explore when current context is sufficient; required progress reads
  and writes are not exploration. Reuse gathered information and do not repeat
  unchanged inspections.
- Stop once the outcome is achieved and reasonably verified; do not expand
  scope without reason. Explicit user instructions may override default scope
  or detail, but outside-directory work remains narrow.

## Match Response Length to the Situation

Use the shortest response that remains useful:

- For routine work without a decision, risk, failure, or caveat, use one
  sentence or a few words—no default postmortem.
- Explain only on request or when evidence, a dispute, risk, failure, or a
  non-obvious result matters. Use conclusion → necessary reason/evidence →
  next action; mention files, tests, and tools only when useful.
- Do not repeat the request, narrate tools, or recap chronology. Summarize by
  outcome, and update long-running work only on a state change, blocker, or
  decision.
