title: R Source Code Blocks in Org Mode
author: Eric Schulte, Dan Davison, Tom Dye, Tyler Smith

[ UP ](https://orgmode.org/worg/org-contrib/babel/languages/index.html) | [ HOME ](https://orgmode.org/worg/)

Support via [Liberapay](https://liberapay.com/org-mode "Donations always help.")

# R Source Code Blocks in Org Mode

## Table of Contents

- [Introduction](#org1026891)
- [Requirements and Setup](#org373dba4)
    - [Note for Windows Users](#orgcdb7860)
- [Org Mode Features for R Source Code Blocks](#org42b8dd7)
    - [Header Arguments](#org4254f38)
    - [Sessions](#org323c940)
    - [Result Types](#org22af9ce)
- [Examples of Use](#org96b93b6)
    - [Debugging](#org3712a69)
    - [Org Mode Output from R](#org3b0e925)
    - [LaTeX code from R](#org2917f9b)
    - [`ess-switch-to-end-of-ESS`](#orga6a88f8)

 Org Mode support for [R](http://www.r-project.org/) 

 [ ![](http://www.r-project.org/Rlogo.jpg) ](http://www.r-project.org/) 

## Introduction {#org1026891}

 R is a free software environment for statistical computing and graphics. R source code blocks are fully supported in Org Mode with a wide variety of R-specific header arguments. 

 R source code blocks in Org Mode can be used to create R packages, carry out statistical analyses, create graphic displays, and produce reproducible research papers. 

## Requirements and Setup {#org373dba4}

 R source code blocks in Org Mode require a working R installation. Precompiled binary distributions for Gnu/Linux, Mac OS X, and Windows are available at the [Comprehensive R Archive Network](http://cran.r-project.org), which also makes available the source code for compilation on other platforms. 

 The Emacs mode, [Emacs Speaks Statistics (ESS)](http://ess.r-project.org/), is required for session-based evaluation. ESS is designed to support editing of scripts and interaction with various statistical analysis programs such as R, S-Plus, SAS, Stata, and JAGS. 

 You can install ESS from the [MELPA repository](https://melpa.org/), using Emacs' package facility. This is usually the easiest way to install it; if you need to, instructions for installing from source are found at the [ESS website](https://ess.r-project.org/index.php?Section=download). 

 To install from Melpa, you need to add it to your `package-archives`. The MELPA maintainers recommend the following code: 

```
(require 'package)
(let* ((no-ssl (and (memq system-type '(windows-nt ms-dos))
                    (not (gnutls-available-p))))
       (proto (if no-ssl "http" "https")))
  (when no-ssl (warn "\
Your version of Emacs does not support SSL connections,
which is unsafe because it allows man-in-the-middle attacks.
There are two things you can do about this warning:
1. Install an Emacs version that does support SSL and be safe.
2. Remove this warning from your init file so you won't see it again."))
  (add-to-list 'package-archives (cons "melpa" (concat proto "://melpa.org/packages/")) t)
  ;; Comment/uncomment this line to enable MELPA Stable if desired.  See `package-archive-priorities`
  ;; and `package-pinned-packages`. Most users will not need or want to do this.
  ;;(add-to-list 'package-archives (cons "melpa-stable" (concat proto "://stable.melpa.org/packages/")) t)
  )
(package-initialize)
```

 Note that the minimal requirement is just to have the line 

```
(add-to-list 'package-archives (cons "melpa" (concat proto "://melpa.org/packages/")) t)
```

 at the appropriate place in your configuration (i.e., before you call `package-initialize`). 

 You also need to ensure that `org-babel-load-languages` includes an entry for R. Typically, `org-babel-load-languages` will contain many entries. The example below omits other languages. 

```
(org-babel-do-load-languages
 'org-babel-load-languages
 '((R . t)))
```

### Note for Windows Users {#orgcdb7860}

 Additional steps may be necessary on Windows. See [this Org-mode mailing list thread](http://www.mail-archive.com/emacs-orgmode@gnu.org/msg57159.html) for background information. If when trying to execute R source code from Org-mode one encouters the message `The
system cannot find the path specified`, it may be necessary to set the variable `org-babel-R-command` in .emacs. Note the addition of `--slave --no-save` to the custom path, which are the R defaults for Babel. 

```
(setq org-babel-R-command "C:/Progra~1/R/R-2.15.0/bin/R --slave --no-save")
```

 Adjust the path appropriately based on your R installation. 

 For 64bit Windows users, the R installation process may have installed both 32bit and 64bit binaries. The above path points to the 32bit paths; for 64bit operating systems the analog is: 

```
(setq org-babel-R-command "C:/Progra~1/R/R-2.15.0/bin/x64/R --slave --no-save")
```

## Org Mode Features for R Source Code Blocks {#org42b8dd7}

### Header Arguments {#org4254f38}

 There are no R-specific *default* header arguments. 

 The `:colnames` header argument is used to specify that both **input** tables (via a `:var` header argument), and output tables (via `:results`) (should) have headers. 

 If a `:file filename.ext` header argument is provided to an R source block, then the output from the source block will go to the named file. What that output is depends on the value of the `:results` header argument. 

 If the value is `:results file graphics` then "base" graphics output is captured to the file specified by the `:file` argument. A link to the file is inserted into the Org Mode buffer. ****NB: as of org version 9.3, the `:results` argument must include both `file` and `graphics`; earlier versions only required the `graphics` argument****. An attempt is made to find an R graphics device corresponding to the file extension. Currently, the following extensions are recognized: `.png`, `.jpg`, `.jpeg`, `.tiff`, `.bmp`, `.pdf`, `.ps`, `.postscript`, `.tex`, and `.svg`. If the extension of the file name passed to `:file` is not recognized, PNG format output is created by default. 

 If the source code block uses grid-based R graphics, e.g., the lattice and ggplot2 packages, then care must be taken either to print() the graphics object, specify `:results output`, or run the code in a `:session`. This is because the graphics functions from lattice and ggplot2 return objects that must be explicitly printed to see them, using the print function. This happens automatically when run interactively, e.g., `:session`, but when called inside another function, it does not. 

 Some years ago, Erik Iverson summarized the different ways of getting this working. His summary, updated to reflect a change in Org Babel syntax, follows: 

```

* does /not/ work; produces a file, but it does
* not contain a valid graphics format
#+begin_src R :file 1.png :results file graphics
library(lattice)
xyplot(1:10 ~ 1:10)
#+end_src
```

```

* does produce a file, by printing object
#+begin_src R :file 2.png :results graphics file
library(lattice)
print(xyplot(1:10 ~ 1:10))
#+end_src
```

```

* does produce a file, by using :results output
#+begin_src R :file 3.png :results output graphics file
library(lattice)
xyplot(1:10 ~ 1:10)
#+end_src
```

```

* does produce a file, by evaluating in :session
#+begin_src R :file 4.png :session :results graphics file
library(lattice)
xyplot(1:10 ~ 1:10)
#+end_src
```

#### Graphics Header Arguments {#orgc2d802f}

 There are many R-specific header arguments used to configure R graphics devices. They include: 

width
:   the width of the graphics region, default value is 7 (inches) or 480 (pixels)
height
:   the height of the graphics region, default value is 7 (inches) or 480 (pixels)
units
:   the units in which width and height are given – `px`, `in`, `cm`, or `mm`. Note that the default units are set by the file type: `in` for pdf and ps, `px` for jpeg, bmp, png, tiff
bg
:   the background color defaults to "white"
fg
:   the foreground color defaults to "black"
pointsize
:   the default point size in the graphics defaults to 12
quality
:   the quality of a JPEG image as a percentage
compression
:   the type of compression to be used
res
:   the nominal resolution in pixels per inch
type
:   the bitmap type, one of "Xlib", "quartz", or "cairo"
antialias
:   the type of antialiasing to be used when `type` \= "cairo" or `type` \= "quartz"
family
:   in normal use, one of "AvantGarde", "Bookman", "Courier", "Helvetica" (default), "Helvetica-Narrow", "NewCenturySchoolbook", "Palatino", or "Times"
title
:   string to embed as the /Title field in the file defaults to "R Graphics Output"
fonts
:   an R graphics font family name – "sans", "serif", or "mono"
version
:   string describing the PDF version required to view the output defaults to "1.4"
paper
:   the target paper size – "special" (default), "default", "a4", "letter", "legal", "us", "executive", "a4r", or "USr", where the latter two are rotated to landscape orientation
encoding
:   the name of an encoding file
pagecentre
:   if paper !\= "special" then a logical that defaults to true and determines whether the graphic device region is centered on the page
colormodel
:   a character string describing the color model, "srgb" (default), "gray", "grey", or "cmyk".
useDingbats
:   if TRUE (default) small circles will be rendered with the Dingbats font
horizontal
:   for the postscript device, a logical that defaults to true and dtermines the orientation of the printed image
R-dev-args
:   for graphics parameters not directly supported by Org Mode (see below)

 See the R help page for the graphics devices (e.g., using `?png`, `?pdf`, `?postscript` in an R session) for additional information on these arguments. 

 Arguments to the R graphics device can also be passed as a string in R argument syntax, using the header arg `:R-dev-args`. This is useful for graphics device arguments that don't have an Org Mode header argument counterpart. 

 The following example source block illustrates use of `:R-dev-args` to pass background and foreground colors. Note that both of these arguments can also be passed directly as header args, using `:fg` and `:bg`. 

```
#+header: :width 8 :height 8 :R-dev-args bg="olivedrab", fg="hotpink"
#+begin_src R :file z.pdf :results graphics file
,plot(matrix(rnorm(100), ncol=2), type="l")
#+end_src
```

### Sessions {#org323c940}

 Sessions are fully supported by R source code blocks. They can be used as one way to preserve state accessed by several source code blocks. Sessions are also useful for debugging, since it is possible to view the values of variables created during the session. 

 `org-babel-R-command` is ignored when using sessions. Instead, ESS-specific setting - `inferior-ess-R-program` - is used. 

### Result Types {#org22af9ce}

 R source code blocks can return text or graphical results. 

 The [ascii package](http://cran.r-project.org/web/packages/ascii/index.html) coerces R objects to Org Mode, among other markup languages. The [Hmisc](http://cran.r-project.org/web/packages/Hmisc/index.html), [xtable](http://cran.r-project.org/web/packages/xtable/index.html) and [tables](http://cran.r-project.org/web/packages/tables/index.html) packages contain functions to write R objects into LaTeX representations. 

 R is capable of creating graphical displays in several formats. The outputs supported by R source code blocks in Org Mode include: 

bmp
:   bitmap image file format commonly used on Microsoft Windows and OS/2
jpg, jpeg
:   Joint Photographics Expert Group method of lossy compression for digital photography widely used in a number of raster image file formats
tex
:   output tikz graphics language which can be typeset by LaTeX so the fonts for text in the plot match the fonts used in the LaTeX document
tiff
:   a sophisticated raster image format that allows multiple pages in a document
png
:   Portable Network Graphics is a lossless raster image file format
svg
:   Scalable Vector Graphics is an open standard vector format that can be embedded in web pages and readily edited in open source software applications such as [Inkscape](http://inkscape.org/)
pdf
:   Portable Document Format can faithfully produce anything R graphics can output
ps, postscript
:   PostScript is a predecessor of PDF that does not support semitransparent colors or hyperlinking

 When using R to produce graphical displays, you will typically set `:results graphics file`. However, if you use the [ggplot implementation of the grammar of graphics in R](http://had.co.nz/ggplot/), then you will need to set `:results
output graphics file` (see above). 

## Examples of Use {#org96b93b6}

### Debugging {#org3712a69}

 This section contains some tips on how to proceed if your R code is not doing what you had hoped. 

#### Use `:session` {#orga8f185e}

 Evaluate your code using the `:session` header argument, then visit the R buffer (i.e. the buffer containing the "inferior ESS" process). Then you can inspect the objects that have been created, and try out some lines of code. Useful R functions for inspecting objects include (in R, type a "?" followed by the name of the function, or use `C-c C-v` to use ESS's help browser, to get help with the function) 

- str
- dim
- summary

#### Use ESS to step through evaluation line-by-line {#org914b90f}

1. Use C-c ' to visit the edit buffer for your code block
2. Use `ess-eval-line-and-step` to evaluate each line in turn

 In addition to `ess-eval-line-and-step`, there are several other ESS functions with names beginning `ess-eval-*`. They evaluate lines and regions in different ways; it's worth looking at their descriptions (C-h f). 

### Org Mode Output from R {#org3b0e925}

 David Hajage's [ascii](http://cran.r-project.org/web/packages/ascii/index.html) R package creates appropriate plain text representations of many R objects. It features an option to specify that the plain text representations should be in Org format. This can be particularly useful for retrieving non-tabular R data structures in Org Mode for export. 

 In R: 

```

> library(ascii)
> options(asciiType = "org")
> library(Hmisc)
> ascii(describe(esoph))
#+CAPTION: esoph
- 5 Variable
- 88 Observations

*agegp*
|  n | missing | unique |
| 88 |       0 |      6 |

|           | 25-34 | 35-44 | 45-54 | 55-64 | 65-74 | 75+ |
| Frequency |    15 |    15 |    16 |    16 |    15 |  11 |
| %         |    17 |    17 |    18 |    18 |    17 |  12 |

*alcgp*
|  n | missing | unique |
| 88 |       0 |      4 |

 0-39g/day (23, 26%), 40-79 (23, 26%), 80-119 (21, 24%), 120+ (21, 24%)

*tobgp*
|  n | missing | unique |
| 88 |       0 |      4 |

 0-9g/day (24, 27%), 10-19 (24, 27%), 20-29 (20, 23%), 30+ (20, 23%)

*ncases*
|  n | missing | unique |  Mean | .05 | .10 | .25 | .50 | .75 | .90 | .95 |
| 88 |       0 |     10 | 2.273 | 0.0 | 0.0 | 0.0 | 1.0 | 4.0 | 5.3 | 6.0 |

|           |  0 |  1 |  2 |  3 | 4 | 5 | 6 | 8 | 9 | 17 |
| Frequency | 29 | 16 | 11 |  9 | 8 | 6 | 5 | 1 | 2 |  1 |
| %         | 33 | 18 | 12 | 10 | 9 | 7 | 6 | 1 | 2 |  1 |

*ncontrols*
|  n | missing | unique |  Mean | .05 | .10 | .25 | .50 |  .75 |  .90 |  .95 |
| 88 |       0 |     30 | 11.08 | 1.0 | 1.0 | 3.0 | 6.0 | 14.0 | 29.1 | 40.0 |

 lowest:  1  2  3  4  5, highest: 40 46 48 49 60
```

 The Org Mode source code block specifies `:results org` so the output is wrapped in `#+BEGIN_ORG` … `#+END_ORG`. This way, arbitrary output can be included and easily replaced on subsequent evaluations of the source code block. 

```

#+begin_src R :results output org
  library(ascii)
  options(asciiType="org")
  ascii(summary(table(1:4, 1:4)))
#+end_src

#+results:
#+BEGIN_ORG
- Number of cases in table: 4
- Number of factors: 2
- Test for independence of all factors:
  - Chisq = 12, df = 9, p-value = 0.2133
  - Chi-squared approximation may be incorrect
#+END_ORG
```

 The results in this case are exported as a nested list structure: 

- Number of cases in table: 4
- Number of factors: 2
- Test for independence of all factors:
    - Chisq \= 12, df \= 9, p-value \= 0.2133
    - Chi-squared approximation may be incorrect

 The `caption`, `header`, and `include.colnames` options are useful. 

```

#+BEGIN_SRC R :results output org
 library(ascii)
 a <- runif(100)
 c <- "Quantiles of 100 random numbers"
 b <- ascii(quantile(a),header=T,include.colnames=T,caption=c)
 print(b,type="org")
 rm(a,b,c)
#+END_SRC

#+RESULTS:
#+BEGIN_ORG
#+CAPTION: Quantiles of 100 random numbers
| 0%   | 25%  | 50%  | 75%  | 100% |
|------+------+------+------+------|
| 0.03 | 0.28 | 0.52 | 0.74 | 1.00 |
#+END_ORG
```

 The output exported to HTML can be quite nice. 

[Table 1: Quantiles of 100 random numbers]
| 0% | 25% | 50% | 75% | 100% |
|----|----|----|----|----|
| 0.03 | 0.28 | 0.52 | 0.74 | 1.00 |

### LaTeX code from R {#org2917f9b}

 This example summarises a linear regression fit. Usually the Org Mode user should not have to be involved in LaTeX code generation, because this is the responsibility of Org Mode's LaTeX export engine. In this example, neither the printed representation, nor the value of `summary(lm(y ~ x))` is tabular, and it would therefore require some work to get the information in to an Org Mode table. However, the `xtable` package can be used to output a LaTeX table. Using `:results
latex` as a header argument to the R source code block ensures that this is returned as a LaTeX block in the Org Mode buffer and thus can be included correctly in LaTex-based export targets. 

```

#+begin_src R :results output latex
library(xtable)
x <- rnorm(100)
y <- x + rnorm(100)
xtable(summary(lm(y ~ x)))
#+end_src
```

```

#+results:
#+BEGIN_LaTeX
% latex table generated in R 2.9.2 by xtable 1.5-5 package
% Wed Dec  9 17:17:53 2009
\begin{table}[ht]
\begin{center}
\begin{tabular}{rrrrr}
  \hline
 & Estimate & Std. Error & t value & Pr($>$$|$t$|$) \\
  \hline
(Intercept) & -0.0743 & 0.0969 & -0.77 & 0.4454 \\
  x & 1.0707 & 0.0923 & 11.60 & 0.0000 \\
   \hline
\end{tabular}
\end{center}
\end{table}
#+END_LaTeX
```

### `ess-switch-to-end-of-ESS` {#orga6a88f8}

 When in an Org Mode R code edit buffer with an associated R session, `M-x ess-switch-to-end-of-ESS` will bring the R session buffer into view and place point at the prompt. ESS binds this to `C-c C-z` and `C-M-r` by default. 

Documentation from the orgmode.org/worg/ website (either in its HTML format or in its Org format) is licensed under the [GNU Free Documentation License version 1.3](http://www.gnu.org/copyleft/fdl.html) or later. The code examples and css stylesheets are licensed under the [GNU General Public License v3](http://www.gnu.org/licenses/gpl.html) or later.
