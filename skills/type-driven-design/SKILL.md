---
name: type-driven-design
description: Language-agnostic guidance on Type-Driven Design — using the type system to make illegal states unrepresentable, parse rather than validate, and eliminate defensive runtime checks. Applies to TypeScript, Rust, Swift, C#, F#, Haskell, Kotlin, Scala, and any language with a meaningfully expressive type system. Use when designing domain models, writing validation logic, reviewing types that use optionals/booleans as flags, adding value-object wrappers around primitives, modelling state machines, designing APIs for correctness-by-construction, or whenever the user mentions "type-driven design", "type-driven development", "parse don't validate", "make illegal states unrepresentable", "smart constructors", "newtype pattern", "branded types", "phantom types", "typestate", "domain primitives", "algebraic data types", "sum types", "discriminated unions", or "push constraints back".
---

# Type-Driven Design

> **Credit & heritage.** This skill stands on the shoulders of the authors who developed, named, and popularized these ideas. The Core Principles section quotes Alexis King's *Parse, Don't Validate* verbatim (with attribution). The pattern catalog draws on work by Matt Parsons, Matt Noonan, Alex Ozun, Mark Seemann, Dominic Codespoti, Ruggero Rebellato, Mike Burns, and Jaime González García. Full bibliography in [bibliography.md](bibliography.md). **Read the originals** — they are the real thing; this skill exists only to make their shared wisdom loadable as in-context guidance.

Type-Driven Design (TyDD) is the discipline of **lifting invariants and proofs of validation into the type system** so that illegal states become compile-time errors instead of runtime bugs. It replaces scattered defensive checks with structural guarantees. Types stop being a bureaucratic annotation layer and become your primary design tool.

## The thesis

> *Parse, don't validate.*  
> — Alexis King (2019)

A **validator** returns a boolean (or throws) and discards what it learned. Every downstream function is then forced either to re-validate or to trust an invariant the compiler cannot see. A **parser** returns a more structured type on success; it **preserves the proof of validation in the type** so the compiler can propagate it through the rest of the program.

Type-Driven Design is what you get when you apply that idea consistently: instead of trusting yourself to check the right things at the right places, you arrange your types so that the wrong things cannot be written at all.

## This is a spectrum, not a religion

**Type-Driven Design is not opposed to Test-Driven Development — they are complementary, and which one "fits" depends on your language's ergonomics.** Mike Burns puts it well: *"TDD purism belongs to the world of dynamic typing; being a TDD purist in Rust is a mistake, but being a type-driven designer in Ruby is also a mistake."*

The payoff from lifting logic into types scales with three things:

1. **How expressive the type system is** — Haskell / Rust / Swift / Kotlin / TypeScript reward heavy investment; Go and Python < 3.10 reward much less; pre-Java-17 rewards even less.
2. **How ergonomic the language makes it** — algebraic data types, pattern matching, refinement syntax, and non-null defaults lower the cost of every pattern below.
3. **How long the code will live and how many people will touch it** — types pay for themselves on load-bearing code that outlives its author.

Match your investment to your ecosystem. In a dynamic language, you can still use many of these patterns (branded types, value objects, tagged unions via classes), but don't pretend you are getting compile-time guarantees you aren't.

Type-Driven Design also **does not replace tests.** Types prove that certain categories of bug cannot occur; they do not prove that your sorting algorithm sorts. Logic errors, algorithmic flaws, and domain misunderstandings still need tests. *"If it compiles, it's correct"* is a dangerous slogan.

## Core Principles (Alexis King, verbatim)

These nine principles come directly from Alexis King's *Parse, Don't Validate* (2019). They are written in the language of Haskell but the ideas are language-agnostic.

1. **Use a data structure that makes illegal states unrepresentable.** Model data using the most precise data structure reasonably possible. Don't stick with inadequate representations just because they're convenient.
2. **Push the burden of proof upward as far as possible, but no further.** Get data into precise representations quickly — ideally at system boundaries — before any action occurs.
3. **Write functions on the data representation you wish you had, not the data representation you are given.** Design by working from both ends until the gap closes.
4. **Let your datatypes inform your code, don't let your code control your datatypes.** Avoid arbitrary `Bool` flags; refactor to use correct representations.
5. **Treat functions that return `()` / `void` with deep suspicion.** They often mask validation that should return meaningful results.
6. **Don't be afraid to parse data in multiple passes.** Context-sensitive parsing is acceptable; what matters is avoiding *action* before full parsing.
7. **Avoid denormalized representations of data, especially if it's mutable.** Duplication creates trivially representable illegal states.
8. **Keep denormalized representations of data behind abstraction boundaries.** If denormalization is necessary, use encapsulation for a single source of truth.
9. **Use abstract datatypes to make validators "look like" parsers.** When illegal states remain impractical to eliminate, use smart constructors via `newtype` / opaque types.

Matt Parsons adds a tenth that I consider equally load-bearing:

10. **Push constraints backward, not forward.** When a function can't handle every input, prefer to restrict its *argument type* (caller must provide valid data) rather than widening its *return type* to `Maybe` / `Option` / `Result`. *"By restricting our past, we gain freedom in the future."*

## The Pattern Catalog

The full pattern catalog — with concrete code examples in multiple languages — lives in [patterns.md](patterns.md). A brief summary for fast reference:

| # | Pattern | Problem it solves | Canonical example |
|---|---|---|---|
| 1 | **Domain primitives** (newtype / branded / value object) | Stringly / numerically typed APIs carrying invariants the compiler cannot see | `Email`, `UserId`, `PositiveInt` |
| 2 | **Sum types** (discriminated unions / ADTs / enums-with-payload) | "Exactly one of these shapes" modelled as co-dependent optionals or boolean flags | `PaymentMethod = Card \| GiftCard` |
| 3 | **Product types** (records / structs) | "All of these together" | `Credentials { email, password }` |
| 4 | **Make illegal states unrepresentable** | A type that permits combinations the business logic rejects | Two optionals that must not both be set → enum |
| 5 | **Option / Maybe over null** | Null-pointer errors and implicit absence | `Option<T>` forcing handling |
| 6 | **Result / Either over exceptions** | Invisible failure paths in function signatures | `Result<User, CreateUserError>` |
| 7 | **Parse, don't validate** | Validation proof discarded the moment the boolean is used | `String -> Option<Email>` |
| 8 | **Phantom types & typed identities** | Interchangeable IDs of different entities being mixed up at runtime | `ID<User>` vs `ID<Song>` |
| 9 | **Typestate** | State machines where the wrong method is called in the wrong state | `Connection<Closed>` can't call `send()` |
| 10 | **NonEmpty / refined collections** | Empty-check scattered through every consumer | `NonEmpty<List<T>>` |
| 11 | **Parse at the boundary** | Untrusted data (JSON, DB, URL params, deep links) flowing raw into business logic | Hand-written `Decodable` that rejects illegal combinations |

## Algebraic distribution: a diagnostic tool

Think of a `struct` as multiplication and an `enum` / sum type as addition. A type like `struct { a: X | Y; b: P | Q }` has `(X+Y) × (P+Q) = XP + XQ + YP + YQ` possible shapes. If fewer than all four combinations are legal, **the type is lying** — it admits states the business logic will reject.

Distribute the product over the sum to reveal all four shapes, then delete the illegal ones. What remains is an enum listing only the legal shapes.

This is the mechanical procedure for finding hidden illegal states in an existing model.

## Types as a design feedback loop

A subtle but important point from Mark Seemann: types are not only a **correctness** mechanism, they are a **design feedback** mechanism. When you write a function signature and the inferred type "looks wrong", that is the compiler telling you the design is wrong, before you have written a line of implementation.

*"The type system implicitly provides a to-do list. There's no reason to keep such a list on a piece of paper on the side."* — Mark Seemann

Use this actively:

- Write the signature first. If it is hard to state clearly, the design is unclear.
- If the signature needs a `Bool` parameter that gates other parameters, you probably want a sum type.
- If the return type is `Unit` / `void` / `()`, ask what the function is actually telling its caller. If the answer is "nothing", it is probably doing I/O you want to name or lifting an invariant you want to return.
- If the signature has five arguments of which only three combinations are legal, push the illegal combinations out with a sum type.
- If you find yourself writing `// this can never happen` — the type is lying.

## Review Checklist

Before merging code that touches types, ask:

- [ ] Does every primitive-typed field that carries an invariant have a dedicated wrapper type? (`Email`, not `String`.)
- [ ] Are any optionals co-dependent? (If yes → sum type.)
- [ ] Are any `Bool` flags gating other fields? (If yes → sum type.)
- [ ] Does any array / string need to be non-empty? (If yes → `NonEmpty`.)
- [ ] Are IDs of different entities the same raw type? (If yes → phantom-typed `ID<T>` / `UserId newtype`.)
- [ ] Are fallible operations signalled in the return type (`Result` / `Either`) or hidden as exceptions?
- [ ] Does the state machine encode forbidden transitions in types, or only check them at runtime?
- [ ] Is parsing of external data concentrated at one boundary?
- [ ] Does any `switch` / `match` have a defaulted `unreachable!` / `fatalError("impossible")`? (If yes → the type admits states the logic rejects.)
- [ ] Do any functions downstream of validation re-validate the same thing?
- [ ] Do any function signatures return `Unit` / `void` / `m ()` where a meaningful type would fit?
- [ ] Does the function push constraints backward (restrictive argument types) or forward (permissive argument types + optional return)?

## Anti-Patterns to Call Out

- **Stringly-typed APIs** — `fn fetch(id: String)` where `id` is really a `UserId`.
- **Boolean blindness** — `fn new(isAdmin: bool, isActive: bool, isDeleted: bool)` should almost always be a sum type over meaningful states.
- **Optional soup** — a record whose fields are mostly optional is usually a badly-expanded enum.
- **Defensive guards in "impossible" branches** — either the branch is possible (and the type is wrong) or it isn't (and the guard is noise).
- **`unwrap` / `!!` / `try!` to paper over invariants the type doesn't express.**
- **`unreachable!` / `fatalError("impossible")` in exhaustive matches** — add the missing case or narrow the enum.
- **Returning `null` / `undefined` / `None` without a `Result` context** — the caller can't distinguish "no data yet" from "lookup failed" from "empty result".
- **Exception-driven control flow for expected failures** — invalid email during signup is not exceptional; encode it in the type.
- **Overcompacted records** — if two fields of a type must be set or unset together, they belong in a single variant of a sum type.
- **Cargo-culting "everything a newtype"** — wrapping a `String` that is displayed once and never compared carries no invariant, so wrapping it is noise.

## Limits and Trade-offs

Type-Driven Design has real costs. The skill is worth nothing if you apply it past the point of diminishing returns.

- **Cost of wrapping.** Every domain primitive is extra code. Reserve wrappers for values that carry invariants or cross API boundaries; don't wrap a string that is only displayed.
- **Decoding friction.** Parsing at the boundary means writing `Decodable` / serializer conformances by hand when the JSON shape doesn't match the domain shape. This friction is **intentional** — it is where illegal states get rejected.
- **Learning curve.** Phantom types, typestate, and GDP-style proof types are unfamiliar to most developers. Introduce them gradually and always with motivating failures.
- **Error messages.** The more the type system carries, the more cryptic the errors get when something goes wrong. Pair heavy type investment with good tooling (Rust Analyzer, Haskell Language Server, Sourcery, etc).
- **Logic errors survive.** Types prove that certain *shapes* of bug cannot occur. They do not prove your business logic is correct. Keep writing tests for algorithmic and behavioural correctness.
- **Third-party APIs.** You cannot force external SDKs to accept your wrappers. Convert at the edge and move on.
- **Over-modelling.** The goal is to capture *essential* domain invariants, not every conceivable property. If an "invariant" isn't actually enforced anywhere in the business logic, don't encode it.
- **Language ceilings.** Not every pattern ports cleanly. Phantom types are trivial in Haskell/Rust/Swift and awkward in Go; sum types need workarounds in Java < 17; dependent types remain impractical outside Idris/Agda/F*. Match your ambition to your tools.
- **Refactoring cost.** Heavily type-encoded designs are rigid — that is the point, but it also means that when the domain model changes you may face a larger refactor than an equivalent dynamically-typed design would.

## How to apply this skill

When writing or reviewing code in any statically-typed language:

1. **Identify the invariants the code is relying on.** Read `guard` / `assert` / `if` statements, comments that say "must be non-empty", force-unwraps, `unreachable!` branches, and `throw` sites for expected failures. Each of these is a potential type lift.
2. **Ask: can this invariant be lifted into a type?** For each one, pick the smallest pattern from the catalog that removes the runtime check.
3. **Work outward from the boundary.** Parse once at the edge, then propagate the parsed type. Don't try to type-drive the core of the system while leaving the boundary loose.
4. **Delete the now-dead runtime checks.** If you added a type and didn't delete any guards, you duplicated work instead of lifting it.
5. **Let the compiler guide the refactor.** Change a type, then fix every resulting compile error. That is the "implicit to-do list" Seemann describes.
6. **Stop when the next refinement stops paying.** Diminishing returns are real.

For the full pattern catalog with multi-language code examples, see [patterns.md](patterns.md). For the annotated source list, see [bibliography.md](bibliography.md).
