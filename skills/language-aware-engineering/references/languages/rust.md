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
- Use `Result<T, E>` with meaningful errors for recoverable failure. Do not
  encode success with booleans, numeric error codes, strings, or `panic!`.
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

## Tests and Verification

- Test names should state the scenario and expected outcome.
- Assert returned values, state changes, errors, and externally observable
  effects. `is_ok()` or `is_some()` alone is insufficient when the actual
  value is meaningful.
- Match nearby test structure and comment conventions. Do not add ritual
  Arrange/Act/Assert comments if they are not customary or useful.
- Run the repository's relevant formatter, Clippy configuration, and tests.
  Treat new warnings as defects unless a narrow, documented exception is
  genuinely required.
