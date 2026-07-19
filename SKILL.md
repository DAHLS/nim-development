---
name: nim-development
description: "Nim development decisions and how to find current, version-correct Nim info. Use when writing, building, or debugging Nim (.nim files, nimble, compile flags, mm/GC choices, C FFI, Nim compiler errors), or when unsure whether a Nim API or idiom is current."
risk: safe
---

# Nim Development

> Decision-making for Nim, plus a research discipline for a small language
> with sparse, version-fragmented docs.
> **Verify against primary sources — do not trust stale priors.**

## When to Use

Use when a task involves Nim: authoring `.nim` code, `nimble` packaging,
compile flags, memory-management choices, C interop, or diagnosing compiler
and runtime errors.

**First, always read `references/nim4friends.txt`** — the authoritative log of
Nim traps already learned (build flags, exception model,
pixie/arraymancer/nimhdf5, times/json/http/os, idioms, footguns). Grep it by
`[category]` tag or error text. This skill does **not** repeat those entries;
it complements them with decision-making and a way to learn *new* traps
correctly.

**And, non-negotiably, feed lessons back into it** — see
[Recording lessons](#recording-lessons-mandatory). The value of this file
compounds only if every session that learns something writes it down.

---

## ⚠️ Why Nim needs a research discipline

Nim is small and its training-data footprint is thin, so model priors on Nim
are often **wrong or version-stale**. Behavior changed materially across
versions (e.g. `CatchableError` is 2.0+, `--mm:orc` became the default in 2.0,
the exception hierarchy was refactored). Never answer a Nim API question from
memory alone — anchor to the version and confirm against a primary source.

### 1. Anchor to the version first

```
nim --version      # this machine: Nim 2.2.4
```

Nim docs are versioned; an idiom that is correct on `devel` may not compile on
the installed compiler, and vice-versa. State the target version before
committing to an API.

### 2. Source hierarchy (most authoritative first)

1. **Official versioned docs**
   - Manual: <https://nim-lang.org/docs/manual.html>
   - Stdlib index: <https://nim-lang.org/docs/lib.html>
   - Per-module: `https://nim-lang.org/docs/<module>.html` (e.g. `/docs/times.html`)
   - Docs index / other versions: <https://nim-lang.org/documentation.html>
2. **The library's own source + `tests/` + `examples/` on GitHub.**
   For sparse-doc libraries (pixie, arraymancer, nimhdf5) the source *is* the
   documentation — the `tests/` directory shows real, compiling API usage.
   Read it before guessing.
3. **Version diffs & community**
   - Nim changelog (release notes / migration): the `changelog.md` in
     `nim-lang/Nim` on GitHub.
   - Forum: <https://forum.nim-lang.org/>
4. **GitHub code search** across `.nim` files for real-world usage patterns
   when docs are silent on an idiom.

### 3. Package discovery

```
nimble search <term>       # CLI is primary — reliable
nimble install <pkg>
```

`nimble.directory` (the web UI) is frequently down (502) — don't depend on it.
Fallback package index: the `packages.json` in `nim-lang/packages` on GitHub
(raw: `raw.githubusercontent.com/nim-lang/packages/master/packages.json`).

### 4. Local fallback when online docs are missing

```
nimble path <pkg>          # locate installed package source, then read it
nim doc <file.nim>         # generate HTML docs from source
nim jsondoc <file.nim>     # machine-readable API dump
```

### 5. Verify, then record

Confirm the API/flag against a primary source before trusting it. When you
discover a new trap, record it — see [Recording lessons](#recording-lessons-mandatory).
This closes the loop: the memory file stays the trap log, this skill stays the
method.

---

## Recording lessons (MANDATORY)

**This is the single most important habit in this skill.** Nim's docs are thin;
`references/nim4friends.txt` is the only thing that stops the next session from
re-hitting a trap you already paid for. A lesson learned and not written down is
wasted.

### When to record — if ANY of these happened this session, you MUST append

- A Nim **compile error or runtime crash** you had to diagnose.
- A result that was **silently wrong** (parsed to zero, dropped rows, off-by-N).
- A **version-specific behavior** you confirmed (e.g. differs across 1.x/2.x).
- A **library/API surprise** where the obvious usage was wrong and you found the
  right one by reading source/tests.

If none of the above occurred, there is nothing to record — do not invent
entries.

### How to record

Append to `references/nim4friends.txt` following **that file's own ADDING
rules** (read its header before writing). In short:

1. **Evidence only** — something you observed or verified this session, never
   from docs or inference alone.
2. **Gate** — you must be able to name the specific Nim API, type, or flag; a
   vague principle is not a lesson.
3. **Generalize** — strip project/file/business names; keep the Nim mechanism
   and the literal error string (it is the next debugger's grep key).
4. **Shape** — `[tag] one-line title → trap → symptom → fix`, ≤15 lines, with
   the minimal snippet showing the fix.
5. **Placement** — grep for the API/flag first; extend an existing entry instead
   of duplicating. Correct an entry in place if it is factually wrong.

Do **not** reorder, reformat, condense, or delete entries you merely disagree
with — the file forbids it.

### Close the loop (this repo)

`references/nim4friends.txt` lives in a git repo. After appending, commit and
push so the lesson reaches your other machines:

```
git -C ~/.config/opencode/skills/nim-development add references/nim4friends.txt
git -C ~/.config/opencode/skills/nim-development commit -m "nim4friends: <what you learned>"
git -C ~/.config/opencode/skills/nim-development push
```

---

## Memory management (`--mm:`)

Decide deliberately; the default changed in 2.0.

```
--mm:orc   → default in Nim 2.0+. ARC + cycle collector. Use for general code
             (graphs, closures, anything that can form reference cycles).
--mm:arc   → deterministic, no cycle collector. Lowest overhead; use when you
             know there are no cycles (or break them with `weak`/manual).
--mm:refc  → legacy tracing GC. Only for old code that depends on its behavior.
--mm:none  → manual. Only for embedded / no-runtime targets.
```

Rule of thumb: leave `orc` unless you have a measured reason. Match the `--mm`
across all compilation units of a project.

## Project structure & nimble

```
Small (script / one tool):
  main.nim
  project.nimble

Library or app:
  src/<pkg>.nim          # entry module, same name as the package
  src/<pkg>/*.nim        # submodules
  tests/t*.nim           # testament picks up test files
  project.nimble
```

- Split into modules once a file grows past ~500 lines (see the `[idiom]`
  module-split entry in `nim4friends.txt`): cleaner boundaries, per-file
  compile caching, no accidental reach into private state.
- The `.nimble` file declares `requires`, `bin`, `srcDir`; `nimble build`,
  `nimble test`, `nimble install` drive it.

## Concurrency — pick the model

```
async / await (std/asyncdispatch or chronos)
  → I/O-bound: many sockets, HTTP, timers on one thread. Single-threaded
    cooperative. chronos is the more actively developed alternative.

threads + channels (std/threads, --threads:on default in 2.x)
  → CPU-bound parallelism or truly independent workers. Share via `Channel`
    or `--mm:orc` isolated refs; avoid sharing mutable GC'd refs across threads.

std/threadpool / malebolgia / weave / taskpools
  → data-parallel workloads; prefer a maintained lib over legacy threadpool.
```

Don't reach for threads to solve an I/O-bound problem — use async.

## Error handling

Standardize handlers on `except CatchableError as e:` — in Nim 2.0 `Defect`
became a child of `Exception`, so `except Exception` swallows programmer-error
crashes you'd rather let propagate. See the `[exn]` entry in
`nim4friends.txt` for the full rationale and the 1.x caveat.

## Build / compile decisions

```
(default)     debug: all checks, no C optimization. Development.
-d:release    C optimization, KEEPS runtime checks (bounds/overflow).
              Correct default for unattended/production code parsing untrusted input.
-d:danger     strips all checks + release. Maximum speed, no safety net.
--checks:off  strips checks without the rest of release.
```

Note: `-d:release` in Nim does **not** disable checks (unlike Rust's
`--release`). For cross-compilation and cache pitfalls, see the `[build]`
entries in `nim4friends.txt` before touching `--cpu`/`--passC`/`--nimcache`.

## C FFI / interop

- Wrap C symbols with `{.importc, header: "foo.h".}`; pass includes/libs via
  `{.passC.}` / `{.passL.}` pragmas or in the `.nimble`/`nim.cfg`.
- For a small, stable C API: write the pragmas by hand.
- For a large header: consider `c2nim` or `futhark` to generate bindings, then
  read and trim the output.
- Match calling convention (`{.cdecl.}`) and struct layout; verify with a tiny
  round-trip test before building on the binding.

## Testing & style (CI gates)

```
nim c -r tests/tfoo.nim        # unittest suites
testament pattern "tests/t*"   # official test runner (categories, spec comments)
nimpretty --verify src/*.nim   # authoritative formatting gate
nim check --styleCheck src/... # naming conventions only
```

`nimpretty --verify` and `nim check --styleCheck` catch **different** things —
run both. See the `[idiom]` style entry in `nim4friends.txt`.

## Anti-patterns

### ❌ DON'T
- Answer a Nim API question from memory without checking the version/docs.
- Rely on `nimble.directory` (web) — use `nimble search`.
- Use `except Exception` (swallows `Defect` in 2.0).
- Assume `-d:release` removes bounds/overflow checks.
- Use threads for an I/O-bound problem.
- Guess a sparse-doc library's API instead of reading its `tests/`.

### ✅ DO
- `nim --version` first; pin the target version.
- Read `references/nim4friends.txt` before starting; **record new traps in it
  (and commit/push) before finishing** — this is mandatory, not optional.
- Read library source/tests when docs are thin.
- Choose `--mm`, concurrency model, and build flags for the actual context.

## Decision checklist

- [ ] Read `nim4friends.txt` for relevant `[tags]`?
- [ ] Confirmed the target Nim version?
- [ ] Verified any uncertain API against a primary source (docs/source/tests)?
- [ ] Chosen `--mm` deliberately (not by accident)?
- [ ] Picked the right concurrency model (I/O vs CPU)?
- [ ] Selected build flags for this context (release keeps checks)?
- [ ] Handlers use `except CatchableError`?
- [ ] Ran `nimpretty --verify` and `nim check --styleCheck`?
- [ ] **Recorded any newly-learned trap** in `references/nim4friends.txt` and committed/pushed it? (mandatory if you hit an error, a silent-wrong result, or a version-specific behavior — see [Recording lessons](#recording-lessons-mandatory))

## Limitations

- Nim evolves; treat any specific API here as needing confirmation against the
  installed compiler's docs/source (that is the whole point of this skill).
- Does not replace `nim4friends.txt` — that file is the authoritative,
  evidence-based trap log; this skill is the method and the decisions.
- Does not authorize destructive or environment-changing actions without
  validation against the user's real sources.
