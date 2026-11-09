# Working with Source Code in Org — Summary of the Org Manual

A condensed reference for the parts of `ORGMANUAL.md` that concern source code blocks,
Org Babel, and executing embedded code.

Sources:

- **`ORGMANUAL.md`** — Chapter 16, "Working with Source Code", plus related pieces of
  Chapters 2, 7, and 17.
- **`ORGBABEL.md`** — Worg's *Introducing Babel* (Schulte, Davison, Dye). Sections
  derived from it are marked **[Worg]**, and it is the source for the tangling details
  below.

> **Note on gaps:** In this copy of `ORGMANUAL.md`, sections **16.3 (Using Header
> Arguments)** and **16.8 (Extracting Source Code / tangling)** are missing from the
> text, even though the chapter's table of contents and index list them. The header
> arguments documented elsewhere in the manual are covered below; the tangling section
> is reconstructed from `ORGBABEL.md` plus the manual's key/function table.

---

## 1. What Org Babel is

"Source code" here means any plain-text collection of computer instructions in a
human-readable programming language. Org manages it when the code is delimited by
`#+BEGIN_SRC` … `#+END_SRC`.

Key points:

- A source block can appear **anywhere** in the document — not restricted to the
  preamble or the end. It **cannot** be managed if it sits inside an Org comment or a
  fixed-width section.
- Source blocks are one of several block types (`center`, `comment`, `dynamic`,
  `example`, `export`, `quote`, `special`, `verse`); Babel concerns only
  `BEGIN_SRC`/`END_SRC`.
- Babel is the facility (by Eric Schulte and Dan Davison) that makes blocks *live*:
  Org can **edit, format, extract (tangle), export, publish, compile, and execute**
  them, and capture their results.
- The distinguishing feature is **language-agnostic interoperability**: a common
  syntax for passing variables, functions, and results between blocks in *any*
  language, so different languages can be mixed in one document.
- Header arguments control how "live" a block is (compilation, execution, extraction,
  export).
- Org can link compiler error messages back to the corresponding line in a block.

**[Worg] Babel's own framing** — three capabilities on top of plain Org code blocks:

1. **interactive and programmatic execution** of code blocks;
2. **code blocks as functions** that accept parameters, refer to other blocks, and can
   be called remotely;
3. **export to files** for literate programming (tangling).

Worg's canonical illustration of the data path: a piece of data goes from an **Org
table** to a **Python** block, then to an **R** block, then is embedded as a value mid-
paragraph, and finally passes through a **Gnuplot** block to become a plot embedded in
the document. Babel executes interpreted languages (shell, Python, R, …) by handing the
code to an interpreter installed on your system; you control what happens to the
results.

Why it matters: Babel turns Org into a tool for **literate programming** and
**reproducible research**, not just a place to keep snippets.

---

## 1b. The three-way split of concerns

Worth internalizing early, since most confusion comes from conflating them:

| Concern | Controlled by | Where documented here |
|---|---|---|
| Does the code **run**, and how | `eval`, `session`, `dir`, `var`, `cache` | §3, §4 |
| What comes **back** and how it's stored | `results` (+ `post`) | §5 |
| What **leaves the document** | `exports` | §6 |
| What becomes a **real file** | `:tangle` family | §7 |
| How blocks **compose** | `var`, `#+CALL:`, noweb, `org-sbe`, Library of Babel | §3, §4, §8, §8c, §10 |

---

## 2. Structure of code blocks

### Block form

```
#+NAME: <name>
#+BEGIN_SRC <language> <switches> <header arguments>
  <body>
#+END_SRC
```

| Element | Required? | Meaning |
|---|---|---|
| `#+NAME: <name>` | optional | Names the block so it can be *called like a function* from other blocks, inline code, other files, or table formulas. Names must be unique (duplicates = undefined behavior). |
| `#+BEGIN_SRC` … `#+END_SRC` | **mandatory** | Delimit the block. |
| `<language>` | optional | Language identifier. Also used for native fontification when `org-src-fontify-natively` is non-`nil`. If omitted, the block **cannot** have switches or header arguments (the first such token would be read as the language). |
| `<switches>` | optional | Finer control of execution, export, and format (see *Literal Examples*). |
| `<header arguments>` | optional | Control evaluation, export, and tangling. Can also be set per-buffer or per-subtree via properties. |
| `<body>` | — | Code in the specified language. |

`#+HEADER: <header arguments>` is an alternative way to attach header arguments to a
block (used in several manual examples), and is especially handy for automatically
generated header arguments.

### Inline form

```
src_<language>{<body>}
src_<language>[<header arguments>]{<body>}
```

Inline blocks **cannot** be named. Their results are wrapped according to
`org-babel-inline-result-wrap` (default `"=%s="`, i.e. verbatim markup).

### Inserting blocks

- `C-c C-,` (`org-insert-structure-template`): insert an empty block at point, or wrap
  the active region. Types come from `org-structure-template-alist`; `s` = `SRC`.
- Org Tempo: typing `<s` then `TAB` expands to a source block (enable `org-tempo`).

---

## 3. Environment of a block (header arguments)

### 3.1 `var` — passing arguments

Syntax (identical across languages):

```
:var NAME=ASSIGN
:var NAME=FILE:REFERENCE
```

`ASSIGN` may be a literal (string, number), a reference to a named table, a list, a
literal example, another code block (with or without arguments), or the results of a
block evaluation. With `FILE:` the reference is looked up in another file; if `FILE`
does not exist the verbatim reference is searched in the current buffer.

Referenceable objects:

- **Table** named via `#+NAME`. Special handling:
  - `:colnames yes|no|nil` — default `nil`: if the table has column names (second row
    is a horizontal rule), Org strips them, processes, and restores them. `yes` treats
    the first non-hline row as headers; `no` disables pre-processing.
  - `:rownames yes|no` — default `no`. `yes` strips/restores the first column.
    Emacs Lisp blocks ignore `rownames`.
  - `:hlines yes|no` — default `no`: strip horizontal rules from the input table
    (otherwise hline symbols can cause unbound-variable errors).
- **Plain list** named via `#+NAME` (only top-level items are passed; nested items are
  ignored).
- **Code block without arguments** — `:var x=block-name()`.
- **Code block with arguments** — `:var x=block-name(arg=val)`; the block is evaluated
  with point at its location.
- **Literal example / block contents** — `:var x=example-name[]` (brackets optional for
  `EXAMPLE` blocks).
- **Indexing** into values: 0-based, negatives count from the end, commas separate
  dimensions, `[i:j]` is an inclusive range, `[,0]` takes a column, `[1,,1]` indexes a
  3-D structure, `[*]` or `[0:-1]` takes everything. Indexing happens **before**
  `hlines`/`colnames`/`rownames` are applied, so row/column names must be accounted
  for manually.
- **Emacs Lisp values**: any value beginning with `(`, `[`, `'` or `` ` `` is treated
  as Emacs Lisp and evaluated, e.g. `:var filename=(buffer-file-name)`. Values read
  from tables/lists are *not* evaluated.

### 3.2 `session` — shared interpreter

- `none` (default): each block gets a fresh interpreter process that terminates after
  evaluation.
- Any other string names the session; blocks with the same session name share one
  interpreter process and thus state. With `:session` and no value, the session name
  derives from the language identifier.
- Only languages with interactive evaluation support sessions (not C or ditaa). Even
  then, interactive limitations apply.
- **[Worg]** Available for languages that can run an "inferior process" in Emacs —
  Python, R, Ruby, shell, and others. Sessions are especially useful for **prototyping
  and debugging**, and `org-babel-pop-to-session` (`M-<down>` in a block) switches to the
  live session buffer.
- **Once a block is finished, prefer running it outside a session**, so the state of the
  execution environment is certain. Lingering session state is a common source of
  "works on my machine, not on a fresh run" surprises.
- With R, the session is controlled by **ESS** as usual, so full ESS power is available
  both in the session and in the `C-c '` edit buffer.

### 3.3 `dir` / `mkdirp` — working directory

- `:dir DIRECTORY` sets `default-directory` during execution; otherwise the current
  buffer's directory is used. Temporarily equivalent to `M-x cd`.
- `:mkdirp yes` creates the directory if needed.
- `:dir attach` (or `"'attach"`) uses `(org-attach-dir)`, implies `:mkdirp yes`, and
  inserts paths under the attachment directory as `attachment:` links.
- Tramp syntax gives remote execution, e.g. `:dir /scp:user@host:`.
- `dir` with `session` only sets the directory for *new* sessions.
- Do **not** combine `dir` with `:exports results`/`:exports both` (Org does not
  expand `default-directory`, producing wrong remote links).

### 3.4 `prologue` / `epilogue`

- `:prologue "..."` prepends text before execution (e.g. `reset` for gnuplot).
- `:epilogue "..."` appends text after the body.
- Both can be set globally per language via `org-babel-default-header-args:<lang>`.

---

## 4. Evaluating code blocks

### 4.1 Invoking evaluation

- `C-c C-c` or `C-c C-v e` on a block → `org-babel-execute-src-block`: executes, collects
  results, inserts them after the block following a `#+RESULTS:` keyword (created if
  absent).
- **Security:** Org prompts for permission before executing code by default.
- **Only Emacs Lisp is enabled by default.** Other languages must be enabled (see §9).

### 4.2 Calling named blocks — `#+CALL:` and `call_`

```
#+CALL: <name>(<arguments>)
#+CALL: <name>[<inside header arguments>](<arguments>) <end header arguments>
```

Inline form:

```
... call_<name>(<arguments>) ...
... call_<name>[<inside header arguments>](<arguments>)[<end header arguments>] ...
```

- `<name>` is the block's `NAME`. For a block in another file, prefix with the file
  name and a colon: `#+CALL: file.org:clear-data()`.
- `<arguments>` use **function-call** syntax, e.g. `#+CALL: double(n=4)` — different
  from header-argument syntax.
- `<inside header arguments>` use **header-argument** syntax and apply to *evaluation*,
  e.g. `[:results output]`.
- `<end header arguments>` affect how returned results are inserted (e.g.
  `:results html` wraps them in a `BEGIN_EXPORT html` block).
- Named blocks can also be called from table formulas and from the Library of Babel.

### 4.3 Limiting evaluation — `eval`

| Value | Effect |
|---|---|
| `yes` | Evaluate (subject to `org-confirm-babel-evaluate`). |
| `never` / `no` | Never evaluate. |
| `query` | Prompt for permission. |
| `never-export` / `no-export` | Do not evaluate during export; interactive evaluation still allowed. |
| `query-export` | Prompt during export. |

If `eval` is unset, `org-confirm-babel-evaluate` decides.

### 4.4 Caching — `cache`

- `no` (default): always evaluate.
- `yes`: compare a SHA1 hash of block text + arguments + `var` values against the hash
  stored on the `#+RESULTS[...]:` line; skip evaluation if it matches.
  ```
  #+RESULTS[a2a72cd647ad44515fab62e144796432793d68e1]: random
  ```
- Requires an existing result in the buffer and no changes to header args, `var`
  values, or body.
- Best for pure functions with no side effects. Caching **does not expand noweb
  references** when hashing, and `cache` inside a `session` can give unexpected
  results.

---

## 5. Results of evaluation

The `results` header argument takes one option from each of four classes.

### 5.1 Collection (mutually exclusive)

- `value` — *functional mode*: Org wraps the code in a function definition; the result
  is the value returned by the last statement. In Python an explicit `return` is
  mandatory. Default for most Babel libraries. In a session, Org obtains the value via
  language-specific means (Ruby `_`, R `.Last.value`).
- `output` — *scripting mode*: code runs in an external process; the contents of stdout
  are returned as text. In a session, output from the inferior process is concatenated.

**[Worg] Functional vs. scripting mode — the clearest way to see the difference.** Given
this Python block:

```python
import time
print("Hello, today's date is %s" % time.ctime())
print('Two plus two is')
return 2 + 2
```

- `:results value` (functional) captures **only the return value**: `4`. Everything
  printed to stdout is discarded. The block behaves as a function with a return value.
- `:results output` (scripting) captures **only stdout**:

  ```
  Hello, today's date is Sat Oct 16 10:48:47 2021
  Two plus two is
  ```

  The trailing `2 + 2` never appears because there is no `print()` for it — a scripting-
  mode block has no return value apart from what it emits.

Practical consequences: in functional mode a tabular return value becomes an Org table
and can feed another block in another language; in scripting mode you get exactly what
the program printed, including prompts and progress chatter. Arguments (`var`) work in
both modes. Functional mode is the default in most languages; scripting mode is default
for shell and other "just run it" languages.

### 5.2 Type (mutually exclusive)

Default is auto-detected per language.

- `table`, `vector` — interpret as an Org table (a single value becomes a 1×1 table).
  Usage: `:results value table`.
- `list` — interpret as an Org list (single value → one-element list).
- `scalar`, `verbatim` — insert literally as quoted text; no table.
- `file` — save output to a file and insert a **link** to it.
  - Filename from `:file`, resolved under `:output-dir` (default: current directory).
  - If `:file` is absent, the base name comes from the block name and the extension
    from `:file-ext`; both are then mandatory.
  - `:file-desc` sets the link description (empty vector `[]` for an empty description);
    `:sep` sets the table separator (default TAB); `:file-mode` sets permissions, e.g.
    `:file-mode (identity #o755)` to make a script executable.

### 5.3 Format (mutually exclusive; default derives from type)

- `raw` — inserted as raw Org mode, tables aligned.
- `code` — enclosed in a code block.
- `drawer` — inserted into a `RESULTS` drawer / results macro (good for scripting).
- `html` / `latex` — enclosed in a `BEGIN_EXPORT html` / `latex` block.
- `link`, `graphics` — with `file`: produce a link *without* writing output to disk
  (the block must create the file by side effects). Without `:file`, the result is
  interpreted as a path.
- `org` — enclosed in a `BEGIN_SRC org` block.
- `pp` — pretty-printed source (Emacs Lisp, Python, Ruby).
- `wrap` — unconditionally wraps results by appending to `#+BEGIN_`/`#+END_`
  (e.g. `:wrap EXPORT markdown`); takes precedence over `results`.

### 5.4 Handling (mutually exclusive)

- `replace` (default) — insert, removing previous results.
- `silent` — don't insert; echo in the minibuffer.
- `none` — compute but neither insert nor echo (still usable by other blocks).
- `discard` — ignore results entirely; programmatic/by-reference calls always yield `nil`.
- `append` — append newest results at the bottom.
- `prepend` — prepend newest results at the top.

### 5.5 Post-processing — `post`

`post` runs a *named* block over the results, binding them to `*this*` so they can be fed
to `var` of the post-processing block. Useful for attaching attributes (e.g. an
`#+ATTR_LATEX: :width 5cm` line) or reformatting tables.

```
#+BEGIN_SRC dot :post attr_wrap(width="5cm", data=*this*) :results drawer
```

---

## 6. Exporting code blocks

`exports` decides what leaves Org:

| Value | Exports |
|---|---|
| `code` | **Default.** The code body. |
| `results` | Only the evaluation results. |
| `both` | Code and results. |
| `none` | Neither. |

- Defaults: *code* for most languages, *results* for inline blocks (and always
  *results* for languages like ditaa).
- A named block's results inherit the block's name, so fuzzy links stay valid under
  `:exports results`. Results can also be named explicitly with their own `#+NAME:`
  keyword (preferred over the parent block's name) — useful with `:exports both` to
  disambiguate where links point:
  ```
  #+NAME: code name
  #+BEGIN_SRC emacs-lisp :exports both value
  (+ 1 2)
  #+END_SRC

  #+NAME: results name
  #+RESULTS: code name
  3
  ```
- Links pointing to a block exported with `:exports none` will be **broken** and can
  fail the export.
- To skip evaluation: `:eval never-export` (header arguments still evaluated) or set
  `org-export-use-babel` to `nil` (header arguments then have no effect at all).
- Results not marked for export are not evaluated even with `:eval yes` — except in
  `session` blocks, which follow `eval`.
- Blocks in **commented subtrees** are never evaluated; blocks in subtrees excluded
  from export **are** evaluated.
- To export just the body (no `BEGIN_SRC` wrapper), see *Literal Examples*.

---

## 7. Extracting source code — tangling

*Manual §16.8 is absent from this copy; the following is from Worg's "Literate
Programming" section plus the manual's index and key tables.*

**Tangling** = extracting the embedded source from an Org document into source-code
files (the counterpart of **weaving**, which exports the documentation via Org's normal
export). Babel is therefore a literate-programming tool: the same document is readable
by humans (exported to HTML/LaTeX) and by computers (tangled to a `.sh`, `.py`, etc.).

### 7.1 The `:tangle` family of header arguments **[Worg]**

- `:tangle` controls tangling and can be set per **code block** or per **heading level**
  (via `header-args`, which then applies to all blocks in that subtree).
- Without `:tangle`, a block produces no file — but it can still be *pulled into* a
  file by another block that does tangle, via noweb references.
- `:tangle <filename>` names the output file for that block.
- `:shebang` sets the interpreter line prepended to the tangled file.
- `:tangle-mode` sets the resulting file's permissions.
- `:no-expand` inhibits body expansion (variable assignment code) during tangling.
- Hooks: `org-babel-pre-tangle-hook`, `org-babel-tangle-body-hook`,
  `org-babel-post-tangle-hook`, `org-babel-tangle-finished-hook`.
- `org-babel-tangle-jump-to-org` jumps from a tangled file back to the block.

### 7.2 Invoking tangling

| Key | Command |
|---|---|
| `C-c C-v t` / `C-c C-v C-t` | `org-babel-tangle` (current file) |
| `C-c C-v f` / `C-c C-v C-f` | `org-babel-tangle-file` (prompt for a file) |

**Batch tangling** from the shell (manual §16.14):

```sh
emacs -Q --batch --eval "
    (progn
      (require 'ob-tangle)
      (dolist (file command-line-args-left)
        (with-current-buffer (find-file-noselect file)
          (org-babel-tangle))))
  " "$@"
```

### 7.3 Worked example **[Worg]**

Two blocks with **no** `:tangle` argument supply reusable fragments:

```
#+name: hello-world-prefix
#+begin_src shell :exports none
  echo "/-----------------------------------------------------------\\"
#+end_src

#+name: hello-world-postfix
#+begin_src shell :exports none
  echo "\-----------------------------------------------------------/"
#+end_src
```

A third block names the output file and pulls the fragments in with noweb:

```
#+name: hello-world
#+begin_src shell :tangle hello :exports results :noweb yes :results raw
  <<hello-world-prefix>>
  echo "|                       hello world                         |"
  <<hello-world-postfix>>
#+end_src
```

`org-babel-tangle` writes `hello.sh`:

```sh
#!/usr/bin/env sh

# [[file:~/org/temp/index.org::*Noweb test][hello-world]]

echo "/-----------------------------------------------------------\\"
echo "|                       hello world                         |"
echo "\-----------------------------------------------------------/"
# hello-world ends here
```

Note the generated **back-link comment** pointing at the originating Org file and
heading — tangled files remain navigable back to their source.

### 7.4 Tangling expanded bodies **[Worg]**

Tangling normally writes the block **body**; but a block can be tangled *already
expanded*, i.e. with the code that assigns `var` values to variables inlined as literal
data. Those values may come from other code blocks, from headline properties, or from
tables.

This makes Org a way to keep secrets/config out of the code while producing a
self-contained file — e.g. a literate Emacs init file where usernames and passwords
live in a table:

```
username   john-doe
password   abc123
```

and the block referring to them

```
(setq my-special-username (first (first data)))
(setq my-special-password (first (second data)))
```

**expands** to (per `C-c M-b p`):

```
(let ((data (quote (("john-doe") ("abc123")))))
  (setq my-special-username (first (first data)))
  (setq my-special-password (first (second data))))
```

Use `:no-expand` to prevent this at tangle time.

### 7.5 Indentation during tangling

`org-src-preserve-indentation` (see §11) governs whether leading whitespace survives
export and tangling. Leave it `nil` for most languages; set it non-`nil` for
whitespace-sensitive languages such as Python.

---

## 8. Noweb reference syntax

Include other blocks' contents (or results) with:

```
<<CODE-BLOCK-ID>>
<<NAME(optional arguments)>>
```

`CODE-BLOCK-ID` is either a block's `NAME` or a set of blocks sharing the same
`noweb-ref` header argument (settable at file, subtree, or block level).

The `noweb` header argument controls expansion when **evaluating, tangling, or
exporting**:

| Value | Evaluating | Tangling | Exporting |
|---|---|---|---|
| `no` (default) | — | — | — |
| `yes` | yes | yes | yes |
| `tangle` | — | yes | — |
| `strip-tangle` | yes | — | yes, then references removed |
| `no-export` | yes | yes | — |
| `strip-export` | yes | yes | yes, then removed |
| `eval` | yes | — | — |

Details:

- Parentheses with the name (`<<NAME(args)>>`) insert the **results** of evaluating
  that block instead of its body; this requires a `NAME` (a `noweb-ref` reference will
  not work).
- Prefix characters before a reference are honored: with an SQL-comment prefix `---`,
  every expanded line is commented out; this also controls indentation of expanded
  Python, etc. Disable with `:noweb-prefix no`.
- `noweb` expansion does **not** carry over `:var` header arguments.
- Noweb arguments are passed **literally**, not evaluated in the current block: with
  `<<example-block(a=9)>>`, `a` equals 9 inside `example-block`, regardless of any local
  bindings.
- `C-c C-v v` (`org-babel-expand-src-block`) and `C-c M-b p` preview the expansion in a
  buffer — very useful for debugging why a reference expanded unexpectedly.
- `cache` hashing does not expand noweb references.

---

## 8b. Code block body expansion

Before evaluation, Babel **expands** a block: header arguments and `var` references are
turned into actual code that assigns the referenced data to the named variables, and the
body is that code plus your original statements.

- Preview with `C-c C-v v` (`org-babel-expand-src-block`), or `C-c M-b p` **[Worg]**,
  with point inside the block.
- Expansion is also applied when tangling (see §7.4) unless `:no-expand` is set.
- Expansion resolves, in order of what can appear in a `var` position:

  1. results of other code blocks,
  2. values stored in headline properties,
  3. Org tables.

This is the mechanism that makes Babel a meta-programming layer: because the return
value of a function in one language can be passed to a function in another language, or
to an Org table (itself programmable), functions from many languages compose.

---

## 8c. Babel in Org table formulas — `org-sbe` **[Worg]**

Babel adds a third data path with tables (besides passing tables *in* as `var` and
writing results *out* as tables): the `#+TBLFM` formula line can call a code block in
any language via the `org-sbe` ("source block evaluate") macro.

```
#+name: tbl-example-data
#+begin_src R
runif(n=5, min=0, max=1)
#+end_src

#+name: R-mean
#+begin_src R :var x=""
colMeans(x)
#+end_src

#+name: summaries
|              mean |
|-------------------|
| 0.779619386699051 |
#+TBLFM: @2$1='(org-sbe "R-mean" (x "tbl-example-data()"))
```

Notes:

- `(org-sbe "block-name" (arg1 value1) (arg2 value2))` evaluates the block and returns
  its value to the formula.
- Arguments can be other block calls (`"tbl-example-data()"`), so blocks compose inside
  formulas.
- To recalculate a table formula, use `C-u C-c C-c` in the table.
- If the data exists both as a *table* and as the *block* that generates it, the table
  wins and the value is static. Delete the data table and the reference resolves to the
  generating block instead, so the value is recomputed on each recalculation.
- The same `#+TBLFM` machinery was used to build Babel's own test suite: a single table
  where column 5 calls the block named in column 2 and column 6 compares results with
  expectations (`C-u C-c C-c` runs the whole suite).

```org
#+TBLFM: $5='(if (= (length $3) 1) (sbe $2 (n $3)) (sbe $2)) :: $6='(if (string= $4 $5) "pass" (format "expected %S but was %S" $4 $5))
```

Note `(sbe ...)` is the shorter form usable inside table formulas.

---

## 9. Languages

- Dozens of languages are supported; see the Worg
  [language-specific documentation](https://orgmode.org/worg/org-contrib/babel/languages/index.html).
- **Only Emacs Lisp is enabled by default.** Enable others with
  `org-babel-load-languages`:
  ```elisp
  (org-babel-do-load-languages
   'org-babel-load-languages
   '((emacs-lisp . nil)
     (R . t)))
  ```
- `require`-ing a language's `ob-<lang>` library also enables it, e.g.
  `(require 'ob-clojure)`.

---

## 10. Library of Babel

- A user-extensible collection of reusable, **named** code blocks callable from *any*
  Org buffer, not just the one they are defined in — "like a function library" **[Worg]**.
  Babel searches the current buffer first, then the files registered in the library.
- Register files with `org-babel-lob-ingest`:
  ```elisp
  (org-babel-lob-ingest "path/to/file.org")
  ```
  The manual binds this to `C-c C-v i`; Worg documents it as `C-c C-v l` (that key is
  also bound to `org-babel-load-in-session` in the manual's table, so check
  `C-c C-v h` / `org-babel-describe-bindings` in your Emacs).
- Once registered, a block is callable with `#+call: block-name(args)` from any buffer.
- Table values or the output of other blocks can be passed to library functions, and
  library functions can be referenced inside arguments to other blocks.
- Org once shipped an eponymous `library-of-babel.org`; it now lives at
  <https://git.sr.ht/~bzg/worg/tree/master/item/library-of-babel.org>.
- Blocks in another *specific* file can be called with `file.org:block-name()`.

---

## 11. Editing source code

- `C-c '` opens the block body in a dedicated major-mode edit buffer; `C-c '` again
  returns to the Org buffer. `org-edit-special` is the general command also used for
  `table.el` tables.
- `C-x C-s` saves the buffer and updates the Org buffer.
- Variables:
  - `org-edit-src-auto-save-idle-delay` — save the base buffer after idle time.
  - `org-edit-src-turn-on-auto-save` — auto-save the edit buffer to a separate file.
  - `org-src-lang-modes` — map language identifiers to major modes (`<LANG>-mode` when
    available, else Fundamental mode).
  - `org-src-window-setup` — window arrangement for the edit buffer.
  - `org-src-preserve-indentation` — default `nil` (code indented). Set non-`nil` to
    align at the leftmost column and modify no lines during export/tangling — essential
    for whitespace-sensitive languages such as Python.
  - `org-src-ask-before-returning-to-edit-buffer` — default prompts before returning.
  - `org-src-fontify-natively` — native fontification in the Org buffer.
  - `org-src-block-faces` — per-language `org-block` face customization.
  - Further variables live in the `org-edit-structure` customization group.

---

## 12. Quick reference: keys and functions

**With point in a code block**

| Key | Command |
|---|---|
| `C-c C-c` | `org-babel-execute-src-block` |
| `C-c C-o` | `org-babel-open-src-block-result` |
| `M-<up>` | `org-babel-load-in-session` |
| `M-<down>` | `org-babel-pop-to-session` |
| `C-c '` | edit the block in a dedicated buffer |

**In an Org buffer (`C-c C-v` prefix)**

| Key | Command |
|---|---|
| `C-c C-v p` / `C-c C-v C-p` | `org-babel-previous-src-block` |
| `C-c C-v n` / `C-c C-v C-n` | `org-babel-next-src-block` |
| `C-c C-v e` / `C-c C-v C-e` | `org-babel-execute-maybe` |
| `C-c C-v o` / `C-c C-v C-o` | `org-babel-open-src-block-result` |
| `C-c C-v v` / `C-c C-v C-v` | `org-babel-expand-src-block` (noweb preview) |
| `C-c C-v u` / `C-c C-v C-u` | `org-babel-goto-src-block-head` |
| `C-c C-v g` / `C-c C-v C-g` | `org-babel-goto-named-src-block` |
| `C-c C-v r` / `C-c C-v C-r` | `org-babel-goto-named-result` |
| `C-c C-v b` / `C-c C-v C-b` | `org-babel-execute-buffer` |
| `C-c C-v s` / `C-c C-v C-s` | `org-babel-execute-subtree` |
| `C-c C-v d` / `C-c C-v C-d` | `org-babel-demarcate-block` |
| `C-c C-v t` / `C-c C-v C-t` | `org-babel-tangle` |
| `C-c C-v f` / `C-c C-v C-f` | `org-babel-tangle-file` |
| `C-c C-v c` / `C-c C-v C-c` | `org-babel-check-src-block` |
| `C-c C-v j` / `C-c C-v C-j` | `org-babel-insert-header-arg` |
| `C-c C-v l` / `C-c C-v C-l` | `org-babel-load-in-session` |
| `C-c C-v i` / `C-c C-v C-i` | `org-babel-lob-ingest` |
| `C-c C-v I` / `C-c C-v C-I` | `org-babel-view-src-block-info` |
| `C-c C-v z` / `C-c C-v C-z` | `org-babel-switch-to-session-with-code` |
| `C-c C-v a` / `C-c C-v C-a` | `org-babel-sha1-hash` |
| `C-c C-v h` / `C-c C-v C-h` | `org-babel-describe-bindings` |
| `C-c C-v x` / `C-c C-v C-x` | `org-babel-do-key-sequence-in-edit-buffer` |

Other relevant keys: `C-c C-,` (insert/wrap structure template), `C-c C-x p`
(`org-set-property`, useful for setting `header-args` on a subtree).

**`C-c M-b` prefix** — Org's "block" prefix, distinct from `C-c C-v`:

| Key | Command |
|---|---|
| `C-c M-b p` | `org-babel-expand-src-block` (preview expansion) |
| `C-c M-b` followed by other keys | block navigation/insertion commands (`org-previous-block`, etc.; see `C-c M-b h` / the manual's speed-key table) |

`C-c M-b p` is the shortcut Worg documents; the manual binds the same command to
`C-c C-v v`. Both work.

---

## 13. Security

- Each source block is, in terms of risk, equivalent to an executable file.
- `org-confirm-babel-evaluate`:
  - `t` — prompt before executing each block.
  - `nil` — execute without prompting.
  - a function — called with `(LANG BODY)`, must return `t`/`nil` to decide whether to
    prompt. Example: never prompt for ditaa.
    ```elisp
    (defun my-org-confirm-babel-evaluate (lang body)
      (not (string= lang "ditaa")))
    (setq org-confirm-babel-evaluate #'my-org-confirm-babel-evaluate)
    ```
- Two link types evaluate code directly and are also gated:
  `org-link-shell-confirm-function` (`shell:` links) and
  `org-link-elisp-confirm-function` (`elisp:` links).
- Table formulas are code too — evaluated by the Calc or Emacs Lisp interpreter.
- Org evaluates blocks during **export** as well as interactively; only open files from
  trusted sources.

---

## 14. Literate Emacs initialization **[Worg]**

Babel has dedicated support for keeping your Emacs config in Org. `org-babel-load-file`
loads the Emacs Lisp code blocks embedded in a literate Org file exactly as if it were a
plain `.el` file, so you get folding, tags, notes, and HTML export for your config.

Minimal `~/.emacs.d/init.el`:

```elisp
;;; init.el --- Where all the magic begins

;; Load up Org Mode and (now included) Org Babel for elisp embedded in Org Mode files
(setq dotfiles-dir (file-name-directory (or (buffer-file-name) load-file-name)))

(require 'org)
(require 'ob-tangle)

;; load up all literate org-mode files in this directory
(mapc #'org-babel-load-file (directory-files dotfiles-dir t "\\.org$"))

;;; init.el ends here
```

Then put all customizations in Emacs Lisp `#+BEGIN_SRC emacs-lisp` blocks inside `.org`
files in that directory, and restart Emacs.

This pattern generalizes: any language whose config you want to keep literate can be
tangled (`:tangle`) into a real source file while the Org document remains the source of
truth and the documentation.

---

## 15. Reproducible research **[Worg]**

- Reproducible research (RR) = shipping, alongside a publication, all data, software, and
  tools needed to reproduce its results — the paper becomes a complete laboratory rather
  than "advertising for the scholarship" (Donoho).
- Org's export gives the *weave*; Babel's activation of embedded code and data gives the
  *executable* part: the whole document becomes runnable.
- Advantages over Sweave (R + LaTeX): multi-language blocks; flexible powerful export
  (HTML as well as LaTeX); plus Org's planning/task features for managing the project.
- Babel's design goal is verification and maintenance: keep text, data, code,
  environment configuration, results, narratives, claims, references, and links in **one**
  Org document.

---

## 16. Related manual sections worth knowing

- **Properties (§7)** — header arguments can be applied to a whole buffer or subtree
  using `header-args` / `header-args:LANG` properties, or globally with
  `#+PROPERTY: var foo` and the append form `#+PROPERTY: var+ bar`. Inheritance is
  governed by `org-use-property-inheritance`. Headline properties are also a legitimate
  source of `var` data (see §8b).
- **Blocks (§2.8)** — `#+STARTUP: nohideblocks` and related block visibility.
- **Literal Examples** — switches and export of block bodies (`-n` numbering, references,
  etc.). Org's "code blocks" (`src` blocks) are introduced there.
- **Structure Templates (§17.2)** — `C-c C-,` and Org Tempo `< s TAB`.
- **The Very Busy `C-c C-c` Key (§17.7)** — the consolidated context-dependent behavior
  list (source blocks are executed when point is in one).
- **In-Buffer Settings (§17.8)** — keywords such as `#+PROPERTY`, `#+STARTUP`,
  `#+NAME`, `#+HEADER`, `#+CALL`, `#+RESULTS`.
- **Export backends (§13)** — e.g. "Source blocks in LaTeX export".
- **Batch execution (§16.14)** — `emacs -Q --batch` scripts driving `ob-tangle`.
- **The Spreadsheet (§3.5)** — `#+TBLFM`, Calc, and Emacs Lisp formulas that `org-sbe`
  plugs into.
- **Installation (§1.2)** — how to get a newer Org/Babel than the one bundled with Emacs;
  Babel has shipped with Org since **Org 7.0** and is available out of the box in
  **Emacs 24+** **[Worg]**.

