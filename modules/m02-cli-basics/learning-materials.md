# M2 Learning Materials — CLI Basics

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own `write-up.md`, made from
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, who have completed M1 or at least
> have the M1 environment running. Every command is explained before it is used. You need the
> course's Linux environment (WSL2, a VM, or Linux-in-a-container) and a terminal.
>
> **Scope note.** Every command output in this guide is **illustrative** — it shows the shape of
> what a command prints, not a record of a completed lab. Values such as file sizes, dates,
> directory listings, and home paths differ on your machine. Run each command and paste your own
> output before you use any example as evidence.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **96 minutes** — 12 + 12 + 12 + 15
> + 25 for the four concept sections and the worked example, and 5 each for the
> putting-it-together section, the checklist, the best practices, and the references. M2 shares
> **Week 1** of the course with M1, in Layer 1 (Foundations). Commands assume **bash** on
> **WSL2 Ubuntu 26.04**, the course environment; on a Virtual Machine or in a container the same
> commands report your own environment's values.
>
> **Prerequisites.** The M1 environment and a bash terminal, plus M1's model of the shell and the
> terminal emulator as two separate programs — [§4](#4-built-in-help) relies on it. No tool beyond
> the shell is needed: every command in this guide is already on the machine.

## Contents

1. [Command anatomy](#1-command-anatomy)
2. [Pipes](#2-pipes)
3. [Redirection](#3-redirection)
4. [Built-in help](#4-built-in-help)
5. [Worked example: the M2 lab](#5-worked-example-the-m2-lab)
6. [Putting it together: from keypress to pipeline output](#6-putting-it-together-from-keypress-to-pipeline-output)
7. [Completion checklist](#7-completion-checklist)
8. [Best practices and troubleshooting](#8-best-practices-and-troubleshooting)
9. [References](#9-references)

---

## 1. Command anatomy

_Estimated: 12 minutes._

**Understand.** Nearly every command has the same shape, and that shared shape is why one habit
works across thousands of tools:

| Part | What it is | Example in `ls -l /etc` |
| --- | --- | --- |
| **Name** | The program to run | `ls` |
| **Flags (options)** | Change *how* the command behaves | `-l` |
| **Arguments** | Say *what* the command acts on | `/etc` |

A **short flag** is a single dash plus a letter (`ls -l`). Short flags can be combined into one
token: `ls -la` means `-l` and `-a` together. They are quick to type and harder to read.

A **long flag** is two dashes plus a word (`ls --all`). Long flags read better and are easier to
remember, but they cannot be combined into one token, and some take a value written with `=` —
`--color=auto`.

The two spellings often name the same option: `ls -a` and `ls --all` are one option written two
ways. What separates a flag from an argument is what it does. **A flag changes how the command
behaves; an argument is what the command acts on.** In `ls -l /etc`, `-l` is a flag and `/etc` is
the argument. A value given to a *flag* — `auto` in `--color=auto` — belongs to the flag, not to the
command.

**Try.** Run the same option both ways on your home directory:

```bash
ls -a ~          # short flag: include entries whose names start with a dot
ls --all ~       # long flag: the same option, spelled out
ls -la ~         # two short flags combined into one token
```

An illustrative example of what those commands print on a fresh Ubuntu home:

```text
$ ls -a ~
.  ..  .bash_logout  .bashrc  .profile

$ ls --all ~
.  ..  .bash_logout  .bashrc  .profile
```

The two listings are identical: the short flag and the long flag are two names for one option.
Both show the three dotfiles a new Ubuntu account starts with — `.bash_logout`, `.bashrc`, and
`.profile` — because plain `ls` hides anything whose name begins with a dot. The same three files
are what a new account is copied from, so you can check them yourself with `ls -a /etc/skel`. A
home does not stay that sparse: the shell writes `.bash_history` as you type, and programs create
folders such as `.cache`, so the listing grows with use as well as with anything you add
yourself.

Combined short flags give you the long listing and the dotfiles together:

```text
$ ls -la ~
total <n>
drwxr-x--- <n> trainee trainee 4096 <date> .
drwxr-xr-x <n> root root 4096 <date> ..
-rw-r--r-- 1 trainee trainee  <size> <date> .bash_logout
-rw-r--r-- 1 trainee trainee  <size> <date> .bashrc
-rw-r--r-- 1 trainee trainee  <size> <date> .profile
```

The two directory lines carry the structure of the filesystem. Your home is `drwxr-x---`, owned by
you. Mode 750 lets only you and your own group into it; M5 explains what the digits mean. Its
link count is above 1 because every subdirectory inside it adds to that count. The `..` entry is
the parent directory, `/home`, which is owned by `root` and readable by everyone.

Now take any command you run today and name its three parts. For `ls -l /etc`: the name is `ls`,
the flag is `-l`, the argument is `/etc`.

**Verify.** Your two commands print the same list, including `.` and `..`. You can point at the
name, the flag, and the argument in a command you ran today, and say what each part changed.

**Continue** to [§2](#2-pipes) to feed one command's output straight into another.

---

## 2. Pipes

_Estimated: 12 minutes._

**Understand.** The pipe symbol `|` takes the **standard output** of the command on its left —
what would have hit the screen — and connects it to the **standard input** of the command on its
right, with no file in between. `man bash` puts it in one sentence: *"The standard output of
command1 is connected via a pipe to the standard input of command2."* That is why small,
single-purpose tools combine into something larger.

Two details that matter:

- Only **standard output** (file descriptor 1) travels through a pipe. Errors (file descriptor 2)
  stay on the screen, because `|` does not connect them.
- A pipeline reads **left to right** as a sequence of transformations, like a sentence: each
  command is one step that changes or filters what came before it.

In `ls -l /etc | grep ".conf"`:

| Step | Command | What it produces |
| --- | --- | --- |
| 1 | `ls -l /etc` | a long listing of every entry in `/etc` |
| 2 | `grep ".conf"` | keeps only the lines matching `.conf`, drops the rest |

In the second command, `grep` is the name and `".conf"` is its argument, the pattern it looks
for. It has no flag yet; `-F` comes later in this section.

**Try.**

```bash
ls -l /etc | grep ".conf"
```

An illustrative example of what that prints:

```text
$ ls -l /etc | grep ".conf"
-rw-r--r-- 1 root root  <size> <date> adduser.conf
-rw-r--r-- 1 root root  <size> <date> ca-certificates.conf
drwxr-xr-x 3 root root  <size> <date> dconf
-rw-r--r-- 1 root root  <size> <date> debconf.conf
-rw-r--r-- 1 root root  <size> <date> host.conf
-rw-r--r-- 1 root root  <size> <date> ld.so.conf
drwxr-xr-x 2 root root  <size> <date> ld.so.conf.d
-rw-r--r-- 1 root root  <size> <date> manpath.config
-rw-r--r-- 1 root root  <size> <date> netconfig
-rw-r--r-- 1 root root  <size> <date> pam.conf
lrwxrwxrwx 1 root root  <size> <date> vconsole.conf -> default/keyboard
```

Read it left to right: `ls -l /etc` produced the full listing, and `grep ".conf"` kept the lines
that matched. The listing on your machine is longer and carries different sizes and dates.

Two entries in that example are worth a second look. `dconf` and `netconfig` matched even though
neither name contains a literal `.conf`. The reason is that **grep treats its pattern as a
regular expression by default**, and in a regular expression `.` means *any character* — so
`.conf` also matches `dconf` (any character + "conf") and `netconfig` (the "t" before "conf").
When you want the pattern matched as plain text, say so:

```bash
ls -l /etc | grep -F ".conf"
```

```text
$ ls -l /etc | grep -F ".conf"
-rw-r--r-- 1 root root  <size> <date> adduser.conf
-rw-r--r-- 1 root root  <size> <date> ca-certificates.conf
-rw-r--r-- 1 root root  <size> <date> debconf.conf
-rw-r--r-- 1 root root  <size> <date> host.conf
-rw-r--r-- 1 root root  <size> <date> ld.so.conf
drwxr-xr-x 2 root root  <size> <date> ld.so.conf.d
-rw-r--r-- 1 root root  <size> <date> manpath.config
-rw-r--r-- 1 root root  <size> <date> pam.conf
lrwxrwxrwx 1 root root  <size> <date> vconsole.conf -> default/keyboard
```

With `-F` (`--fixed-strings`) the two extra entries are gone, and `manpath.config` stays because
it really does contain `.conf` inside `.config`. The `-F` option comes from the same help output
you will meet in [§4](#4-built-in-help).

**Verify.** Your output contains only lines matching the pattern, and you can read the pipeline
left to right as two steps — what `ls -l` produces, what `grep` keeps. You can also explain why
the plain run matched entries whose names do not contain a literal `.conf`.

**Continue** to [§3](#3-redirection) to decide where command output ends up: screen or file.

---

## 3. Redirection

_Estimated: 12 minutes._

**Understand.** Output can go to a file instead of the screen, and whether the file is overwritten
or appended to decides whether the earlier content is kept. `man bash` documents both operators
in its REDIRECTION section:

| Operator | What it does to an existing file | What it does if the file is missing |
| --- | --- | --- |
| `>` | Opens the file for writing and **truncates it to zero size** — previous content is destroyed | Creates the file |
| `>>` | Opens the file for **appending** — previous content stays | Creates the file |

Both create a file when there is none. The difference only shows when the file already exists, and
the direction of the mistake decides what is lost:

- Using `>` when you meant `>>`: the file's previous content is destroyed the moment the command
  runs, and everything that was in it is gone.
- Using `>>` when you meant `>`: nothing is lost, but the old lines are still there, mixed in with
  the new output, because the new output was added at the end.

**Try.** Create the working directory for the course, then write two lines — one with `>` and one
with `>>`:

```bash
mkdir -p ~/linux-course/m2
cd ~/linux-course/m2
echo "line one" > practice.txt
echo "line two" >> practice.txt
cat practice.txt
```

```text
$ mkdir -p ~/linux-course/m2
$ cd ~/linux-course/m2
$ echo "line one" > practice.txt
$ echo "line two" >> practice.txt
$ cat practice.txt
line one
line two
```

Both lines are in `practice.txt`. The first `echo` used `>`, which created the file and wrote
"line one" into it. The second used `>>`, which added "line two" at the end and left the first
line alone. Had the second command used `>` as well, it would have truncated the file before
writing — "line one" would be gone and the file would hold only "line two".

Two details about the commands themselves. `cd` moved the working directory, which is why the
bare filename `practice.txt` is enough for the `echo` commands and for `cat` to find the file
again. And `>` resets the file every time it runs, so re-running the pair reproduces exactly the
same two lines, in the same order.

**Verify.** Your `cat` prints both lines. In your notes, state which line was written with `>` and
which with `>>`, and what would have been lost if `>` had been used for both.

**Continue** to [§4](#4-built-in-help) to learn how to check a command's behavior before you run
it.

---

## 4. Built-in help

_Estimated: 15 minutes._

**Understand.** Three built-ins let you work without memorizing every flag — which matters on a
server with no internet:

| Tool | What it gives you | Form |
| --- | --- | --- |
| `man <command>` | The full manual, offline, searchable | Opens a **pager** (a viewer) |
| `<command> --help` | A short summary of options, printed straight to the screen | No viewer |
| **Tab** | Finishes what you are typing | Handled by the shell, or by the terminal in Warp |

`man` displays the formatted page with a pager — `less` on this course image — and a pager is a
**viewer, not an editor**. The keys, from `man less`:

- `Space` — next page
- `b` — back a page
- `/word` — search
- `n` — next match
- `q` — quit

`--help` prints a short summary of a command's options and exits; no viewer opens at all.

**Try.**

```bash
man grep | head -15     # print the page as plain text instead of opening the pager
grep --help | head -10   # the short summary
```

```text
$ man grep | head -15
GREP(1)                          User Commands                          GREP(1)

NAME
       grep, egrep, fgrep, rgrep - print lines that match patterns

SYNOPSIS
       grep [OPTION]... PATTERNS [FILE]...
       grep [OPTION]... -e PATTERNS ... [FILE]...
       grep [OPTION]... -f PATTERN_FILE ... [FILE]...

DESCRIPTION
       grep  searches for patterns in each FILE.  In the synopsis's first form,
       which is used if no -e or -f options  are  present,  the  first  operand
       PATTERNS  is  one  or more patterns separated by newline characters, and
       grep prints each line that matches a pattern.  Typically PATTERNS should
```

```text
$ grep --help | head -10
Usage: grep [OPTION]... PATTERNS [FILE]...
Search for PATTERNS in each FILE.
Example: grep -i 'hello world' menu.h main.c
PATTERNS can contain multiple patterns separated by newlines.

Pattern selection and interpretation:
  -E, --extended-regexp     PATTERNS are extended regular expressions
  -F, --fixed-strings       PATTERNS are strings
  -G, --basic-regexp        PATTERNS are basic regular expressions
  -P, --perl-regexp         PATTERNS are Perl regular expressions
```

Both forms explain grep's options. `man grep` is the full manual you can scroll and search;
`grep --help` is a short summary printed straight to the screen. Read the SYNOPSIS line in the
manual and you can pick the parts apart with the anatomy from [§1](#1-command-anatomy): `grep` is
the name, `OPTION` and `PATTERNS` and `FILE` are the parts of the shape it expects, and `-F` from
the second output is the option that made [§2](#2-pipes) match plain text.

Now tab completion. Type a partial path and press Tab, and the shell finishes it when the match is
unique:

```bash
cd ~/linux-cou<Tab>       # one match: the shell completes the rest of the path
```

```text
$ cd ~/linux-cou<Tab>
cd ~/linux-course/
$ pwd
/home/trainee/linux-course
```

The shell replaced what you had typed with the full path `~/linux-course/`, and the command then
ran from that directory.

When more than one entry could match, the shell cannot finish the word, so it lists the
candidates instead:

```bash
ls /etc/sys<Tab><Tab>     # more than one match: the shell lists the candidates
```

```text
$ ls /etc/sys<Tab><Tab>
sysctl.d/  systemd/
```

The second command is a **prefix you type, not a path that is expected to exist** — `sys` is
where you stop typing, and Tab is showing you the two entries in `/etc` that could complete it.
Which of them the shell does next depends on the terminal, and the terminal and the shell are
separate programs, as in M1:

- **Plain Ubuntu terminal (bash):** the first Tab only rings the bell; the second Tab prints the
  list. That is bash's default.
- **Warp:** the list appears on the first Tab, because Warp has its own completion menu.

Neither terminal picks a match for you and completes it on its own.

**Verify.** Your notes contain output from both `man grep` (or its piped capture) and
`grep --help`, plus one thing either revealed that you did not already know. You have seen what
Tab does with one match — it completes the path — and with several — it lists the candidates,
after a bell on the first press in plain bash.

**Continue** to [§5](#5-worked-example-the-m2-lab) to produce the lab evidence.

---

## 5. Worked example: the M2 lab

_Estimated: 25 minutes._

**Understand.** The five lab steps come from the Lab section of [`brief.md`](./brief.md); the
deliverable steps come from the checklist in [`tasks.md`](./tasks.md). Each part gives you the
commands, an illustrative example of what they print, and the evidence to capture in your own
notes.

### Step 1 — Write two lines with `>` and `>>`

**Understand.** This step is the whole overwrite-versus-append contrast in four commands. `>` is
the one that destroys earlier content, `>>` is the one that keeps it.

**Try.**

```bash
mkdir -p ~/linux-course/m2
cd ~/linux-course/m2
echo "line one" > practice.txt
echo "line two" >> practice.txt
```

```text
$ mkdir -p ~/linux-course/m2
$ cd ~/linux-course/m2
$ echo "line one" > practice.txt
$ echo "line two" >> practice.txt
```

There is no output from these four commands — the evidence is in the file, which step 2 reads.

**Capture.** Record the four commands, and in your own notes state which line was written with `>`
and which with `>>`, plus what would have been lost if `>` had been used for both.

### Step 2 — Confirm both lines are present

**Understand.** `cat` prints a file's contents. This is the proof that `>` created the file and
`>>` added to it.

**Try.**

```bash
cat practice.txt
```

```text
$ cat practice.txt
line one
line two
```

**Capture.** Paste the raw `cat` output with the command line included — not a paraphrase. This is
DoD-01 evidence, and it only counts if it came from your own terminal.

### Step 3 — Filter with a pipe

**Understand.** Chain two commands so that `grep` receives `ls`'s output directly, with no file in
between.

**Try.**

```bash
ls -l /etc | grep ".conf"
```

```text
$ ls -l /etc | grep ".conf"
-rw-r--r-- 1 root root  <size> <date> adduser.conf
-rw-r--r-- 1 root root  <size> <date> ca-certificates.conf
drwxr-xr-x 3 root root  <size> <date> dconf
-rw-r--r-- 1 root root  <size> <date> debconf.conf
-rw-r--r-- 1 root root  <size> <date> host.conf
-rw-r--r-- 1 root root  <size> <date> ld.so.conf
drwxr-xr-x 2 root root  <size> <date> ld.so.conf.d
-rw-r--r-- 1 root root  <size> <date> manpath.config
-rw-r--r-- 1 root root  <size> <date> netconfig
-rw-r--r-- 1 root root  <size> <date> pam.conf
lrwxrwxrwx 1 root root  <size> <date> vconsole.conf -> default/keyboard
```

Your listing will have more entries, with different sizes and dates.

**Capture.** Paste the full output, prompt line included, and write the left-to-right reading in
your own words: what `ls -l /etc` produced, what `grep ".conf"` kept. If your output contains
entries that do not carry a literal `.conf` — `dconf` and `netconfig` are the usual two — explain
why in your notes, rather than deleting them from the paste.

### Step 4 — Read both forms of help

**Understand.** Open the same command's manual and its short help, and find one thing you did not
already know.

**Try.**

```bash
man grep        # read a little, then press q to quit
grep --help
```

```text
$ grep --help | head -10
Usage: grep [OPTION]... PATTERNS [FILE]...
Search for PATTERNS in each FILE.
Example: grep -i 'hello world' menu.h main.c
PATTERNS can contain multiple patterns separated by newlines.

Pattern selection and interpretation:
  -E, --extended-regexp     PATTERNS are extended regular expressions
  -F, --fixed-strings       PATTERNS are strings
  -G, --basic-regexp        PATTERNS are basic regular expressions
  -P, --perl-regexp         PATTERNS are Perl regular expressions
```

The `man grep` capture is the one in [§4](#4-built-in-help). Run `man grep` yourself and read the
page in the pager — `Space` to move on, `/` to search, `q` to quit.

**Capture.** Show that you opened both `man grep` and `grep --help`, and state something one of
them revealed that you did not already know. The `--help` output above shows one candidate: grep
treats the pattern as a regular expression by default, and `-F` (`--fixed-strings`) makes it match
plain text instead.

### Step 5 — Complete a path with Tab

**Understand.** Tab finishes your typing when the match is unique, and reports the candidates
when it is not.

**Try.**

```bash
cd ~/linux-cou<Tab>       # one match: the shell completes the rest of the path
ls /etc/sys<Tab><Tab>     # more than one match: the shell lists the candidates
```

```text
$ cd ~/linux-cou<Tab>
cd ~/linux-course/
$ pwd
/home/trainee/linux-course
```

```text
$ ls /etc/sys<Tab><Tab>
sysctl.d/  systemd/
```

**Capture.** Record the path Tab completed, and what happened on the ambiguous prefix — and say
which terminal you used, because the first press behaves differently in each: in the plain Ubuntu
terminal the first Tab rings the bell and the second lists the candidates, while in Warp the
list appears on the first press.

### Deliverable steps from tasks.md

The lab produces the evidence; the deliverable is the redirected file listing plus your notes and
write-up. The checklist in [`tasks.md`](./tasks.md) is the order of work.

**Setup.** Confirm with your trainer what counts as a real command and data set to filter, if you
are unsure. Then choose the real data on your own system that you will filter, rather than a toy
example.

**Build.** Research the "Research:" questions under each Scope bullet in [`brief.md`](./brief.md)
before you start. Run commands with options and arguments, and note how each part changes the
output. Chain at least two commands with a pipe, reading the pipeline left to right. Redirect
output to a file, and try both overwrite and append so you see the difference. Use `man`,
`--help`, and tab-completion to find something you did not already know.

Then produce the deliverable — a redirected file listing: a file produced by piping and/or
redirecting the output of at least one command into another, showing deliberate control over where
output goes. A pipe and a redirect in one command is the shortest way to show that control:

```bash
ls -l /etc | grep ".conf" > conf-list.txt
cat conf-list.txt
```

```text
$ ls -l /etc | grep ".conf" > conf-list.txt
$ cat conf-list.txt
-rw-r--r-- 1 root root  <size> <date> adduser.conf
-rw-r--r-- 1 root root  <size> <date> ca-certificates.conf
drwxr-xr-x 3 root root  <size> <date> dconf
-rw-r--r-- 1 root root  <size> <date> debconf.conf
-rw-r--r-- 1 root root  <size> <date> host.conf
-rw-r--r-- 1 root root  <size> <date> ld.so.conf
drwxr-xr-x 2 root root  <size> <date> ld.so.conf.d
-rw-r--r-- 1 root root  <size> <date> manpath.config
-rw-r--r-- 1 root root  <size> <date> netconfig
-rw-r--r-- 1 root root  <size> <date> pam.conf
lrwxrwxrwx 1 root root  <size> <date> vconsole.conf -> default/keyboard
```

The list is the same one step 3 printed to the screen, except that this time it went into
`conf-list.txt`. Record the exact command that creates the file — re-running it must reproduce
the file's contents.

**Verify.** Confirm the deliverable file exists on disk and that re-running your documented command
reproduces its contents. Confirm your notes say when you used overwrite versus append, and why.
Practice explaining, unprompted, something `man` or tab-completion told you that you did not
already know.

**Write-up.** Fill in every heading of [`write-up-template.md`](./write-up-template.md) in your
own `write-up.md`: "What I built," "Why it's built this way," "How to build it," "Concepts worth
explaining," "What tripped me up," "Checkpoint evidence," and "What I'd do differently." Under
"Checkpoint evidence," show the exact command you ran and the resulting file's contents, pasted
rather than transcribed.

### Clean-up

Nothing to clean up. The practice files stay under `~/linux-course/`: `practice.txt` and
`conf-list.txt` are left in place on purpose, because they are the artifacts your notes refer to.
The clean-up does not reach the other half of the module — your notes and your write-up are yours
to keep, and they are what a reviewer reads.

**Continue** to [§6](#6-putting-it-together-from-keypress-to-pipeline-output) to tie the pipeline
back to the course's self-check question.

---

## 6. Putting it together: from keypress to pipeline output

_Estimated: 5 minutes._

**Understand.** The architecture document's self-check at the M1–M2 boundary is: *"What happens
between typing a command and seeing output?"* M1 answered it for a single command. M2's pipeline
version, using the deliverable command `ls -l /etc | grep ".conf" > conf-list.txt`:

1. **The shell parses the whole line first.** Before starting any program, bash creates the pipe
   connecting `ls`'s output to `grep`'s input, and opens `conf-list.txt` for writing — truncating
   it first, because the operator is `>` and not `>>` — as `grep`'s file descriptor 1.
2. **Then the programs run.** `ls -l /etc` writes its listing to file descriptor 1, which now goes
   into the pipe instead of to the screen.
3. `grep` reads that stream from standard input, keeps the matching lines, and writes them to its
   file descriptor 1 — which is `conf-list.txt`.
4. Nothing appears on the screen, because `grep`'s standard output points at the file. The shell
   then prints the prompt again.

The M1 layers are all still here — the shell interprets, the kernel starts both processes — with
one M2 addition: **the shell wires the streams before the programs start**, so each program can
just read file descriptor 0 and write file descriptor 1 without knowing where its data came from
or where it is going.

**Try.** Tell the same story with your own deliverable command: name the pipe, name the redirect,
and say which program's file descriptor 1 is the file.

**Verify.** You can answer, unprompted: *"What happens between typing a command and seeing
output?"* — including what changes when the line contains a pipe or a redirect.

**Continue** to [§7](#7-completion-checklist) to confirm the deliverable is complete.

---

## 7. Completion checklist

_Estimated: 5 minutes._

Map your notes to the definition of done in [`brief.md`](./brief.md):

- [ ] DoD-01: `practice.txt` exists on disk and the trainee's pasted `cat` output shows both
      lines (Lab steps 1–2). Re-running the documented commands reproduces the file. *Verify via
      [§5](#5-worked-example-the-m2-lab), Steps 1–2.*
- [ ] DoD-02: The trainee's notes state which line was written with `>` and which with `>>`, and
      what would have been lost if `>` had been used for both (Lab step 1). *Verify via
      [§3](#3-redirection) and [§5](#5-worked-example-the-m2-lab), Step 1.*
- [ ] DoD-03: The pasted output of `ls -l /etc | grep ".conf"` shows only matching entries, and
      the trainee can read the pipeline left to right as two steps: what `ls -l` produces and what
      `grep` keeps (Lab step 3). *Verify via [§2](#2-pipes) and
      [§5](#5-worked-example-the-m2-lab), Step 3.*
- [ ] DoD-04: The notes show that the trainee opened both `man grep` and `grep --help`, and state
      something that one of them revealed which the trainee did not already know (Lab step 4).
      *Verify via [§4](#4-built-in-help) and [§5](#5-worked-example-the-m2-lab), Step 4.*
- [ ] DoD-05: The notes show what Tab-completion did with the partial path `~/linux-cou` (Lab
      step 5). *Verify via [§4](#4-built-in-help) and [§5](#5-worked-example-the-m2-lab), Step 5.*

When all five are ticked, transfer the evidence into your `write-up.md` and continue to M3.

---

## 8. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- Prefer `>>` over `>` when appending to a log or an existing file
  ([§3](#3-redirection)).
- Read a pipeline left to right as a sequence of transformations
  ([§2](#2-pipes)).
- Check `man` or `--help` before guessing a flag's behavior
  ([§4](#4-built-in-help)).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| The first Tab on an ambiguous prefix rings the bell and prints nothing, in the plain Ubuntu terminal | bash's default completion: there is nothing it can add beyond what you typed | Press Tab a second time to list the candidates |
| The candidate list appears on the first Tab, in Warp | The terminal's own completion menu is handling Tab, not bash — expected behavior, not a broken shell | Expected; open the plain Ubuntu terminal to see bash's default: the bell, then the list on the second Tab |
| `grep ".conf"` also matched entries such as `dconf` and `netconfig` | The pattern is a regular expression by default, and in a regular expression `.` means *any character*, so `.conf` also matches `xconf` | Use `grep -F ".conf"` (`--fixed-strings`) to match the pattern as plain text — the `-F` run in [§2](#2-pipes) drops those two entries |
| The earlier contents of a file disappeared after a redirect | `>` opens the file for writing and truncates it to zero size if it already exists (`man bash`, REDIRECTION) | Use `>>` to append, and check with `cat` first — once `>` has run, the old content is gone |
| The screen shows a pager that will not edit anything | `man` displays the formatted page with a pager, which is a viewer and not an editor (`man man`) | `Space` next page, `b` back a page, `/` search, `n` next match, `q` quit (`man less`) |

---

## 9. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man bash` — the **REDIRECTION** section (`>` truncates, `>>` appends) and **Pipelines**, the
  subsection under SHELL GRAMMAR that describes `|`.
- `man grep` — the full manual: synopsis, options, and how patterns are matched.
- `man ls` — every flag of the listing command used throughout this module, including `-a`/`--all`.
  On this course image the local page is the uutils (`rust-ls`) page, not the GNU coreutils one.
- `man man` — how the manual viewer itself works: the pager and how a page is displayed.
- `man less` — the pager behind `man` here, and its keys (`Space`, `b`, `/`, `n`, `q`).

Online copies of the same pages:

- [`bash(1)`](https://man7.org/linux/man-pages/man1/bash.1.html)
- [`grep(1)`](https://man7.org/linux/man-pages/man1/grep.1.html)
- [`ls(1)`](https://man7.org/linux/man-pages/man1/ls.1.html)
- [`man(1)`](https://man7.org/linux/man-pages/man1/man.1.html)
- [`less(1)`](https://man7.org/linux/man-pages/man1/less.1.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — the template your `write-up.md` is made from.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md)
  — where M2 sits in the curriculum (Layer 1, Week 1, with M1).

*No curriculum resource is cited for this module — the architecture document's Resource Map lists
one for M1 only.*
