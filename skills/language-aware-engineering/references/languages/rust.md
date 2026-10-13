# Rust Profile

Write Rust that uses its type system, ownership model, enums, traits, and
standard library to express the domain. Do not write C-shaped control flow or
dynamic JSON-shaped domain logic merely because it is faster to patch.

Project conventions and the configured Rust version remain authoritative. Use
these rules when creating or evolving a module, not to impose unrelated
rewrites.

## Model Known Data With Types

- Deserialize known external data into named request, event, configuration, or
  persistence types at the adapter boundary. Serialize named types only at an
  outgoing boundary.
- Do not pass `serde_json::Value`, `serde_json::Map`, raw JSON strings, or
  `HashMap<String, Value>` / `HashMap<String, String>` through business logic
  when the schema is known. A known response must be a named response type,
  not a constructed JSON value.
- Dynamic JSON is allowed only in a module whose explicit job is to handle
  arbitrary JSON, such as schema-agnostic tooling, a generic proxy, logging,
  or a fragment whose schema is genuinely unknown. Convert it to a typed
  representation as soon as the schema becomes known.
- Use `Option<T>` for absence. Do not use sentinel strings, numbers, empty
  collections, or null-like placeholders to represent a missing value.
- For an operation that can fail recoverably, return `Result<T, E>` with a
  meaningful error instead of a bare success boolean, numeric error code,
  string, or `panic!`.
- Model a finite set of mutually exclusive business states with an `enum`, not
  string tags, integer codes, magic constants, or combinations of flags.
- Keep a genuine binary fact as `bool`; do not manufacture an enum merely to
  avoid booleans. Use an enum once a value represents a named state, carries
  data, has transition rules, or is likely to gain variants.
- Use named newtypes or value objects where a primitive alone does not
  communicate a domain role or invariant: identifiers, validated names,
  quantities, money, externally supplied keys, and similar values are common
  examples. Put validation in construction where practical so invalid states
  are hard to represent.
- Use a named struct rather than an opaque tuple for stable public inputs or
  outputs with several meaningful fields. Tuples remain appropriate for short,
  local transformations and iterator or closure plumbing where they improve
  readability.

## Errors, Ownership, and Interfaces

- Prefer domain-specific error types or contextually useful library errors to
  stringly typed failures. Application-level code may use an application error
  wrapper where the project convention supports it; library interfaces should
  preserve meaningful error semantics.
- Avoid `unwrap()` and `expect()` outside tests unless an invariant is both
  established locally and a panic is the intended contract. Propagate or
  handle recoverable errors instead.
- Do not use `.clone()` just to satisfy the borrow checker. First narrow a
  borrow, move ownership, borrow the needed field, or restructure the data
  flow. Clone only when a second owned value is semantically correct.
- Keep visibility as narrow as possible. Prefer private items, `pub(super)`,
  or `pub(crate)` over public exports unless external consumers need the item.
- Do not add `unsafe` unless the task explicitly requires it or its necessity
  has been established and explained before use.
- Do not add generic `AsRef`, `Into`, or `impl Trait` parameters by reflex.
  Use them at a boundary when they make a stable interface easier to call;
  favor a concrete, readable signature for local code and when generics add no
  caller value.
- Prefer concrete types and direct calls in local code. Introduce a custom
  trait or generic abstraction only for a demonstrated current variation or
  stable boundary; a one-implementation custom trait, one-use generic, or
  forwarding wrapper is not enough. Implement established standard or library
  traits when they express the type's real semantics. Hypothetical reuse and
  testing convenience alone do not qualify.

## Cargo Packages and Modules

- Keep related application or library behavior in its existing Cargo package
  by default. Organize internal responsibilities with `src/` modules and
  directories; a domain, storage, transport, application, or feature area is
  not by itself a reason to create a package or workspace member.
- Create a new Cargo package only when it solves a demonstrated Cargo-level
  need: an independently consumed, published, or versioned library; an
  independently deployed product; a dependency direction that must be
  enforced at Cargo level; required target, platform, native-dependency, or
  artifact isolation; or a Rust-required boundary such as a procedural macro
  crate.
  Do not split for naming layers, organizing files, testing convenience, one
  implementation, or hypothetical reuse.
- A package boundary must justify the dependency edge and any public interface
  it requires. Keep implementation details in the existing package; do not
  widen visibility or add re-exports solely to make a new boundary work.
- When several executable entry points share application code and lifecycle,
  prefer one package with a library crate and thin `src/bin/` binaries. Split
  packages only when the executables have an independent product, deployment,
  target, or dependency boundary.

## Choose Idioms for Clarity, Not Performance

- Use `match` or `if let` when variants or destructuring are central to the
  decision. Handle meaningful variants explicitly; use a catch-all only when
  the omitted variants are intentionally equivalent.
- Use `for` for clear collection traversal. Use iterator combinators when the
  transformation is simpler to read as a compact pipeline. Avoid index-based
  loops unless the index itself is semantically necessary.
- Use `From` and `TryFrom` for real conversions instead of a scattering of
  ad-hoc conversion methods when doing so makes construction and error flow
  clearer.
- Derive traits only when their semantics and costs are appropriate. In
  particular, do not derive `Clone` merely because it is convenient.
- Keep a type with its inherent implementations and closely related behavior
  together unless established module organization provides a clearer shape.
- Split modules by responsibility when that improves locality. Do not split a
  cohesive module by line count or create a new module for a one-use wrapper.

## Macros and Declarative Code

- Use established language or library macros and derives when they directly
  express the construct: trait derivation, collection or formatting literals,
  assertions, and `bitflags!` for genuinely independent flags are common
  examples.
- Use a small private `macro_rules!` macro when a declarative call or
  declaration schema is clearer than an equivalent type or function. Repeated
  syntax is one signal, not a requirement. Its invocation must name one
  operation. Do not swallow errors or introduce hidden non-local control flow;
  make fallibility and material effects evident at the invocation.
- When values vary but the operation is the same, use a named specification
  type with a helper or loop instead. Keep explicit phases when ordering or an
  intervening effect matters.
- Define an exported or procedural macro only for a stable cross-crate API or
  unavoidable compile-time generation, not local convenience. Do not add any
  macro merely to remove a few lines, conceal a poor interface, or make code
  look clever.

## Let `rustfmt` Expose Expression Overload

The formatted file, not an unformatted draft, defines the visual result.
After `cargo fmt`, inspect changed expressions that have become multi-line.

- Use `matches!` only when the complete predicate stays short and reads as one
  visual unit. Do not use a tuple pattern or a multi-line macro just to appear
  concise.
- A short, single-concept predicate may remain direct:

  ```rust
  if user.can_approve() && order.is_pending() {
      // ...
  }
  ```

- If a condition wraps because it represents one stable business rule, put the
  rule on the type or module that owns it and use an intent-revealing call.
  The method name must describe a real concept, not merely conceal operators.

  ```rust
  if order.can_be_approved_by(&user, now) {
      // ...
  }
  ```

- If wrapped conditions are independent validation gates, use guards or early
  returns with specific errors. If an enum state determines the result, use a
  straightforward `match`.
- Multi-line struct literals, function arguments, `match` arms, and pipelines
  are desirable when their lines express separate fields, inputs, branches, or
  stages. Do not collapse these merely to reduce line count.
- Do not raise `max_width`, hand-format against `rustfmt`, or add an arbitrary
  helper solely to force a dense expression onto one line. A wider project
  line-width policy is valid only when adopted consistently for the project.

## Limit Control-Flow Nesting

- Keep active control-flow nesting at **three levels or fewer** in ordinary
  production code. A fourth nested decision or iteration is a refactoring
  trigger. Five or more levels require a concrete constraint—such as a
  framework callback shape, protocol parser, or table-driven traversal—that
  prevents a coherent reduction. State that constraint in the delivery summary;
  do not add a code comment solely to justify nesting.
- Count each active `if` / `if let`, `match`, `for`, `while`, `while let`, or
  `loop` that a reader must hold in mind to understand a line. A `match` adds
  one level, but its arms do not each add another; an `else if` chain remains
  one conditional level. Do not count `fn`, `impl`, struct literals, argument
  lists, or guard exits such as `?` and `let ... else { return; }`. A
  single-purpose `if` guard that exits and immediately closes, such as
  `if !ready { return; }`, does not add nesting depth to the following main
  path, although it remains a branch for complexity purposes.
- Do not satisfy the limit cosmetically. A helper is justified only when it
  names a cohesive operation; a helper whose only purpose is to hide one more
  nested block has not reduced the cognitive load.

- When an outer `if` exists only to contain the successful path, invert it and
  exit at the narrowest correct scope. Use early `return`, `?`, or `let ...
  else` in functions; use `continue` in loops; and use `break` only when
  ending the enclosing loop is the intended behavior. The point is to keep the
  main path left-aligned, not to prefer any one exit keyword.

  ```rust
  fn render_cell(cell: Option<Cell>) {
      let Some(cell) = cell else {
          return;
      };

      render(cell);
  }
  ```

  ```rust
  for entry in entries {
      let Some(entry) = entry else {
          continue;
      };

      if !entry.is_visible() {
          continue;
      }

      render(entry);
  }
  ```

- Do not introduce an early exit merely to eliminate indentation. It is wrong
  if code after the nested condition must update state, flush a buffered value,
  release a resource, emit required output, or perform other work in the same
  scope. In that case, keep the control flow explicit or extract the cohesive
  operation that owns both the condition and the required follow-up.
- If several guards only conceal a single state transition or business rule,
  use a named operation or `match` instead of turning the flow into an
  unstructured sequence of exits.

## Tests and Verification

- For Rust code governed by this profile, keep production code in `src/` and
  test code in the Cargo package's `tests/` directory, overriding an existing
  source-local test convention. Do not add or grow `#[cfg(test)] mod tests`;
  documentation examples remain with the public API they document.
- Test behavior through the crate's public, domain-oriented interface. Do not
  widen visibility merely to reach an implementation detail from a test. If a
  critical rule cannot be expressed through that interface, reconsider the
  boundary rather than exposing an arbitrary helper.
- For a binary-only package, exercise its CLI, protocol, or other external
  boundary from `tests/`; do not add a test-only library surface.
- Put shared fixtures, test applications, and test-only helpers under
  `tests/support/`. When modifying an existing source-local test, migrate it;
  do not perform a mechanical repository-wide move without need.
- Test names should state the scenario and expected outcome.
- Assert returned values, state changes, errors, and externally observable
  effects. `is_ok()` or `is_some()` alone is insufficient when the actual
  value is meaningful.
- Match relevant conventions within `tests/`. Do not add ritual Arrange/Act/
  Assert comments if they are not customary or useful.
- Run the repository's relevant formatter, Clippy configuration, and tests.
  Treat new warnings as defects unless a narrow, documented exception is
  genuinely required.
