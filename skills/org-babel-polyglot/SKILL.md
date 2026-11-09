---
name: org-babel-polyglot
description: Guide for building multi-language data analysis pipelines, cross-block variable passing, table formulas with org-sbe, and remote block calls in Org Babel.
---

# Polyglot Data Pipelines & Interoperability with Org Babel

One of Org Babel's most powerful capabilities is **language-agnostic interoperability**: data can flow seamlessly across different programming languages, Org tables, and formulas within the same document.

A single notebook can query data in **SQL**, transform and clean it in **Python**, perform statistical modeling in **R**, execute system commands in **Shell**, and visualize results in **Gnuplot** or **Matplotlib** — with Org Mode acting as the universal data bus.

---

## 1. The Universal Data Passing Mechanism: `:var`

The `:var` header argument has identical syntax across all supported languages:

```org
:var VARIABLE_NAME=ASSIGNMENT
:var VARIABLE_NAME=FILE:REFERENCE
```

### 1.1 Types of Assignments Supported by `:var`

| Input Source | Syntax Example | Notes |
|---|---|---|
| **Literal Value** | `:var n=42 text="sample"` | Numbers, booleans, and strings. |
| **Named Org Table** | `:var data=raw_measurements` | Passes the 2D table as a nested list/data frame. |
| **Named Plain List** | `:var items=todo_list` | Passes top-level list items as an array/vector. |
| **Code Block without Args** | `:var stats=compute_stats()` | Runs `compute_stats` and passes its return value. |
| **Code Block with Args** | `:var res=calc(rate=0.05, n=12)` | Evaluates target block with specified parameters. |
| **Block Contents / Literal** | `:var template=config_template[]` | Passes verbatim body of an example or code block. |
| **Emacs Lisp Evaluation** | `:var file=(buffer-file-name)` | Any value starting with `(`, `[`, `'` or `` ` `` is evaluated as Elisp. |
| **External Org File** | `:var ext=other.org:data_table` | Resolves reference inside an external file. |

---

## 2. Passing and Formatting Org Tables

Org tables are converted automatically into native structures (Python lists of lists, R data frames, etc.). Header arguments control table preprocessing:

### 2.1 Table Preprocessing Arguments

- `:colnames yes | no | nil` (Default: `nil`):
  - When `nil`: If the table has a horizontal rule under row 1, Babel strips headers, converts rows, and re-applies headers to returned tables.
  - When `yes`: Explicitly treats the first row as column names / headers.
  - When `no`: Treats the first row as normal data.
- `:rownames yes | no` (Default: `no`):
  - When `yes`: Strips the first column before processing and restores it as row names on return.
- `:hlines yes | no` (Default: `no`):
  - When `no` (recommended): Strips all horizontal rules (`|-...`) before passing data to the interpreter. This prevents type errors (such as `None` or `'hline'` strings crashing numeric routines).
  - When `yes`: Leaves horizontal rules in place as `'hline` symbols.

### 2.2 Slicing and Indexing Tables in `:var`

You can slice rows and columns directly in the `:var` declaration using 0-based index notation `[rows, columns]`:

```org
#+NAME: full_dataset
| ID | Age | Score |
|----+-----+-------|
|  1 |  25 |  88.5 |
|  2 |  30 |  92.0 |
|  3 |  22 |  79.0 |
|  4 |  35 |  95.5 |

# Passing only the 'Score' column (column index 2, rows 1 through end):
#+BEGIN_SRC python :var scores=full_dataset[1:, 2] :results value
  return sum(scores) / len(scores)
#+END_SRC

#+RESULTS:
: 88.75
```

- Indexing syntax:
  - `data[0]` : First row.
  - `data[,1]` : Second column across all rows.
  - `data[1:3, :]` : Rows 1 through 3, all columns (inclusive).
  - `data[-1]` : Last row.
  - `data[*]` or `data[0:-1]` : All rows.
- **Important:** Indexing occurs **before** `:colnames` or `:hlines` stripping.

---

## 3. Polyglot Pipeline Pattern: SQL -> Python -> R -> Graphics

Here is a complete multi-language data workflow:

```org
#+TITLE: Multi-Language Pipeline

* Step 1: Database Query (SQLite)
Extract raw event counts:

#+NAME: raw_events
#+BEGIN_SRC sqlite :db telemetry.db :results table :colnames yes
  SELECT device_id, event_type, count(*) as count
  FROM events
  WHERE event_date >= '2025-01-01'
  GROUP BY device_id, event_type;
#+END_SRC

#+RESULTS: raw_events
| device_id | event_type | count |
|-----------+------------+-------|
| dev-01    | heartbeat  |  1420 |
| dev-01    | alert      |    12 |
| dev-02    | heartbeat  |  1380 |
| dev-02    | alert      |    45 |

* Step 2: Data Cleaning & Normalization (Python)
Consume SQLite table into Python:

#+NAME: clean_telemetry
#+BEGIN_SRC python :var events=raw_events :colnames yes :results value table
  cleaned = []
  for row in events:
      dev_id, ev_type, count = row[0], row[1], int(row[2])
      rate = round(count / 1440.0, 4)  # events per minute
      cleaned.append([dev_id, ev_type, count, rate])
  return [["Device", "Event", "Count", "RatePerMin"]] + cleaned
#+END_SRC

#+RESULTS: clean_telemetry
| Device | Event     | Count | RatePerMin |
|--------+-----------+-------+------------|
| dev-01 | heartbeat |  1420 |     0.9861 |
| dev-01 | alert     |    12 |     0.0083 |
| dev-02 | heartbeat |  1380 |     0.9583 |
| dev-02 | alert     |    45 |     0.0313 |

* Step 3: Statistical Modeling (R)
Consume cleaned Python output into R:

#+NAME: r_stat_summary
#+BEGIN_SRC R :var df=clean_telemetry :colnames yes :results value table
  summary_df <- aggregate(RatePerMin ~ Event, data=df, FUN=mean)
  colnames(summary_df) <- c("Event", "AvgRatePerMin")
  summary_df
#+END_SRC

#+RESULTS: r_stat_summary
| Event     | AvgRatePerMin |
|-----------+---------------|
| alert     |        0.0198 |
| heartbeat |        0.9722 |
```

---

## 4. Calling Blocks Programmatically: `#+CALL:` and Inline `call_`

You can treat any named block as a reusable function and invoke it without rewriting the block.

### 4.1 Syntax

```org
# Block form:
#+CALL: <name>[<inside_eval_headers>](<arguments>) <end_result_headers>

# Inline form:
... call_<name>[<inside_eval_headers>](<arguments>)[<end_result_headers>] ...
```

- `<name>`: The `#+NAME:` of the target block.
- `<arguments>`: Function-style parameters, e.g., `(n=5, metric="accuracy")`.
- `[<inside_eval_headers>]`: Applied during execution (e.g., `[:results output]`).
- `[<end_result_headers>]`: Applied to formatting the returned result (e.g., `:results raw`).

### 4.2 Example

```org
#+NAME: power_calc
#+BEGIN_SRC python :var base=2 exp=3 :results value
  return base ** exp
#+END_SRC

# Calling with custom arguments:
#+CALL: power_calc(base=3, exp=4)

#+RESULTS:
: 81

# Inline invocation:
The total capacity is call_power_calc(base=2, exp=8) units.
```

---

## 5. Reactive Spreadsheets: Babel in Table Formulas (`org-sbe`)

Org Mode tables have a built-in spreadsheet calculator (`#+TBLFM:`). The `org-sbe` macro allows any table cell formula to call an arbitrary Babel block in any language:

```org
(org-sbe "block-name" (param1 value1) (param2 value2))
;; Shorthand:
(sbe "block-name" (param1 value1))
```

### Example: Table Powered by Python & R Formulas

```org
#+NAME: py_tax_rate
#+BEGIN_SRC python :var country="US" :results value
  rates = {"US": 0.08, "UK": 0.20, "DE": 0.19}
  return rates.get(country, 0.0)
#+END_SRC

#+NAME: invoice_table
| Item        | Country | Base Price | Tax Rate | Total Price |
|-------------+---------+------------+----------+-------------|
| Server Node | US      |       1200 |     0.08 |      1296.0 |
| Storage SAN | UK      |       3000 |      0.2 |      3600.0 |
| Gateway     | DE      |        800 |     0.19 |       952.0 |
#+TBLFM: $4='(org-sbe "py_tax_rate" (country $"$2")) :: $5=$3 * (1 + $4)
```

- In the table, pressing `C-u C-c C-c` on the `#+TBLFM` line recalculates all formulas, evaluating the Python block dynamically for every row!
- `$"$2"` passes the value of column 2 as a string literal.

---

## 6. Result Post-Processing with `:post`

The `:post` header argument chains a block's output through a secondary formatting block before it is inserted into the document. The output of the primary block is bound to `*this*`:

```org
# Post-processor block that adds custom LaTeX export attributes:
#+NAME: wrap_latex_figure
#+BEGIN_SRC emacs-lisp :var data="" caption="Figure"
  (format "#+CAPTION: %s\n#+ATTR_LATEX: :width 0.8\\textwidth\n%s" caption data)
#+END_SRC

# Primary plotting block using :post:
#+BEGIN_SRC python :results file link :file figures/out.pdf :post wrap_latex_figure(caption="Model Accuracy", data=*this*)
  import matplotlib.pyplot as plt
  plt.plot([1, 2, 3], [4, 5, 6])
  plt.savefig("figures/out.pdf")
  plt.close()
  return "figures/out.pdf"
#+END_SRC
```

---

## 7. The Library of Babel: Shared Global Function Libraries

The **Library of Babel** lets you import and share named blocks across multiple distinct `.org` files without duplicating code.

1. Create a shared library file (e.g., `~/org/shared_babel.org`) containing named blocks.
2. Ingest the library in your Emacs session:
   ```elisp
   (org-babel-lob-ingest "~/org/shared_babel.org")
   ```
   Or via keyboard shortcut: `C-c C-v i` (`org-babel-lob-ingest`).
3. Call any block from that file using `#+CALL: block_name(args)` in any buffer!

---

## 8. Common Polyglot Pitfalls & Best Practices

| Pitfall | Consequence | Remedy |
|---|---|---|
| Mismatched column headers | Numeric calculations fail because header strings are read as row 0 data | Always specify `:colnames yes` on blocks consuming tabular inputs. |
| Table horizontal rules (`|-`) causing crashes | Python/R receives `'hline'` symbol where number was expected | Use `:hlines no` to strip separators automatically. |
| Unnamed blocks in `:var` | `Reference 'xyz' not found in buffer` error | Ensure the input block or table has a matching `#+NAME: xyz` keyword directly above it. |
| Quoting table cell variables in `org-sbe` | Text cells without quotes are read as undefined Lisp symbols | Wrap column references in strings: `(country $"$2")`. |
| Indexing order with headers | `data[0]` returns header row when indexing | Note that table slicing happens *before* `:colnames` stripping. |
