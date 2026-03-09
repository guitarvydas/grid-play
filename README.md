This is a quickie example. I'm cutting corners, e.g. I don't handle multiple interpolations in one string, I don't handle Python indentation (there's a way to do this, but I want to elide nuances for now (I can show this later)), ....

# usage
`./@make`

# notes
The objective of this stage is to rewrite shorthand .grid code to long-hand .grid code. This makes the .grid compiler simpler (it only needs to handle one verbose case for string handling)

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

pythonize:
```
first = "Jane"
last = "Doe"

rtlib.cellAssign (A1, "Hello, " + str(first) + "!" + " Your surname is " + last)
```

I'm not using Python F-strings, since, someday, you might want to target some other language.
