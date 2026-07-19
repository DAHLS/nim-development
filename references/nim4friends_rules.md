# Nim Notes — Rules

Persistent memory of Nim gotchas, idioms, and lessons learned from real
projects, maintained across many LLM sessions and models.

## Reading

Read this file in full — it is short and stable. For entries in
`nim4friends.txt`, `grep -n '^\['` lists every `[tag]` title with line
numbers for proactive awareness. For `[tags]` relevant to your task, read
those entries in full (use the line numbers to target them). When
debugging, grep by `[tag]` and error text. Titles are the awareness
layer; bodies are the detail layer.

## Adding an entry

1. **Evidence** — add only what you observed this session (an error, a
   wrong result, a crash) or verified by a targeted experiment. Never
   from docs or inference alone.
2. **Gate** — if you can't name the specific Nim API, type, or flag, you
   have a principle, not a lesson; principles (DRY, "test your math",
   "avoid hardcoded paths") don't belong here. (`[idiom]` entries must
   name a Nim construct AND a concrete consequence — warning, wrong
   result, compile failure, or footgun. "Cleaner code" alone is a style
   preference; reject.)
3. **Generalization** — strip project/domain references (file names,
   endpoints, business terms); preserve the Nim mechanism (library,
   signature, type, flag, literal error text — the error string is the
   next debugger's grep key).
4. **Shape** — `[tag]` one-line title → trap → symptom → fix, with the
   minimal snippet showing the fix. Aim for ≤15 lines.
5. **Placement** — grep for the API/flag first. If an entry on it exists,
   extend it by reference ("extends the X entry") instead of duplicating.
   Otherwise append at the end of `nim4friends.txt`. Position carries no
   meaning; the `[tag]` is the retrieval key, not layout.
6. **Versioning** — note the Nim version when behavior is version-dependent
   (e.g. "Nim 2.0+").

## Editing existing entries

- If an entry is factually wrong (the API, flag, or fix does not work as
  stated), correct it in place — accuracy beats append-only. Keep the
  correction minimal and factual; do not rewrite style.
- All other edits are forbidden: no reordering, reformatting, condensing,
  or deleting entries you merely disagree with.

## Categories (add new as needed)

| Tag | Description |
|---|---|
| `[build]` | Compile flags, build script issues |
| `[ambiguity]` | Name/symbol collisions between modules |
| `[pixie]` | pixie 2D graphics library |
| `[arraymancer]` | Arraymancer tensor library |
| `[nimhdf5]` | nimhdf5 HDF5 wrapper |
| `[times]` | std/times module |
| `[json]` | std/json module |
| `[http]` | std/httpclient module |
| `[os]` | std/os module |
| `[exn]` | Exception model and `except` handlers |
| `[idiom]` | Style/convention lessons |
| `[footgun]` | Silent surprises that cost a debug cycle |
| `[cli]` | CLI argument parsing |
| `[nimble]` | nimble package manager |
