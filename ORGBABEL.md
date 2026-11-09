---
Title: Introducing Babel
URL: https://orgmode.org/worg/org-contrib/babel/intro.html
Author: Eric Schulte, Dan Davison, Tom Dye
---

## Table of Contents

## Preamble

Babel is about letting [many different languages](https://orgmode.org/worg/org-contrib/babel/languages/index.html) work together. Programming languages live in code blocks inside natural language [Org](https://orgmode.org/)documents. A piece of data might pass from an Org table to a Python code block, then maybe move on to an R code block, and finally end up embedded as a value in the middle of a paragraph or possibly pass through a Gnuplot code block and end up as a plot embedded in the document.

Through extending [Org](https://orgmode.org/) with several features for editing, exporting, and executing source code, Babel transforms Org into a tool for literate programming and reproducible research.

Babel augments [Org code blocks](https://orgmode.org/manual/Working-with-Source-Code.html) by providing:

 * interactive and programmatic execution of code blocks;
 * code blocks as functions that accept parameters, refer to other code blocks, and can be called remotely; and
 * export to files for literate programming.

## Overview

Babel provides new features on a few different fronts, and different people may want to start in different places.

Using code blocks in Org modeIf you are not familiar with creating code blocks in an Org buffer, and moving between that buffer and the language major-mode edit buffer, then you should have a look at the [Org manual's Literal Examples section](https://orgmode.org/manual/Literal-Examples.html#Literal-Examples) and Section [Code Blocks](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#source-code-blocks), try it out, and come back.Executing codeThe core of Babel is its ability to execute source code in Org code blocks, taking input from other blocks and tables, and with output to yet other blocks and tables. This is described in Section[Executing Code Blocks](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#source-code-execution).Literate programmingIf you are a programmer writing code that you would normally execute in some other way (e.g. from the command line, or sourcing it into an interactive session), then a simple introduction to Babel is to place your code in blocks in an Org file, and to use Babel's [literate programming](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#literate-programming) support to extract pure code from your Org files.

All of these use cases, as well as exhaustive documentation of the features of Babel are covered in the [Working with Source Code](https://orgmode.org/manual/Working-with-Source-Code.html)section of the Org manual.

## Configuring Babel

If you have a working Emacs installation, then getting started with Babel is a simple process.

 1. If you are running Emacs24 or later a version of Org with Babel is already available by default. As of Org 7.0, Babel is included as part of Org. If you would like to use a more recent Babel than the one that ships with your Emacs, then please follow [these instructions](https://orgmode.org/manual/Installation.html#Installation).
 2. Optionally, activate the subset of languages that you want to use with Babel. See the instructions in Section [Activate a Language](https://orgmode.org/worg/org-contrib/babel/languages/index.html#configure). Note that Emacs Lisp is activated by default so this step can be skipped if you intend to work exclusively with it.
 3. If you have made changes to your setup, please remember to evaluate modified configuration file(s).

## Code Blocks

### Code Blocks in Org

Babel is all about code blocks in Org. If you are unfamiliar with the notion of a code block in Org, where they are called 'src' blocks, please have a look at the [Org manual](https://orgmode.org/manual/Literal-Examples.html#Literal-Examples) before proceeding.

Code blocks in supported languages can occur anywhere in an Org file. Code blocks can be entered directly into the Org file, but it is often easier to enter code with the function `org-edit-src-code`, which is called with the keyboard shortcut, `C-c '`. This places the code block in a new buffer with the appropriate mode activated.

```
#+begin_src <language> <switches> 
<body>
#+end_src

```

For example, a [Ruby](http://www.ruby-lang.org/) code block looks like this in an Org file:

```
#+begin_src ruby
require 'date'
"This file was last evaluated on #{Date.today}"
#+end_src

```

### Code Blocks in Babel

Babel adds some new elements to code blocks. The basic structure becomes:

```
#+begin_src <language>  <switches> <header arguments>
<body>
#+end_src

```

<language>An identifier for the code block language (see the[list of core languages](https://orgmode.org/worg/org-contrib/babel/languages/index.html#orgf05f403) and the [list of contributed languages](https://orgmode.org/worg/org-contrib/babel/languages/index.html#org8f1cf6c) for identifiers).<switches>Control code execution, export, and format.<header arguments>Header arguments control many facets of code block behavior, including [tangling](https://orgmode.org/manual/Header-arguments.html#Header-arguments), [evaluation](https://orgmode.org/manual/Evaluating-Code-Blocks.html), handling[results of evaluation](https://orgmode.org/manual/Results-of-Evaluation.html), and [exporting](https://orgmode.org/manual/Exporting-Code-Blocks.html). In addition, the language of the code block might define additional header arguments (see[Babel: Languages](https://orgmode.org/worg/org-contrib/babel/languages/index.html)).<body>The source code to be evaluated. An important key-binding is `C-c '`. This calls `org-edit-src-code`, a function that brings up an edit buffer containing the code using the Emacs major mode appropriate to the language. You can edit your code block as you regularly would in Emacs.

## Executing Code Blocks

Babel executes code blocks for interpreted languages such as shell, Python, R, etc. by passing code to the interpreter, which must be installed on your system. You control what is done with the results of execution.

Here are examples of code blocks in four different languages, followed by their output. If you are viewing the Org version of this document in Emacs, place point anywhere inside a block and press`C-c C-c` to run the code[1](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#fn.1) (and feel free to alter it!).

### Ruby

In the Org file:

    #+begin_src ruby
"This block was last evaluated on #{Date.today}"
#+end_src

HTML export of code:

```
"This block was last evaluated on #{Date.today}"

```

HTML export of the resulting string:

    This block was last evaluated on 2009-08-09

### Shell

In the Org file:

    #+begin_src shell
  echo "This file takes up `du -h org-babel.org |sed 's/\([0-9k]*\)[ ]*org-babel.org/\1/'`"
#+end_src

HTML export of code:

```
echo "This file takes up `du -h org-babel.org |sed 's/\([0-9k]*\)[ ]*org-babel.org/\1/'`"

```

HTML export of the resulting string:

    This file takes up 4.0K

### R

What are the most common words in this file?

In the Org file:

    #+begin_src R :colnames yes
  words <- tolower(scan("intro.org", what="", na.strings=c("|",":")))
  t(sort(table(words[nchar(words) > 3]), decreasing=TRUE)[1:10])
#+end_src

HTML export of code:

```
words <- tolower(scan("intro.org", what="", na.strings=c("|",":")))
t(sort(table(words[nchar(words) > 3]), decreasing=TRUE)[1:10])

```

HTML export of the resulting table:

code   #+end_src   #+name:   #+begin_src   babel   with   block   this   that   blocks   
90     47          45        44            43      43     41      37     36     27       

### ditaa

In the Org file:

    #+begin_src ditaa :file blue.png :cmdline -r
+---------+
| cBLU    |
|         |
|    +----+
|    |cPNK|
|    |    |
+----+----+
#+end_src

HTML export of code:

```
+---------+
| cBLU    |
|         |
|    +----+
|    |cPNK|
|    |    |
+----+----+

```

![blue.png](https://orgmode.org/worg/images/babel/blue.png)Figure 1: HTML export of the ditaa image.

## Capturing the Results of Code Evaluation

Babel provides two fundamentally different modes for capturing the results of code evaluation: functional mode and scripting mode. The choice of mode is specified by the `:results` header argument.

### Functional Mode

The 'result' of code evaluation is the value of the last statement in the code block. In functional mode, the code block is a function with a return value. Functional mode is indicated by setting the header argument `:results value`.

The return value of one code block can be used as input for another code block, even one in a different language. In this way, Babel becomes a [meta-programming language](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#meta-programming-language). If the block returns tabular data (a vector, array or table of some sort) then this will be held as an Org table in the buffer. This setting is the default.

For example, consider the following block of Python code and its output.

```
import time
print("Hello, today's date is %s" % time.ctime())
print('Two plus two is')
return 2 + 2

```

    4

Notice that, in functional mode, the output consists of the value of the last statement and nothing else.

### Scripting Mode

In scripting mode, Babel captures the text output of the code block and places it in the Org buffer. Scripting mode is indicated by setting the header argument `:results output`.

It is called scripting mode because the code block contains a series of commands, and the output of each command is returned. Unlike functional mode, the code block itself has no return value apart from the output of the commands it contains.[2](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#fn.2)

Consider the result of evaluating this code block with scripting mode.

```
import time
print("Hello, today's date is %s" % time.ctime())
print('Two plus two is')
2 + 2

```

    Hello, today's date is Sat Oct 16 10:48:47 2021
Two plus two is

Here, scripting mode returned the text that Python sent to`stdout` with the two `print()` statements. Because the code block doesn't include a `print()` statement for the last value,`(2 + 2)`, 4 does not appear in the results.

## Session-based Evaluation

For some languages, such as Python, R, Ruby and shell, it is possible to run an interactive session as an "inferior process" within Emacs. This means that an environment is created containing data objects that persist between different source code blocks. Babel supports evaluation of code within such sessions with the `:session` header argument. If the header argument is given a value, then that will be used as the name of the session. Thus, it is possible to run simultaneous sessions in the same language.

Session-based evaluation is particularly useful for prototyping and debugging. The function `org-babel-pop-to-session` can be used to switch to the session buffer.

Once a code block is finished, it is often best to execute it outside of a session, so the state of the environment in which it executes will be certain.

With R, the session will be under the control of [Emacs Speaks Statistics](http://ess.r-project.org/) as usual, and the full power of ESS is thus still available, both in the R session, and when switching to the R code edit buffer with `C-c '`.

## Arguments to Code Blocks

Babel supports parameterisation of code blocks, i.e., arguments can be passed to code blocks, which gives them the status of functions. Arguments can be passed to code blocks in both functional and scripting modes.

### Using a Code Block as a Function

First let's look at a very simple example. The following Python code block defines a function that squares its argument.

```
return x*x

```

In the Org file, the function looks like this:

    #+name: square
#+header: :var x=0
#+begin_src python
return x*x
#+end_src

Now we use the Python code block with a `#+call:` line (for information on the `#+call:` syntax see [Evaluating Code Blocks](https://orgmode.org/manual/Evaluating-Code-Blocks.html#Evaluating-Code-Blocks)):

    #+call: square(x=6)

36

### Using an Org Table as Input

In this example we define a function called `fibonacci-seq`, using Emacs Lisp. The function `fibonacci-seq` computes a Fibonacci sequence. The function takes a single argument, in this case, a reference to an Org table.

Here is the Org table that is passed to `fibonacci-seq`:

1   2   3   4   5    6    7    8    9    10   
2   4   6   8   10   12   14   16   18   20   

The table looks like this in the Org buffer:

    #+name: fibonacci-inputs
| 1 | 2 | 3 | 4 |  5 |  6 |  7 |  8 |  9 | 10 |
| 2 | 4 | 6 | 8 | 10 | 12 | 14 | 16 | 18 | 20 |

The [Emacs Lisp](http://www.gnu.org/software/emacs/manual/elisp.html) source code:

```
(defun fibonacci (n)
  (if (or (= n 0) (= n 1))
      n
    (+ (fibonacci (- n 1)) (fibonacci (- n 2)))))

(mapcar (lambda (row)
          (mapcar #'fibonacci row)) fib-inputs)

```

In the Org buffer the function looks like this:

    #+name: fibonacci-seq
#+begin_src emacs-lisp :var fib-inputs=fibonacci-inputs 
  (defun fibonacci (n)
    (if (or (= n 0) (= n 1))
        n
      (+ (fibonacci (- n 1)) (fibonacci (- n 2)))))

  (mapcar (lambda (row)
            (mapcar #'fibonacci row)) fib-inputs)
#+end_src

The return value of `fibonacci-seq` is a table:

1   1   2   3    5    8     13    21    34     55     
1   3   8   21   55   144   377   987   2584   6765   

## In-line Code Blocks

Code can be evaluated in-line using the following syntax:

    Without header args: src_<lang>{<code>} or with header args: src_<lang[<args>]{<code>},
for example src_python[:session]{10*x}, where x is a variable existing in the
python session.

## Code Block Body Expansion

Babel "expands" code blocks prior to evaluation, i.e., the evaluated code comprises the code block contents augmented with code that assigns referenced data to variables. It is possible to preview expanded contents, and also to expand code during tangling. Expansion takes into account header arguments and variables.

previewThe shortcut, `C-c M-b p`, is bound to the function,`org-babel-expand-src-block`. It can be used inside a code block to preview the expanded contents. This facility is useful for debugging.tangling

The expanded body can be tangled. Tangling this way includes variable values that may be

 * the results of other code blocks,
 * variables stored in headline properties, or
 * tables.

One possible use for tangling expanded code block is for `Emacs`initialization. Values such as user names and passwords can be stored in headline properties or in tables. The `:no-expand`header argument can be used to inhibit expansion of a code block during tangling.

Here is an example of a code block and its resulting expanded body.

The data are kept in a table:

username   john-doe   
password   abc123     

The code block refers to the data table:

```
(setq my-special-username (first (first data)))
(setq my-special-password (first (second data)))

```

With point inside the code block, `C-c M-b p` expands the contents:

```
(let ((data (quote (("john-doe") ("abc123")))))
(setq my-special-username (first (first data)))
(setq my-special-password (first (second data)))
)

```

Because the return value of a function written in one language can be passed to a function written in another language, or to an Org table, which is itself programmable, Babel can be used as a meta-functional programming language. With Babel, functions from many languages can work together. You can mix and match languages, using each language for the tasks to which it is best suited.

For example, let's take some system diagnostics in the shell and graph them with R.

First, create a code block, using shell code, to list directories in our home directory together with their sizes. Babel automatically converts the output into an Org table.

    #+name: directories
#+begin_src shell :results replace
  cd ~ && du -sc * |grep -v total
#+end_src

72         "Desktop"     
12156104   "Documents"   
3482440    "Downloads"   
2901720    "Library"     
57344      "Movies"      
16548024   "Music"       
120        "News"        
7649472    "Pictures"    
0          "Public"      
152224     "Sites"       
8          "System"      
56         "bin"         
3821872    "mail"        
10605392   "src"         
1264       "tools"       

Next write a function with a single line of R code that plots the data in the Org table as a dot chart. Note how this code block uses the `name` of the previous code block to obtain the data.

In the Org file:

    #+name: directory-dot-chart
#+header: :var dirs=directories() :exports both
#+begin_src R :results graphics file :file ../../images/babel/dirs.png
  dotchart(dirs[,1], labels = dirs[,2])
#+end_src

HTML export of code:

```
dotchart(dirs[,1], labels = dirs[,2])

```

![dirs.png](https://orgmode.org/worg/images/babel/dirs.png)Figure 2: HTML export of the directories dot chart.

## Using Code Blocks in Org Tables

In addition to passing data from tables as [arguments](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#arguments-to-source-code-blocks) to code blocks, and [storing](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#results-value) results as tables, Babel can be used in a third way with Org tables. First note that Org's [spreadsheet](https://orgmode.org/manual/The-Spreadsheet.html) is able to compute cell values from the values of other cells using a `#+TBLFM`formula line. In this way, table computations can be carried out using [Calc](https://orgmode.org/manual/Formula-syntax-for-Calc.html#Formula-syntax-for-Calc) and [Emacs Lisp](https://orgmode.org/manual/Formula-syntax-for-Lisp.html#Formula-syntax-for-Lisp).

What Babel adds is the ability to use code blocks (in whatever language) in the `#+TBLFM` line to perform the necessary computation.

### Example 1: Data Summaries Using R

As a simple example, we'll fill in a cell in an Org table with the average value of a few numbers. First, let's make some data. The following code block creates an Org table filled with five random numbers between 0 and 1.

    #+name: tbl-example-data
#+begin_src R
runif(n=5, min=0, max=1)
#+end_src

0.836685163900256   
0.696652316721156   
0.382423302158713   
0.987541858805344   
0.994794291909784   

Now we define a code block to calculate the mean of a table column.

In the Org file:

    #+name: R-mean
#+begin_src R :var x=""
colMeans(x)
#+end_src

HTML export of code:

```
colMeans(x)

```

Finally, we create the table which is going to make use of the R code. This is done using the `org-sbe` ('source block evaluate') macro in the table formula line.

In the Org file:

    #+name: summaries
|              mean |
|-------------------|
| 0.779619386699051 |
#+TBLFM: @2$1='(org-sbe "R-mean" (x "tbl-example-data()"))

HTML export of code:

mean   
0.78   

To recalculate the table formula, use `C-u C-c C-c` in the table. Notice that as things stand the calculated value doesn't change, because the data (held in the table above named`tbl-example-data`) are static. However, if you delete that data table, then the reference will be interpreted as a reference to the code block responsible for generating the data; each time the table formula is recalculated the code block will be evaluated again, and therefore the calculated average value will change.

### Example 2: Org Babel Test Suite

While developing Babel, we used a suite of tests implemented as a large Org table. To run the entire test suite we simply evaluate the table with `C-u C-c C-c`: all of the tests are run, the results are compared with expectations, and the table is updated with results and pass/fail statistics.

Here's a sample of our test suite.

In the Org file:

    #+name: org-babel-tests
| functionality    | block        | arg |    expected |     results | pass |
|------------------+--------------+-----+-------------+-------------+------|
| basic evaluation |              |     |             |             | pass |
|------------------+--------------+-----+-------------+-------------+------|
| emacs lisp       | basic-elisp  |   2 |           4 |           4 | pass |
| shell            | basic-shell  |     |           6 |           6 | pass |
| ruby             | basic-ruby   |     |   org-babel |   org-babel | pass |
| python           | basic-python |     | hello world | hello world | pass |
| R                | basic-R      |     |          13 |          13 | pass |
#+TBLFM: $5='(if (= (length $3) 1) (sbe $2 (n $3)) (sbe $2)) :: $6='(if (string= $4 $5) "pass" (format "expected %S but was %S" $4 $5))

HTML export of code:

functionality      block          arg   expected      results       pass   
basic evaluation                                                    pass   
emacs lisp         basic-elisp    2     4             4             pass   
shell              basic-shell          6             6             pass   
ruby               basic-ruby           org-babel     org-babel     pass   
python             basic-python         hello world   hello world   pass   
R                  basic-R              13            13            pass   

#### Code Blocks for Tests

In the Org file:

    #+name: basic-elisp(n)
#+begin_src emacs-lisp
(* 2 n)
#+end_src

HTML export of code:

```
(* 2 n)

```

In the Org file:

    #+name: basic-shell
#+begin_src shell :results silent
expr 1 + 5
#+end_src

HTML export of code:

```
expr 1 + 5

```

In the Org file:

    #+name: date-simple
#+begin_src shell :results silent
date
#+end_src

HTML export of code:

```
date

```

In the Org file:

    #+name: basic-ruby
#+begin_src ruby :results silent
"org-babel"
#+end_src

HTML export of code:

```
"org-babel"

```

In the Org file

    #+name: basic-python
#+begin_src python :results silent
'hello world'
#+end_src

HTML export of code:

```
'hello world'

```

In the Org file:

    #+name: basic-R
#+begin_src R :results silent
b <- 9
b + 4
#+end_src

HTML export of code:

```
b <- 9
b + 4

```

## The Library of Babel

As we saw with the [square](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#orga335aa1) example, once a code block function has been defined in the buffer it can be called using the `#+call:`notation:

    #+call: square(x=6)

But what about code blocks that you want to make available to every Org buffer?

In addition to the current buffer, Babel searches for pre-defined code block functions in files that have been assigned to the Library of Babel, a user-extensible collection of code blocks.

In practice, you are free to register as many files as you wish to your Library of Babel using the function, `org-babel-lob-ingest`, which is bound to `C-c C-v l`.

```
(org-babel-lob-ingest "path/to/file.org")

```

Note that it is possible to pass table values or the output of a code block to registered Library of Babel functions. It is also possible to reference registered Library of Babel functions in arguments to code blocks.

Once upon a time, Org was distributed with the eponymous Library of Babel file. This file, which includes a wide variety of code blocks for common tasks, is now available at [library-of-babel.org](https://git.sr.ht/~bzg/worg/tree/master/item/library-of-babel.org).

For more information, see [Library-of-Babel](https://orgmode.org/manual/Library-of-Babel.html#Library-of-Babel).

## Literate Programming

> Let us change our traditional attitude to the construction of programs: Instead of imagining that our main task is to instruct a_computer_ what to do, let us concentrate rather on explaining to_human beings_ what we want a computer to do.
> 
> The practitioner of literate programming can be regarded as an essayist, whose main concern is with exposition and excellence of style. Such an author, with thesaurus in hand, chooses the names of variables carefully and explains what each variable means. He or she strives for a program that is comprehensible because its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reinforce each other.
> 
> – Donald Knuth
> 
> 

Babel supports [literate programming](https://en.wikipedia.org/wiki/Literate_programming) (LP) by allowing the act of programming to take place inside an Org document. The Org document can then be exported (woven in LP speak) to HTML or LaTeX for consumption by a human, and the embedded source code can be extracted (tangled in LP speak) into source code files for consumption by a computer.

To support these operations Babel relies on Org's [export functionality](https://orgmode.org/manual/Exporting.html#Exporting) for weaving documentation, and on the`org-babel-tangle` function, which makes use of [Noweb](http://www.cs.tufts.edu/~nr/noweb/) [reference syntax](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#literate-programming-example), for tangling code files.

The [following example](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#literate-programming-example) demonstrates the process of tangling in Babel.

### Simple Literate Programming Example (Noweb Syntax)

Tangling functionality is controlled by the `:tangle` family of header arguments. These arguments can be used to turn tangling on or off (the default), either for the code block or the Org heading level.

The following code blocks demonstrate how to tangle them into a single source code file using `org-babel-tangle`.

The following two code blocks have no `:tangle` header arguments and so will not, by themselves, create source code files. They are included in the source code file by the third code block, which does have a `:tangle` header argument.

In the Org file:

    #+name: hello-world-prefix
#+begin_src shell :exports none
  echo "/-----------------------------------------------------------\\"
#+end_src

HTML export of code: In the Org file

    #+name: hello-world-postfix
#+begin_src shell :exports none
  echo "\-----------------------------------------------------------/"
#+end_src

HTML export of code: The third code block does have a `:tangle` header argument indicating the name of the file to which the tangled source code will be written. It also has [Noweb](http://www.cs.tufts.edu/~nr/noweb/) style references to the two previous code blocks. These references will be expanded during tangling to include them in the output file as well.

In the Org file:

    #+name: hello-world
#+begin_src shell :tangle hello :exports results :noweb yes :results raw
  <<hello-world-prefix>>
  echo "|                       hello world                         |"
  <<hello-world-postfix>>
#+end_src

HTML export of code:

```
echo "/-----------------------------------------------------------\\"
echo "|                       hello world                         |"
echo "\-----------------------------------------------------------/"

```

HTML export of results:

```
/-----------------------------------------------------------\
|                       hello world                         |
\-----------------------------------------------------------/

```

Calling `org-babel-tangle` will result in the following shell source code being written to the `hello.sh` file:

```
#!/usr/bin/env sh

# [[file:~/org/temp/index.org::*Noweb test][hello-world]]

echo "/-----------------------------------------------------------\\"
echo "|                       hello world                         |"
echo "\-----------------------------------------------------------/"
# hello-world ends here

```

In addition, the following Noweb syntax can be used to insert the results of evaluating a code block, in this case one named `example-block()`.

    # <<example-block()>>

Any optional arguments can be passed to `example-block()` by placing the arguments inside the parentheses following the convention defined when calling code block functions (see the [Library of Babel](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#library-of-babel)). For example,

    # <<example-block(a=9)>>

sets the value of argument `a` equal to `9`. Note that these arguments are not evaluated in the current code block but are passed literally to `example-block()`.

## Emacs Initialization with Babel

Babel has special support for embedding your Emacs initialization into Org files. The `org-babel-load-file` function can be used to load the Emacs Lisp code blocks embedded in a literate Org file in the same way that you might load a regular Emacs Lisp file, such as `.emacs`.

This allows you to make use of Org features, such as folding, tags, notes, HTML export, etc., to organize and maintain your Emacs initialization.

![dot-emacs.png](https://orgmode.org/worg/images/babel/dot-emacs.png)Figure 3: Org mode provides convenient facilities for organizing an initialization file.

To try this out, see the simple [Literate Emacs Initialization](file:///Users/andrewpatrick/Library/Containers/com.ngocluu.goodlinks/Data/Library/Application%20Support/articles/bbd7c0363818a051b3fa7f5f271d5c40/index.html#literate-emacs-init)example, check out the literate programming version of Phil Hagelberg's excellent [emacs-starter-kit](http://github.com/technomancy/emacs-starter-kit/tree/master), the [Emacs 24 starter kit](https://github.com/eschulte/emacs24-starter-kit), contributed by one of the Babel authors, or visit any one of the several sites found by an Internet search for the phrase "literate Emacs configuration."

### Literate Emacs Initialization

For a simple example of usage, follow these steps:

 * create a directory named `.emacs.d` in the base of your home directory;

   ```
   mkdir ~/.emacs.d

   ```

* checkout the latest version of Org into the src subdirectory of this new directory;

   ```
   cd ~/.emacs.d
   mkdir src
   cd src
   git clone https://git.savannah.nongnu.org/git/org-mode.git

   ```

* place the following code block in a file called `init.el` in your Emacs initialization directory (`~/.emacs.d`).

   ```
   ;;; init.el --- Where all the magic begins
   ;;
   ;; This file loads Org and then loads the rest of our Emacs initialization from Emacs lisp
   ;; embedded in literate Org files.

   ;; Load up Org Mode and (now included) Org Babel for elisp embedded in Org Mode files
   (setq dotfiles-dir (file-name-directory (or (buffer-file-name) load-file-name)))

   (let* ((org-dir (expand-file-name
                       "lisp" (expand-file-name
                               "org" (expand-file-name
                                      "src" dotfiles-dir))))
             (org-contrib-dir (expand-file-name
                               "lisp" (expand-file-name
                                       "contrib" (expand-file-name
                                                  ".." org-dir))))
             (load-path (append (list org-dir org-contrib-dir)
                                (or load-path nil))))
     ;; load up Org and Org-babel
     (require 'org)
     (require 'ob-tangle))

   ;; load up all literate org-mode files in this directory
   (mapc #'org-babel-load-file (directory-files dotfiles-dir t "\\.org$"))

   ;;; init.el ends here

   ```

* implement all of your Emacs customizations inside of Emacs Lisp code blocks embedded in Org files in this directory; and
 * re-start Emacs to load the customizations.

## Reproducible Research

> An article about computational science in a scientific publication is not the scholarship itself, it is merely advertising of the scholarship. The actual scholarship is the complete software development environment and the complete set of instructions which generated the figures.
> 
> – D. Donoho
> 
> 

[Reproducible research](http://reproducibleresearch.net/) (RR) is the practice of distributing, along with a research publication, all data, software source code, and tools required to reproduce the results discussed in the publication. As such the RR package not only describes the research and its results, but becomes a complete laboratory in which the research can be reproduced and extended.

Org already has exceptional support for [exporting to HTML and LaTeX](https://orgmode.org/manual/Exporting.html#Exporting). Babel makes Org a tool for RR by activating the data and code blocks embedded in Org documents; the entire document becomes executable. This makes it possible, and natural, to distribute research in a format that encourages readers to recreate results and perform their own analyses.

One notable existing RR tool is [Sweave](https://en.wikipedia.org/wiki/Sweave), which provides a mechanism for embedding [R](http://www.r-project.org/) code into LaTeX documents. Sweave is a mature and very useful tool, but we believe that Babel has several advantages:

 * it supports multiple languages (we're not aware of other RR tools that do this);
 * the [export process](https://orgmode.org/manual/Exporting.html#Exporting) is flexible and powerful, including HTML as a target in addition to LaTeX; and
 * the document can make use of Org features that support [project planning](https://orgmode.org/manual/Agenda-Views.html) and [task management](https://orgmode.org/manual/TODO-Items.html).
