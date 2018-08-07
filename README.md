# LITPROG

`litprog` — literate programming tool.

- [Introduction](#introduction)
- [Other Documents](#other-documents)

## Introduction

`litprog` is a utility for [literate
programming](https://www-cs-faculty.stanford.edu/~knuth/lp.html), a
meta-paradigm invented by [Donald
Knuth](https://www-cs-faculty.stanford.edu/~knuth/).
`litprog` supports [`noweb`](https://www.cs.tufts.edu/~nr/noweb/)-style
literate programs: sequences of documentation- and code-chunks.
Unlike `noweb`, `litprog` uses *HTML* as its documentation language,
instead of *LaTeX*.

`litprog` directly supports

  - **weaving** a hypertext document from a literate program, and
  - **tangling** compiler-ready (or interpreter-ready) code from a
    literate program.

`litprog` processes literate programs using an in-memory
[SQLite](https://www.sqlite.org/) database, which can be dumped on disk
for 3rd party applications to interact with.
The application file format (the database schema) will be documented and
frozen to allow for 3rd party programs to analyze a literate program.

