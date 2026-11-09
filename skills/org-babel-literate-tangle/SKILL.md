---
name: org-babel-literate-tangle
description: Guide for literate programming, code extraction (tangling), noweb macro expansion, batch compilation, and configuration management with Org Babel.
---

# Literate Programming & Tangling with Org Babel

Org Babel is an implementation of Donald Knuth's **Literate Programming** paradigm. Instead of writing code with interspersed comments, you write human-oriented prose, architecture notes, and design rationale, embedding the functional code blocks inside.

A single Org file serves as the definitive source of truth for two distinct outputs:
1. **Weaving (Human Documentation):** Exporting the notebook to formatted HTML, LaTeX/PDF, or Markdown for readers.
2. **Tangling (Machine Execution):** Extracting pure, runnable source code files (`.py`, `.sh`, `.c`, `.conf`, `.el`) for compilers, interpreters, and deployment systems.

---

## 1. Tangling Basics: The `:tangle` Header Argument

By default, code blocks are not tangled. The `:tangle` family of header arguments specifies where and how blocks are extracted to disk.

### 1.1 The `:tangle` Argument Values

| Value | Behavior |
|---|---|
| `:tangle no` (Default) | Block is not written to any file. (It can still be included inside other blocks via `<<noweb>>`). |
| `:tangle "path/to/file.py"` | Writes the block body directly to the specified file path (relative to buffer directory or absolute). |
| `:tangle yes` | Tangles to a file matching the base name of the Org file with the language's standard file extension (e.g., `model.org` -> `model.py`). |

### 1.2 Additional Tangling Header Arguments

- `:shebang "#!/usr/bin/env python3"`:
  Prepends the specified shebang string as the first line of the tangled file.
- `:tangle-mode (identity #o755)`:
  Sets filesystem permissions on the generated file (e.g., `#o755` makes a script executable).
- `:no-expand`:
  Prevents variable assignments (`:var`) from being inlined into the tangled code; writes only the raw block body.
- `:mkdirp yes`:
  Automatically creates any non-existent parent directories in the tangle path.

---

## 2. Tangling Scope: Block, Subtree, and Buffer Level

Tangling arguments can be scoped cleanly across an entire project:

### 2.1 Subtree-Level Tangling (Recommended Pattern)
Different headings can tangle into different files or modules:

```org
* Core Utilities
:PROPERTIES:
:header-args:python: :tangle src/utils.py :mkdirp yes
:END:

#+BEGIN_SRC python
  def slugify(text: str) -> str:
      return "-".join(text.lower().split())
#+END_SRC

* Data Processing Pipeline
:PROPERTIES:
:header-args:python: :tangle src/pipeline.py :mkdirp yes
:END:

#+BEGIN_SRC python
  from utils import slugify

  def process_record(rec: dict) -> dict:
      rec["slug"] = slugify(rec["name"])
      return rec
#+END_SRC
```

### 2.2 Buffer-Wide Tangling Default
Set at the top of the file:
```org
#+PROPERTY: header-args:python :tangle src/main.py
```

---

## 3. Noweb Syntax: Modular Code Assembly

**Noweb** enables you to assemble larger source files from smaller, modular, easily explained fragments. You reference one code block inside another using `<<block_name>>`.

### 3.1 Enabling Noweb Expansion
Noweb is disabled by default. Control it with the `:noweb` header argument:

| `:noweb` Value | Evaluated in Emacs? | Tangled to Disk? | Exported in Doc? |
|---|---|---|---|
| `no` (Default) | No | No | No |
| `yes` | Yes | Yes | Yes |
| `tangle` | No | **Yes** | No |
| `strip-tangle` | Yes | No (references stripped) | Yes |
| `no-export` | Yes | Yes | No |
| `eval` | **Yes** | No | No |

### 3.2 Noweb Example: Literate Script Construction

```org
#+NAME: imports
#+BEGIN_SRC python :exports none
  import sys
  import argparse
  from pathlib import Path
#+END_SRC

#+NAME: argument-parser
#+BEGIN_SRC python :exports code
  parser = argparse.ArgumentParser(description="Process dataset.")
  parser.add_argument("--input", required=True, type=Path, help="Input CSV path")
  parser.add_argument("--threshold", default=0.5, type=float, help="Score cutoff")
  args = parser.parse_args()
#+END_SRC

#+NAME: process-data
#+BEGIN_SRC python :exports code
  print(f"Loading data from {args.input}...")
  # Processing logic here
#+END_SRC

# Assembling the final executable script:
#+NAME: main-cli
#+BEGIN_SRC python :tangle bin/run_pipeline.py :shebang "#!/usr/bin/env python3" :tangle-mode (identity #o755) :noweb yes :exports none
  <<imports>>

  def main():
      <<argument-parser>>
      <<process-data>>

  if __name__ == "__main__":
      main()
#+END_SRC
```

When tangled, `bin/run_pipeline.py` is written as a clean, executable script with all fragments properly placed and indented!

### 3.3 Noweb Prefix & Indentation
- Noweb honors indentation: if `<<argument-parser>>` is indented by 4 spaces inside `def main():`, every line of the expanded block will be indented by 4 spaces.
- To disable prefixing behavior: `:noweb-prefix no`.
- To insert the **results** of a block's execution instead of its source code, append parentheses:
  `<<compute_constant()>>`

### 3.4 Debugging Expansion
To verify how noweb blocks will expand before tangling or running:
- Place point inside the block and press `C-c C-v v` (`org-babel-expand-src-block`).
- Org opens a temporary buffer showing the exact code after all noweb and `:var` substitutions.

---

## 4. Inlining Config and Data into Tangled Files

Babel can inline variables from Org tables or headline properties directly into tangled code:

```org
# Configuration stored in an Org Table:
#+NAME: server_config
| Key     | Value            |
|---------+------------------|
| host    | 10.0.1.25        |
| port    | 8080             |
| timeout | 30               |

# The tangled configuration module:
#+BEGIN_SRC python :tangle config.py :var cfg=server_config :colnames yes
  HOST = cfg[0][1]
  PORT = int(cfg[1][1])
  TIMEOUT = int(cfg[2][1])
#+END_SRC
```

To prevent Babel from generating variable wrappers in the tangled code, pass `:no-expand`.

---

## 5. Indentation Handling: Whitespace-Sensitive Languages

In whitespace-sensitive languages like Python or YAML, automatic Org re-indentation can break syntax.

### Critical Rule for Python Tangling:
Set `org-src-preserve-indentation` to `t`. When `t`, Org does not strip or add indentation during export or tangling:

```elisp
;; In Emacs configuration:
(setq org-src-preserve-indentation t)
```
Or per block with switch `-i`:
```org
#+BEGIN_SRC python -i :tangle script.py
```

---

## 6. How to Execute Tangling

### 6.1 Interactively Inside Emacs

| Keybinding | Command | Description |
|---|---|---|
| `C-c C-v t` / `C-c C-v C-t` | `org-babel-tangle` | Tangles all code blocks in the current buffer. |
| `C-c C-v f` / `C-c C-v C-f` | `org-babel-tangle-file` | Prompts for an Org file on disk and tangles it. |

### 6.2 Navigating from Tangled Files back to Org
When tangled, Babel inserts back-link comments into generated files:
```python
# [[file:~/project/notebook.org::*Core Utilities][core-slugify]]
def slugify(text: str) -> str:
...
# core-slugify ends here
```
In Emacs, opening the generated file and running `M-x org-babel-tangle-jump-to-org` jumps instantly back to the exact heading and block in the source `.org` document!

### 6.3 Headless Command-Line / CI / Make Batch Tangling
Tangle files without opening an Emacs GUI window (ideal for build scripts, Makefiles, and continuous integration):

```bash
# Tangle a single file:
emacs -Q --batch \
  --eval "(require 'org)" \
  --eval "(require 'ob-tangle)" \
  --eval '(org-babel-tangle-file "analysis.org")'

# Tangle multiple files from a shell loop or Makefile:
emacs -Q --batch --eval "
  (progn
    (require 'org)
    (require 'ob-tangle)
    (dolist (file command-line-args-left)
      (with-current-buffer (find-file-noselect file)
        (org-babel-tangle))))" *.org
```

---

## 7. Literate Configuration Management: `init.org` Pattern

A popular use case for Org Babel literate programming is managing Emacs configurations or system dotfiles.

In `~/.emacs.d/init.el`:
```elisp
;; Bootstrap Org Babel literate configuration
(require 'org)
(require 'ob-tangle)
(org-babel-load-file (expand-file-name "config.org" user-emacs-directory))
```

`org-babel-load-file` automatically checks if `config.org` has been modified since `config.el` was last generated, tangles the new version if needed, and loads the resulting `.el` file.

---

## 8. Summary Checklist for Tangling Projects

1. **Explicit Tangling Paths:** Use explicit filenames (e.g. `:tangle "src/core.py"`).
2. **Use Noweb for Readability:** Break complex algorithms into logical steps (`<<init-weights>>`, `<<forward-pass>>`, `<<backprop>>`) so the narrative flows naturally for readers.
3. **Preserve Indentation:** Always ensure `org-src-preserve-indentation` is `t` for Python code.
4. **Automate with Make/CI:** Include a headless `emacs --batch` tangling step in your build system to keep generated source code synchronized with the Org document.
