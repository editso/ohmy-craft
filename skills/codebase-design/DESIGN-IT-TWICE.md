# Design It Twice

When the user wants to explore alternative interfaces for a chosen deepening
candidate, compare three deliberately different designs. Prefer parallel
sub-agents when delegation is available and appropriate; otherwise produce the
same three constrained designs serially. Based on "Design It Twice"
(Ousterhout): your first idea is unlikely to be the best.

Uses the vocabulary in [SKILL.md](SKILL.md): **module**, **interface**, **seam**, **adapter**, **leverage**.

## Process

### 1. Frame the problem space

Before producing candidates, write a user-facing explanation of the problem space for the chosen candidate:

- The constraints any new interface would need to satisfy
- The dependencies it would rely on, and which category they fall into (see [DEEPENING.md](DEEPENING.md))
- A rough illustrative code sketch to ground the constraints, not a proposal, just a way to make the constraints concrete

Show this to the user, then proceed to Step 2. When candidates run in parallel,
the user can read and think while they are produced.

### 2. Produce independent designs

Use 3+ sub-agents in parallel when available and permitted. Otherwise, produce
three radically different interfaces serially, keeping each candidate's
constraint and reasoning independent of the earlier candidates.

Give each agent—or each serial candidate—a separate technical brief (file
paths, coupling details, dependency category from [DEEPENING.md](DEEPENING.md),
what sits behind the seam). The brief is independent of the user-facing
problem-space explanation in Step 1. Use different design constraints:

- Agent 1: "Minimize the interface: aim for 1–3 entry points max. Maximise leverage per entry point."
- Agent 2: "Maximise flexibility: support many use cases and extension."
- Agent 3: "Optimise for the most common caller: make the default case trivial."
- Agent 4 (if applicable): "Design around ports & adapters for cross-seam dependencies."

Include [SKILL.md](SKILL.md) vocabulary and, if the project has a
`CONTEXT.md`, its domain vocabulary. Otherwise derive a short domain vocabulary
from the identified target files so candidates name things consistently.

Each candidate outputs:

1. Interface (types, methods, params, plus invariants, ordering, error modes)
2. Usage example showing how callers use it
3. What the implementation hides behind the seam
4. Dependency strategy and adapters (see [DEEPENING.md](DEEPENING.md))
5. Trade-offs: where leverage is high, where it's thin

### 3. Present and compare

Present designs sequentially so the user can absorb each one, then compare them in prose. Contrast by **depth** (leverage at the interface), **locality** (where change concentrates), and **seam placement**.

After comparing, give your own recommendation: which design you think is strongest and why. If elements from different designs would combine well, propose a hybrid. Be opinionated: the user wants a strong read, not a menu.
