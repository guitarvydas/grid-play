
---

# Staged Computation: Whittling Down Source Code with t2t

**TL;DR** — Instead of writing one big compiler, build a pipeline of small text-to-text stages. Each stage simplifies the source code. The final stage is easy because all the hard cases have already been handled upstream.

---

**L;R**

A compiler doesn't have to be a single monolithic pass. It can be a pipeline of small, focused stages, each one inhaling source code and exhaling simpler source code. By the time the source reaches the final stage, the hard cases have already been eliminated. The downstream compiler only needs to handle one clean, normalized form.

This is staged computation. Each stage is a transmogrifier — it pattern-matches the input and rewrites it.

## t2t

`t2t` (text-to-text) is a small tool that applies a single transmogrification to its stdin and writes the result to stdout. It uses OhmJS twice: once to pattern-match the input, and once to rewrite it. Because each stage is a standalone unix filter, stages compose naturally into bash pipelines:

```bash
cat source.grid | t2t strexpander | t2t pythonize
```

Each stage is defined by two files:

- a grammar file (`.ohm`) — describes what to match
- a rewrite file (`.rwr`) — describes what to emit

The worked example is at https://github.com/guitarvydas/grid-play.

## RWR

RWR is a small DSL for writing rewrite rules. In a traditional OhmJS compiler, you write grammar rules in OhmJS syntax and then write Javascript functions to process the resulting parse tree. RWR replaces the bulk of that Javascript.

RWR is intentionally limited:

- **String rewriting only** — no full Javascript type system, no complex data structures
- **Very few operations** — simple string constants and interpolations
- **Parameter stacks** — for passing information down during the parse tree walk. The actual syntax is given in the RWR documentation.
- **One rewrite rule per grammar rule** — including one per branch of each `|` alternation
- **One level of iteration** — handles OhmJS `*`, `+`, `?` operators, but only one level deep; deeply nested iteration sometimes requires flattening the grammar rules first

For anything outside RWR's scope, there is an escape hatch: `support.mjs`. This is a plain Javascript file that the generated transmogrifier imports. It typically contains a handful of simple, one-line helper functions callable from RWR rules. In simple examples it is empty, but it must exist.

RWR transpiles to a Javascript module that is fully compatible with OhmJS's semantics processor. If you comment out the `rm -f temp.*` line in `@make`, the generated file (`temp.???.nanodsl.mjs`) is left on disk so you can inspect it directly. Note that a pipeline with multiple `t2t` invocations will produce one `nanodsl` file per stage — be careful to identify which file corresponds to which stage when debugging.

Full RWR documentation is at https://github.com/guitarvydas/pbp-dev/blob/main/t2td/doc/rwr/RWR%20Spec.pdf.

## A Worked Example

The example compiles a snippet of Grid (a new language by Denis Bredelet) into Python in two stages.

**Stage 1 — `strexpander`:** Rewrites `$"Hello, {first}!"` into `"Hello, " & first & "!"`. The `$`-string interpolation syntax is unwound into explicit `&` concatenation. The output is still valid Grid — just normalized, with the syntactic sugar removed.

**Stage 2 — `pythonize`:** The downstream compiler is easy to write because `strexpander` already handled the irregular case. It is not shown in detail here, to keep things simple — we simply pipe the normalized Grid output into a `pythonize` stage to show how this small example might be converted to working Python. Many semantic nuances of Grid have been elided to keep things simple. This is the core payoff of staged computation: each stage only has to solve one problem.

## Why Is This Important?

The goal is to make compilation 10x simpler. If building a DSL takes minutes instead of months, it changes how we approach projects.

We could build small, focused DSLs to help us think through and express the hard parts of a problem. We could use multiple little languages in a single project, each one suited to the sub-problem it addresses. This is already how we use REGEXs — Python supports them, Javascript supports them, and although they were originally developed for compiler work, they turned out to be useful everywhere. OhmJS, RWR, and PEG could be used the same way.

Drawing editors output XML and SVG. Both formats can be parsed directly by `t2t`. This opens the door to little languages with diagrammatic syntaxes — not just syntaxes inspired by 500 years of typesetting technology.

Instead of being constrained by what general-purpose language designers decided to implement for us, we could whip up small, project-specific languages based on what our project actually needs. This breaks us away from 20th century thinking about programming — one general-purpose language to rule them all — and opens up a 21st century approach: many small, special-purpose languages, composed together.

Programming languages are just tools that help us create machine code. In the 21st century, we can afford to expand our toolset well beyond the concept of the general-purpose programming language. We have the technology to build and compose many small, precise tools — we should use it.

## Reuse

The example is self-contained. The `./pbp` directory holds all the tools. To start your own project, copy `./pbp`, `@make`, and `@makec` into a new directory and build from there. A template repo (`pbp-sdk`) is also available.

A previous video walkthrough (slightly older tooling, which still works) is in the PBP Cookbook playlist: https://www.youtube.com/watch?v=EFTzFA82YRc&list=PLHh2_dCKBPjbBN2R8xwBiS4nHlo5iQjqS

