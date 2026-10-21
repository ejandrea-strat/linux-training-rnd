# M1 Learning Materials — Linux Fundamentals

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own `write-up.md`, made from
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals. No prior exposure to kernels,
> processes, or the Linux filesystem is assumed, and every command is explained before it is used.
> You need the Linux environment set up in M0 (WSL2, a VM, or Linux-in-a-container) and a terminal.
>
> **Scope note.** Every command output in this guide is **illustrative** — it shows the shape of
> what a command prints, not a record of a completed lab. Values such as hostname, PID, kernel
> release, distro version, CPU model, and process count differ on your machine. Run each command
> and paste your own output before you use any example as evidence.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **90 minutes** — 15 + 10 + 10 + 10
> for the four concept sections, 25 for the worked example, and 5 each for the putting-it-together
> section, the checklist, the best practices, and the references. M1 shares **Week 1** of the
> course with M2, in Layer 1 (Foundations). Commands assume **bash** on **WSL2 Ubuntu 26.04**, the
> course environment; on a Virtual Machine or in a container the same commands report your own
> environment's values.
>
> **Prerequisites.** The Linux environment set up in M0 and a bash terminal. No earlier module
> comes before M1, and no tool beyond the shell is needed — every command here is already on the
> machine.

## Contents

1. [Kernel, shell, and distro](#1-kernel-shell-and-distro)
2. [System identity](#2-system-identity)
3. [The filesystem as an abstraction](#3-the-filesystem-as-an-abstraction)
4. [A live system](#4-a-live-system)
5. [Worked example: the M1 lab](#5-worked-example-the-m1-lab)
6. [Putting it together: from keypress to output](#6-putting-it-together-from-keypress-to-output)
7. [Completion checklist](#7-completion-checklist)
8. [Best practices and troubleshooting](#8-best-practices-and-troubleshooting)
9. [References](#9-references)

---

## 1. Kernel, shell, and distro

_Estimated: 15 minutes._

**Understand.** Three things are easy to blur together, so separate them first:

| Thing | What it is | Job in one line |
| --- | --- | --- |
| **Kernel** | The core program that owns the hardware | Decides which process runs, and performs every privileged operation on their behalf |
| **Shell** | An ordinary program (e.g. `bash`) | Reads what you type, interprets it, and starts other programs |
| **Distro** | A packaged collection of kernel + tools + shell + defaults (e.g. Ubuntu) | Decides which programs exist and how they behave by default |

### What a system call is

A **system call** is how a program asks the kernel to do something it cannot do by itself — read a
file, write to the terminal, start another process. The kernel runs in a privileged mode; user
programs cannot touch hardware directly, so they must ask.

When a command runs, the shell parses the command line and asks the kernel to start the program.
That request is made through system calls. The program then makes its *own* system calls to do its
work — `ls` asks the kernel to read the directory, then asks it to write the result to the
terminal. The shell does not make those calls on the program's behalf.

### Shell vs. terminal emulator

The **shell** and the **terminal emulator** are two different programs:

- The **terminal emulator** is the window. It displays text and collects keystrokes.
- The **shell** is the command interpreter running inside that window. It reads the keystrokes,
  parses them, and starts programs.

The two are connected by a **pseudo-terminal** — a device file under `/dev/pts/` that carries input
in one direction and output in the other. Neither program has a kernel of its own: under WSL2 the
terminal application runs on Windows and the shell runs on the Linux kernel, and they still meet
at a `/dev/pts/` device.

### Which shell am I running?

**Try.** Inspect your own shell instead of taking this guide's word for it:

```bash
ps -p $$ -o pid,ppid,comm,args      # your shell: PID, parent PID, name, full command
ps -p $$ -o ppid=                   # just the parent PID, for the next command
ls -l /proc/$$/fd                   # the files your shell has open
type cd                             # builtin, or a separate program?
type -a ls                          # every meaning of ls: alias, then real paths
```

An illustrative example of what those commands print:

```text
$ ps -p $$ -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
   <pid>   <ppid> bash            bash
```

Some terminal applications start bash with an extra option, so the COMMAND column may read
`bash --rcfile /dev/fd/<n>` instead of plain `bash`. The point does not change either way: your
shell is an ordinary program with a PID and a parent.

```text
$ ls -l /proc/$$/fd
total 0
lrwx------ 1 trainee trainee 64 <date> 0 -> /dev/pts/<n>
lrwx------ 1 trainee trainee 64 <date> 1 -> /dev/pts/<n>
lrwx------ 1 trainee trainee 64 <date> 2 -> /dev/pts/<n>
lrwx------ 1 trainee trainee 64 <date> 255 -> /dev/pts/<n>
```

File descriptors 0, 1, and 2 — standard input, output, and error — all point at the same
`/dev/pts/<n>` device, which is the link between the shell and the terminal window. A terminal
application that starts bash with its own startup file adds a descriptor for that file's pipe and
one for `/dev/ptmx`, so the exact list of descriptors varies with the terminal.

```text
$ type cd
cd is a shell builtin

$ type -a ls
ls is aliased to `ls --color=auto'
ls is /usr/bin/ls
ls is /bin/ls
```

`cd` is handled by the shell itself (a *builtin* — no separate program runs), while `ls` is a real
program on disk that the shell must ask the kernel to start. The two paths `/usr/bin/ls` and
`/bin/ls` are the same program: on Ubuntu, `/bin` is a link to `/usr/bin`. The alias line appears
when the shell is interactive and has read its startup file; a non-interactive shell — a script, or
a command run by another tool — shows only the two paths.

Finally, ask what started your shell:

```bash
ps -p <ppid> -o pid,ppid,comm,args    # replace <ppid> with the PPID printed above
```

```text
$ ps -p <ppid> -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
   <ppid> <pppid> Relay(<pid>)    /init
```

Under WSL2 the parent is not the terminal application but a relay process that passes text
between Windows and your Linux session. On a Virtual Machine the parent is the login session or
the SSH server; in a container it is whatever started the shell.

**Verify.** `ps -p $$` shows a shell process with a PID and a PPID; `ls -l /proc/$$/fd` shows
descriptors 0, 1, and 2 pointing at a `/dev/pts/<n>` device; `type cd` says *builtin*; `type -a
ls` shows a path.

**Continue** to [§2](#2-system-identity) to learn where the kernel and the distribution announce
their identities.

---

## 2. System identity

_Estimated: 10 minutes._

**Understand.** Two commands answer "what am I running?" from two different angles:

- `uname -a` reports facts about the **running kernel**: kernel name, hostname, kernel release,
  build string, machine architecture, and operating system name.
- `cat /etc/os-release` reports facts written by the **distribution**: its name, version, ID, and
  links. It is a plain file of `KEY=value` lines, so you can read it with `cat`.

They are not the same fact because they do not describe the same layer. The kernel can be replaced
(a VM boots the distribution's own kernel; WSL2 runs a kernel built by Microsoft) while the
distribution's files stay put, and the other way round.

### Reading `uname -a`

**Try.**

```bash
uname -a
uname -r      # kernel release on its own
uname -m      # machine architecture on its own
```

```text
$ uname -a
Linux linux-pc <version> #1 SMP PREEMPT_DYNAMIC <build-date> x86_64 GNU/Linux
```

| `uname -a` field | Meaning |
| --- | --- |
| `Linux` | Kernel name |
| `linux-pc` | Hostname — under WSL2 this is the Windows computer name by default; your own will differ |
| `<version>` | Kernel release — the *version* of the kernel |
| `#1 SMP PREEMPT_DYNAMIC <build-date>` | Build number and build date — `man uname` calls this the "kernel version" (`uname -v`); the kernel version this lab asks for is the kernel release (`uname -r`) |
| `x86_64` | Machine architecture (CPU family the kernel was built for) |
| `GNU/Linux` | Operating system name reported by the kernel |

Under WSL2 the kernel release carries a `-microsoft-standard-WSL2` suffix, because Microsoft builds
the kernel rather than the distribution. That is expected, and it is covered in
[§8](#8-best-practices-and-troubleshooting).

### Reading `/etc/os-release`

**Try.**

```bash
cat /etc/os-release
```

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.x LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.x LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=resolute
LOGO=ubuntu-logo
```

This is Ubuntu 26.04's set of keys, in the order Ubuntu writes them; `x` stands for the point
release, so the values differ slightly between point releases. The key set is the distribution's
choice — another distribution's file carries different keys.

| `/etc/os-release` field | Meaning |
| --- | --- |
| `PRETTY_NAME` / `NAME` | Human-readable distribution name — `"Ubuntu 26.04.x LTS"` and `"Ubuntu"` |
| `VERSION` / `VERSION_ID` | Distribution version — `"26.04.x LTS (Resolute Raccoon)"` — and the script-friendly version number, `"26.04"` |
| `VERSION_CODENAME` / `UBUNTU_CODENAME` | Release nicknames — `resolute` |
| `ID` / `ID_LIKE` | Machine-readable distro identifier — `ubuntu` — and the family it derives from, `debian` |
| `HOME_URL`, `SUPPORT_URL`, `BUG_REPORT_URL`, `PRIVACY_POLICY_URL` | Where the distribution points for help — ubuntu.com, help.ubuntu.com, and the bug tracker at bugs.launchpad.net/ubuntu/ |
| `LOGO` | Icon name used by graphical front-ends — `ubuntu-logo` |

This file identifies the distribution. It carries names, identifiers, and links — not a release
date, and nothing about the kernel. `man 5 os-release` is the authority on the format, and it
recommends reading `ID` and `VERSION_ID` for decisions and `PRETTY_NAME` for display.

**Verify.** Both commands print output on your machine. In the examples above, `uname -a` says
nothing about the distribution, and `/etc/os-release` says nothing about the kernel. Pulling the
four headline facts out of your own output, and comparing WSL2 with a Virtual Machine, happens in
[§5](#5-worked-example-the-m1-lab).

**Continue** to [§3](#3-the-filesystem-as-an-abstraction) to see what else the kernel exposes — as
files.

---

## 3. The filesystem as an abstraction

_Estimated: 10 minutes._

**Understand.** In Linux, "a file" is better read as "a thing you can open, read, write, and
close". Two directories make that literal:

- **`/dev`** holds **device files**. Each one represents a piece of hardware or a kernel service —
  disks, terminals, `/dev/null`. In `ls -l` output the first character of the mode tells you the
  type: `c` = character device, read and written as a stream of bytes (terminals, consoles);
  `b` = block device, read and written in fixed-size blocks (disks); `d` = directory; `l` = link.
- **`/proc`** is a **pseudo-filesystem**: a window into the kernel's own data structures. There are
  no bytes on disk behind it. The kernel generates the contents the moment you read them, and the
  per-process directories (`/proc/<pid>/`) are how tools like `ps` learn what is running.

"What about 'everything is a file'?" It means these non-file resources — devices, processes,
kernel tunables — are reached with the **same operations** used for ordinary files: open, read,
write, close. It does not mean every entry is an editable document: permissions still apply, and
`man 5 proc` notes that most files in the proc filesystem are read-only.

**Try.**

```bash
ls -l /dev | head -10              # device files and their types (c, b, d, l)
ls -l /proc/cpuinfo                # a kernel-generated file: note the 0-byte size
head -5 /proc/cpuinfo              # reading it returns real CPU details
```

```text
$ ls -l /dev | head -10
total 0
crw-r--r-- 1 root root     10, 235 <date> autofs
drwxr-xr-x 2 root root        600 <date> block
drwxr-xr-x 2 root root        120 <date> bsg
crw------- 1 root root     10, 234 <date> btrfs-control
drwxr-xr-x 2 root root       2860 <date> char
crw------- 1 root root      5,   1 <date> console
lrwxrwxrwx 1 root root         11 <date> core -> /proc/kcore
crw------- 1 root root     10, 260 <date> cpu_dma_latency
crw------- 1 root root     10, 203 <date> cuse
```

Most entries start with `c`, meaning character devices, so `/dev` holds devices shown as files.
The `core` entry is a link pointing from `/dev` into `/proc`, so the two directories are not as
separate as they first look.

```text
$ ls -l /proc/cpuinfo
-r--r--r-- 1 root root 0 <date> /proc/cpuinfo

$ head -5 /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: <model>
model name	: <cpu model>
```

The `/proc/cpuinfo` example is the whole idea in two commands: its size is **0 bytes**, yet
reading it returns real CPU details. Nothing is stored on disk — the kernel produces the content
on demand.

**Verify.** Your `/dev` listing contains entries whose mode starts with `c` or `b`, and
`/proc/cpuinfo` reports a 0-byte size while still printing CPU lines.

**Continue** to [§4](#4-a-live-system) to use `/proc` to see what is running right now.

---

## 4. A live system

_Estimated: 10 minutes._

**Understand.** Every running program is a **process**: a program in memory with a **PID** (process
ID) and a **PPID** (the PID of the process that started it). The kernel creates processes, decides
which one runs on the CPU and when, tracks their state, and exposes them through `/proc`. `ps` is
just a reader of that information; `top` is the live view of the same thing. `ls` has no option for
listing processes; `ps` reads each process's entry under `/proc` and prints them as a table.

**Try.**

```bash
ps -ef | head -20                              # snapshot of processes, full format
ps -e --no-headers | wc -l                     # how many processes are running
cat /proc/1/comm                               # what is PID 1?
top -b -n 1                                    # one snapshot of top, then it exits
```

```text
$ ps -ef | head -20
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 <time> ?        00:00:01 /sbin/init
root           2       1  0 <time> hvc0     00:00:01 /init
root         <pid>      2  0 <time> hvc0     00:00:00 init-watcher
root         <pid>      2  0 <time> hvc0     00:00:31 plan9 --control-socket 7 --log-level 4 --server-fd 8 --pipe-fd 10 --log-truncate
root         <pid>      1  0 <time> ?        00:00:00 /usr/lib/systemd/systemd-journald
systemd+     <pid>      1  0 <time> ?        00:00:00 /usr/lib/systemd/systemd-resolved
trainee      <pid>      1  0 <time> ?        00:00:00 /usr/lib/systemd/systemd --user
```

Read the columns: `PID` and `PPID` give the process and its parent, and the PPID column traces the
parent-child tree that PID 1 starts. PID 1 appears as `/sbin/init`, which *is* systemd on Ubuntu,
and its child PID 2 is `/init`. `/init`, `init-watcher`, and `plan9` are WSL2's own processes —
`/init` starts the session and `plan9` is what lets Windows read the Linux filesystem — so they do
not appear on a Virtual Machine or in a container.

```text
$ ps -e --no-headers | wc -l
<count>

$ cat /proc/1/comm
systemd
```

The count is machine-specific. `cat /proc/1/comm` names the init system of your environment: on
Ubuntu with systemd it prints `systemd`, while a plain container often has its own entry point
running as PID 1.

`top` normally redraws the screen until you quit it. The `-b` (batch) and `-n 1` options print one
snapshot and exit, so the output can be pasted into notes:

```text
$ top -b -n 1
top - <time> up <uptime>,  <n> user,  load average: <load>
Tasks:  <n> total,   <n> running,  <n> sleeping,   <n> stopped,   <n> zombie
%Cpu(s):  <n> us,  <n> sy,  <n> ni, <n> id,  <n> wa,  <n> hi,  <n> si,  <n> st
MiB Mem :   <n> total,   <n> free,   <n> used,    <n> buff/cache
MiB Swap:  <n> total,  <n> free,      <n> used.   <n> avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  <pid> trainee   <n>   0      <n>     <n>    <n> S   <n>   <n>       <n> <program>
      1 root      <n>   0      <n>     <n>    <n> S   <n>   <n>       <n> systemd
```

**Verify.** `ps -ef` shows a PID/PPID tree headed by PID 1; your process count prints a number;
`cat /proc/1/comm` names your init system; `top -b -n 1` prints a snapshot and exits.

**Continue** to [§5](#5-worked-example-the-m1-lab), where you produce the lab evidence.

---

## 5. Worked example: the M1 lab

_Estimated: 25 minutes._

**Understand.** The lab is a measurement, not a summary. The three steps come from the Lab section
of [`brief.md`](./brief.md); the deliverable steps come from the checklist in
[`tasks.md`](./tasks.md).

### Step 1 — Capture the raw output

**Try.** Run both commands in your own Linux environment and capture the output exactly as it
appears.

```bash
uname -a
cat /etc/os-release
```

```text
$ uname -a
Linux linux-pc <version> #1 SMP PREEMPT_DYNAMIC <build-date> x86_64 GNU/Linux
```

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.x LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.x LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=resolute
LOGO=ubuntu-logo
```

**Capture.** Paste the raw output verbatim into your observation notes, command line included:

- Keep the `$ uname -a` and `$ cat /etc/os-release` prompts next to their output, so a reader can
  tell which command produced what.
- Do not paraphrase, reorder, or tidy the output — an edited transcript is not evidence.
- The full `/etc/os-release` is short; paste all of it rather than trimming to two lines.
- If you copy from a terminal, copy the output as text, not a screenshot, so it stays searchable.

### Step 2 — Identify the four facts

**Understand.** Four facts live in that output, spread across both commands.

**Try.** Fill in the table from your own captured output.

| Fact | Source command | Where in the output | Your value |
| --- | --- | --- | --- |
| Kernel version | `uname -a` | the kernel release field | |
| Kernel architecture | `uname -a` | the machine field, just before `GNU/Linux` | |
| Distribution name | `cat /etc/os-release` | the `NAME=` line | |
| Distribution version | `cat /etc/os-release` | the `VERSION=` line | |

**Capture.** Record all four values *and* the position you read each one from. The "where" column
is the part that proves the fact came from the output rather than from memory.

### Step 3 — WSL2 versus Virtual Machine

**Understand.** The point of this step is that **kernel and distribution are independent facts**.
They are supplied by different layers, so changing one path need not change the other.

**Try.** Answer in one or two sentences: what would change in step 1's output if you switched
between the WSL2 path and the Virtual Machine path?

For contrast, here is what the kernel line looks like on a Virtual Machine running the same
distribution — the kernel release now names the distribution's own kernel:

```text
$ uname -a
Linux linux-pc <version>-generic #<build>-Ubuntu SMP PREEMPT_DYNAMIC <build-date> x86_64 GNU/Linux
```

Same distribution, different kernel: `uname -a` changes, `/etc/os-release` need not. The hostname
changes too, and the architecture stays the same if it is the same CPU.

**Capture.** Your one- or two-sentence explanation, in your observation notes, naming which line
of the output changes and which stays the same.

### Deliverable steps from tasks.md

The lab produces the evidence; the deliverable is the observation notes plus the write-up. The
checklist in [`tasks.md`](./tasks.md) is the order of work.

**Setup.** Confirm with the trainer how much detail the observation notes need before you begin.
The brief lists the expected length and depth as still open, so settle that question first.

**Build.**

- Research the "Research:" questions under each Scope bullet in [`brief.md`](./brief.md) before
  you start investigating, and jot down what you expect to find.
- Investigate the kernel, the shell, and your distribution, and trace how a typed command travels
  from the shell to the kernel and back as output.
- Investigate your system's kernel and distribution identity directly from the command line, and
  record the real output — not a paraphrase.
- Explore the filesystem as an abstraction, and note what distinguishes it from the physical
  storage underneath.
- Investigate what's actually running on your system right now, and note anything that differs from
  what you expected before you looked.
- Compile your written observation notes from the investigations above into one coherent record.

**Verify.**

- Confirm your notes name your system's actual kernel version and distribution, not generic facts
  copied from documentation.
- Confirm every claim in your notes is backed by real terminal output you captured, not written
  from memory.
- Practice answering, unprompted: *"What happens between typing a command and seeing output?"* —
  this is also the course's self-check at the M1–M2 boundary.

**Write-up.** Fill in every heading of [`write-up-template.md`](./write-up-template.md) in your own
`write-up.md`: "What I built," "Why it's built this way," "How to build it," "Concepts worth
explaining," "What tripped me up," "Checkpoint evidence," and "What I'd do differently." Under
"Checkpoint evidence," paste the real terminal output you used, not a transcription:

```text
$ uname -a
Linux linux-pc <version> #1 SMP PREEMPT_DYNAMIC <build-date> x86_64 GNU/Linux
```

Finally, re-read your notes and write-up against M1's definition of done, and confirm you can
answer the kernel-to-output question unprompted.

### Clean-up

Nothing to clean up: this lab only reads system information. No files, users, or packages are
created, so nothing is left behind on the machine. The notes and the write-up are the artifacts
you keep, and they are the ones a reviewer reads.

**Continue** to [§6](#6-putting-it-together-from-keypress-to-output) to tie the lab together into
the course's headline question.

---

## 6. Putting it together: from keypress to output

_Estimated: 5 minutes._

**Understand.** The whole module resolves into one sequence. Typing `ls` and pressing Enter:

1. The terminal application sends the keystrokes through the pseudo-terminal
   `/dev/pts/<n>`, and the shell reads them as file descriptor 0.
2. The shell sees that `ls` is not a builtin, finds the program at `/usr/bin/ls`, and asks the
   kernel to start it. That request is made through system calls.
3. `ls` asks the kernel to read the directory — more system calls.
4. `ls` writes the result to file descriptor 1 (`/dev/pts/<n>`), and the terminal application
   displays it.
5. The shell prints the prompt again.

Every layer from [§1](#1-kernel-shell-and-distro) to [§4](#4-a-live-system) appears here: the
**terminal emulator** collects the input, the **shell** interprets it, the **kernel** starts the
process and performs the directory read, the **filesystem as an abstraction** supplies
`/dev/pts/<n>` and `/usr/bin/ls`, and the **running processes** are the shell and `ls` themselves.
All of it runs on the kernel your `uname -a` reported, with the programs supplied by the
distribution your `/etc/os-release` reported.

**Try.** Tell the same story using your own two outputs: name your kernel from `uname -a` and your
distribution from `/etc/os-release` as the two layers involved.

**Verify.** You can answer, unprompted: *"What happens between typing a command and seeing
output?"* This is the course's self-check at the M1–M2 boundary.

**Continue** to [§7](#7-completion-checklist) to confirm the deliverable is complete.

---

## 7. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] DoD-01: The trainee's notes show the raw, pasted output of `uname -a` and `cat
      /etc/os-release` from the trainee's own terminal (Lab step 1), not facts copied from
      documentation or paraphrased from memory. *Verify via
      [§5](#5-worked-example-the-m1-lab), Step 1.*
- [ ] DoD-02: From that output, the notes name the kernel version, the kernel architecture, the
      distro name, and the distro version, and show where in the output each one appears (Lab step
      2). *Verify via [§5](#5-worked-example-the-m1-lab), Step 2.*
- [ ] DoD-03: The notes explain in one or two sentences what would change in step 1's output if
      the trainee switched between the WSL2 and Virtual Machine paths, showing that kernel and
      distro are independent facts (Lab step 3). *Verify via
      [§5](#5-worked-example-the-m1-lab), Step 3.*
- [ ] DoD-04: Using the output from steps 1–3 as evidence, the trainee can answer unprompted,
      "what happens between typing a command and seeing output?" This also serves as the course's
      self-check at the M1–M2 boundary. *Verify via
      [§6](#6-putting-it-together-from-keypress-to-output).*

When all four are ticked, transfer the evidence into your `write-up.md` and continue to M2.

---

## 8. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Treat kernel and distro version as the first two facts to check when something behaves
  differently across machines** ([§2](#2-system-identity)).
- **Do not assume that distro-specific defaults (package manager, default shell, service manager)
  transfer to another distro** ([§1](#1-kernel-shell-and-distro)).
- **Keep "the shell" and "the terminal emulator" distinct**
  ([§1](#1-kernel-shell-and-distro)).

### Troubleshooting

| Symptom | Likely cause | Recommended fix |
| --- | --- | --- |
| `uname -a` shows a kernel release ending in `-microsoft-standard-WSL2` | Expected under WSL2: Microsoft builds the kernel, while the distribution's files come from the distribution. It is not a mis-installed distribution — the two layers are independent ([§2](#2-system-identity)). | Nothing to fix. If you need the distribution's own kernel, use the Virtual Machine path instead: `uname -a` changes, `/etc/os-release` need not. |
| `cat /proc/1/comm` prints something other than `systemd` | The environment does not run systemd as PID 1 — a plain container runs its own entry point, and a WSL2 instance without systemd enabled runs another init. | Check `cat /etc/wsl.conf` for a `[boot]` section with `systemd=true` under WSL2, then restart the instance from Windows with `wsl --shutdown` and re-run `cat /proc/1/comm`. In a container, treat PID 1 as your environment's entry point. |
| `uname --version` prints `uname (uutils coreutils) <version>` instead of the GNU string, and `man uname` shows a different page than the one online | This course image ships the Rust uutils implementations of the coreutils, so `man uname` resolves to the uutils page rather than the GNU coreutils page. The options are the same. | Read the local page — `man uname` — for the option list; compare it with the GNU `uname(1)` page only when you need the wording from coreutils itself. |
| `man 5 proc` has no `/proc/cpuinfo` section | The proc man pages were split up: `/proc/cpuinfo` moved to its own page, and per-process data to `proc_pid(5)`. | Read `man 5 proc_cpuinfo` for `/proc/cpuinfo` and `man 5 proc_pid` for the per-process directories; `man 5 proc` still covers the pseudo-filesystem itself. |

---

## 9. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man uname` — every field `uname -a` prints. On this course image the local page is the Rust
  uutils page, not the GNU coreutils one.
- `man 5 os-release` — the `/etc/os-release` file format and all its keys.
- `man 5 proc` — the `/proc` pseudo-filesystem. On this image, `/proc/cpuinfo` is documented in
  `man 5 proc_cpuinfo` and per-process data in `man 5 proc_pid`.
- `man ps` — listing processes and the meaning of each output column.
- `man top` — the live process view and its options.
- `man bash` — the shell itself, including the `type` builtin.

Online copies of the same pages:

- [`uname(1)`](https://man7.org/linux/man-pages/man1/uname.1.html)
- [`os-release(5)`](https://man7.org/linux/man-pages/man5/os-release.5.html)
- [`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html)
- [`ps(1)`](https://man7.org/linux/man-pages/man1/ps.1.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — the template your `write-up.md` is made from.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) —
  where M1 sits in the curriculum (Layer 1, Week 1, with M2).
- Curriculum resource for this stage: [What Is Linux? From Kernel to IoT, DevOps, &
  Supercomputers](https://www.youtube.com/watch?v=H8Q64gHNp6Y).
