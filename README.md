
# t2t Example: Grid to Python

This example uses `t2t` (text-to-text transmogrification) to compile a snippet of Grid language source code into working Python. The goal is to show how a two-pass pipeline of simple text rewrites can serve as a lightweight compiler for a new language or as a prepass stage to lighten the load for a subsequent, downstream compiler pass.

## Background

Grid is a new language by Denis Bredelet (https://github.com/grid-lang/grid-lang.cc). It uses `[A1]`-style cell references and `$"..."` for string interpolation, similar to Python f-strings.

## The Pipeline

Compilation happens in two passes, expressed as a bash pipeline:

```bash
cat ${test}.grid | ${PBP}/t2t strexpander | ${PBP}/t2t pythonize
```

**Pass 1 — `strexpander`:** Expands syntactic sugar into normalized, verbose Grid. No `$`-strings after this pass — all string interpolation is unwound into explicit `&` concatenation.

**Pass 2 — `pythonize`:** Converts normalized Grid into Python, emitting calls to a small runtime library (`rtlib`).

Each pass is defined by two files: a grammar (`.ohm`) and a rewrite rule set (`.rwr`). You can read them directly in a text editor.

## Example

**Input** (human-written Grid):

```
: first = "Jane"
: last = "Doe"
[A1] := $"Hello, {first}!" & " Your surname is " & last
```

**After `strexpander`** (normalized Grid — no `$`-strings):

```
:first="Jane"
:last="Doe"
[A1] := "Hello, " & first & "!" & " Your surname is " & last
```

**After `pythonize`** (Python):

```python
first="Jane"
last="Doe"
import rtlib
rtlib.cellAssign("A", "1", "Hello, " + str(first) + "!" + " Your surname is " + str(last))
```

## How `pythonize` Handles String Concatenation

Grid's `&` operator covers four cases: string & string, string & variable, variable & string, variable & variable. Each case has an explicit rewrite rule. This is intentionally verbose — clarity is preferred over cleverness for a teaching example.

The output uses `rtlib` rather than Python f-strings, to show that string interpolation can be expressed as plain concatenation. This makes the example easier to port to another target language (e.g. JavaScript).

## The `xcontinue` Loop

`pythonize` uses a rewrite loop. Each time a rule rewrites part of the source, it calls `⎨xcontinue⎬`, which signals `t2t` to make another pass over the now-modified source. This is needed because `t2t` finds and applies only the first matching rule per pass — so rewrites that produce code matchable by other rules require subsequent passes to complete.

`⎨xcontinue⎬` returns the empty string, so it can be inserted anywhere in a rewrite without affecting output. Use it with care: any rule that fires on every pass will apply repeatedly. In this example, `import rtlib` is emitted from the `AssignCell` rule (which fires once) rather than from `main` (which fires on every pass) — otherwise `import rtlib` would appear multiple times in the output.

## Notes

This example cuts corners deliberately. It does not handle all Grid semantics, does not perform semantic analysis, and does not cover every string interpolation case. The focus is narrowly on showing how `t2t` rewrites can simplify a downstream compiler.

A real Grid compiler would add semantic analysis in the host language (JavaScript, given the use of OhmJS). The `t2t` tools handle syntax transformation only.

## Usage

```bash
./@install   # run once to install JS dependencies
./@make
```