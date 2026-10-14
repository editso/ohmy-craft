---
name: coding-standards
description: Guidance for writing clear, typed, idiomatic, and maintainable code.
---

# Coding Standards

Treat source-code work as design work. Let the target language and local
project guide types, interfaces, control flow, and final visual shape.

## 1. Establish the Local Contract

Before implementation work, identify target files and languages; then read
only the needed local context: requested behavior, target module and direct
callers, adjacent tests, and relevant formatter, linter, or build
configuration. For a new module, identify its entry point and external
interface instead. Preserve local conventions unless an applicable profile
overrides them or they cause the problem being fixed.

Read the available profile for each target language before writing or editing
source:

- [Rust](references/languages/rust.md)

If a target language has no profile, follow its project conventions and native
practices; never apply Rust rules to it.

Do not scan an entire repository merely to find a pattern. Expand beyond this
local context only when the change crosses a real module interface.

## 2. Choose a Coherent Shape

Before writing code, decide which of these shapes best expresses the work:

- Extend the existing module when behavior belongs to its responsibility.
  Otherwise define a clear responsibility and typed boundary before adding
  handlers, storage, or control flow; expose a caller-facing operation only
  when callers, variation, or an external boundary require one.
- Add a small, deep module when callers would otherwise repeat a rule or need
  an implementation detail. Change the domain type or state model when the
  existing representation cannot express behavior safely.
- Make a direct local edit only when it creates no parallel flow, duplicate
  business rule, implementation leak, or overloaded responsibility.

Extract or move behavior only when its name describes a real domain concept or
concentrates a spreading rule—not merely to hide an unreadable condition. Use
the smallest refactor that fixes a demonstrated design problem; a requested
change does not authorize unrelated rewrites.

### Require Evidence for an Abstraction

- Start concrete; do not pre-create an extension point for hypothetical
  variation. Add an interface, type parameter, builder, factory, or forwarding
  wrapper only for a demonstrated need: real callers or implementations, a
  stable external boundary, a domain invariant, or a duplicated caller rule.
  A type protecting a real invariant is not speculative.
- Add a module only for a distinct current responsibility, not file length or
  a one-use wrapper. Every abstraction must remove caller knowledge or a
  duplicated rule; a one-use forwarding layer, one-implementation custom
  interface, or singly instantiated type parameter is insufficient alone.
  Future reuse or testing convenience is not evidence.
- Current callers can justify one narrow, concrete operation; caller count
  alone does not justify a trait, generic abstraction, factory, or wrapper.

When work redistributes responsibilities across modules or establishes a stable
interface for multiple callers, use
[`codebase-design`](../codebase-design/SKILL.md) before implementation.

## 3. Implement at the Right Seam

For every new field, parameter, branch, flag, conversion, or duplicate block,
ask: should known data be typed rather than dynamic or primitive; is a finite
state encoded as a string, number, boolean combination, or magic constant;
would callers repeat the rule; is the condition a rule, validation gates, or a
state branch; and does convenience widen visibility or leak implementation?
Answer in code without speculative abstraction. Keep the public interface
small and test new behavior through the caller-facing interface.

## 4. Format, Then Review the Actual Code

Run the formatter before review; its output is the final source shape. Read
the formatted diff as a maintainer and revisit each changed:

- multi-line `&&` or `||` chain;
- multi-line macro invocation, especially `matches!`;
- tuple pattern used solely to compact state logic;
- a fourth active level of control-flow nesting;
- nested `if` blocks whose outer condition exists only to guard the successful
  path;
- nested closure, generic expression, or builder chain that became a syntax
  staircase;
- duplicate or near-duplicate branch; and
- newly dynamic JSON, string tag, numeric code, or boolean flag.

When formatting turns one fact into visual noise, use the smallest semantic
rewrite with a natural shape:

1. Keep a predicate on one line when it is short and expresses one idea.
2. Move a stable business rule behind an intent-revealing operation when that
   operation has a real domain name.
3. Use guards or early returns when checks are independent validation gates.
4. Use a normal `match` when the current state or variant determines control
   flow.
5. Keep multi-line code when each line is a meaningful field, branch, pipeline
   stage, or argument—not merely one fragment of an overlong expression.

When an outer `if` only wraps the successful path, invert it and exit at the
narrowest correct scope: function `return` or error propagation, loop
`continue`, or another language-native exit. Do not exit merely to reduce
indentation when later work must update state, flush, clean up, or preserve an
effect.

Treat a fourth active control-flow level as a refactoring trigger: flatten the
main path with guards, separate validation gates, branch on state, or extract
a real operation—not a nameless helper. A guard that exits immediately does
not add nesting to the following main path.

Do not override the project formatter or widen its configured line width solely
to preserve a one-line expression. Do not introduce a meaningless abstraction
for the same purpose. Conversely, do not use language features as syntax
theater when a direct expression is clearer.

## 5. Verify the Change

Run the repository's relevant formatter, linter, tests, and project checks,
using its documented commands and versions. Before delivery, confirm the
formatted diff has one path per behavior, typed known-data boundaries,
domain-intent names, no unnecessary public surface, tests for observable
behavior and changed failures, and no unrelated changes without reason.
