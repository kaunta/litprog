# LITPROG SQLITE FORMAT

`litprog` processes literate programs using an in-memory
[SQLite](https://www.sqlite.org/) database, which can be dumped on disk
for 3rd party applications to interact with.

## Overview

Conceptually, a literate program consists of a series of code- and
documentation-chunks.
Documentation-chunks are prose descriptions of the program, and
code-chunks are blocks of code that can link to other code-blocks.
Each chunk consists of lines, with documentation-chunks consisting just
of “verbatim” lines—without any semantics—and code-chunks consisting of
both “verbatim” lines and “reference” lines that link to other
code-chunks.

The general hierarchy of the database is:

  - The SQLite database represents the **Document**.
  - The **Document** contains **Chunks** at various **Positions.**
  - Each **Chunk** contains **Lines** at various **Positions.**

And the speciation in the database is:

  - Each **Chunk** is either a **Documentation Chunk** or a **Code
    Chunk**.
  - Each **Line** is either a **Verbatim Line** or a **Reference Line**.
  - Each **Documentation Chunk** only allows **Vertbatim Lines**.
  - Each **Code Chunk** has a **Name** and allows both **Verbatim
    Lines** and **Reference Lines**.
  - Each **Verbatim Line** has **Content**.
  - Each **Reference Line** has a **Prefix**, a **Reference**, and a
    **Suffix**.

The **Prefix**, **Reference**, and **Suffix** refer to the three parts
of a reference line.
For example, given a **Reference Line** represented in plain text as

```
int x = <<value of x>>;
```

the **Prefix** is `int x =`, the **Reference** is `value of x`, and the
**Suffix** is `;`.

The **Position** of a **Chunk** in the **Document** (or of a **Line** in
a **Chunk**) is only used for the relative ordering: the specific value
of a **Chunk‘s** **Position** can be changed to any other value as long
as all pre-existing inequalities are preserved.

## Schema

Here is the full schema, with all tables annotated with their
corresponding formal sentences:

```SQL
PRAGMA foreign_keys = ON;

-- Position Relations
---------------------

-- Position_Chunk(position, chunk_id)
-- : “Chunk «chunk_id» is at position «position» in the document.”
--
CREATE TABLE Position_Chunk (
  position  INTEGER  PRIMARY KEY,
  chunk_id  INTEGER  NOT NULL,

  FOREIGN KEY (chunk_id)
              REFERENCES Chunk(id)
);

-- Position_Line(position, chunk_id, line_id)
-- : “Line «line_id» is at position «position» in chunk «chunk_id».”
--
CREATE TABLE Position_Line (
  position  INTEGER  PRIMARY KEY,
  chunk_id  INTEGER  NOT NULL,
  line_id   INTEGER  NOT NULL,

  FOREIGN KEY (chunk_id)
              REFERENCES Chunk(id),

  FOREIGN KEY (line_id)
              REFERENCES Line(id)
);

-- Chunk Relations
------------------

-- Chunk(id, species)
-- : “Chunk «id» exists and is of species «species».”
--
CREATE TABLE Chunk (
  id       INTEGER  PRIMARY KEY,
  species  TEXT     NOT NULL,

  CHECK (
    species IN ('DOCUMENTATION', 'CODE')
  )
);

-- Chunk_Name(chunk_id, name)
-- : “If chunk «chunk_id» is of species ‘CODE’, then it has the name «name».”
--
CREATE TABLE Chunk_Name (
  chunk_id  INTEGER  PRIMARY KEY,
  name      TEXT     NOT NULL,

  FOREIGN KEY (chunk_id)
              REFERENCES Chunk(id)
);

-- Line Relations
-----------------

-- Line(id, species)
-- : “Line «id» exists and is of species «species».”
--
CREATE TABLE Line (
  id       INTEGER  PRIMARY KEY,
  species  TEXT     NOT NULL,

  CHECK (
    species IN ('VERBATIM', 'REFERENCE')
  )
);

-- Line_Verbatim(line_id, content)
-- : “If line «line_id» is of species ‘VERBATIM’, then its content
--    is «content».”
--
CREATE TABLE Line_Verbatim (
  line_id  INTEGER  PRIMARY KEY,
  content  TEXT     NOT NULL,

  FOREIGN KEY (line_id)
              REFERENCES Line(id)
);

-- Line_Reference(line_id, prefix, reference, suffix)
-- : “If line «line_id» is of species ‘REFERENCE’, then it has prefix «prefix»,
--    reference «reference», and suffix «suffix».”
--
CREATE TABLE Line_Reference (
  line_id    INTEGER  PRIMARY KEY,
  prefix     TEXT     NOT NULL,
  reference  TEXT     NOT NULL,
  suffix     TEXT     NOT NULL,

  FOREIGN KEY (line_id)
              REFERENCES Lines(id)
);
```
