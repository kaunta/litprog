# LITPROG COMMAND LINE INTERFACE

- [Command Line Usage](#command-line-usage)

`litprog` is a utility for [literate
programming](https://www-cs-faculty.stanford.edu/~knuth/lp.html), a
meta-paradigm invented by [Donald
Knuth](https://www-cs-faculty.stanford.edu/~knuth/).
`litprog` directly supports

  - **weaving** a hypertext document from a literate program, and
  - **tangling** compiler-ready (or interpreter-ready) code from a
    literate program.

Additionally, `litprog` can dump the in-memory SQLite database
(documented in [LITPROG SQLITE FORMAT](sqlite_format.md)).

## Command Line Usage

  - `$ litprog weave <prog.txt >render.html`
      - Use `litprog` to **weave** an HTML document `render.html` from
        literate program `prog.txt`.
  - `$ litprog tangle main.c <prog.txt >code.c`
      - Use `litprog` to **tangle** the node `main.c` out of literate
        program `prog.txt` and into file `code.c`.
  - `$ litprog dump db.sqlite <prog.txt`
      - Use `litprog` to parse literate program `prog.txt` and dump the
        corresponding SQLite database into `db.sqlite`.
