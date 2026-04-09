# Type-Driven Design

This extension provides language-agnostic guidance on **Type-Driven Design** — using the type system to make illegal states unrepresentable, parse rather than validate, and eliminate defensive runtime checks. Applies to TypeScript, Rust, Swift, C#, F#, Haskell, Kotlin, Scala, and any language with a meaningfully expressive type system.

## When to apply

Load [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md) whenever you are writing, reviewing, or refactoring statically-typed code and you see:

- Primitive-typed fields (`string`, `int`, `UUID`) carrying invariants the compiler cannot see
- Types that use optionals or `bool` flags as implicit state machines
- `Result` / `Option` absent in signatures for operations that can fail
- Validation logic returning `bool` or `void`
- Code that parses external data (JSON, database rows, deep links, URL params)
- `unreachable!` / `fatalError("impossible")` in switch / match defaults
- Repeated emptiness / positivity / shape checks at every call site
- Stringly-typed IDs (`UserId = string`) passing through many layers
- State machines where the wrong method is legal on the wrong state

## Core principle

> *Parse, don't validate.*  
> *Make illegal states unrepresentable.*  
> *Push constraints backward, not forward.*

Information acquired through validation must be preserved by the type system, not discarded. Instead of trusting yourself to check the right things at the right places, arrange your types so that the wrong things cannot be written at all.

## Reference files

- [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md) — core principles, checklist, anti-patterns, limits
- [`skills/type-driven-design/patterns.md`](skills/type-driven-design/patterns.md) — full pattern catalog with code examples in TypeScript, Rust, Swift, C#, Haskell, Kotlin
- [`skills/type-driven-design/bibliography.md`](skills/type-driven-design/bibliography.md) — annotated list of the sources this skill is distilled from

## Credit

This skill is an index into the work of Alexis King, Matt Parsons, Matt Noonan, Alex Ozun, Mark Seemann, Dominic Codespoti, Ruggero Rebellato, Mike Burns, and Jaime González García. Full attribution lives in the bibliography. When citing an idea from this skill, cite the original source, not this file.
