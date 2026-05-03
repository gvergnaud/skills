---
name: typescript-code-quality
description: Use when writing or refactoring TypeScript for maintainability, type safety, explicit error handling, and strong module boundaries. Emphasize local reasoning, explicit dependencies, separation of side effects from pure logic, separation of control flow from data flow, pragmatic functional patterns over stateful object-oriented design, and stable module APIs.
---

# TypeScript Code Quality

## Core Principle: Local Reasoning

Local reasoning is the ability to understand what a piece of code does by reading only that piece of code and its immediate inputs/outputs, without tracing through the rest of the codebase. When you can reason locally, you can change code with confidence.

A pure domain function is easy to reason about locally when:

- Its behavior is fully determined by its arguments
- Everything it produces comes through its return value
- Its type signature is its complete contract (no hidden side-contracts like "must call init() first" or "also updates the cache")

A function is hard to reason about locally when it has implicit dependencies: things it depends on or affects that aren't visible in its signature. Every implicit dependency is a trap for the next person who changes the code. They read the signature, believe they understand the contract, make a reasonable change, and break something because the real contract was wider than what was declared.

### Common sources of broken local reasoning

1. **Shared mutable state.** A function reads or writes a variable that other functions also read or write. You can't know what the function will do without knowing the current state, and you can't know what it will break without finding every other reader/writer. This includes global variables, singleton caches, mutable class fields accessed by multiple methods, and module-level state.

2. **Action at a distance.** Event emitters, pub/sub, observer patterns, middleware chains. A function emits an event; some handler somewhere reacts. The emitter's signature says nothing about the consequences.

3. **Implicit ordering constraints.** Code that only works if you call `init()` before `process()`, or set `this.config` before calling `this.run()`. The type system doesn't enforce the order, so correctness lives in the caller's head, not in the code.

4. **Non-local control flow.** Exceptions thrown deep in a call stack and caught somewhere far above. The intermediate layers don't declare that they can fail this way. You can't tell from reading a function whether it might throw or what kind of error.

5. **Ambient authority.** Functions that reach into the environment: reading env vars, checking the clock, hitting the network, accessing the filesystem. Their behavior changes depending on the world outside the process.

6. **Type lies.** When a parameter is `string` but actually must be a specific format, or `any` or un-narrowed `unknown` is used to sidestep real contracts. The type signature says one thing; the runtime behavior depends on something else.

### Making dependencies explicit

The distinction:

- Explicit dependency: appears in the function signature (parameters, return type). Visible, compiler-checked.
- Implicit dependency: exists but invisible at the call site. You must read the implementation (or other files) to discover it.

Making dependencies explicit is not about being verbose. It's about making the code honest.

### Blast radius of a change

When you change a function that has only explicit dependencies and no side effects, the blast radius is exactly the set of its callers, which is trivially findable. When you change a function that mutates shared state, emits events, or relies on implicit ordering, the blast radius is unknowable without whole-program analysis.

If a function's declared contract is honest and its dependencies are explicit, the compiler can catch contract breaks and point you to the affected callers. It does not prove behavior correct; it keeps the blast radius visible.

## Principle: Separate side effects from pure logic

Push I/O, network calls, filesystem access, DB writes, logging, and framework integration to the edges. Domain logic should stay pure; edge code may be effectful, but the dependencies and side effects should be explicit.

Prefer:

- Pure functions for parsing, validation, mapping, planning, normalization, and decision-making
- Thin effectful wrappers that call pure logic and then perform the side effect

Good shape:

1. Read inputs
2. Build a pure plan
3. Execute the plan

Avoid:

- Mixing fetch/write/log calls into transformation logic
- Hidden mutation inside utility functions
- Business logic embedded in framework handlers

## Principle: Separate control flow from data flow

Keep orchestration code distinct from transformation code.

Prefer:

- Orchestrators that answer: "In what order do steps happen?"
- Pure helpers that answer: "How is data transformed?"
- Data structures that make decisions explicit

Good shape:

- `buildPlan(input) -> Plan`
- `executePlan(plan) -> Result`

Avoid:

- Branch-heavy functions that both decide and mutate
- Recomputing the same derived state in multiple places
- Encoding important logic implicitly in string manipulation or ad hoc conditionals

## Principle: Prefer pragmatic functional patterns over stateful OO

Default to functions and data structures unless objects clearly reduce complexity.

Prefer:

- Plain functions
- Immutable data flow
- Tagged unions / discriminated unions
- Explicit inputs and outputs
- Small composable helpers
- No shared or global mutation

Use objects/classes only when they model real lifecycle or stateful resources cleanly.

Avoid:

- Classes that mostly act as namespaces
- Stateful helpers with hidden internal behavior
- Methods that depend on initialization order or mutation history

## Principle: Build modules with clear, stable APIs

Each module should have one job and expose a small surface area.

Prefer modules like:

- `planner`: derive normalized plans from raw inputs
- `executor`: perform effectful work from plans
- `formatter`: transform data for presentation
- `repository`: read/write external systems

Module boundaries should make it obvious:

- what goes in
- what comes out
- what side effects happen
- what invariants are guaranteed

Avoid:

- leaking transport/framework types across layers
- requiring callers to understand internal representation details

## Principle: Import paths encode module boundaries

- **Relative imports** (`./`, `./sub/`): private code within the current directory or direct children.
- **Absolute imports**: public APIs from other modules.
- **Avoid `../` across module boundaries**: use absolute imports for public APIs when the repo supports them. If the repo lacks aliases, follow local convention but do not use path traversal to pierce ownership.

## Principle: Type safety must be honest

Type-safe TypeScript means the static type describes what is actually true at runtime. Do not silence the compiler; make the contract true.

Avoid unsafe escapes:

- unchecked assertions: `value as SomeType`, `<SomeType>value`, double assertions, and non-null assertions
- `any`, `Function`, `{}` catch-alls, and vague records where the shape is known
- broad `@ts-ignore`, `@ts-expect-error`, and lint disables
- casts around `JSON.parse`, network responses, database rows, env vars, messages, or other untrusted input

`as const` for literal narrowing is fine. Unavoidable assertions around broken external types must be tiny, adapter-local, and downstream of validation.

Preferred moves:

- infer locally, annotate boundaries, and use `satisfies` for checked object literals
- accept `unknown` at trust boundaries and decode it with type guards or Zod schemas
- use discriminated unions, branded/refined primitives, and exhaustive switches
- treat lookup APIs as returning maybe: handle `Map.get`, `Array.find`, indexed access, and optional fields explicitly
- wrap weakly typed libraries in small adapters that return honest types

Expected failures are data. Validation errors, parse errors, missing data, permission failures, conflicts, and retryable external failures should appear in the return type as `Result<T, E>` or a domain-specific discriminated union.

Throw only for genuinely unexpected situations: violated internal invariants, impossible states, programmer misuse, corrupted process state, or failures this layer cannot sensibly model. Do not throw random `Error`s for ordinary business flow. When wrapping unexpected exceptions, preserve the cause.

Catch errors as `unknown`, narrow before reading, and never cast with `as Error`. Keep error variants structured enough that callers do not need string matching.

## Practical Heuristics

### Use explicit intermediate data

If an intermediate data representation makes the logic easier to understand and the performance cost is ok, start by creating this intermediate structure.

### Prefer collection transforms over local accumulation

When transforming one collection into another, prefer declarative operators over manual accumulation.

Prefer:

```ts
const activeNames = users
  .filter((user) => user.isActive)
  .map((user) => user.name);

const allCommands = groups.flatMap((group) => group.commands);
```

Avoid:

```ts
const activeNames: string[] = [];
for (const user of users) {
  if (user.isActive) activeNames.push(user.name);
}
```

The declarative version makes the intent more obvious. The mutating version makes the reader track control flow and intermediate state. That cost matters even when the mutation is technically local.

Use local mutation only when there is a clear payoff, such as a measured performance need, an algorithm that is genuinely clearer imperatively, or an API that is inherently mutating. In those cases, isolate the mutation tightly and do not let it leak into surrounding design.
