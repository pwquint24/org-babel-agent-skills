---
name: org-babel-notebook
description: Comprehensive guide for authoring, structuring, executing, and publishing interactive computational notebooks in Emacs Org Mode using Org Babel.
---

# Authoring Computational Notebooks with Org Babel

Org Babel turns Emacs Org Mode into an interactive, multi-language computational notebook environment. Like Jupyter or Mathematica, Org Babel interleaves executable code, live execution results, figures, tables, and narrative prose. Unlike JSON-based notebooks, Org notebooks are plain text, human-readable, natively version-controllable with Git, and support mixing multiple programming languages within a single document.

---

## 1. Core Architecture & Separation of Concerns

Every Org Babel notebook operates across five distinct concerns governed by header arguments. Conflating these is the primary source of bugs in Org notebooks:

| Concern | Key Header Arguments | What It Governs |
|---|---|---|
| **Execution** | `:session`, `:eval`, `:dir`, `:var`, `:cache` | Does the block run, in which process, where, and with what inputs? |
| **Results** | `:results` (`value` vs `output`, format, type, handling) | What is captured from execution and how is it rendered in the buffer? |
| **Export** | `:exports` (`code`, `results`, `both`, `none`) | What appears in exported reports (HTML, PDF, Markdown)? |
| **Tangling** | `:tangle`, `:shebang`, `:noweb` | What is extracted to standalone source code files on disk? |
| **Composition** | `:var`, `#+CALL:`, `<<noweb>>`, `org-sbe` | How do blocks feed data or code into one another? |

---

## 2. Notebook Structure & Preamble Setup

An Org notebook file (`.org`) starts with document metadata and buffer-wide default settings, followed by hierarchical headings containing narrative text, mathematical formulas, and code blocks.

### 2.1 Standard Notebook Preamble Template

```org
#+TITLE: Computational Analysis of Experiment X
#+AUTHOR: Jane Doe
#+DATE: [2025-05-18 Sun]
#+OPTIONS: toc:2 num:t ^:{}
#+STARTUP: overview hideblocks inlineimages

# Global Babel defaults for all blocks in this buffer:
#+PROPERTY: header-args :eval yes :exports both :results drawer
# Language-specific defaults:
#+PROPERTY: header-args:python :session *py-analysis* :results value :exports both
#+PROPERTY: header-args:R :session *R-analysis* :results value :exports both
#+PROPERTY: header-args:sh :session none :results output :exports both
```

### Key In-Buffer Settings:
- `#+STARTUP: inlineimages`: Automatically displays generated plots/images inline when results are inserted or when the buffer is opened.
- `#+STARTUP: hideblocks`: Starts with code blocks folded, displaying only narrative and results for readability.
- `#+STARTUP: nohideblocks`: Shows all code blocks expanded.
- `#+PROPERTY: header-args ...`: Buffer-wide defaults. Can be appended to with `header-args+`.

### 2.2 Enabling Languages in Emacs Configuration
Org only enables Emacs Lisp evaluation by default. All other languages must be explicitly loaded in your Emacs configuration:

```elisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (python     . t)
   (R          . t)
   (shell      . t)
   (sqlite     . t)
   (gnuplot    . t)
   (C          . t)))
```

To avoid interactive execution confirmation dialogs for trusted notebooks:
```elisp
;; nil allows blocks to run without confirmation prompts
(setq org-confirm-babel-evaluate nil)
```

---

## 3. Code Block Anatomy

```org
#+NAME: <block_name>
#+HEADER: <additional_header_args>
#+BEGIN_SRC <language> <switches> <header_arguments>
  <body>
#+END_SRC
```

- `#+NAME: <name>`: Optional unique identifier. Mandatory if the block will be referenced as a function, passed as input to another block via `:var`, or called via `#+CALL:`.
- `<language>`: Identifier recognized by Babel (`python`, `R`, `sh`, `emacs-lisp`, `sqlite`, `sql`, `julia`, `calc`, `gnuplot`, etc.).
- `<switches>`: Optional switches (e.g., `-n` for line numbering during export, `-r` to strip labels).
- `<header_arguments>`: Key-value pairs controlling execution, results, and export.
- `#+HEADER:`: Secondary lines to specify long lists of header arguments cleanly.

### Inline Code Blocks
For embedding dynamic computed values inside a paragraph:
```org
The final computed accuracy was src_python[:session *py* :results value]{f"{acc * 100:.2f}%"}.
```

---

## 4. Header Arguments Reference

### 4.1 Session Management: Interactive Exploration vs Hermetic Reproducibility
- `:session none` (Default): Evaluates the block in a one-off external interpreter process. Hermetic, clean, and avoids state pollution, but re-initializes imports and environment on every run.
- `:session <name>`: Evaluates in a persistent Emacs inferior REPL buffer (e.g., `*py-analysis*`).
  - Variables, functions, and imported modules persist across multiple blocks sharing the same session name.
  - Essential for heavy datasets, large model weights, and exploratory workflows.
  - **Best Practice:** Prototype with `:session <name>`; for final verification, restart the session or test critical sections with `:session none` to guarantee reproducibility.

### 4.2 Working Directories & Remote Environments
- `:dir /path/to/project`: Runs the block inside the specified directory.
- `:mkdirp yes`: Creates the target directory if it does not already exist.
- **Remote Execution (TRAMP):**
  `:dir /ssh:user@gpu-cluster:/workspace/data` executes the block remotely on a server while rendering output inside your local Emacs buffer.

### 4.3 Caching Expensive Computations
- `:cache yes`: Computes a SHA1 hash of the block text, inputs, and `:var` parameters.
- If the hash matches the one stored on the `#+RESULTS[hash]:` line, evaluation is skipped and cached results are retained.
- Use only for deterministic, pure computations without external side-effects.

---

## 5. Result Handling: The Critical Distinction

The `:results` argument takes up to four orthogonal components:

```
:results [collection] [type] [format] [handling]
```

### 5.1 Collection: Functional (`value`) vs Scripting (`output`)
Understanding this difference eliminates 90% of Babel confusion:

| Collection Mode | What is Captured | Python Behavior | Shell Behavior |
|---|---|---|---|
| `:results value` (Default for Python, R, elisp) | **Return value** of the last expression or explicit `return`. | Must use an explicit `return <val>` statement. All `print()` output is **discarded**. | Returns the exit code or string value. |
| `:results output` (Default for shell, bash) | **Stdout stream** emitted during execution. | Captures all `print()` statements. Any `return` is ignored. | Captures all command stdout (`echo`, `ls`, etc.). |

#### Python Functional Example (`:results value`):
```org
#+BEGIN_SRC python :results value
  data = [1, 2, 3, 4]
  # In functional mode, an explicit return is REQUIRED in Python:
  return sum(data)
#+END_SRC

#+RESULTS:
: 10
```

#### Python Scripting Example (`:results output`):
```org
#+BEGIN_SRC python :results output
  for i in range(3):
      print(f"Epoch {i}: loss={0.5/(i+1):.4f}")
#+END_SRC

#+RESULTS:
: Epoch 0: loss=0.5000
: Epoch 1: loss=0.2500
: Epoch 2: loss=0.1667
```

### 5.2 Result Types
- `table` / `vector`: Formats returned lists/arrays as an Org Mode table.
- `scalar` / `verbatim`: Formats output as raw quoted text without table formatting.
- `file`: Directs output to a file on disk (specified with `:file <filename>`) and inserts an Org file link into the results.

### 5.3 Result Formats
- `drawer` (Recommended): Wraps results in `:RESULTS:` … `:END:` delimiters. Keeps documents tidy, especially for multi-line outputs.
- `raw`: Inserts output directly as unescaped Org text (useful for generating dynamic headlines or tables).
- `code`: Wraps output in `#+BEGIN_SRC <lang>` … `#+END_SRC`.
- `link` / `graphics`: Treats the result as a path to an image/file and generates `[[file:path]]`.

### 5.4 Handling
- `replace` (Default): Replaces the previous `#+RESULTS:` block.
- `append`: Appends new results below existing results.
- `prepend`: Inserts new results above existing results.
- `silent`: Displays results in the Emacs echo area / minibuffer without writing to the Org buffer.

---

## 6. Visualizations & Figures in Notebooks

To generate plots and figures that render directly in the notebook:

### 6.1 Python Matplotlib Plot
```org
#+NAME: plot-normal-distribution
#+BEGIN_SRC python :session *py-vis* :results graphics file :file figures/normal_dist.png :exports both
  import matplotlib.pyplot as plt
  import numpy as np

  plt.figure(figsize=(6, 3.5))
  x = np.linspace(-3, 3, 200)
  plt.plot(x, np.exp(-x**2 / 2) / np.sqrt(2 * np.pi), color="#2b5c8f", lw=2)
  plt.title("Standard Normal Distribution")
  plt.xlabel("z")
  plt.ylabel("Density")
  plt.tight_layout()
  plt.savefig("figures/normal_dist.png", dpi=150)
  plt.close()
#+END_SRC

#+RESULTS: plot-normal-distribution
[[file:figures/normal_dist.png]]
```

### 6.2 R ggplot2 Plot
```org
#+NAME: r-scatter-plot
#+BEGIN_SRC R :session *r-vis* :results graphics file :file figures/r_scatter.png :exports both
  library(ggplot2)
  p <- ggplot(mtcars, aes(x=wt, y=mpg, color=factor(cyl))) +
    geom_point(size=3) +
    theme_minimal() +
    labs(title="Fuel Efficiency vs Weight", x="Weight (1000 lbs)", y="MPG")
  ggsave("figures/r_scatter.png", plot=p, width=6, height=4, dpi=150)
#+END_SRC

#+RESULTS: r-scatter-plot
[[file:figures/r_scatter.png]]
```

**Tip:** Run `M-x org-toggle-inline-images` (`C-c C-x C-v`) in Emacs to toggle the display of generated graphics right inside the buffer.

---

## 7. Interactive Execution & Editing Workflow

### Essential Keybindings

| Keybinding | Command | Description |
|---|---|---|
| `C-c C-c` | `org-babel-execute-src-block` | Executes the block under point and inserts/updates results. |
| `C-c '` | `org-edit-special` | Opens the block body in a dedicated major-mode buffer (full LSP, syntax highlighting, autocompletion). Press `C-c '` again to save and return. |
| `C-c C-v b` | `org-babel-execute-buffer` | Executes **all** code blocks in the notebook sequentially (like "Run All Cells"). |
| `C-c C-v s` | `org-babel-execute-subtree` | Executes all code blocks under the current heading. |
| `C-c C-o` | `org-open-at-point` | On a `#+RESULTS:` link, opens the image or generated file in an external viewer or buffer. |
| `M-<down>` | `org-babel-pop-to-session` | Switches focus directly to the interactive REPL buffer for this block's session. |
| `C-c C-v v` | `org-babel-expand-src-block` | Previews the fully expanded block (with all `:var` and noweb expansions resolved). |

---

## 8. Exporting & Publishing Reports

A notebook can be weaved into standalone HTML, PDF, or Markdown documents.

### 8.1 Controlling Export Content with `:exports`
- `:exports code`: Exports only the code block.
- `:exports results`: Exports only the output/figure (hides code, ideal for executive reports).
- `:exports both`: Exports both code and output (ideal for technical documentation).
- `:exports none`: Hides both code and results from the exported document (runs during evaluation if needed).

### 8.2 Controlling Export Evaluation with `:eval`
- `:eval never-export`: Prevents re-running blocks during export, using existing `#+RESULTS:` in the buffer. Speeds up export and prevents non-deterministic changes.
- `:eval query-export`: Prompts before executing during export.

### 8.3 Export Commands
- `C-c C-e h h` (`org-html-export-to-html`): Generates an HTML report.
- `C-c C-e h o` (`org-html-export-to-html-and-open`): Generates and opens HTML in your browser.
- `C-c C-e l p` (`org-latex-export-to-pdf`): Generates a typeset PDF report via LaTeX.

---

## 9. Common Pitfalls & Troubleshooting Checklist

| Symptom / Error | Cause | Fix |
|---|---|---|
| Python block returns no results | Used `:results value` with `print(...)` instead of `return ...` | Replace `print(...)` with `return ...`, or change header to `:results output`. |
| `No org-babel-execute function for <lang>` | Language is not registered in `org-babel-load-languages` | Add `(<lang> . t)` to `org-babel-load-languages` in your Emacs config and evaluate it. |
| Image link is produced but image is not displayed | Inline images are toggled off in Org Mode | Run `M-x org-toggle-inline-images` (`C-c C-x C-v`) or add `#+STARTUP: inlineimages`. |
| Session outputs contain repeated prompts (`>>>`) | Lingering interactive prompt text in REPL buffer | Clean or restart the session buffer (`*session-name*`). |
| Export is slow or hangs | Org is re-executing all notebook blocks during export | Set `:eval never-export` on long-running blocks or globally via `#+PROPERTY: header-args :eval never-export`. |
| Table input produces error on horizontal rules | Horizontal rules (`|-`) treated as raw rows | Add `:hlines no` to the block header to automatically strip table separator lines. |

---

## 10. Complete Annotated Notebook Example

```org
#+TITLE: Solar Flare Frequency Analysis
#+AUTHOR: Dr. Alex Rivera
#+DATE: [2025-05-18 Sun]
#+OPTIONS: toc:nil num:nil
#+STARTUP: inlineimages nohideblocks
#+PROPERTY: header-args :session *solar* :results drawer :exports both

* Introduction
We analyze observational flare frequencies recorded over a 6-month observation window.

* Raw Observations
Here is the baseline measurement table:

#+NAME: raw_observations
| Month | C-Class | M-Class | X-Class |
|-------+---------+---------+---------|
| Jan   |      12 |       3 |       0 |
| Feb   |      19 |       5 |       1 |
| Mar   |      15 |       2 |       0 |
| Apr   |      27 |       8 |       2 |
| May   |      31 |      11 |       3 |
| Jun   |      22 |       7 |       1 |

* Data Processing & Summary Statistics
We compute total solar activity using Python:

#+NAME: compute_totals
#+BEGIN_SRC python :var obs=raw_observations :colnames yes :results value table
  total_events = []
  for row in obs:
      month = row[0]
      total = sum(row[1:])
      total_events.append([month, total])
  return [["Month", "Total Events"]] + total_events
#+END_SRC

#+RESULTS: compute_totals
:RESULTS:
| Month | Total Events |
|-------+--------------|
| Jan   |           15 |
| Feb   |           25 |
| Mar   |           17 |
| Apr   |           37 |
| May   |           45 |
| Jun   |           30 |
:END:

* Visualization
We generate a plot tracking flare trend over time:

#+NAME: plot_flare_trend
#+BEGIN_SRC python :var totals=compute_totals :colnames yes :results graphics file :file figures/flare_trend.png
  import matplotlib.pyplot as plt

  months = [row[0] for row in totals]
  counts = [row[1] for row in totals]

  plt.figure(figsize=(7, 4))
  plt.plot(months, counts, marker='o', color='#d95f02', lw=2)
  plt.title("Solar Event Trends (Jan-Jun 2025)")
  plt.xlabel("Month")
  plt.ylabel("Observed Events")
  plt.grid(True, linestyle='--', alpha=0.6)
  plt.tight_layout()
  plt.savefig("figures/flare_trend.png", dpi=150)
  plt.close()
#+END_SRC

#+RESULTS: plot_flare_trend
:RESULTS:
[[file:figures/flare_trend.png]]
:END:

* Conclusions
Solar activity exhibited a noticeable peak in May 2025.
The peak month had src_python[:session *solar* :results value]{max(row[1] for row in totals)} total detected events.
```
