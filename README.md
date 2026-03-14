This is a short example of using t2t (text-to-text transmogrification) to convert a snippet of a new language design into working Python code and how this can simplify writing a "compiler" for such a new language.

The new language is called "grid". ( Denis Bredelet https://github.com/grid-lang/grid-lang.cc )

You don't need to understand what `strexpander.grid` does nor its syntax. The point of this example is to show how to map a non-traditional syntax into valid Python. "$" strings in `grid` have a syntax similar to that of Python F-strings, where `{...}` causes string interpolation. (Grid's semantics are somewhat different for what actually happens here, but, you don't need to understand those kinds of nuances to get the point of this example).

The objective of this stage is to rewrite shorthand .grid code to long-hand .grid code. This makes the downstream .grid compiler simpler. It only needs to handle only one case, albeit verbose from users' perspective, for string handling. After transmogrification, we can "compile" the new language (without syntactic sugar niceties) into Python.

This is kind of like "macros" that transform syntax of a language into the same language, but simpler to "compile" ("interpret").

The first step - simplification of the language - is seen in `@makec` as `${PBP}/t2t strexpander`. This uses the `t2t` tools with `stexpander.ohm` and `strexpander.rwr`. The `strexpander.ohm` file is a grammar (in OhmJS syntax) while the `strexpander.rwr` is a set of rewrite rules with one set of rewrite rules corresponding to each grammar rule. You can examine the grammar and rewrite rules by opening the files in a text editor.

The second step - rewriting the simplified `.grid` code into Python - is seen in `@makec` as `${PBP}/t2t pythonize`. This uses the grammar `pythonize.ohm` and the rewrite rule set `pythonize.rwr`. The only complication is that I handle the dissectiondeconstruction of string concatenation in 4 pieces (1. string & string, 2. variable & string, 3. string & variable, 4. variable & variable) [_There are many other ways to solve this, I chose to be very explicit to keep this example "simple" and "readable"._]

The whole process is seen as a simple `bash` pipeline in `@makec`
```
cat ${test}.grid | ${PBP}/t2t strexpander | ${PBP}/t2t pythonize
```

1. push the source code as a big text string into the front of the pipeline (`cat ...`), 
2. then macro-process using the `strexpander` step, 
3. then convert to Python using the `pythonize` step.

Usually, I draw this as a diagram (see the appendix) but in this simple example, there are no feedback loops nor fan-out. The pipeline is a simple sequential pipeline which can be expressed in `bash` textual syntax. To keep things simple, I chose to write this example as a `bash` script instead of as a full PBP diagram.

# The Transmogrifications:

Human-writable Grid:
```
: first = "Jane"
: last = "Doe"
[A1] := $"Hello, {first}!" & " Your surname is " & last
```

verbose, but, normalized Grid code:
```
:first="Jane"
:last="Doe"
[A1] := "Hello, " & "str(first)" & "!" & " Your surname is " & last
```

Note that this version is still `.grid` code. It uses "`&`" for string concatenation an the `[A1]` syntax. The difference is that this code does *not* use `"$"` syntax. String interpolation has been unwound into simple string concatenation of strings and variables. This version might be considered "verbose" from a human perspective, but is machine-readable and normalized. 

pythonize:
```
first = "Jane"
last = "Doe"

rtlib.cellAssign ("A1", "Hello, " + str(first) + "!" + " Your surname is " + last)
```

I'm not using Python F-strings, since, someday, you might want to target some other language.

[_This is but a quickie example. I'm cutting corners, e.g. I don't handle all of the nuances of the new language (like multiple string interpolations in the same string). The focus here is on how to use transmogrification to simplify the work in downstream passes._]

With respect to creating a new language, you would probably want to do semantic analysis before spitting out Python - I don't do that here, to maintain clarity, focussing only on the use of the t2t tools to show how to transmogrify text code into other text code. The stock `t2t` tools don't help much with semantic analysis, anyway - semantic analysis needs to be done mostly with helper routines in the host language (say, Javascript in this case (not Python, Javascript, due to the use of OhmJS and the way the rest of the rwr stuff is written). Including such details would have made this example difficult to read.

# usage
run `./@install` once to install various JS libs
`./@make`

# notes
