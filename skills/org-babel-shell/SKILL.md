---
name: org-babel-shell
description: Complete guide for system automation, DevOps, remote execution, and shell scripting (Bash, Zsh, Fish, Dash, Sh) with Org Babel.
---

# Shell & Bash Code Blocks in Org Babel

Org Babel provides comprehensive shell scripting support across Unix and POSIX shells: **Bash**, **Zsh**, **Sh**, **Fish**, **Dash**, **Csh**, **Ksh**, and **Ash**.

Shell blocks turn Org Mode into an interactive DevOps notebook, runbook, and system administration console capable of local execution, background asynchronous processing, and remote server administration over SSH via TRAMP.

---

## 1. Requirements & Configuration

### 1.1 Emacs Configuration
Unlike other Babel languages that require separate modules, all shells are governed by a single Babel library (`ob-shell`). You only need `(shell . t)` in your `org-babel-load-languages`:

```elisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (shell      . t)))
```

*(Note: Prior to Org 8.2, this was named `sh`. Use `shell` in all modern Org versions).*

### 1.2 Specifying the Shell
Specify the exact shell binary name as the block language tag (`#+BEGIN_SRC <shell>`):

```org
# Bash:
#+BEGIN_SRC bash
  echo "Running in $BASH_VERSION"
#+END_SRC

# Zsh:
#+BEGIN_SRC zsh
  echo "Running in Zsh $ZSH_VERSION"
#+END_SRC

# POSIX / Dash (faster, lightweight execution):
#+BEGIN_SRC dash
  echo "Running in dash"
#+END_SRC

# Fish shell:
#+BEGIN_SRC fish
  set greeting "Hello from fish"
  echo $greeting
#+END_SRC
```
The specified shell binary must be accessible on your system `$PATH`.

---

## 2. Execution Modes: Isolated, Session, & Asynchronous

### 2.1 Isolated Subshell (Default, `:session none`)
Each block runs in a fresh, temporary subshell process. Environment variables and working directories do not carry over:

```org
#+BEGIN_SRC bash :results output
  echo "PID: $$"
  export WORKSPACE="/tmp/build"
#+END_SRC

#+BEGIN_SRC bash :results output
  # $WORKSPACE is empty here because this runs in a new subshell process:
  echo "PID: $$, WORKSPACE: $WORKSPACE"
#+END_SRC
```

### 2.2 Interactive Sessions (`:session <name>`)
Runs inside a persistent Emacs `comint` buffer (e.g. `*dev-session*`). Variables, functions, and working directories persist across multiple blocks:

```org
#+BEGIN_SRC bash :session *dev-session* :results output
  export CLUSTER_ID="prod-east-01"
  cd /var/log
#+END_SRC

#+BEGIN_SRC bash :session *dev-session* :results output
  # Uses the state set in the previous block:
  echo "Deploying to $CLUSTER_ID from $(pwd)"
#+END_SRC

#+RESULTS:
: Deploying to prod-east-01 from /var/log
```

### 2.3 Non-Blocking Asynchronous Execution (`:async yes`)
Long-running scripts (backups, container builds, model training) freeze Emacs when run synchronously. Add `:async yes` to run them in the background without locking the editor!

**Rules:**
1. `:async` **requires** `:session <name>`.
2. While running, Babel inserts a temporary UUID token in the `#+RESULTS:` block.
3. Once execution finishes in the background, the UUID is automatically replaced with the command's stdout.

```org
#+BEGIN_SRC bash :session *bg-job* :async yes :results output
  echo "Starting heavy database backup at $(date)..."
  sleep 5
  echo "Backup finished successfully at $(date)!"
#+END_SRC

#+RESULTS:
: Starting heavy database backup at Sun May 18 14:00:00 UTC 2025...
: Backup finished successfully at Sun May 18 14:00:05 UTC 2025!
```

To run synchronously in a session, pass `:async no`.

---

## 3. Shell-Specific Header Arguments

Org Babel provides three unique header arguments specifically tailored to shell operations:

### 3.1 `:stdin <element-name>` (Standard Input)
Pipes named Org elements (tables, raw text, or other code block results) into the shell command's `stdin`:

```org
# Raw text input:
#+NAME: raw_server_payload
{"node": "alpha", "cores": 8, "status": "active"}

# Pipe payload directly into jq:
#+BEGIN_SRC bash :stdin raw_server_payload :results output
  jq -r '.node + " has " + (.cores|tostring) + " cores"'
#+END_SRC

#+RESULTS:
: alpha has 8 cores
```

*(Note: Blocks using `:stdin` evaluate in a temporary subshell, ignoring `:session`).*

### 3.2 `:cmdline <args>` (Command-Line Arguments)
Passes positional arguments (`$1`, `$2`, `$@`) into the script body:

```org
#+BEGIN_SRC bash :cmdline "staging" 8080 :results output
  ENV="$1"
  PORT="$2"
  echo "Starting server on port $PORT in $ENV mode..."
#+END_SRC

#+RESULTS:
: Starting server on port 8080 in staging mode...
```

⚠️ **Security Warning:** `:cmdline` arguments are expanded directly by the host shell. Do not pass untrusted or unescaped strings to avoid command injection.

### 3.3 `:shebang <directive>` (Standalone Scripts & Clean Output)
Forces the block to be written to a temporary executable file and run via its shebang directive:

```org
#+BEGIN_SRC bash :shebang "#!/usr/bin/env bash" :results output
  for i in {1..3}; do
      echo "Step $i"
  done
#+END_SRC
```

**Why `:shebang` is essential:**
In multi-line `:session` blocks without a shebang, lines are sent line-by-line to the interactive shell, which can produce stray prompt strings (e.g., `bash-5.1$ `) in your results. Specifying `:shebang` or running without a session produces completely clean output.

---

## 4. Remote Server Administration via TRAMP (`:dir`)

You can execute commands on remote servers, virtual machines, or Docker containers by specifying a TRAMP path in `:dir`:

```org
:dir /ssh:<user>@<host>:<remote-path>
:dir /docker:<container-id>:<path>
:dir /sudo::/etc
```

### Example: Remote Sysadmin Workflow
```org
# Check disk usage and memory on a remote server:
#+BEGIN_SRC bash :dir /ssh:admin@bastion.example.com:/tmp :results output
  hostname
  uptime
  df -h / | awk 'NR==2 {print "Root Disk Usage: " $5}'
#+END_SRC

#+RESULTS:
: bastion.example.com
: 14:22:05 up 42 days,  3:18,  1 user,  load average: 0.12, 0.08, 0.05
: Root Disk Usage: 48%
```

### Remote Sessions
Combine `:dir` with `:session` to maintain an active SSH connection across multiple blocks without reconnecting on each step:

```org
#+BEGIN_SRC bash :dir /ssh:admin@bastion.example.com:/home/admin :session *remote-bastion* :results output
  WORKDIR=$(mktemp -d)
  cd "$WORKDIR"
  pwd
#+END_SRC
```

---

## 5. Result Formats & Tabular Data Extraction

### 5.1 Tab-Delimited Output to Org Table (`:results output table`)
When shell commands emit tab-separated or structured tabular output, Babel parses them into native Org tables:

```org
#+BEGIN_SRC bash :results output table
  printf "PID\tUser\tCommand\n"
  printf "101\troot\tsystemd\n"
  printf "405\tpostgres\tpostgres\n"
  printf "892\tnginx\tnginx: worker\n"
#+END_SRC

#+RESULTS:
| PID | User     | Command       |
|-----+----------+---------------|
| 101 | root     | systemd       |
| 405 | postgres | postgres      |
| 892 | nginx    | nginx: worker |
```

### 5.2 Directing Output to a File Link (`:results file :file <path>`)
```org
#+BEGIN_SRC bash :results file :file reports/sysinfo.txt
  uname -a > reports/sysinfo.txt
  cat /etc/os-release >> reports/sysinfo.txt
#+END_SRC

#+RESULTS:
[[file:reports/sysinfo.txt]]
```

---

## 6. Passing Variables & Bash Arrays (`:var`)

### 6.1 Scalars and Strings
```org
#+BEGIN_SRC bash :var port=8000 host="localhost" :results output
  echo "Connecting to $host on port $port..."
#+END_SRC
```

### 6.2 Bash 1D Arrays
Bash supports 1D array parameters defined with list syntax:

```org
#+BEGIN_SRC bash :results output :var servers='("web-01" "web-02" "db-01")'
  echo "Total servers: ${#servers[@]}"
  for s in "${servers[@]}"; do
      echo "Pinging $s..."
  done
#+END_SRC

#+RESULTS:
: Total servers: 3
: Pinging web-01...
: Pinging web-02...
: Pinging db-01...
```

---

## 7. Common Pitfalls & Safety Best Practices

| Pitfall | Cause | Solution |
|---|---|---|
| Emacs hangs on SSH or ffmpeg command | Command attempts to read stdin from the subshell | Redirect stdin explicitly (`</dev/null`) or use `ssh -n`. |
| Stray `bash-5.1$` prompt characters in results | Interactive line-by-line echo in `:session` | Add `:shebang #!/usr/bin/env bash` or evaluate outside of session. |
| Inadvertent variable expansion in heredocs | Using `<<EOF` allows shell to expand `$VARS` prematurely | Use quoted delimiter `<<"EOF"` to keep text literal and prevent early expansion. |
| Non-POSIX syntax failing in `sh` blocks | `#+BEGIN_SRC sh` maps to `/bin/sh` (often dash on Debian/Ubuntu) | Use `#+BEGIN_SRC bash` if your script uses bashisms (`[[`, `((`, arrays). |
| Command injection via `:cmdline` | Shell arguments evaluated verbatim | Quote arguments carefully or pass data via `:stdin` or `:var` instead. |
| Asynchronous block output not appearing | `:async yes` requires an active `:session` | Always provide a `:session <name>` when using `:async yes`. |

---

## 8. Complete DevOps Runbook Example

```org
#+TITLE: Production Service Health Runbook
#+PROPERTY: header-args:bash :session *runbook* :results output

* Step 1: Health Check Configuration
#+NAME: cluster_nodes
| Hostname        | Role    |
|-----------------+---------|
| prod-app-01.lan | backend |
| prod-app-02.lan | backend |
| prod-db-01.lan  | storage |

* Step 2: Probe Cluster Nodes
#+BEGIN_SRC bash :var nodes=cluster_nodes :colnames yes :results output table
  printf "Host\tRole\tStatus\n"
  for row in "${nodes[@]}"; do
      # Parse tab/space separated row data:
      host=$(echo "$row" | awk '{print $1}')
      role=$(echo "$row" | awk '{print $2}')
      
      # Quick ping probe:
      if ping -c 1 -W 1 "$host" > /dev/null 2>&1; then
          status="ONLINE"
      else
          status="OFFLINE"
      fi
      printf "%s\t%s\t%s\n" "$host" "$role" "$status"
  done
#+END_SRC

#+RESULTS:
| Host            | Role    | Status |
|-----------------+---------+--------|
| prod-app-01.lan | backend | ONLINE |
| prod-app-02.lan | backend | ONLINE |
| prod-db-01.lan  | storage | ONLINE |

* Step 3: Run Service Maintenance Asynchronously
#+BEGIN_SRC bash :session *runbook* :async yes
  echo "Initiating rolling cache purge across nodes..."
  sleep 4
  echo "Cache purge completed successfully."
#+END_SRC
```
