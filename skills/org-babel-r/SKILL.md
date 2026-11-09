---
name: org-babel-r
description: Complete guide for data science, statistical computing, graphics devices, ESS integration, and reproducible research using R with Org Babel.
---

# R Source Code Blocks in Org Babel

Using R with Org Babel provides an environment for statistical computing, data analysis, graphics generation, and reproducible research papers. R code blocks in Org Mode integrate with **ESS (Emacs Speaks Statistics)** to support interactive execution, multi-block state sharing, publication-quality graphics, and structured table exports.

---

## 1. Setup & Environment Configuration

### 1.1 Prerequisites
1. A working **R** installation (CRAN binary or built from source).
2. **ESS (Emacs Speaks Statistics)** installed via MELPA (required for session-based evaluation and stepping).
3. `ob-R` enabled in `org-babel-load-languages`.

### 1.2 Emacs Configuration Template
Add the following to your `init.el`:

```elisp
;; 1. Configure MELPA
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)
(package-initialize)

;; 2. Ensure ESS is loaded
(unless (package-installed-p 'ess)
  (package-refresh-contents)
  (package-install 'ess))
(require 'ess-r-mode)

;; 3. Enable R in Org Babel
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (R          . t)))

;; 4. Optional: Run blocks without interactive security confirmation prompts
(setq org-confirm-babel-evaluate nil)
```

### 1.3 Windows-Specific Setup
On Microsoft Windows, if Emacs reports `The system cannot find the path specified`, explicitly set the R binary path (including default Babel flags `--slave --no-save`):

```elisp
;; For 64-bit Windows:
(setq org-babel-R-command "C:/Progra~1/R/R-4.3.0/bin/x64/R --slave --no-save")
```
*Note:* When using `:session`, `org-babel-R-command` is ignored and ESS's `inferior-ess-R-program` is used instead.

---

## 2. Execution Modes: Sessions & ESS Integration

### 2.1 Non-Session (`:session none`, default)
Runs in a separate batch R process (`R --slave --no-save`). Clean, isolated, and side-effect free, but re-executes all libraries and data loading every time.

### 2.2 Interactive Sessions (`:session <name>`)
Runs inside an inferior ESS R process (`*<name>*`).
- Datasets, libraries, models, and variables persist across multiple blocks sharing the same session name.
- Enables interactive inspection, debugging, and fast iterative workflows.

```org
#+PROPERTY: header-args:R :session *r-analysis* :results value :exports both
```

### 2.3 Interactive Debugging & Stepping with ESS
When troubleshooting R blocks:
1. **Switch to Session:** With point in an R block, press `M-<down>` (`org-babel-pop-to-session`) or `C-c C-z` (`ess-switch-to-end-of-ESS`) to jump to the R REPL buffer.
2. **Inspect Environment:** Use `ls()`, `str(x)`, `dim(x)`, `summary(x)`, or `head(x)`.
3. **Line-by-Line Stepping:**
   - Press `C-c '` (`org-edit-special`) inside the Org block to enter the dedicated ESS R buffer.
   - Use `C-c C-n` (`ess-eval-line-and-step`) or `C-c C-c` (`ess-eval-region-or-function-or-paragraph`) to step through code line by line in the live R process.
   - Press `C-c '` to return to the Org document.

---

## 3. Results Handling: Data Frames, Values, and Tables

### 3.1 Returning Tabular Data (`:results value table`)
When returning data frames, matrices, or vectors, Org Babel converts them directly into native Org tables:

```org
#+NAME: mtcars_summary
#+BEGIN_SRC R :session *r* :results value table :colnames yes
  aggregate(mpg ~ cyl + gear, data=mtcars, FUN=mean)
#+END_SRC

#+RESULTS: mtcars_summary
| cyl | gear |              mpg |
|-----+------+------------------|
|   4 |    3 |            21.5  |
|   6 |    3 | 19.7500000000000 |
|   8 |    3 | 15.0500000000000 |
|   4 |    4 |            26.92 |
|   6 |    4 |            19.75 |
|   4 |    5 |             28.2 |
|   6 |    5 |             19.7 |
|   8 |    5 |            15.4  |
```

- Always add `:colnames yes` when working with data frames to preserve and display column names correctly.
- In R, the value of the last evaluated expression is automatically returned (no explicit `return()` required).

### 3.2 Capturing Printed Console Output (`:results output`)
Use `:results output` to capture stdout, model printouts, or summary statistics:

```org
#+BEGIN_SRC R :session *r* :results output
  fit <- lm(mpg ~ wt + hp, data=mtcars)
  summary(fit)$coefficients
#+END_SRC

#+RESULTS:
:               Estimate Std. Error   t value     Pr(>|t|)
: (Intercept) 37.2272694 1.59878754 23.284688 2.565459e-20
: wt          -3.8778307 0.63273349 -6.128695 1.119647e-06
: hp          -0.0317729 0.00902971 -3.518708 1.451229e-03
```

---

## 4. Graphics & Visualizations

### 4.1 Base Graphics Rule (Org $\ge$ 9.3)
To produce graphics, specify **both** `file` and `graphics` in `:results`, plus the output filename in `:file`:

```org
:results graphics file :file <path.ext>
```
Supported file extensions and corresponding R devices:
- Vector: `.pdf`, `.svg`, `.ps` / `.postscript`, `.tex` (TikZ)
- Raster: `.png`, `.jpg` / `.jpeg`, `.tiff`, `.bmp`
- Default fallback: `.png`

```org
#+BEGIN_SRC R :file figures/base_plot.png :results graphics file :width 600 :height 400 :res 120
  plot(mtcars$wt, mtcars$mpg, pch=19, col="darkblue",
       xlab="Weight (1000 lbs)", ylab="Miles per Gallon",
       main="Fuel Economy")
  abline(lm(mpg ~ wt, data=mtcars), col="red", lwd=2)
#+END_SRC

#+RESULTS:
[[file:figures/base_plot.png]]
```

### 4.2 The Critical ggplot2 / Lattice "Grid Graphics" Rule
**Trap:** Lattice (`xyplot`) and ggplot2 functions return graphical *objects*. In non-interactive batch evaluation (non-session), returning the object without printing it creates an empty/corrupted graphics file!

There are **three reliable ways** to render ggplot2 / Lattice plots in Org Babel:

#### Method A: Explicit `print()` (Recommended for non-session)
```org
#+BEGIN_SRC R :file figures/dist.png :results graphics file
  library(ggplot2)
  p <- ggplot(mtcars, aes(x=mpg)) + geom_histogram(bins=10, fill="steelblue")
  print(p)  # Explicit print() triggers the graphics device rendering
#+END_SRC
```

#### Method B: Use `:session` (Auto-prints interactively)
```org
#+BEGIN_SRC R :session *r* :file figures/dist.png :results graphics file
  library(ggplot2)
  ggplot(mtcars, aes(x=mpg)) + geom_histogram(bins=10, fill="steelblue")
#+END_SRC
```

#### Method C: Use `:results output graphics file`
```org
#+BEGIN_SRC R :file figures/dist.png :results output graphics file
  library(lattice)
  xyplot(mpg ~ wt, data=mtcars)
#+END_SRC
```

---

## 5. R Graphics Device Header Arguments

Org Babel provides first-class header arguments mapping directly to parameters of R graphics devices (`png()`, `pdf()`, `svg()`, etc.):

| Header Argument | Default | Description & Allowed Values |
|---|---|---|
| `:width` | 7 (in) / 480 (px) | Device width. |
| `:height` | 7 (in) / 480 (px) | Device height. |
| `:units` | `px` (raster), `in` (PDF) | Units: `"px"`, `"in"`, `"cm"`, `"mm"`. |
| `:res` | 72 | Nominal resolution in pixels per inch (DPI). (Essential for crisp raster plots: use 150-300). |
| `:bg` | `"white"` | Background color (e.g. `"transparent"`, `"#f7f7f7"`). |
| `:fg` | `"black"` | Foreground color. |
| `:pointsize` | 12 | Base point size for text rendering. |
| `:family` | `"Helvetica"` | Font family (`"Helvetica"`, `"Times"`, `"Palatino"`, `"Courier"`, `"sans"`, `"serif"`, `"mono"`). |
| `:quality` | 75 | JPEG quality percentage (1-100). |
| `:type` | Platform default | Bitmap renderer: `"cairo"`, `"quartz"`, or `"Xlib"`. |
| `:paper` | `"special"` | PDF paper size: `"special"`, `"a4"`, `"letter"`, `"a4r"`, etc. |
| `:R-dev-args` | `nil` | Raw string of additional R device arguments passed directly to the device function. |

### Example: High-DPI Cairo SVG/PNG with Custom Dimensions
```org
#+NAME: custom_styled_plot
#+HEADER: :width 8 :height 5 :units in :res 300 :type cairo
#+HEADER: :R-dev-args bg="#fcfcfc"
#+BEGIN_SRC R :session *r* :file figures/styled_plot.png :results graphics file
  library(ggplot2)
  ggplot(diamonds[1:1000, ], aes(x=carat, y=price, color=cut)) +
    geom_point(alpha=0.6) +
    theme_minimal(base_family="sans") +
    labs(title="Diamond Prices by Carat and Cut")
#+END_SRC
```

### 5.3 TikZ LaTeX Graphics for Matching Document Fonts
To generate plots where axis labels and mathematical annotations share the exact same LaTeX font as your document:

```org
#+BEGIN_SRC R :file figures/model_plot.tex :results graphics file
  plot(1:10, type="b", main="\\textbf{TikZ Plot with Math: } $\\int_0^1 x^2 dx$")
#+END_SRC
```
When exported to LaTeX/PDF, the `.tex` file is included natively via `\input{figures/model_plot.tex}`!

---

## 6. Advanced Formatted Outputs (ASCII, Org, & LaTeX)

### 6.1 Formatted Org Output with the `ascii` Package
The R package `ascii` can coerce complex non-tabular R objects (like `describe()`, `table()`, or `summary()`) into native Org syntax:

```org
#+BEGIN_SRC R :session *r* :results output org
  library(ascii)
  options(asciiType = "org")
  tab <- table(mtcars$cyl, mtcars$gear)
  ascii(tab, caption="Cylinders vs Forward Gears")
#+END_SRC

#+RESULTS:
#+BEGIN_ORG
#+CAPTION: Cylinders vs Forward Gears
|   | 3  | 4 | 5 |
|---+----+---+---|
| 4 | 1  | 8 | 2 |
| 6 | 2  | 4 | 1 |
| 8 | 12 | 0 | 2 |
#+END_ORG
```

### 6.2 Publication Tables with `xtable` for LaTeX Export
For documents targeting PDF via LaTeX export, use `xtable` with `:results output latex`:

```org
#+BEGIN_SRC R :results output latex :exports results
  library(xtable)
  fit <- lm(mpg ~ wt + hp, data=mtcars)
  print(xtable(summary(fit), caption="Regression Estimates", label="tab:reg"),
        comment=FALSE)
#+END_SRC

#+RESULTS:
#+BEGIN_LaTeX
\begin{table}[ht]
\centering
\begin{tabular}{rrrrr}
  \hline
 & Estimate & Std. Error & t value & Pr($>$$|$t$|$) \\ 
  \hline
(Intercept) & 37.2273 & 1.5988 & 23.28 & 0.0000 \\ 
  wt & -3.8778 & 0.6327 & -6.13 & 0.0000 \\ 
  hp & -0.0318 & 0.0090 & -3.52 & 0.0015 \\ 
   \hline
\end{tabular}
\caption{Regression Estimates} 
\label{tab:reg}
\end{table}
#+END_LaTeX
```

---

## 7. Troubleshooting & Common Pitfalls

| Issue | Cause | Solution |
|---|---|---|
| ggplot2 or lattice produces blank/corrupt image | Returned graphics object was not explicitly printed in non-session mode | Wrap plot in `print(p)`, or switch to `:session`, or add `output` to `:results output graphics file`. |
| Graphics file produced but no link inserted | Missing `file` or `graphics` keyword | Ensure header has `:results graphics file` (both words required in Org $\ge$ 9.3). |
| `The system cannot find the path specified` (Windows) | R executable not found in PATH or custom location | Set `(setq org-babel-R-command "path/to/R --slave --no-save")` in `init.el`. |
| Columns converted to row 0 in R data frames | Table column headers treated as observation data | Add `:colnames yes` to the block header arguments. |
| In-session R hangs on large computation | Block is blocking the Emacs process | Switch to inferior ESS buffer (`C-c C-z`) to monitor progress or interrupt with `C-c C-c`. |
| Blurry plots in exported documents | Default raster DPI (72) is too low | Add `:res 300` and adjust `:width` and `:height` accordingly. |
