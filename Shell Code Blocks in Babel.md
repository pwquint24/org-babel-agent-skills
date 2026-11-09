title: Shell Code Blocks in Babel
author: Matt Trzcinski

[ UP ](https://orgmode.org/worg/org-contrib/babel/languages/index.html) | [ HOME ](https://orgmode.org/worg/)

Support via [Liberapay](https://liberapay.com/org-mode "Donations always help.")

# Shell Code Blocks in Babel

 Babel shell support, including: [sh](https://www.gnu.org/software/bash/), [bash](https://www.gnu.org/software/bash/), [zsh](https://www.zsh.org/), [fish](https://fishshell.com/), [csh](https://www.grymoire.com/Unix/Csh.html), [ash](https://www.in-ulm.de/~mascheck/various/ash/), [dash](http://gondor.apana.org.au/~herbert/dash/), [ksh](http://www.kornshell.org/), [mksh](https://www.mirbsd.org/mksh.htm), and [posh](https://packages.qa.debian.org/p/posh.html). 

 [![](https://tiswww.case.edu/php/chet/img/bash-logo-web.png){width=180px}](https://www.gnu.org/software/bash/) [![](https://zsh.sourceforge.io/Images/wizard.gif){width=150px}](https://www.zsh.org/) [![](http://www.mirbsd.org/pics/logo-grey.png){width=150px}](https://www.mirbsd.org/mksh.htm) [![](https://fishshell.com/assets/img/Terminal_Logo2_CRT_Flat.png){width=150px}](https://fishshell.com/) [![](http://www.kornshell.org/kornshell.gif){width=180px}](http://www.kornshell.org/) 

## Introduction {#org39eb2fb}

 A shell is a user interface for interacting with system services. File management, process execution, and operating system monitoring can all be done with a shell. Many shells are plain text whereas some support graphics or are themselves fully graphical. Shells often provide a programming language as well as access to system utilities, such as the GNU Core Utilities^[1](#fn.1)^. Users can stitch utilities together and create their own. Similarities exist between shells and, though standards exist (such as POSIX), there is no guarantee that what works in one shell will work in another^[2](#fn.2)^. Each shell is a separate application. 

 [Org Babel](https://orgmode.org/manual/Structure-of-Code-Blocks.html) works with several text based shells. 

 Run commands in separate shells…^[3](#fn.3)^ 

```

#+begin_src sh :results output
  echo PID: "$$"
#+end_src

#+RESULTS:
: PID: 19056

#+begin_src sh :results output
  echo PID: "$$"
#+end_src

#+RESULTS:
: PID: 19059
```

 …as part of a single instance… 

```

#+begin_src sh :results output :session shared
  echo PID: "$$"
  X=1
#+end_src

#+RESULTS:
: PID: 19066

#+begin_src sh :results output :session shared
  echo PID: "$$"
  echo X was set to "$X"
#+end_src

#+RESULTS:
: PID: 19066
: X was set to 1
```

 …asynchronously… 

```

#+begin_src sh :session execute-asynchronously :async
echo "Execute session blocks in the background"
sleep 3
echo "Using the :async header"
#+end_src

#+RESULTS:
: Execute session blocks in the background
: Using the :async header
```

 …or as a standalone shell command. 

```

#+title: shell command (shebang) example

This is a literate script file.  The script we'll make can be used to ask
someone what operating system they're using.  First, we define a reference.

#+name: their-os
Linux

The reference is then passed as stdin into a script which gets executed as a
command according to the shebang (within a shell specified by the block
language).  We get a different response depending on who we ask.

Let's test it by asking RMS:

#+begin_src bash :results output :shebang #!/usr/bin/env bash :stdin their-os :cmdline RMS :tangle ask_for_os.sh

  # call as ./ask_for_os.sh NAME, where NAME is who to ask

  if [ -z "$1" ]; then
      asked="$USER"
  else
      asked="$1"
  fi

  echo Hi, "$asked"! What operating system are you using?
  read my_os

  if [ "$asked" = "RMS" ]; then
      echo You\'re using GNU/"$my_os"!
  elif [ "$asked" = "Linus" ]; then
      echo You\'re using "$my_os"!
  else
      echo You\'re using `uname -o`!
  fi
#+end_src

#+RESULTS:
: Hi, RMS! What operating system are you using?
: You're using GNU/Linux!
```

 Shell blocks can accept args, work with stdin, be tangled, use noweb, and more^[4](#fn.4)^. 

 Some commands try to read standard input if it is available. In interactive sessions it may be hidden due to time interval between typed commands, but be careful when they are used in scripts Org source blocks. [BASH FAQ #89](https://mywiki.wooledge.org/BashFAQ/089) warns concerning `ssh` and `ffmpeg`. Either explicitly specify `</dev/null` as input or use here-document syntax. 

```

ssh example.org 'sed -i -e s/foo/bar/ file.txt' </dev/null
ssh example.org 'tee >>file.log' <<"EOF"
Added by org-babel
EOF
```

 (Quotes around "EOF" delimiter suppress variable expansion in the text.) Tools may have dedicated options, for example `ssh -n` is a more concise way to avoid the pitfall. 

## Requirements and Setup {#orgb5b6234}

 Org Babel can run many different shells such as [sh](https://www.gnu.org/software/bash/), [bash](https://www.gnu.org/software/bash/), [zsh](https://www.zsh.org/), [fish](https://fishshell.com/), [csh](https://www.grymoire.com/Unix/Csh.html), [ash](https://www.in-ulm.de/~mascheck/various/ash/), [dash](http://gondor.apana.org.au/~herbert/dash/), [ksh](http://www.kornshell.org/), [mksh](https://www.mirbsd.org/mksh.htm), and [posh](https://packages.qa.debian.org/p/posh.html). 

1. The shell must be present on the system and be accessable from the $PATH.
2.  The Org Babel language facility must be set to load shell functionality^[5](#fn.5)^: 
    ```
    
    ;; active Babel languages
    (org-babel-do-load-languages
    'org-babel-load-languages
    '((shell . t)))
    ```
     **TIP:** All that's needed is `(shell . t)`, regardless of the intended shell (e.g. dash, fish, etc.).^[6](#fn.6)^ 
     **NOTE:** The `ob-shell.el` module used to be named `ob-sh.el`. This was changed in Org 8.2^[7](#fn.7)^. 
3.  The desired shell should be given in place of the \<lang> header arg. 
     For example, to run using dash^[8](#fn.8)^: 
    ```
    
    #+begin_src dash :result output
      if [ `basename $SHELL` == bash ]; then
        echo bash
      else
        echo dash
      fi
    #+end_src
    
    #+RESULTS:
    : dash
    ```

## Header Arguments {#org78a3549}

 Shell blocks support many common headers, as well as several headers specific only to shells. 

 Common: 

- [:results](#orga1abc96)
- [:dir](#org3e96d08)
- [:session](#orgdc91965)
    - [:async](#org8fe50fc)
- [:var](#org02d2c52)
- [:noweb](https://orgmode.org/manual/Noweb-Reference-Syntax.html)
- [:tangle](https://orgmode.org/manual/Extracting-Source-Code.html)

 Shell specific: 

- [:stdin](#org5be6337)
- [:cmdline](#org5dfcbd2)
- [:shebang](#orgfa4710a)

### Results  {#orgb191aec}

 `:results {output, value}` 

- `output` returns stdout, default
- `value` returns exit code

 Supported types: table, list, and file. 

 *Examples:* 

 Define the following 2D-array: 

```

#+name: make_array
#+begin_src bash
declare -a array

m=4
n=3
for ((i=0; i<m; i++))
do
    for ((j=0; j<n; j++))
    do
        a[${i},${j}]=$RANDOM
    done
done
for ((i=0; i<m; i++))
do
    for ((j=0; j<n; j++))
    do
        echo -ne "${a[${i},${j}]}\t"
    done
    echo
done
#+end_src
```

 When called with `:results output table` or `:results table` 

```

#+RESULTS:
| 19323 | 14951 | 2805 |
| 19323 | 14951 | 2805 |
| 19323 | 14951 | 2805 |
| 19323 | 14951 | 2805 |
```

 When called with `:results output list` or `:results list` 

```

#+RESULTS:
- (29607 15726 14035)
- (29607 15726 14035)
- (29607 15726 14035)
- (29607 15726 14035)
```

 When called with `:results file :file my_output.txt`, a file named "my\_output.txt" is created which contains the output. The results contains a link to the file. 

```

#+RESULTS:
file:my_output.txt

#+begin_src sh :results output
  cat my_output.txt
#+end_src

#+RESULTS:
: 1526	5064	6484
: 1526	5064	6484
: 1526	5064	6484
: 1526	5064	6484
```

### Dir  {#org3aab6a9}

 `:dir <path>` 

 Use the `:dir` header to specify the default directory to use during execution. 

 **TIP** Use `:dir` to execute commands on a [remote server](https://www.gnu.org/software/emacs/manual/html_node/emacs/Remote-Files.html). See [Choosing a working directory](https://www.gnu.org/software/emacs/manual/html_node/org/Environment-of-a-Code-Block.html#Choosing-a-working-directory-1) to learn more about `:dir`. 

 *Example:* 

 Open an SSH connection to a server and create a file if it doesn't already exist. 

```

#+begin_src bash :results output :dir /ssh:user@localhost:/home/user
if [ ! -e  "foo_file" ];
then
    echo "foo" > foo_file
    echo "Created foo_file"
else
    echo "foo_file already exists!"
fi
#+end_src

#+RESULTS:
: Created foo_file
```

 *Example (with sessions):* 

 It also works with sessions! 

```

#+begin_src bash :results output :dir /ssh:user@localhost:/home/user :session *remote*
if [ ! -e  "foo_file" ];
then
    echo "foo" > foo_file
    echo "Created foo_file"
else
    echo "foo_file already exists!"
fi
#+end_src

#+RESULTS:
: foo_file already exists!
```

### Sessions  {#org02cf0c3}

 `:session <session-name>` 

 Use the header `:session <session-name>` to run different code blocks in the same shell instance. If `<session-name>` is "none", blank, or the `:session` header arg is left out altogether, then the code block evaluates in a temporary shell instance. Otherwise, a comint (command-line interpreter) buffer with `<session-name>` is created and reused. The code block is then sent for evaluation and the results inserted into the Org buffer. Blocks are processed synchronously (i.e. they freeze Emacs until done). 

 *Examples:* 

 The following creates a session called "`*my-session*`" and defines a variable^[9](#fn.9)^. The next block uses the same shell instance and has access to the variable from the previous block. Finally, a block with another session is given. It uses a different shell instance and, therefore, has no knowledge of the variable from the other session. 

```

#+begin_src sh :results none :session *my-session*
  X=1
#+end_src

#+begin_src sh :results output :session *my-session*
  echo X was set to "$X"
#+end_src

#+RESULTS:
: X was set to 1

#+begin_src sh :results output :session *another-session*
  echo X was set to "$X"
#+end_src

#+RESULTS:
: X was set to
```

 Sessions are useful for isolating processes and for incremental development. 

### Async  {#orgb6f34fd}

 `:async <[yes|no]>` 

 Run session blocks asynchronously with the `:async` header. This means Emacs will not freeze while waiting for the block to execute. 

 The `:async` header accepts optional arguments of "yes" and "no". When "yes", blocks are executed in a background process. When "no", the block behaves like `:session` (Emacs freezes until execution completes). A "yes" argument is assumed unless the argument is "no". 

 **NOTE:** `:async` requires the `:session` header! 

 *Examples:* 

 A universally unique identifier (UUID) appears while an async block executes: 

```

#+begin_src sh :results output :session test :async yes
echo "Hello, world!"
sleep 3
echo "Good-bye, cruel World..."
#+end_src

#+RESULTS:
: d3aec7bd-c2a0-41f4-bd4d-edad7143cbbc
```

 When the process completes, the UUID is replaced with the results: 

```

#+begin_src sh :results output :session test :async yes
echo "Hello, world!"
sleep 3
echo "Good-bye, cruel World..."
#+end_src

#+RESULTS:
: Hello, world!
: Good-bye, cruel World...
```

 Disable asynchronous execution by passing "no" as an argument: 

```

#+begin_src sh :results output :session test :async no
echo "Changing the :async argument to 'no'"
sleep 3
echo "will run the block synchronously (freezing Emacs"
echo "until the block completes)."
#+end_src

#+RESULTS:
: Changing the :async argument to 'no'
: will run the block synchronously (freezing Emacs
: until the block completes).
```

### Variables  {#orgf33a2ae}

 `:var <name_1>=<value_1> ... [<name_n>=<value_n>]` 

 Use the `:var` header to define variables in the shell environment. 

 Variables are defined using the following forms. Separate multiple variables with a space. 

| type | form |
|----|----|
| scalar | `<name>=<value>` |
| 1D-array\* | `<name>='(element1 element2 ...)` |

 `*` Only works with Bash. 

 *Example (scalar):* 

```

#+begin_src dash :var by_two=0  x=2
  if [ "$by_two" = "0" ]; then
    echo $(($x * 2))
  else
    echo $(($x * 3))
  fi
#+end_src

#+RESULTS:
: 4
```

 *Example (1D-array):* 

```

#+begin_src bash :results output :var arr='("apple" "banana" "cherry")
  echo The first element is...
  echo \"${arr[1]}\"
#+end_src

#+RESULTS:
: The first element is...
: "banana"
```

 The `:var` header lets results be passed between blocks, which allows blocks to act like functions. 

```

#+name: multiply_by_2
#+begin_src bash :var data="" :results output
  echo $(($data * 2))
#+end_src

#+begin_src bash :post multiply_by_2(data=*this*)
  echo 3
#+end_src

#+results:
: 6
```

### Standard Input  {#org46197cd}

 `:stdin <element-name>` 

 Use the `:stdin` header arg to pass named Org elements, such as code block results, to a shell process as standard input. 

 **NOTE** When `:stdin` is used, the block is evaluated in a temporary shell, regardless of whether a `:session` is declared. 

 *Example:* 

```

#+name: my-org-element
This is something referenced as stdin.
The stdin can even be multiple lines!

#+begin_src sh :stdin my-org-element :results output
cat
#+end_src

#+RESULTS:
: This is something referenced as stdin.
: The stdin can even be multiple lines!
```

 *Example:* 

```

#+name: my-input
3

#+begin_src bash :stdin my-input :results output
read n
echo You entered: "$n"
#+end_src

#+RESULTS:
: You entered: 3
```

 *Example:* 

```

#+name: my-input-block
#+begin_src bash
  echo "$(uname -o)"
  x=$(ps -p $PPID -o comm=)
  echo "${x:1:5}"
#+end_src

#+RESULTS: my-input-block
| GNU/Linux |
| emacs     |

#+begin_src bash :stdin my-input-block
  echo $(cut -f 1 -d "/") rocks!
#+end_src

#+RESULTS:
: GNU emacs rocks!
```

### Command-line Arguments  {#org9320496}

 `:cmdline <arg_1> ... [arg_n]` 

 Use the `:cmdline` header arg to pass arguments to a shell command. The arguments are passed verbatim as 

```

<system shell> -c <temporary file with src block contents> <arguments>
```

 `:cmdline` is used verbatim and is interpreted by shell. Org mode does not prevent shell expansion/interpretation. This, for example, means that 

```

#+begin_src sh :cmdline arg ; touch /tmp/not-an-arg
...
#+end_src
```

 will evaluate `touch` command as well. 

 When `:cmdline` is used, the block is evaluated in a temporary shell, regardless of whether a `:session` is declared. 

```

#+begin_src sh :cmdline arg1 arg2
  echo "$1"
  echo "$2"
#+end_src

#+RESULTS:
| arg1 |
| arg2 |
```

### Shebang (standalone scripts)  {#org16b36f7}

 `:shebang [shebang]` 

 Use the `:shebang` header arg to process a block as a command^[10](#fn.10)^. 

 Pass an interpreter directive (e.g. `#!/bin/bash`) to override the declared shell. If no shebang is provided, the block language is used (e.g. `#+begin_src bash`). 

 **NOTE** When the `:shebang` header arg is used, the block is evaluated in a temporary shell, regardless of whether a `:session` is declared. 

 **TIP** The `:shebang` header arg is useful for when a script spans several lines. Without the `:shebang` header arg, block lines are sent to the shell process in sequence which may result in unwanted output characters. 

 *Example (remove unwanted characters):* 

```

#+begin_src sh :results output :session unwanted-chars
  X=1
  echo Setting X...
  sleep 3
  echo X was set to "$X"
#+end_src

#+RESULTS:
: Setting X...
: sh-5.1$ X was set to 1

#+begin_src sh :results output :shebang
  X=1
  echo Setting X...
  sleep 3
  echo X was set to "$X"
#+end_src

#+RESULTS:
: Setting X...
: X was set to 1
```

 *Example (override):* 

```

#+begin_src bash :shebang #!/usr/bin/env dash
  if [ `basename $SHELL` == bash ]; then
    echo bash;
  else
    echo dash;
  fi
#+end_src

#+RESULTS:
: dash
```

## Footnotes: 

^[1](#fnr.1)^

 [https://en.wikipedia.org/wiki/List\_of\_GNU\_Core\_Utilities\_commands](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) 

^[2](#fnr.2)^

 [https://opensource.com/article/19/7/what-posix-richard-stallman-explains](https://opensource.com/article/19/7/what-posix-richard-stallman-explains) 

^[3](#fnr.3)^

 [https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-\_0024\_0024](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-_0024_0024) 

^[4](#fnr.4)^

 Tangle means to extract various blocks into a single source file. Noweb is a syntax used to insert/reference other code blocks. See the following for more details: 

- Tangle: [https://orgmode.org/manual/Extracting-Source-Code.html](https://orgmode.org/manual/Extracting-Source-Code.html)
- NoWeb: [https://orgmode.org/manual/Noweb-Reference-Syntax.html](https://orgmode.org/manual/Noweb-Reference-Syntax.html)
- Examples: [https://orgmode.org/worg/org-contrib/babel/intro.html#literate-programming-example](https://orgmode.org/worg/org-contrib/babel/intro.html#literate-programming-example)

^[5](#fnr.5)^

 [https://orgmode.org/worg/org-contrib/babel/languages/index.html#configure](https://orgmode.org/worg/org-contrib/babel/languages/index.html#configure) 

^[6](#fnr.6)^

 The symbol "shell" loads functionality for *all* supported shells. Specifically, the car of a `org-babel-load-languages` element is used to load the corresponding module. The car of `(shell . t)` is "shell" which is used to load `ob-shell.el`, the Org Babel module that handles shell interactions. There is not a separate module for each shell. 

^[7](#fnr.7)^

 [https://git.savannah.nongnu.org/cgit/org-mode.git/tree/etc/ORG-NEWS#n3995](https://git.savannah.nongnu.org/cgit/org-mode.git/tree/etc/ORG-NEWS#n3995) 

```

* Version 8.2

** Incompatible changes
*** =ob-sh.el= renamed to =ob-shell=
This may require two changes in user config.

1. In ~org-babel-do-load-languages~, change =(sh . t)= to =(shell . t)=.
2. Edit =local.mk= files to change the value of =BTEST_OB_LANGUAGES=
   to remove "sh" and include "shell".
```

^[8](#fnr.8)^

 This is a weird example. A better one might simply check the $SHELL environment variable. The reason the example doesn't is that the shell process created by a source block is a subprocess of Emacs. As such, it retains some environment variables, namely $SHELL. This document was written on a system using bash as it's main shell. This means that although dash is used to create the subprocess, the $SHELL variable still points to bash. To be more precise, the example uses the fact that "\=\=" is valid bash syntax, but invalid in dash. The first condition is met on bash, echoing "bash", whereas it fails on dash, echoing "dash". This way, we can be sure that the block is really using dash. 

^[9](#fnr.9)^

 Using asterisks, or "earmuffs", for the session name is not mandatory. 

>  In some other systems there is a convention of choosing variable names that begin and end with ‘\*’. We don’t use that convention in Emacs Lisp, so please don’t use it in your programs. (Emacs uses such names only for special-purpose buffers.) 
>  [https://www.gnu.org/software/emacs/manual/html\_node/elisp/Coding-Conventions.html](https://www.gnu.org/software/emacs/manual/html_node/elisp/Coding-Conventions.html) 

^[10](#fnr.10)^

 [https://en.wikipedia.org/wiki/Shebang\_(Unix)](https://en.wikipedia.org/wiki/Shebang_%28Unix%29) 

Documentation from the orgmode.org/worg/ website (either in its HTML format or in its Org format) is licensed under the [GNU Free Documentation License version 1.3](http://www.gnu.org/copyleft/fdl.html) or later. The code examples and css stylesheets are licensed under the [GNU General Public License v3](http://www.gnu.org/licenses/gpl.html) or later.
