# Type-Driven Design — Annotated Bibliography

All ideas, patterns, and examples in this skill are distilled from the sources below. The skill is an **index** into these works; they are the real thing.

## Foundational / canonical sources

### Alexis King — *Parse, Don't Validate* (2019)
<https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/>  
Language: Haskell. **The slogan, the thesis, and nine quotable principles** — the canonical modern statement of type-driven design. The Core Principles section of this skill is lifted verbatim from this article (with attribution). If you only read one source, read this one.

### Matt Parsons — *Type Safety Back and Forth* (2017)
<https://www.parsonsmatt.org/2017/10/11/type_safety_back_and_forth.html>  
Language: Haskell. Introduces the **push-forward vs. push-back** framing for where in a program to handle partial functions. The slogan: *"By restricting our past, we gain freedom in the future."* Essential companion to King.

### Matt Noonan — *Ghosts of Departed Proofs* (Haskell 2018)
<https://kataskeue.com/gdp.pdf>  
Language: Haskell. Formalizes the use of **phantom types as proof certificates**: compile-time invariants carried through the type system with zero runtime cost. The theoretical grounding for typestate and phantom-typed identities across all modern type-driven-design work.

---

## Practitioner sources (by language)

### Alex Ozun — *Type-Driven Design with Swift* (series, swiftology.io)
<https://swiftology.io/collections/type-driven-design/>  
Language: Swift. **A complete pattern catalog** applied to a mainstream mobile language, including smart constructors, sum types for contact methods / payments, `NonEmpty`, phantom-typed IDs, `Never` for impossible failures, and parse-at-the-boundary decoders. Highly recommended even if you don't write Swift.

Five-part series:
1. [Fundamentals of type-driven code](https://swiftology.io/articles/tydd-part-1-fundamentals)
2. [Type-safe validation](https://swiftology.io/articles/tydd-part-2)
3. [Witness pattern — type-safe access control](https://swiftology.io/articles/tydd-part-3)
4. [Domain modeling with types](https://swiftology.io/articles/tydd-part-4)
5. [Making illegal states unrepresentable](https://swiftology.io/articles/making-illegal-states-unrepresentable)

### Mark Seemann — *Type Driven Development* (2015, blog.ploeh.dk)
<https://blog.ploeh.dk/2015/08/10/type-driven-development/>  
Language: F#. Frames types as a **design feedback loop** rather than merely a correctness mechanism. Source of the *"types as an implicit to-do list"* framing used in this skill. Uses a polling-consumer state machine as a worked example.

### Dominic Codespoti — *Type-Driven Development for Robust Code* (2024, arinco.com.au)
<https://arinco.com.au/blog/type-driven-development-for-robust-code/>  
Language: C#. **Proves the patterns port to mainstream enterprise OO**, including `Result<T>` implemented as a record struct, `Email` as a value type with `Create` returning a `Result`, and the interface-based workaround for discriminated unions in C#. Re-cites Lambda and Parsons.

### Ruggero Rebellato — *Type-Driven Development Guide* (2025, ruggero.io)
<https://www.ruggero.io/blog/rust_type_driven_development_guide/>  
Language: Rust. **The most complete typestate treatment** (`Connection<Closed>` / `Connection<Open>` / `Connection<Authenticated>`), phantom types for units of measure, const generics for dimension-checked matrices, and an explicit Rust-vs-Haskell-vs-Scala-vs-TypeScript-vs-Idris comparison. Also includes empirical case studies (Microsoft / Google Android 70% memory-safety bug figure) that keep the skill grounded in reality, not theory.

### Mike Burns / thoughtbot — *Type-Driven Design, Test-Driven Design* (nuanced-tdd)
<https://thoughtbot.com/blog/nuanced-tdd>  
Languages: Rust vs. Ruby. The **meta-framing**: TyDD and TDD are complementary, not opposed. Which one "fits" depends on your language ecosystem. Source of the aphorism *"TDD purism belongs to the world of dynamic typing; being a TDD purist in Rust is a mistake, but being a type-driven designer in Ruby is also a mistake."*

### Jaime González García — *Type Drive Design* (notes, barbarianmeetscoding.com)
<https://www.barbarianmeetscoding.com/notes/type-drive-design/>  
Language-agnostic notes. A one-paragraph definition plus an excellent additional-resources list — this skill's bibliography was seeded from its "Additional Resources" section.

---

## Skeptical / balancing voices

### Anonymous forum post on the nebulousness of the term
Defines TyDD as *"lift as much of the logic of your program into your types as possible so that bugs and errors can be caught at type-check time instead of runtime"*, gives canonical examples (`SanitizedSqlString` vs `UnsanitizedSqlString`; state-as-type), and importantly notes that the term is not rigorously defined and that dependent types remain impractical outside research languages. The source of the **limits and trade-offs** framing in this skill.

---

## Further reading referenced but not directly cited

- **Edwin Brady — *Type-Driven Development with Idris*** (book, Manning 2017). The book that coined the phrase in its modern form. Demonstrates the technique at the dependent-types end of the spectrum. Worth reading to see where the ideas go when taken to their logical conclusion.
- **Yaron Minsky — *Effective ML: Make illegal states unrepresentable*** (blog.janestreet.com). The OCaml/ML tradition's statement of the same idea, predating most of the Haskell literature on the topic.
- **Language-Theoretic Security / LangSec** — *The Seven Turrets of Babel* (2016). Defines "shotgun parsing" as the antipattern that parse-don't-validate exists to fix.
- **Danielsson et al. — *Fast and Loose Reasoning is Morally Correct*** (Oxford). Theoretical underpinning for reasoning about Haskell programs as if they were total.
- **RustBelt** (Jung et al., POPL 2018), **Liquid Haskell** (Vazou et al., 2014), **Prusti / Creusot** (formal verification for Rust). Research directions pushing type-driven design further.

---

## Sources the skill could not fetch

- **Becoming Functional — *Type Driven Design in Elm*** (<https://becoming-functional.com/type-driven-design-in-elm-f8ad90e642aa>). Returned 404 at the time this skill was compiled. If a live mirror is found, the Elm perspective would round out the skill's coverage of the pure-FP-frontend end of the spectrum.

---

## How to cite this skill

This skill is a **compilation and index**, not an original work. When referencing an idea from this skill, cite the original source above, not the skill file. The one thing the skill adds is the cross-language pattern catalog in [patterns.md](patterns.md), which you are welcome to reuse with attribution to the skill and to the underlying sources.
