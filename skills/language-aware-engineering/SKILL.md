---
name: language-aware-engineering
description: Use before adding or modifying application source code. Read the profile for each target language and the relevant project context; design typed boundaries and a coherent module shape, then review formatted code for type-modeling gaps, duplicate flows, and visual overload. Skip documentation, generated files, and purely mechanical edits.
---

# Language-Aware Engineering

Treat source-code work as design work, whether it creates a new module or
evolves an existing one. Let the target language and the local project guide
the types, interfaces, control flow, and final visual shape.

## 1. Establish the Local Contract

Before designing or editing implementation code:

1. Identify the source files that will be added or changed and the language of
   each file.
2. Read the profile for every target language listed below. A profile is a
   required constraint for code in that language, not optional inspiration.
3. Read only the context needed to place the work: the requested behavior,
   target module and direct callers when they exist, adjacent tests, and
   relevant local formatter, linter, or build configuration. For a new
   module, identify its intended entry point and external interface instead.
4. Infer and preserve established local conventions unless they are the source
   of the problem being fixed.

Read [Rust](references/languages/rust.md) before writing or editing Rust
source. If no profile exists for a target language, follow its project
conventions and language-native practices; do not apply Rust rules to it.

Do not scan an entire repository merely to find a pattern. Expand beyond this
local context only when the change crosses a real module interface.

## 2. Choose a Coherent Shape

Before writing code, decide which of these shapes best expresses the work:

- Extend an existing module interface when the new behavior belongs to its
  current responsibility.
- For new behavior with no existing home, establish a small interface and
  typed boundary before adding handlers, storage code, or control flow.
- Add a small, deep module when callers would otherwise repeat a new rule or
  need to know an implementation detail.
- Change a domain type or state model when the current representation cannot
  express the new behavior safely.
- Make a direct local edit only when it does not create a parallel flow,
  duplicate a business rule, leak an implementation detail, or overload the
  module's responsibility.

Do not extract a helper just to hide an unreadable condition. Extract or move
behavior only when the name describes a real domain concept or concentrates a
rule that would otherwise spread.

Prefer the smallest refactor that removes a demonstrated design problem. Do
not use a requested change as permission to rewrite unrelated code.

## 3. Implement at the Right Seam

While implementing, ask these questions whenever code introduces a new
field, parameter, branch, flag, data conversion, or duplicated block:

- Is this a known domain value that should be represented by a type instead of
  dynamic data or a primitive?
- Is a finite business state being encoded as a string, number, boolean
  combination, or magic constant?
- Would another caller need to repeat this rule or transformation?
- Is this condition one named business rule, several independent validation
  gates, or a branch on state?
- Does this change widen visibility or expose an implementation detail merely
  to make the work convenient?

Answer in code, not in a speculative abstraction. Keep the public interface
small and make new behavior testable through the same interface callers use.

## 4. Format, Then Review the Actual Code

Run the project's formatter before final review. Formatter output is the final
source shape, not a mechanical afterthought.

Read the formatted diff as a maintainer. In particular, revisit each changed:

- multi-line `&&` or `||` chain;
- multi-line macro invocation, especially `matches!`;
- tuple pattern used solely to compact state logic;
- nested closure, generic expression, or builder chain that became a syntax
  staircase;
- duplicate or near-duplicate branch; and
- newly dynamic JSON, string tag, numeric code, or boolean flag.

When formatting turns one conceptual fact into visual noise, use the smallest
semantic rewrite that gives the code a natural shape:

1. Keep a predicate on one line when it is short and expresses one idea.
2. Move a stable business rule behind an intent-revealing operation when that
   operation has a real domain name.
3. Use guards or early returns when checks are independent validation gates.
4. Use a normal `match` when the current state or variant determines control
   flow.
5. Keep multi-line code when each line is a meaningful field, branch, pipeline
   stage, or argument—not merely one fragment of an overlong expression.

Do not increase line width, suppress formatting, or introduce a meaningless
abstraction merely to preserve a one-line expression. Conversely, do not use
language features as syntax theater when a direct expression is clearer.

## 5. Verify the Change

Run the formatter, linter, tests, and project-specific checks relevant to the
changed code. Follow the repository's documented command and version policy.

Before delivery, confirm that the final formatted diff:

- has one clear path for each behavior rather than a parallel implementation;
- keeps known data typed at its boundaries;
- has names that communicate domain intent rather than implementation steps;
- exposes no unnecessary public surface;
- contains tests that assert observable behavior and changed failure cases; and
- changes no unrelated code without an explicit reason.
