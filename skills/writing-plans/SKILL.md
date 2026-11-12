---
name: writing-plans
description: Turn an approved architectural design into a precise, reviewable implementation plan before code changes begin.
---

# Writing Implementation Plans

Turn an approved design into the decisions an implementer cannot safely infer:
file responsibilities, interfaces, ordering, observable tests, and verification.
Use this after an Architectural `brainstorming` handoff, or when the user
explicitly requests a plan for substantial multi-step work. Do not create a
plan for a routine, focused change or a `brainstorming` Bounded path unless the
user asks for one.

This skill writes a plan, not source code. The plan is a static design artifact;
execution status belongs only in the durable progress record governed by
`working-efficiently`.

## Establish the Plan Boundary

Read the approved spec or requirements, then inspect only the target files,
direct callers, adjacent tests, and relevant project configuration needed to
make the plan precise. Follow `working-efficiently` for bounded exploration.

If the approved design leaves a material architectural decision unresolved, do
not invent it in the plan: stop and ask the user to resolve it in design. If
independent subsystems cannot produce useful, testable increments together,
propose separate plans.

Save the plan to `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` unless the
user or project specifies another location. Do not modify production or test
code while writing it, and do not make a commit an automatic plan task.

## Make the Plan Executable

Start the document with:

```markdown
# <Feature> Implementation Plan

**Goal:** <one sentence>
**Spec:** <approved spec path or requirements source>
**Architecture:** <the already-approved implementation shape>

## Global Constraints

- <exact version, compatibility, dependency, naming, or platform constraint>

## Review Focus

- <important input, failure mode, or behavior the plan must test>
```

Include only constraints and risks that affect the work. `Review Focus` names
the most likely untested inputs or failure modes and assigns each to a task.

Before tasks, provide a concise file map: every created or modified production
file and test file, with its responsibility. Follow the established project
shape and applicable language profile; do not use a plan to introduce a
speculative layer, package, or test surface.

Each task is a smallest independently reviewable, testable increment—not a
micro-step or a future commit. Use this shape when the information applies:

````markdown
### Task N: <deliverable>

**Files:**
- Create: `path/to/file`
- Modify: `path/to/file` — `<symbol or responsibility>`
- Test: `tests/path/to/test`

**Interfaces:**
- Consumes: `<exact prior type, function, command, or contract>`
- Produces: `<exact signature or behavior used later>`

1. Define the observable test scenario, assertions, and relevant fixed values.
2. Implement `<exact type, function, module, or boundary>` with the decisions
   the spec fixes; leave an obvious implementation body to the implementer.
3. Run `<project command>` and expect `<observable passing result>`.
````

Omit inapplicable fields rather than inventing them. A task must name exact
files, symbols, signatures, values, error behavior, compatibility constraints,
and test assertions whenever the implementer could otherwise make a different
reasonable choice. Do not pad it with "handle errors", "add validation", or
"write tests". Conversely, do not transcribe routine function bodies or repeat
another task's details. Refer to that task's declared interface instead.

Use the repository's test conventions and documented commands. Applicable
language profiles govern code shape and test placement at implementation time;
the plan records their effects only when they change a task's decision.

## Review Before Handoff

Review the completed plan against the approved spec:

- Every requirement and Review Focus item has an owning task and observable
  verification.
- Task order and declared interfaces agree on names, types, and ownership.
- Each task has one coherent, independently testable deliverable.
- The plan decides what the implementer cannot infer, without becoming a code
  transcript or a second architecture proposal.

Fix findings inline. Then give the user the saved plan for review and wait for
approval before implementation. Execute the approved plan through the normal
workflow; do not require a separate execution skill or track completion in the
plan document.
