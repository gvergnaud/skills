---
name: code-quality
description: Use when writing, reviewing, or refactoring code in any programming language for maintainability, local reasoning, low integration complexity, narrow module APIs, honest domain models, low invalid-state cardinality, explicit dependencies, and manageable cyclomatic complexity.
---

# Code Quality

Use this skill to make code easier to understand, change, and test. Prefer changes that make contracts honest, reduce what callers must know, and keep valid states explicit.

## Preserve Local Reasoning

Code supports local reasoning when behavior can be understood from the local code, its inputs, its outputs, and nearby types or contracts. A reader should not need whole-program knowledge to make a safe change.

Broken local reasoning looks like:

- Shared mutable state: globals, singletons, mutable module variables, mutable object fields used across unrelated methods
- Hidden dependencies: environment, clock, filesystem, network, current user, framework context, service locator, dependency registry
- Action at a distance: events, observers, callbacks, middleware, hooks, or reflection whose effects are invisible at the call site
- Implicit ordering: code only works after `init`, `configure`, `open`, `start`, or some previous mutation
- Non-local control flow: ordinary business failures thrown deep and handled far away
- Representation leaks: callers must know internal layout, storage details, lifecycle states, or protocol phases

Prefer explicit dependencies, explicit results, narrow contracts, and small effectful edges around pure domain logic. If a function depends on or changes something important, make that visible in its signature, receiver, return value, or module API.

## Minimize Module Integration

Integration complexity is the amount of cross-boundary information one part of the codebase must rely on from another to use it correctly. In the same sense that an integrated system is one whose parts causally constrain each other, integrated code has parts whose behavior depends on facts, ordering, invariants, or representations outside the local module.

The recommendation is not to eliminate integration. Useful software needs parts to cooperate. The recommendation is to make necessary integration explicit and small: callers should depend on a stable public contract, not on the callee's internal state machine, data layout, protocol steps, framework context, or storage model.

Reduce integration complexity by designing deep modules: small public interfaces that hide substantial implementation knowledge.

Prefer modules whose public API makes clear:

- what inputs are accepted
- what outputs are produced
- what side effects may happen
- what invariants are guaranteed
- which errors or failure results can occur

Avoid shallow modules that expose many names but hide little. They increase integration cost because callers must still understand the underlying protocol, data shape, ordering rules, framework types, or storage model.

Good module boundaries:

- expose stable domain operations, not internal helper steps
- keep invariants and representation private
- translate framework, transport, persistence, and vendor types at the edge
- make invalid call sequences impossible or hard to express
- keep public names few, cohesive, and hard to misuse

When proposing a refactor, separate the new public API from private implementation helpers. A useful public API might be one or two domain operations; the validation, planning, parsing, and execution helpers behind it should usually remain private unless another module genuinely needs them.

Do not split code just to make files smaller. A new module is useful when it hides decisions behind a smaller interface. If extracting code creates more public protocol for callers to learn, it may make the system worse.

## Align Cardinality With Valid States

The cardinality of an API is the number of different values or call combinations it permits. Good code keeps representable states close to valid domain states.

Watch for product shapes that accidentally multiply possibilities:

- many independent optional or nullable fields
- boolean flags that choose different modes
- option bags where only some combinations are valid
- functions whose arguments are only meaningful in certain combinations
- return values where callers must inspect several fields to know what happened

Use a rough count when useful: independent booleans, nullable fields, enum choices, and optional payloads multiply possible states. If most combinations are invalid, the API is lying about the domain.

Prefer explicit variants:

- tagged records or discriminated unions
- sealed subclasses or enum variants with payloads
- separate constructors or named factory functions
- separate functions for genuinely different operations
- command/result objects validated at the boundary

In languages without strong sum types, use clear tags, private constructors, validators, assertions at trust boundaries, and exhaustive tests to keep invalid states out. Optional fields are fine only when absence is truly valid in that exact case.

## Keep Function Contracts Narrow

A function's signature is part of its design. Pass only the information needed for the job and return only the information callers need.

Avoid:

- passing a whole context, request, model, dependency bag, or options object when the function reads a few fields
- returning broad mutable objects when a precise result would do
- adding flags that make one function implement multiple protocols
- accepting unvalidated primitives whose valid range, format, or lifecycle state is only in comments

If a function needs many inputs, ask whether:

- it is doing too much
- those inputs form a meaningful domain value
- the caller should derive a smaller intermediate value first
- separate variants or operations would better match the domain

Do not hide complexity by replacing many parameters with one vague object. A parameter object helps only when it names a real concept and rules out invalid combinations.

## Limit Cyclomatic Complexity

Branch-heavy code makes readers simulate too many paths. Lower cyclomatic complexity when it clarifies the domain and reduces the chance of missed cases.

Prefer:

- guard clauses for preconditions and early exits
- pure decision functions separate from effectful execution
- explicit variants with exhaustive handling
- tables or maps for regular case dispatch
- small helpers when a branch has a nameable domain meaning

Avoid:

- deeply nested conditionals that mix validation, decisions, mutation, and I/O
- long functions where different branches operate under different hidden assumptions
- boolean flag combinations that create many paths but few valid cases
- clever indirection that hides simple domain branches

First reduce accidental paths by making invalid states unrepresentable. Then simplify the remaining control flow. Splitting a complex function into many public helpers does not help if callers must now understand the same state machine across more names.

Some complexity is essential. A clear exhaustive branch over real domain variants is better than scattering those cases across many files.

## Separate Effects From Decisions

Keep orchestration, pure logic, and side effects distinct when practical.

Good shape:

1. Read or decode inputs at the edge
2. Build a pure plan or result from explicit data
3. Execute the side effects from that plan

Use pure functions for parsing, validation, normalization, mapping, planning, and policy decisions. Keep database access, network calls, filesystem access, logging, framework hooks, and clock reads near boundaries or adapters.

## Review Workflow

When reviewing or refactoring, ask:

- What must a reader know outside this function or module to change it safely?
- Which dependencies or effects are real but invisible?
- Which public names, parameters, fields, or call sequences could be private or impossible?
- What invalid states can this API represent?
- Do product-shaped inputs match the domain, or should some cases be explicit variants?
- Is branching complexity representing real domain cases, or accidental control flow?
- Would a deeper module with a smaller API reduce what callers need to know?

Prefer the smallest change that makes the contract more honest and the next change easier.
