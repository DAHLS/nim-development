# nim-development

An [OpenCode](https://opencode.ai/) skill for writing, building, and debugging
**Nim**, plus a research discipline for finding *current, version-correct* Nim
information — important because Nim's training-data footprint is thin and its
behavior changed materially across versions.

## What's in here

| Path | Purpose |
|------|---------|
| `SKILL.md` | Decision-making guidance (memory model, project/nimble structure, concurrency, error handling, build flags, C FFI, testing) and the verify-first research method. Auto-invoked by OpenCode on Nim tasks. |
| `references/nim4friends_rules.md` | Reading, adding, and editing rules for the trap log. Read in full before using `nim4friends.txt`. |
| `references/nim4friends.txt` | The entries — a persistent, evidence-based log of Nim traps learned across sessions (build flags, exception model, pixie/arraymancer/nimhdf5, times/json/http/os, idioms, footguns). See `nim4friends_rules.md` for rules. |
| `LICENSE` | GPL-3.0. |

## How it works in OpenCode

OpenCode reads `SKILL.md` natively from the `skills/` directory. The skill
triggers automatically when a task involves Nim (`.nim` files, `nimble`, compile
flags, Nim errors). `references/nim4friends_rules.md` is read in full;
`references/nim4friends.txt` is grepped on demand, not loaded into context
unless something opens it.

The companion OpenCode Skills Collection plugin only manages
`*-category-pointer` folders — it never touches this folder.

## Install / sync on another machine

Clone directly into the OpenCode skills directory:

```bash
git clone https://github.com/DAHLS/nim-development.git \
  ~/.config/opencode/skills/nim-development
```

## Recording lessons (the important part)

`references/nim4friends.txt` compounds in value only if every session writes
down what it learned. When you hit a Nim error, a silently-wrong result, or a
version-specific behavior, append the lesson **per the rules in
`references/nim4friends_rules.md`**, then commit and push so it reaches other
machines:

```bash
cd ~/.config/opencode/skills/nim-development
git add references/nim4friends.txt references/nim4friends_rules.md
git commit -m "nim4friends: <what you learned>"
git push
```

Never reorder, reformat, condense, or delete entries you merely disagree with.
Correct factually-wrong entries in place.

## Maintaining this skill

- **Validate after frontmatter edits** — run the official checker from
  `agentskills/agentskills`:
  `skills-ref validate ~/.config/opencode/skills/nim-development`
- **`risk:` frontmatter field** is a local house convention (every skill in
  `~/.config/opencode/skills/` carries it). Spec-compliant loaders ignore
  unknown fields. If this skill is ever published, move it under the spec's
  `metadata:` mapping.
- **Smoke-test after SKILL.md changes** — run a fresh agent session on a
  small Nim task and watch whether the guidance lands. A prompt that has
  worked: "plan a Nim CLI that takes a date (YYYY/MM/DD) and prints seconds
  from now to that date, leap-year correct". Check: rules file read first?
  Known traps avoided on the first attempt? New lessons appended and pushed?

## License

GPL-3.0. See `LICENSE`.
