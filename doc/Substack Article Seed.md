I would like to write a short Substack article that talks about this example and this technique.

The idea is to used "staged computation" to whittle down incoming source code in a way that makes the downstream compiler easier to manage. 

We create a pipeline of t2t (text to text) transmogrification stages that inhale source code and spit out valid source code, but simplified. 

T2t uses OhmJS twice
1. to pattern match the input source code
2. then, to rewrite the source code in some - hopefully simpler - way.

The rewrite rules are written in a little DSL syntax that I call '.rwr'. The RWR DSL is transpiled to Javascript code that is compatible with OhmJS's "semantic" processor.

In a traditional OhmJS compiler, you write grammar rules in OhmJS syntax, then write Javascript functions to process the resulting AST.

# RWR
RWR replaces the bulk of the Javascript manual work.
- string rewriting only - doesn't provide the full gamut of Javascript operations and data types
- very few operations - simple to write a DSL
- constants, interpolations, %parameter stacks for passing information *down* during the AST tree-walk.

- support.mjs
	- the generated transmogrifier leans on a javascript file called `support.mjs` to perform certain semantic actions, like creating stacks of names, etc.
	- this is often empty, or very simple Javascript, mostly 1-line functions that can be called from RWR
	- in this example, no support functions are needed, so support.mjs is empty (it needs to exist, though) 
- one rewrite rule for each grammar rule (and for each branch of each "|" branching grammar rule)
- stunted, one-level of "iteration"
	- OhmJS allows deeply nested iterations, but, for now, RWR handles only two cases 1. non-nested or 2. nested one-level deep `(+/*/?)`
		- handles most cases, but sometimes requires rewriting grammar rules that use deep iteration

If you comment out the `rm -f temp.*` line in `@make` the script will leave temp files lying around. The file `temp.???.nanodsl.mjs` contains the generated Javascript app that is the full transmogrifier that uses OhmJS for pattern matching and rewrites the input source.

# PBP Tools
This example is completely stand-alone and contains all of the required tools in `./pbp`.

If you want to build your own project using the tools, simply copy (recursively) the ./pbp directory and the `@make` and `@makec` scripts into your own project directory.

I intend to create a revised video showing how to use `pbp-sdk` as a template repo to build new projects. For now, a previous video, showing a slightly older way of doing this is in the 'PBP Cookbook' playlist: `https://www.youtube.com/watch?v=EFTzFA82YRc&list=PLHh2_dCKBPjbBN2R8xwBiS4nHlo5iQjqS`

---

need to add references to the example repo and to RWR documentation

example repo: https://github.com/guitarvydas/grid-play

RWR documentation: https://github.com/guitarvydas/pbp-dev/blob/main/t2td/doc/rwr/RWR%20Spec.pdf

---

> **Parameter stacks** — for passing information down during the parse tree walk **(`%push`, `%pop`, `%get`)**

becomes

> **Parameter stacks** — for passing information down during the parse tree walk.

The actual syntax for these stacks is given in the RWR documentation.


---

# Why Is This Important?

A goal is to make compilation 10x simpler. This might change the way that we work on projects.

We could, for example, build little DSLs to help us think through and express hoary parts of the problem space.  If we believed that building a DSL is "easy" and takes minutes instead of months, and generates code in whatever language we need, we might begin using multiple little languages in our project.

This would be like using REGEXs in our code. Python supports REGEXs. Javascript supports REGEXs. REGEXs were originally developed for compiler development, but turned out to be useful for all sorts of things.

OhmJS, RWR, PEG, etc. could be used in this way, too. Instead of using someone else's ideas about what DSLs and PLs we need, we could whip up little languages for our projects based on what our project needs instead of what someone else decided to implement for us.

This would break us away from 20th century ideas about using only general purpose languages and vault us into the 21st century building special purpose languages.

---

> left on disk so you can inspect it directly.

add something like: if you do try to capture and inspect the intermediate code, remember that there will be one `nanodsl` file for each `t2t` invocation in a pipeline - you need to be cognizant which version is applicable to which part of the pipeline.

---

> The downstream compiler (`pythonize`) is easy to write ...

The downstream compiler is easy to write. The downstream compiler is not shown in this example, to keep things simple. We simply pipe the output into a `pythonize` stage to show how this small, simple example might be converted to working Python (again, many semantic nuances of Grid have been elided to keep things simple).

---

> slightly older tooling

which still works

---
---

Add to why this is important: Programming languages are just tools that help us create machine code. In the 21st century, we can afford to expand our toolset to go beyond simple concepts like general purpose programming languages.


---
---
Drawing editors use XML and SVG. Output from these editors can be directly parsed by `t2t`. This opens the door to making little languages that have diagrammatic syntaxes instead of syntaxes solely inspired by 500-year typesetting technology.

