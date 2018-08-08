# LITPROG TEXT FORMAT

  - [NOWEB Syntax](#noweb-syntax)
  - [LITPROG Syntax](#litprog-syntax)
  - [Parsing LITPROG](#parsing-litprog)

`litprog` parses literate programs from plain text files.
The syntax used by `litprog` is a near superset of the syntax used by
[`noweb`](https://www.cs.tufts.edu/~nr/noweb/).
This document first discusses the `noweb` syntax, then highlights the
differences and extensions used for `litprog`.
Finally, pseudo code for a parser is given.

## NOWEB Syntax

For **Documentation Chunks:**

  - `@` in column 1 of a line by itself starts a documentation chunk.
  - Text is in LaTeX; use `[[…]]` for embedded code fragments.

For **Code Chunks:**

  - `<<…chunk name…>>=` in column 1 of a line by itself starts a code
    chunk.
  - Code is in any language; use `<<…>>` to include other code chunks by
    reference.
  - *You can write multiple code chunks with the same name*; they are
    concatenated.

Here is an example document conforming to `noweb` syntax:

```
The arms of the case statement have some extra information.
The file and line number help with error message and make it possible
to generate [[#line]] statements that identify the source of the code.
The original arm gives the arm from which the current arm is derived,
and is useful for many of the heuristics.
<<*>>=
record caserec(arms,valcode,trailer)
		# case arms, code to compute value, trailing code
record arm(file, line, pattern, code, original)
		# pattern and code are the content
		# line, file, original(pattern) are used for error reporting
@
Each node of the decision tree is associated with a particular case
statement.
Internal nodes have children, and a [[field]] which says which field
we decided to test on.  The edges that point to the children record
the interval of values for the particular child.
Leaf nodes have a [[name]] that records the name of the pattern known
to match at that leaf node.
<<*>>=
record node(cs, children, field, name)
	# case statement, list of edges to children, field chosen, pattern name
	#	(name field used to support name operator, assigned only to leaves)
record edge(node, lo, hi)
	# node pointed to and lo and hi interval of field for this edge
@
To create a decision tree, I begin with a node containing the full,
original case statement.  I then use a ``work queue'' approach to check
each node and see if it needs to be split.
```

## LITPROG Syntax

For **Documentation Chunks:**

  - `@` in column 1 of a line by itself starts a documentation chunk.

For **Code Chunks:**

  - `<<…chunk name…>>=` (or `<-<…chunk name…>->`, `<------<…chunk
    name…>------>`, etc.) in column 1 of a line by itself starts a code chunk.
  - Code is in any language; use `<<…>>` (or `<-<…>->`,
    `<------<…>------>`, etc.) to include other code chunks by
    reference.
  - *You can write multiple code chunks with the same name*; they are
    concatenated.

Here are the differences between `litprog` and `noweb`:

  - Text is in LaTeX in `noweb`, and text is in HTML (by default) in
    `litprog`.
      - Text is actually ambiguous in `litprog`, but the builtin weaver
        assumes HTML.
        Creating a 3rd party extension can afford using other languages.
  - `noweb` uses `[[…]]` for embedded code fragments, but `litprog`
    expects any code fragments in the document to be properly escaped already.
  - `noweb` allows code chunks to begin with `<<…>>=` and code
    references to be of the form `<<…>>`, whereas `litprog` allows code
    chunks to begin with any of `<<…>>=`, `<-<…>->=`, `<--<…>-->=`, and
    so on, with the corresponding code references being `<<…>>`,
    `<-<…>->`, `<--<…>-->`, and so on.
      - The reason for this is to make writing code chunks including
        `<<` and `>>` (which is a valid operator in *Python* and *C++*)
        easier, by not having to worry if the `<<` operator will be
        misinterpreted as part of a code reference.

Here is an example document conforming to `litprog` syntax:

```
<p>
The arms of the case statement have some extra information. The file and line
number help with error message and make it possible to generate
<code>#line</code> statements that identify the source of the code. The
original arm gives the arm from which the current arm is derived, and is useful
for many of the heuristics.
</p>

<<*>>=
record caserec(arms,valcode,trailer)
		# case arms, code to compute value, trailing code
record arm(file, line, pattern, code, original)
		# pattern and code are the content
		# line, file, original(pattern) are used for error reporting
@

<p>
Each node of the decision tree is associated with a particular case statement.
Internal nodes have children, and a <code>field</code> which says which field
we decided to test on. The edges that point to the children record the interval
of values for the particular child. Leaf nodes have a <code>name</code> that
records the name of the pattern known to match at that leaf node.
</p>

<<*>>=
record node(cs, children, field, name)
	# case statement, list of edges to children, field chosen, pattern name
	#	(name field used to support name operator, assigned only to leaves)
record edge(node, lo, hi)
	# node pointed to and lo and hi interval of field for this edge
@

<p>
To create a decision tree, I begin with a node containing the full, original
case statement. I then use a “work queue” approach to check each node and see
if it needs to be split.
</p>
```
## Parsing LITPROG

Here is a pseudocode algorithm for parsing `litprog` style literate
programs:

```
procedure Parse is
  set parser state to ‘documentation’

  for each line
    if parser state is ‘documentation’
    then
    |
    | if at END-OF-FILE
    | then
    | |    • store currently parsed documentation chunk
    | end if
    |
    | if line is a code header (like “<<…>>=”)
    | then
    | |    • store currently parsed documentation chunk
    | |    • set parser state to ‘code’
    | |    • note how many ‘-’ are used (“<<…>>=”, “<--<…>-->”, etc.)
    | |    • prepare a new chunk
    | end if
    |
    | if line is a documentation header (“@”)
    | then
    | |    • store currently parsed documentation chunk
    | |    • prepare a new chunk
    | end if
    |
    | otherwise add verbatim line to current chunk
    |
    or if parse state is ‘code’
    then
    |
    | if at END-OF-FILE
    | then
    | |    • store currently parsed code chunk
    | end if
    |
    | if line is a code header (like “<<…>>=”)
    | then
    | |    • store currently parsed code chunk
    | |    • note how many ‘-’ are used (“<<…>>=”, “<--<…>-->”, etc.)
    | |    • prepare a new chunk
    | end if
    |
    | if line is a documentation header (“@”)
    | then
    | |    • store currently parsed code chunk
    | |    • set parser state to ‘documentation’
    | |    • prepare a new chunk
    | end if
    |
    | if line is a code reference (includes “<<…>>” or “<-<…>->”, etc.)
    | then
    | |    • determine the prefix, suffix, and reference of the line
    | |    • add the reference line to the chunk
    | end if
    |
    | otherwise add verbatim line to current chunk
    |
    end if
  end loop
end procedure
```
