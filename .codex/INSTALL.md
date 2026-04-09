# Installing type-driven-design for Codex

Enable the Type-Driven Design skill in Codex via native skill discovery. Just clone and symlink.

## Prerequisites

- Git
- Codex CLI

## Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/MathisDetourbet/type-driven-design-skill.git \
     ~/.codex/type-driven-design-skill
   ```

2. **Create the skill symlink:**

   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/type-driven-design-skill/skills/type-driven-design \
     ~/.agents/skills/type-driven-design
   ```

   **Windows (PowerShell):**

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\type-driven-design" `
     "$env:USERPROFILE\.codex\type-driven-design-skill\skills\type-driven-design"
   ```

3. **Restart Codex** to discover the new skill.

## Verify

```bash
ls -la ~/.agents/skills/type-driven-design
```

You should see a symlink (or junction on Windows) pointing to the cloned skill directory.

Then ask Codex: *"Review this type definition using type-driven-design principles."* The skill should auto-load from its description.

## Updating

```bash
cd ~/.codex/type-driven-design-skill && git pull
```

## Uninstalling

```bash
rm ~/.agents/skills/type-driven-design
rm -rf ~/.codex/type-driven-design-skill
```
