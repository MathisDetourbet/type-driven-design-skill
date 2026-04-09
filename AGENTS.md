# Type-Driven Design

This repository provides a single language-agnostic AI coding-agent skill that teaches **Type-Driven Design**: lifting invariants into the type system so illegal states become compile-time errors rather than runtime bugs.

**Full skill content:** [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md)  
**Pattern catalog with multi-language code examples:** [`skills/type-driven-design/patterns.md`](skills/type-driven-design/patterns.md)  
**Annotated bibliography:** [`skills/type-driven-design/bibliography.md`](skills/type-driven-design/bibliography.md)

## When the skill applies

Any time you are writing, reviewing, or refactoring statically-typed code and you encounter:

- Domain model design (records, structs, enums, identity types)
- Validation logic returning `bool` or `void`
- Optionals / `bool` flags acting as implicit state machines
- Decoding external data (JSON, database rows, deep links, URL params)
- `unreachable!` / `fatalError("impossible")` / defaulted match arms
- Repeated emptiness / positivity / shape checks at every consumer
- Stringly-typed APIs carrying invariants the compiler cannot see
- State machines where the wrong method is legal on the wrong state
- Functions that return `Unit` / `void` / `()` where a meaningful type would fit
- Fallible operations signalled as thrown exceptions instead of `Result` / `Either`

...load `skills/type-driven-design/SKILL.md` for the core principles, review checklist, and anti-patterns, and `skills/type-driven-design/patterns.md` for the full catalog with code examples in **TypeScript, Rust, Swift, C#, Haskell, Kotlin, F#, Scala**.

## Core principles

1. **Parse, don't validate.** Preserve the proof of validation in the type.
2. **Make illegal states unrepresentable.** Model data so impossible states cannot be constructed.
3. **Push constraints backward, not forward.** Restrict your argument types rather than widening your return types.
4. **This is a spectrum, not a religion.** Match your investment to your language's type system, your ergonomics, and the lifespan of the code.

## Credit

All ideas are distilled from the work of Alexis King (*Parse, Don't Validate*), Matt Parsons (*Type Safety Back and Forth*), Matt Noonan (*Ghosts of Departed Proofs*), Alex Ozun (swiftology.io), Mark Seemann (blog.ploeh.dk), Dominic Codespoti (arinco.com.au), Ruggero Rebellato (ruggero.io), Mike Burns (thoughtbot.com), and Jaime González García (barbarianmeetscoding.com). Full bibliography in [`skills/type-driven-design/bibliography.md`](skills/type-driven-design/bibliography.md).
