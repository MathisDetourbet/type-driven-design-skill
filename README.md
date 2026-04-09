# Type-Driven Design — A Language-Agnostic AI Coding-Agent Skill

A skill for [Claude Code](https://code.claude.com/), Cursor, Codex, OpenCode, Gemini CLI, and any other AI coding agent, that teaches the agent to apply **Type-Driven Design** when writing and reviewing code: lifting invariants into the type system so illegal states become compile-time errors rather than runtime bugs.

Applies to **TypeScript, Rust, Swift, C#, F#, Haskell, Kotlin, Scala**, and any language with a meaningfully expressive type system.

## What it does

Once installed, the agent will automatically reach for type-driven patterns when you:

- Design domain models (records, structs, enums, identity types)
- Write validation logic or failable constructors
- Review code that uses optionals or `bool` flags as implicit state machines
- Decode external data (JSON, database rows, deep links, URL params)
- Model state machines or resource lifecycles
- Refactor stringly-typed APIs or deeply-nested optionals
- Mention "parse don't validate", "make illegal states unrepresentable", "smart constructors", "newtype pattern", "branded types", "phantom types", "typestate", "algebraic data types", "discriminated unions", or any of their variants

The skill includes:

- **Core principles** — the nine rules from Alexis King's *Parse, Don't Validate* plus Matt Parsons' tenth on pushing constraints backward, quoted verbatim with attribution.
- **A pattern catalog** with concrete code examples in TypeScript, Rust, Swift, C#, Haskell, Kotlin, and F#: domain primitives / newtypes, sum types, parse-don't-validate, `Result`/`Either`, `Option`/`Maybe`, phantom types, typestate, `NonEmpty`, parse-at-the-boundary.
- **A review checklist** and a list of anti-patterns.
- **A deliberate limits-and-trade-offs section** — this skill is a spectrum, not a religion.
- **An annotated bibliography** pointing back to the original sources.

## Credit

This skill is an **index into existing work**, not an original contribution. All ideas, patterns, and examples are distilled from:

- **Alexis King** — [*Parse, Don't Validate*](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) (the canonical modern statement of the thesis)
- **Matt Parsons** — [*Type Safety Back and Forth*](https://www.parsonsmatt.org/2017/10/11/type_safety_back_and_forth.html)
- **Matt Noonan** — [*Ghosts of Departed Proofs*](https://kataskeue.com/gdp.pdf) (Haskell 2018)
- **Alex Ozun** — [Type-Driven Design with Swift](https://swiftology.io/collections/type-driven-design/) (swiftology.io)
- **Mark Seemann** — [Type Driven Development](https://blog.ploeh.dk/2015/08/10/type-driven-development/) (ploeh.dk, F#)
- **Dominic Codespoti** — [Type-Driven Development for Robust Code](https://arinco.com.au/blog/type-driven-development-for-robust-code/) (Arinco, C#)
- **Ruggero Rebellato** — [Type-Driven Development Guide](https://www.ruggero.io/blog/rust_type_driven_development_guide/) (ruggero.io, Rust)
- **Mike Burns / thoughtbot** — [*Type-Driven Design, Test-Driven Design*](https://thoughtbot.com/blog/nuanced-tdd)
- **Jaime González García** — [Type Drive Design notes](https://www.barbarianmeetscoding.com/notes/type-drive-design/)

Full annotated bibliography in [`skills/type-driven-design/bibliography.md`](skills/type-driven-design/bibliography.md). If you find the skill useful, **go read the originals** — they are the real thing.

A language-specific companion skill focused on Swift is available at [swift-type-driven-design-skill](https://github.com/MathisDetourbet/swift-type-driven-design-skill).

## Installation

Installation differs by AI agent. Claude Code and Cursor have plugin systems; Codex, OpenCode, and Gemini use clone-and-configure.

### Claude Code (plugin marketplace)

```
/plugin marketplace add MathisDetourbet/type-driven-design-skill
/plugin install type-driven-design@type-driven-design-marketplace
```

Update later with:

```
/plugin marketplace update type-driven-design-marketplace
```

### Cursor (plugin)

Install directly from the repo via Cursor's plugin system — point it at this GitHub URL. Cursor reads `.cursor-plugin/plugin.json`, which registers the skill from `skills/`.

### Codex

See [`.codex/INSTALL.md`](.codex/INSTALL.md) — a git clone + symlink into `~/.agents/skills/`. Or tell Codex:

> Fetch and follow instructions from https://raw.githubusercontent.com/MathisDetourbet/type-driven-design-skill/main/.codex/INSTALL.md

### OpenCode

See [`.opencode/INSTALL.md`](.opencode/INSTALL.md) — clone the repo and add its `skills/` path to `skills.paths` in your `opencode.json`. Or tell OpenCode:

> Fetch and follow instructions from https://raw.githubusercontent.com/MathisDetourbet/type-driven-design-skill/main/.opencode/INSTALL.md

### Gemini CLI

This repo ships a Gemini extension ([`gemini-extension.json`](gemini-extension.json) + [`GEMINI.md`](GEMINI.md)). Install with:

```bash
gemini extensions install https://github.com/MathisDetourbet/type-driven-design-skill
```

### Generic (any agent honoring `AGENTS.md`)

Many agents auto-read `AGENTS.md` from the repository root. Clone the repo into your project (or next to it) and the agent will pick up the [`AGENTS.md`](AGENTS.md) pointer, which in turn references [`skills/type-driven-design/SKILL.md`](skills/type-driven-design/SKILL.md).

### Manual install (no plugin system)

Drop the skill files directly into your user skills directory:

```bash
mkdir -p ~/.claude/skills/type-driven-design
curl -fsSL https://raw.githubusercontent.com/MathisDetourbet/type-driven-design-skill/main/skills/type-driven-design/SKILL.md \
  -o ~/.claude/skills/type-driven-design/SKILL.md
curl -fsSL https://raw.githubusercontent.com/MathisDetourbet/type-driven-design-skill/main/skills/type-driven-design/patterns.md \
  -o ~/.claude/skills/type-driven-design/patterns.md
curl -fsSL https://raw.githubusercontent.com/MathisDetourbet/type-driven-design-skill/main/skills/type-driven-design/bibliography.md \
  -o ~/.claude/skills/type-driven-design/bibliography.md
```

### Verifying

After installing through any path, ask the agent:

> Review this type definition using type-driven-design principles.

The skill should auto-load from its description and produce concrete suggestions grounded in the pattern catalog.

## Contributing

Issues and PRs welcome, especially for:

- **Additional languages** in the pattern catalog (Kotlin, Scala, F#, and Elm are currently underrepresented)
- **Real-world "we went too far with types" stories** for the Limits section
- **Missing canonical sources** — if I'm citing someone, I want to get it right
- **Broken or moved links** in the bibliography
