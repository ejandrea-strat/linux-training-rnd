# M2 Learning Materials — CLI Basics

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, who have completed M1 or at least
> have the M1 environment running. Every command is explained before it is used. You need the
> course's Linux environment (WSL2, a VM, or Linux-in-a-container) and a terminal.
>
> **Scope note.** Every example output below is **illustrative** unless it is explicitly labeled as
> coming from the author's observation notes
> ([`knowledge/observations/m02-cli-basics.md`](../../knowledge/observations/m02-cli-basics.md)).
> The labeled outputs were captured on WSL2 on a specific machine; dates, file listings, symlinks,
> and the contents of `/etc` will differ on yours. The Tab-completion comparison was captured in two
> different terminals (Warp and the plain Ubuntu terminal) and says which is which. Never copy an
> example as evidence — run the command and paste your own output.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes roughly **90–100 minutes**. M2 shares
> Week 1 of the course with M1 (5–6 estimated hours in total).

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

**Understand.** Nearly every command has the same shape, and one habit works across thousands of
tools because of it:

| Part | What it is | Example in `ls -l /etc` |
| --- | --- | --- |
| **Name** | The program to run | `ls` |
| **Flags (options)** | Change *how* the command behaves | `-l` |
| **Arguments** | Say *what* the command acts on | `/etc` |

A **short flag** is a single dash plus a letter (`ls -l`). Short flags can be combined:
`ls -la` means `-l` and `-a` together. They are fast to type but harder to read.

A **long flag** is two dashes plus a word (`ls --all`). Long flags are easier to read and remember,
but they cannot be combined into one token, and some take a value written with `=` (e.g.
`--color=auto`). A short flag and a long flag can be two names for the same option — `ls -a` and
`ls --all` are the same option spelled two ways.

The rule that separates the last two: **a flag changes how the command behaves; an argument is what
the command acts on.** In `ls -l /etc`, `-l` is a flag and `/etc` is the argument. A value given to
a *flag* (like `auto` in `--color=auto`) belongs to the flag, not to the command.

**Try.** Run the same option both ways on your home directory:

```bash
ls -a ~
ls --all ~
```

Output from the author's observation notes — captured on `~/linux-course/m2` after the lab:

```text
$ ls -a ~/linux-course/m2
.  ..  conf-list.txt  practice.txt

$ ls --all ~/linux-course/m2
.  ..  conf-list.txt  practice.txt
```

Both print `.` and `..`, which plain `ls` hides — same option, two spellings. The file list shown is
the author's, captured in the course directory after the lab; on your home directory the entries
will be your own files.

Now take any command you run this week and name its three parts. For `ls -l /etc`: name `ls`,
flag `-l`, argument `/etc`.

> **Common misconception:** *the only difference between a short flag and a long flag is how they
> are typed.* **Actually:** they can do the same job (`ls -a` and `ls --all` print the same list),
> but that isn't the only difference: short flags can be combined (`-la`) and long flags can't.

> **Common misconception:** *an argument is any value passed to a flag or command.*
> **Actually:** an argument is what the command acts on (like `/etc` in `ls -l /etc`). A value
> given to a flag, like `auto` in `--color=auto`, belongs to the flag.

**Verify.** Your two commands print the same list, including `.` and `..`. You can point at the
name, the flag, and the argument in a command you ran today, and explain what each changed.

**Continue** to [§2](#2-pipes) to feed one command's output straight into another.

---

## 2. Pipes

_Estimated: 12 minutes._

**Understand.** The pipe symbol `|` takes the **standard output** of the command on its left — what
would have hit the screen — and feeds it in as **standard input** to the command on its right, with
no file in between. That is why small, single-purpose tools combine into something larger.

Two details that matter:

- Only **standard output** (file descriptor 1) goes through the pipe. Errors (file descriptor 2)
  do not — they still show on the screen.
- A pipeline reads **left to right** as a sequence of transformations, like a sentence: each command
  is one step that changes or filters what came before it.

In `ls -l /etc | grep ".conf"`:

| Step | Command | What it produces |
| --- | --- | --- |
| 1 | `ls -l /etc` | a long listing of every entry in `/etc` |
| 2 | `grep ".conf"` | keeps only the lines matching `.conf`, drops the rest |

**Try.**

```bash
ls -l /etc | grep ".conf"
```

Output from the author's observation notes:

```text
$ ls -l /etc | grep ".conf"
-rw-r--r-- 1 root root       4027 Nov  4  2025 adduser.conf
-rw-r--r-- 1 root root       6274 Aug 27 22:12 ca-certificates.conf
-rw-r--r-- 1 root root       6174 Aug 27 22:11 ca-certificates.conf.dpkg-old
drwxr-xr-x 3 root root       4096 Aug 27 22:12 dconf
-rw-r--r-- 1 root root       2967 Feb 17  2026 debconf.conf
-rw-r--r-- 1 root root       1706 Nov  4  2025 deluser.conf
-rw-r--r-- 1 root root       1289 Jan  7  2026 dhcpcd.conf
-rw-r--r-- 1 root root        685 Feb 13  2026 e2scrub.conf
-rw-r--r-- 1 root root        725 Mar 21  2026 fuse.conf
-rw-r--r-- 1 root root       2584 Jan 24  2026 gai.conf
-rw-r--r-- 1 root root         92 Apr 20 16:46 host.conf
-rw-r--r-- 1 root root         34 Mar 10  2026 ld.so.conf
drwxr-xr-x 2 root root       4096 Oct 12 13:23 ld.so.conf.d
-rw-r--r-- 1 root root        191 Mar 20  2026 libaudit.conf
-rw-r--r-- 1 root root         13 Aug 27 22:11 locale.conf
-rw-r--r-- 1 root root        586 Dec  6  2025 logrotate.conf
-rw-r--r-- 1 root root       5230 Feb  3  2026 manpath.config
-rw-r--r-- 1 root root        775 Feb 13  2026 mke2fs.conf
-rw-r--r-- 1 root root        767 Mar 28  2026 netconfig
-rw-r--r-- 1 root root        526 Aug 27 22:11 nsswitch.conf
-rw-r--r-- 1 root root        552 Jul  4  2025 pam.conf
lrwxrwxrwx 1 root root         20 Oct 20 00:10 resolv.conf -> /mnt/wsl/resolv.conf
-rw-r--r-- 1 root root       1213 Aug 22  2025 rsyslog.conf
-rw-r--r-- 1 root root      10749 Feb  3  2026 sensors3.conf
-rw-r--r-- 1 root root       4354 Mar 13  2026 sudo.conf
-rw-r--r-- 1 root root       9804 Mar 13  2026 sudo_logsrvd.conf
-rw-r--r-- 1 root root       1259 Feb 26  2026 ucf.conf
lrwxrwxrwx 1 root root         16 Aug 27 22:11 vconsole.conf -> default/keyboard
-rw-r--r-- 1 root root        204 Aug 27 22:13 wsl-distribution.conf
-rw-r--r-- 1 root root         45 Oct 12 13:09 wsl.conf
-rw-r--r-- 1 root root        681 Feb 15  2026 xattr.conf
```

Read it left to right: `ls -l /etc` produced the full listing, `grep ".conf"` kept only the lines
that matched. Dates, sizes, and the exact file list are the author's machine — on WSL2 even
`resolv.conf` is a symlink into `/mnt/wsl`. Your list will differ.

One surprise in the author's output: `dconf` and `netconfig` matched even though their names do not
contain `.conf`. In a grep pattern, `.` means **any character**, so `.conf` also matches `xconf`.
The fix is `-F` (`--fixed-strings`), which treats the pattern as plain text.

Output from the author's observation notes:

```text
$ ls -l /etc | grep -F ".conf"
-rw-r--r-- 1 root root       4027 Nov  4  2025 adduser.conf
-rw-r--r-- 1 root root       6274 Aug 27 22:12 ca-certificates.conf
-rw-r--r-- 1 root root       6174 Aug 27 22:11 ca-certificates.conf.dpkg-old
-rw-r--r-- 1 root root       2967 Feb 17  2026 debconf.conf
-rw-r--r-- 1 root root       1706 Nov  4  2025 deluser.conf
-rw-r--r-- 1 root root       1289 Jan  7  2026 dhcpcd.conf
-rw-r--r-- 1 root root        685 Feb 13  2026 e2scrub.conf
-rw-r--r-- 1 root root        725 Mar 21  2026 fuse.conf
-rw-r--r-- 1 root root       2584 Jan 24  2026 gai.conf
-rw-r--r-- 1 root root         92 Apr 20 16:46 host.conf
-rw-r--r-- 1 root root         34 Mar 10  2026 ld.so.conf
drwxr-xr-x 2 root root       4096 Oct 12 13:23 ld.so.conf.d
-rw-r--r-- 1 root root        191 Mar 20  2026 libaudit.conf
-rw-r--r-- 1 root root         13 Aug 27 22:11 locale.conf
-rw-r--r-- 1 root root        586 Dec  6  2025 logrotate.conf
-rw-r--r-- 1 root root       5230 Feb  3  2026 manpath.config
-rw-r--r-- 1 root root        775 Feb 13  2026 mke2fs.conf
-rw-r--r-- 1 root root        526 Aug 27 22:11 nsswitch.conf
-rw-r--r-- 1 root root        552 Jul  4  2025 pam.conf
lrwxrwxrwx 1 root root         20 Oct 20 00:32 resolv.conf -> /mnt/wsl/resolv.conf
-rw-r--r-- 1 root root       1213 Aug 22  2025 rsyslog.conf
-rw-r--r-- 1 root root      10749 Feb  3  2026 sensors3.conf
-rw-r--r-- 1 root root       4354 Mar 13  2026 sudo.conf
-rw-r--r-- 1 root root       9804 Mar 13  2026 sudo_logsrvd.conf
-rw-r--r-- 1 root root       1259 Feb 26  2026 ucf.conf
lrwxrwxrwx 1 root root         16 Aug 27 22:11 vconsole.conf -> default/keyboard
-rw-r--r-- 1 root root        204 Aug 27 22:13 wsl-distribution.conf
-rw-r--r-- 1 root root         45 Oct 12 13:09 wsl.conf
-rw-r--r-- 1 root root        681 Feb 15  2026 xattr.conf
```

With `-F`, `dconf` and `netconfig` are gone; `manpath.config` stays because it really contains
`.conf`.

**Verify.** Your output contains only lines matching the pattern. You can read the pipeline left
to right as two steps — what `ls -l` produces, what `grep` keeps — and you can explain why the
plain run matched two extra entries.

**Continue** to [§3](#3-redirection) to decide where command output ends up: screen or file.

---

## 3. Redirection

_Estimated: 12 minutes._

**Understand.** Output can go to a file instead of the screen, and whether the file is overwritten
or appended to decides whether the earlier content is kept:

| Operator | What it does to an existing file | What it does if the file is missing |
| --- | --- | --- |
| `>` | **Truncates** (empties) the file, then writes — previous content is destroyed | Creates the file |
| `>>` | Appends the new output at the end — previous content stays | Creates the file |

Both create a file when there is none. The difference only shows when the file already exists, and
the direction of the mistake decides what is lost:

- Using `>` when you meant `>>`: the file's previous content is destroyed instantly, and all
  history in that file is gone.
- Using `>>` when you meant `>`: nothing is lost, but the old lines are still there, mixed with the
  new output, since it only appends at the end.

**Try.** Create the working directory, then write two lines — one with `>` and one with `>>`:

```bash
mkdir -p ~/linux-course/m2
cd ~/linux-course/m2
echo "line one" > practice.txt
echo "line two" >> practice.txt
cat practice.txt
```

Output from the author's observation notes:

```text
$ mkdir -p ~/linux-course/m2
$ cd ~/linux-course/m2
$ echo "line one" > practice.txt
$ echo "line two" >> practice.txt
$ cat practice.txt
line one
line two
```

Both lines are in `practice.txt`. Line one was written with `>`, which created the file; line two
was added with `>>` at the end. If `>` had been used for both, the second command would have erased
line one, and the file would have contained only `line two`.

> **Common misconception:** *`>>` is the one that overwrites, so `>` is the safe choice when adding
> a line.* **Actually:** it is the other way around — `>` overwrites and `>>` appends. Choosing `>`
> by mistake is what loses the file's contents; choosing `>>` by mistake only leaves the old lines
> in. That is why line one was still there after the second `echo`.

**Verify.** Your `cat` prints both lines. In your notes, state which line was written with `>` and
which with `>>`, and what would have been lost if `>` had been used for both.

**Continue** to [§4](#4-built-in-help) to learn how to check a command's behavior before running it.

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

A man page opens in a viewer, not an editor. The keys (from the author's research and
observations):

- `Space` — next page
- `b` — back a page
- `/word` — search
- `n` — next match
- `q` — quit

The viewer is `less` on most systems (`man less` documents the keys themselves). `--help` prints a
short summary of a command's options without opening any viewer at all.

**Try.**

```bash
man grep        # read a little, then press q to quit
grep --help     # short summary, straight to the screen
```

The author captured the first 15 lines of the manual by piping it — `man grep | head -15` prints
the page as plain text instead of opening the interactive pager, so the pager itself is **not**
shown in this capture. The pager keys above come from the author's research and observations, not
from this output.

Output from the author's observation notes — first 15 lines printed with `man grep | head -15`:

```text
$ man grep | head -15
GREP(1)                                                   User Commands                                                   GREP(1)

NAME
       grep, egrep, fgrep, rgrep - print lines that match patterns

SYNOPSIS
       grep [OPTION]... PATTERNS [FILE]...
       grep [OPTION]... -e PATTERNS ... [FILE]...
       grep [OPTION]... -f PATTERN_FILE ... [FILE]...

DESCRIPTION
       grep  searches for patterns in each FILE.  In the synopsis's first form, which is used if no -e or -f options are present,
       the first operand PATTERNS is one or more patterns separated by newline characters, and grep prints each line that matches
       a pattern.  Typically PATTERNS should be quoted when grep is used in a shell command.
```

Output from the author's observation notes:

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

Both explain grep's options: `man grep` is a full manual you can scroll and search, while
`grep --help` is a short summary printed to the screen.

Something the author did not already know, visible in that `--help` output: **grep treats the
pattern as a regular expression by default, and `-F` (`--fixed-strings`) makes it match plain
text** — which is exactly the `-F` run shown in [§2](#2-pipes).

Now tab completion. Typing `cd ~/linux-cou` and pressing Tab turned it into
`cd ~/linux-course/` — the shell finished the path, because only one folder starts with `linux-cou`.

When more than one match exists, the shell cannot finish the word:

Output from the author's observation notes:

```text
$ ls /etc/sys    (Tab pressed twice)
sysctl.d/ systemd/
```

**The terminal and the shell are separate programs here too** (from M1), and the difference is
visible in what the *first* Tab press does. The author tried two terminals:

- **Plain Ubuntu terminal (bash):** the first Tab only rings the bell; the second Tab prints the
  list — bash's default.
- **Warp:** the list shows up on the first Tab, because Warp has its own completion menu.

Neither terminal picks a match for you.

> **Common misconception:** *you navigate a man page with editor keys, the way you would in
> nano.* **Actually:** `man` opens a viewer, not an editor. `Space` and `b` move between pages,
> `/` searches, `n` jumps to the next match, and `q` quits.

> **Common misconception:** *when more than one match exists, Tab fills in the first result
> alphabetically.* **Actually:** neither Warp nor plain bash picks one. In the plain Ubuntu
> terminal, the first Tab rings the bell and the second lists the matches; in Warp, the first Tab
> shows Warp's own list.

**Verify.** Your notes contain output from both `man grep` (or its piped capture) and
`grep --help`, plus one thing either revealed that you did not already know. You have seen what Tab
does with one match (it completes) and with several (it lists, on the second press in plain bash).

**Continue** to [§5](#5-worked-example-the-m2-lab) to produce the lab evidence.

---

## 5. Worked example: the M2 lab

> **Purpose.** The five lab steps from [`brief.md`](./brief.md), with the exact evidence to
> capture. The outputs shown are the author's; yours will differ in dates, file lists, and `/etc`
> contents. Record your own.

_Estimated: 25–30 minutes._

### Step 1 — Write two lines with `>` and `>>`

**Understand.** This step is the whole overwrite-versus-append contrast in four commands.

**Try.**

```bash
mkdir -p ~/linux-course/m2
cd ~/linux-course/m2
echo "line one" > practice.txt
echo "line two" >> practice.txt
```

The author's exact command block is shown in [§3](#3-redirection); there is no output until step 2 reads the file.

**Capture.** Record the commands and, in your own notes, which line was written with `>` and which
with `>>`, plus what would have been lost if `>` had been used for both ([§3](#3-redirection)'s misconception box is
the reasoning, not the answer to copy — write it in your own words).

### Step 2 — Confirm both lines are present

**Understand.** `cat` prints a file's contents; this is the proof that `>` created and `>>`
appended.

**Try.**

```bash
cat practice.txt
```

Output from the author's observation notes:

```text
$ cat practice.txt
line one
line two
```

**Capture.** Paste the raw `cat` output with the command line included — not a paraphrase. This is
DoD-01 evidence, and it only counts if it came from your own terminal.

### Step 3 — Filter with a pipe

**Understand.** Chain two commands so `grep` receives `ls`'s output directly, with no file in
between.

**Try.**

```bash
ls -l /etc | grep ".conf"
```

Output from the author's observation notes — the full capture for this step is the listing shown
in [§2](#2-pipes) (same command, same output; it is this step's evidence).

**Capture.** Paste the full output, prompt line included, and write the left-to-right reading in
your own words: what `ls -l /etc` produced, what `grep ".conf"` kept. If your output has entries
the literal string `.conf` does not appear in, explain why ([§2](#2-pipes)) rather than deleting them.

### Step 4 — Read both forms of help

**Understand.** Open the same command's manual and its short help, and find one thing you did not
already know.

**Try.**

```bash
man grep        # then q to quit
grep --help
```

Output from the author's observation notes — both captures are shown in [§4](#4-built-in-help)
(`man grep | head -15` and `grep --help | head -10`).

**Capture.** Show that you opened both `man grep` and `grep --help`, and state something one of
them revealed that you did not already know. The author's answer, for reference: grep treats the
pattern as a regular expression by default, and `-F` (`--fixed-strings`) makes it match plain text.

### Step 5 — Complete a path with Tab

**Understand.** Tab finishes your typing when the match is unique, and reports the candidates when
it is not.

**Try.**

```bash
cd ~/linux-cou<Tab>        # complete the rest of the path
ls /etc/sys<Tab><Tab>      # ambiguous: list the matches
```

Typing `cd ~/linux-cou` and pressing Tab turned it into `cd ~/linux-course/` — the author's
observation.

Output from the author's observation notes:

```text
$ ls /etc/sys    (Tab pressed twice)
sysctl.d/ systemd/
```

**Capture.** Record the path Tab completed and what happened on an ambiguous prefix — and say
which terminal you used (plain Ubuntu terminal or Warp), since the first press behaves differently
in each ([§4](#4-built-in-help)).

### You build and capture — the deliverable

The brief's deliverable is a redirected file listing: a file produced by piping and/or redirecting
the output of at least one command into another. The author's combines a pipe and a redirect in one
command:

```bash
ls -l /etc | grep ".conf" > conf-list.txt
cat conf-list.txt
```

Output from the author's observation notes:

```text
$ ls -l /etc | grep ".conf" > conf-list.txt
$ cat conf-list.txt
-rw-r--r-- 1 root root       4027 Nov  4  2025 adduser.conf
-rw-r--r-- 1 root root       6274 Aug 27 22:12 ca-certificates.conf
-rw-r--r-- 1 root root       6174 Aug 27 22:11 ca-certificates.conf.dpkg-old
drwxr-xr-x 3 root root       4096 Aug 27 22:12 dconf
-rw-r--r-- 1 root root       2967 Feb 17  2026 debconf.conf
-rw-r--r-- 1 root root       1706 Nov  4  2025 deluser.conf
-rw-r--r-- 1 root root       1289 Jan  7  2026 dhcpcd.conf
-rw-r--r-- 1 root root        685 Feb 13  2026 e2scrub.conf
-rw-r--r-- 1 root root        725 Mar 21  2026 fuse.conf
-rw-r--r-- 1 root root       2584 Jan 24  2026 gai.conf
-rw-r--r-- 1 root root         92 Apr 20 16:46 host.conf
-rw-r--r-- 1 root root         34 Mar 10  2026 ld.so.conf
drwxr-xr-x 2 root root       4096 Oct 12 13:23 ld.so.conf.d
-rw-r--r-- 1 root root        191 Mar 20  2026 libaudit.conf
-rw-r--r-- 1 root root         13 Aug 27 22:11 locale.conf
-rw-r--r-- 1 root root        586 Dec  6  2025 logrotate.conf
-rw-r--r-- 1 root root       5230 Feb  3  2026 manpath.config
-rw-r--r-- 1 root root        775 Feb 13  2026 mke2fs.conf
-rw-r--r-- 1 root root        767 Mar 28  2026 netconfig
-rw-r--r-- 1 root root        526 Aug 27 22:11 nsswitch.conf
-rw-r--r-- 1 root root        552 Jul  4  2025 pam.conf
lrwxrwxrwx 1 root root         20 Oct 20 00:12 resolv.conf -> /mnt/wsl/resolv.conf
-rw-r--r-- 1 root root       1213 Aug 22  2025 rsyslog.conf
-rw-r--r-- 1 root root      10749 Feb  3  2026 sensors3.conf
-rw-r--r-- 1 root root       4354 Mar 13  2026 sudo.conf
-rw-r--r-- 1 root root       9804 Mar 13  2026 sudo_logsrvd.conf
-rw-r--r-- 1 root root       1259 Feb 26  2026 ucf.conf
lrwxrwxrwx 1 root root         16 Aug 27 22:11 vconsole.conf -> default/keyboard
-rw-r--r-- 1 root root        204 Aug 27 22:13 wsl-distribution.conf
-rw-r--r-- 1 root root         45 Oct 12 13:09 wsl.conf
-rw-r--r-- 1 root root        681 Feb 15  2026 xattr.conf
```

The same list, but this time it went into `conf-list.txt` instead of the screen. Record the exact
command that creates the file — re-running it must reproduce the file's contents.

**Clean-up.** Nothing to clean up. The practice files stay under `~/linux-course/`.

**Continue** to [§6](#6-putting-it-together-from-keypress-to-pipeline-output) to tie the pipeline back to the course's self-check question.

---

## 6. Putting it together: from keypress to pipeline output

_Estimated: 5 minutes._

**Understand.** The arch doc's self-check at the M1–M2 boundary is: *"What happens between typing a
command and seeing output?"* M1 answered it for a single command. M2's pipeline version, using the
author's real deliverable command `ls -l /etc | grep ".conf" > conf-list.txt`:

1. **The shell parses the whole line first.** Before starting any program, bash creates the pipe
   connecting `ls`'s output to `grep`'s input, and opens `conf-list.txt` for writing (truncating it
   first, because it is `>` not `>>`) as `grep`'s file descriptor 1.
2. **Then the programs run.** `ls -l /etc` writes its listing to file descriptor 1 — which now goes
   into the pipe, not the screen.
3. `grep` reads that stream from standard input, keeps the matching lines, and writes them to its
   file descriptor 1 — which is `conf-list.txt`.
4. Nothing appears on the screen, because `grep`'s standard output points at the file. The shell
   then prints the prompt again.

The M1 layers are all still here — the shell interprets, the kernel starts both processes — with
one M2 addition: **the shell wires the streams before the programs start**, so each program can
just read fd 0 and write fd 1 without knowing where its data came from or where it is going.

**Try.** Tell the same story with your own deliverable command: name the pipe, name the redirect,
and say which program's fd 1 is the file.

**Verify.** You can answer, unprompted: *"What happens between typing a command and seeing
output?"* — including what changes when the line contains a pipe or a redirect.

**Continue** to [§7](#7-completion-checklist) to confirm the deliverable is complete.

---

## 7. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] **DoD-01 — The file exists and reproduces.** `practice.txt` exists on disk and your pasted
      `cat` output shows both lines, and re-running the documented commands reproduces the file
      (Lab steps 1–2). *Verify via [§5](#5-worked-example-the-m2-lab), Steps 1–2.*
- [ ] **DoD-02 — Overwrite vs append, stated.** Your notes state which line was written with `>`
      and which with `>>`, and what would have been lost if `>` had been used for both (Lab
      step 1). *Verify via [§3](#3-redirection) and [§5](#5-worked-example-the-m2-lab), Step 1.*
- [ ] **DoD-03 — The pipeline, read left to right.** Your pasted output of
      `ls -l /etc | grep ".conf"` shows only matching entries, and you can read the pipeline as two
      steps: what `ls -l` produces and what `grep` keeps (Lab step 3). *Verify via [§2](#2-pipes) and [§5](#5-worked-example-the-m2-lab),
      Step 3.*
- [ ] **DoD-04 — Both forms of help, one new fact.** Your notes show that you opened both
      `man grep` and `grep --help`, and state something one of them revealed which you did not
      already know (Lab step 4). *Verify via [§4](#4-built-in-help) and [§5](#5-worked-example-the-m2-lab), Step 4.*
- [ ] **DoD-05 — Tab completion recorded.** Your notes show what Tab-completion did with the
      partial path `~/linux-cou` (Lab step 5). *Verify via [§4](#4-built-in-help) and [§5](#5-worked-example-the-m2-lab), Step 5.*

When all five are ticked, transfer the evidence into your write-up
([`write-up-template.md`](./write-up-template.md)) and continue to M3.

---

## 8. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Prefer `>>` over `>` when appending to a log or an existing file.** `>` truncates before it
  writes; the old content is gone the moment the command runs ([§3](#3-redirection)).
- **Read a pipeline left to right as a sequence of transformations.** Each command is one step that
  changes or filters what came before it — that habit scales to pipelines much longer than two
  commands ([§2](#2-pipes)).
- **Check `man` or `--help` before guessing a flag's behavior.** Both work offline, and guessing is
  how patterns silently match more than you meant ([§4](#4-built-in-help)).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| First Tab on an ambiguous prefix rings the bell and prints nothing (plain Ubuntu terminal) | bash's default completion: there is nothing it can add beyond what you typed | Press Tab a second time to list the matches (author's notes) |
| Tab shows a list on the first press | The terminal's own completion menu (Warp) handles Tab, not bash — expected behavior, not a broken shell | Expected; use the plain Ubuntu terminal to see bash's default (bell, then the list on the second Tab) (author's notes) |
| `grep ".conf"` also matched `dconf` and `netconfig` | In a grep pattern, `.` means *any character*, not a literal dot — `.conf` also matches `xconf` | Use `grep -F ".conf"` (`--fixed-strings`) to match plain text; the author's `-F` run drops those lines ([§2](#2-pipes)) |
| Earlier file contents vanished after a redirect | `>` truncates the file before writing | Use `>>` to append; check with `cat` first — once `>` has run, the old content is gone ([§3](#3-redirection)) |
| Man page keys behave like `nano` — or don't respond as expected | `man` opens a pager (a viewer), not an editor | `Space` next page, `b` back a page, `/` search, `n` next match, `q` quit ([§4](#4-built-in-help)) |

---

## 9. References

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man bash` — the **REDIRECTION** section (`>` and `>>`, truncation vs append) and **Pipelines**,
  the subsection under SHELL GRAMMAR that describes `|`.
- `man grep` — the full manual: synopsis, options, and pattern matching.
- `man ls` — every flag of the listing command used throughout this module, including `-a`/`--all`.
- `man man` — how the manual viewer itself works: navigation and searching inside a man page.
- `man less` — the pager behind `man` on most systems, and its keys (`Space`, `b`, `/`, `n`, `q`).

Online copies of the same pages:

- [`bash(1)`](https://man7.org/linux/man-pages/man1/bash.1.html)
- [`grep(1)`](https://man7.org/linux/man-pages/man1/grep.1.html)
- [`ls(1)`](https://man7.org/linux/man-pages/man1/ls.1.html)
- [`man(1)`](https://man7.org/linux/man-pages/man1/man.1.html)
- [`less(1)`](https://man7.org/linux/man-pages/man1/less.1.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — where the lab evidence goes.
- [`knowledge/observations/m02-cli-basics.md`](../../knowledge/observations/m02-cli-basics.md)
  — the author's captured output, used as the labeled examples in this guide.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md)
  — where M2 sits in the curriculum (Layer 1, Week 1, with M1).

*No curriculum resource is cited for this module — the architecture document's Resource Map lists
one for M1 only.*
