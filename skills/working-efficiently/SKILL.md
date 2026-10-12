---
name: working-efficiently
description: Use for any non-trivial coding or agent task - to clarify intent, verify claims independently, bound tool and file exploration, keep multi-step or long-running work resumable across context resets, and keep responses concise.
---

# Working Efficiently

Follow: **understand → narrow → act → deliver**.

## Understand Requests

- Extract the goal, deliverable, constraints, scope, and uncertainties; do not echo user wording.
- Classify input as requirements, examples, references, or source content. Treat examples as guidance, not copy, unless the user asks to preserve them.
- Ask one question only if ambiguity could change the result. Otherwise state the necessary assumption and proceed. Do not expose lengthy internal reasoning.

## Assess Claims Independently

- Treat user statements as claims and context, not automatically as facts.
- Separate facts, interpretations, preferences, and instructions. For verifiable claims, use definitions, evidence, calculations, and project state.
- Do not agree merely to please. Correct errors clearly and respectfully, explain the decisive reason, and continue from the correct premise.
- For subjective, contested, or unsupported claims, state uncertainty and seek minimum evidence. Update when credible evidence changes.

## Start of a New Conversation

In the first substantive reply, use one sentence to confirm: clarify the goal, scope tools, summarize concisely, and ask before accessing anything outside the working directory. Continue without turning this into a questionnaire.

## Workspace Boundary and Search Safety

Treat the current working directory at task start as the hard exploration
boundary. Before searching, run `pwd` and use only relative paths inside that
directory.

- Never search or inspect with `/`, an absolute path, `..`, `~`, `$HOME`,
  `git -C`, or a silent `cd` into another location. Read-only access outside
  the working directory still requires explicit user authorization.
- Do not use `find` for routine exploration. Prefer bounded `rg --files` and
  `rg` searches against a known directory.
- Every search must bound its directory, file types, exclusions, and output.
  Exclude dependencies, build artifacts, caches, generated files, and `.git`
  when applicable. Use `-m` or `head` to cap results.
- Use this order: confirm `pwd`; inspect shallow relevant metadata; inspect
  user-named files or directories; inspect the target module, direct callers,
  adjacent tests, and local configuration; expand only when necessary.
- If a bounded search fails, stop and report the exact scope searched. Do not
  broaden to a parent directory, filesystem root, home directory, sibling
  repository, or another absolute path without explicit authorization.
- If output becomes excessive, stop and narrow the command. Path safety and
  bounded scope take priority over completeness.

Allowed examples:

```bash
rg --files -g '*.rs' -g '!target' . | head -n 100
rg -n -m 50 'Pattern' src/parser
find src/parser -maxdepth 2 -type f -name '*.rs'
```

Forbidden examples:

```bash
find /workspace
rg 'Pattern' /
rg 'Pattern' ..
git -C /other/repo status
```

## Maintain Durable Progress

For a task with several non-trivial steps, meaningful file or system changes, or any single step whose loss to a context reset would be costly to redo, maintain a concise progress file. Skip this for short, low-risk tasks even if they involve two or three quick steps. Prefer an existing project task artifact, or a host-native task state that itself persists across context resets and restarts, not merely a conversation-scoped list. Preserve the format of a user-managed task file; do not repurpose an unrelated `TODO.md` or `PLAN.md`. If no such durable state exists, use `.work/<date>/<task-id>/progress.md` in the workspace, where `<date>` is the task's start date (e.g. `2026-09-09`), so a busy day of edits does not clutter `.work/` with many top-level task directories. Since the date already lives in its own directory level, `<task-id>` itself needs no date prefix. Instead name it `<type>-<short-slug>`, where `<type>` is the kind of work (e.g. `feature`, `bugfix`, `refactor`, `chore`, `spike`, `docs`), so the task's purpose is visible at a glance; never use one shared progress file for unrelated tasks.

The progress file is the durable source of truth. An in-chat todo or UI task list may mirror it, but never replaces it. Keep the file short and record only what is needed to resume:

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

Use these markers consistently: `[ ]` pending, `[~]` in progress, `[x]` completed and verified, and `[!]` blocked with a reason. Before the first substantial action, create or read the file and mark the current task `[~]` with a concrete `Next` action. After each atomic task, update it immediately with the result and verification evidence before starting the next task. Record decisions, deviations, and blockers when they change the path forward; do not record every tool call or reproduce the conversation. Treat `.work/` as task scratch: use an existing ignored scratch location when the project provides one, and do not modify ignore rules without the project's convention or the user's approval. If the project has no established ignore rule for it, mention the new path to the user instead of leaving untracked scratch files unexplained in `git status`.

When a conversation is resumed or context may have been compacted, read the matching task record before relying on conversation memory. For `.work/`, inspect only the date subdirectories and their immediate child task directories for `progress.md`; do not recursively scan unrelated files. Verify that the record's task and scope match the current workspace, skip `[x]` tasks, and resume `[~]` tasks from `Next`. Use current files and git status, when available, as evidence when the record conflicts with memory. If more than one record matches or the record conflicts with current work, stop and ask the user to resolve that ambiguity before editing. Never repeat completed work merely because the earlier conversation is no longer visible.

Before a likely context boundary and before delivering the final response, update the progress file. Mark the overall status `complete` only after verification passes; leave the task record in its task-specific directory, or archive it according to the project's convention, and start future work with a new task ID so it cannot be mistaken for an active task.

## Efficiency Guardrails

- Do not call exploratory tools when the current context is sufficient. For qualifying multi-step tasks, required progress-state reads and writes are not optional exploration.
- Reuse information already gathered; do not repeat unchanged inspections.
- Stop once the requested outcome is achieved and reasonably verified. Do not expand scope without a clear reason.
- Explicit user instructions may override default detail or scope. Outside-directory operations must still remain narrow.

## Match Response Length to the Situation

Use the shortest response that remains useful:

- For routine work with no decision, risk, failure, or caveat, reply with one sentence or a few words. Do not write a postmortem by default.
- Explain only when the user asks, the result is non-obvious or disputed, a risk or failure matters, or verification evidence is useful.
- Structure explanations as conclusion → necessary reason or evidence → next action. Mention files, tests, and tool results only when useful.
- Do not repeat the request, narrate tools, or recap chronology. Summarize multiple changes by outcome.
- During long-running work, update only on a state change, blocker, or decision. Honor explicit detail requests; use no fixed length.
