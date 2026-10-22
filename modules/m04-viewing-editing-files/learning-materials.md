# M4 Learning Materials — Viewing & Editing Files

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own `write-up.md`, made from
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, working in the course's Linux
> environment (WSL2, a VM, or Linux in a container). You need a terminal, and — for one lab step —
> a second terminal tab or window.
>
> **Scope note.** Every command output in this guide is **illustrative** — it shows the shape of
> what a command prints, not a record of a completed lab. Values such as your username, home path,
> hostname, timestamps, PIDs, addresses, and line counts differ on your machine. Run each command
> and paste your own output before you use any example as evidence.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **91 minutes** — 13 + 13 + 12 + 13
> for the four concept sections, 25 for the worked example, and 5 each for the checklist, the best
> practices, and the references. M4 shares **Week 2** of the course with M3, in **Layer 2
> (Filesystem & Data Handling)**. Commands assume **bash** on **WSL2 Ubuntu 26.04**, the course
> environment; on a Virtual Machine or in a container the same commands report your own
> environment's values.
>
> **Prerequisites.** M2's pipes and `>` redirection, for saving a filtered excerpt to a file. M3's
> project tree `~/linux-course/m3`, with its `data/raw`, `data/processed`, and `docs` folders — the
> lab writes into that tree, and a folder that is missing will stop the lab. The shell, the
> terminal, and `man` from M1 and M2 are assumed throughout.

## Contents

1. [Viewing a file](#1-viewing-a-file)
2. [Searching contents](#2-searching-contents)
3. [Following a live file](#3-following-a-live-file)
4. [Editing in the terminal](#4-editing-in-the-terminal)
5. [Worked example: the M4 lab](#5-worked-example-the-m4-lab)
6. [Completion checklist](#6-completion-checklist)
7. [Best practices and troubleshooting](#7-best-practices-and-troubleshooting)
8. [References](#8-references)

---

## 1. Viewing a file

_Estimated: 13 minutes._

**Understand.** A small set of tools covers nearly every day-to-day viewing case, and which one
fits depends on whether the file is short, long, or still growing:

| Tool | What it does | Choose it when |
| --- | --- | --- |
| `cat` | Prints the whole file to the screen in one go | The file is a screenful or less, and you want the text to stay on screen to read or copy |
| `less` | Opens the file in a full-screen pager | The file is longer than a screenful; you need to scroll or search inside it |
| `head` | Prints the first 10 lines (change the number with `-n`) | You want the start of a file — a log's oldest entries |
| `tail` | Prints the last 10 lines (change the number with `-n`) | You want the end of a file — a log's newest entries |
| `tail -f` | Prints the last lines, then keeps printing new lines as they are written | The file is still growing — see [§3](#3-following-a-live-file) |

`cat` and `less` read the same file and show the same text. The difference is what happens to your
screen afterwards. `cat` writes the file to the terminal and the text stays there, scrolling past if
it does not fit. `less` takes over the screen, shows the file a page at a time, and gives the
screen back exactly as it was when you quit.

Inside `less`: Space moves forward a page, `b` moves back, `/word` searches forward, `n` jumps to
the next match, and `q` quits. `less` is the same pager `man` uses, which you met in M2.

`head` and `tail` are the two ends of a file, without opening it. `head -5` gives the first five
lines and `tail -5` the last five. Neither scrolls, because neither prints more than you asked for.

**Try.**

```bash
cat /etc/os-release
less /etc/os-release
head -5 /etc/os-release
tail -5 /etc/os-release
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

`less /etc/os-release` opens the same file in a full-screen pager: the first lines appear and the
rest waits a page down. Press Space to move forward a page and `b` to move back, type `/` and a
word to search, and press `q` to quit. When you quit, the screen returns to the shell prompt with
none of the file left on it — that is the difference to notice against `cat`.

**Verify.** `cat` leaves the file's text on your screen; `less` takes the screen over and gives it
back. `head -5 /etc/os-release` prints the same first five lines that `cat` printed first, and
`tail -5 /etc/os-release` prints the same last five. It is the same file, read from a different
end.

**Continue** to [§2](#2-searching-contents) to pull the interesting lines out of a file you cannot
read all the way through.

---

## 2. Searching contents

_Estimated: 13 minutes._

**Understand.** `grep` prints every line that contains a match for a pattern and drops the rest.
It does not change the file — it is a filter on the way to your screen, and its output can be
redirected into a file, the way you did in M2.

The pattern is a regular expression by default, which you met in M2. A plain word is a regular
expression too, so `error` matches those five letters anywhere in a line — including inside
"errors", and inside longer words such as "errored". The metacharacters are what add the power.

The options that matter for logs:

| Option | What it does | Why you want it |
| --- | --- | --- |
| `-i` | Ignores case, so `error` also matches `Error` and `ERROR` | Log messages are not capitalized consistently, so `error` alone misses `ERROR` and `Error` |
| `-n` | Prints the line number before each match | Tells you *where* a match is, so you can go read around it |
| `-c` | Prints only the count of matching lines | Answers "how many?" without scrolling the matches |
| `-v` | Inverts the match and prints the lines that do **not** match | Finds the noise, so you can see what is *not* the problem |
| `-F` | Treats the pattern as plain text, not a regular expression | For when your pattern contains `.` or `*` and you mean them literally |

`grep` matches text, not meaning. A line that contains the word `error` inside a phrase like
"Process error reports" is a match, and so is a line that says "no errors detected". Read the
matches before you trust them.

**Try.**

```bash
grep -i "error" /var/log/syslog
grep -i -c "error" /var/log/syslog
grep -i -n "error" /var/log/syslog | head -5
```

```text
$ grep -i -c "error" /var/log/syslog
<count>

$ grep -i -n "error" /var/log/syslog | head -5
<n>:<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
<n>:<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
<n>:<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
<n>:<timestamp> linux-pc systemd[1]: apport-autoreport.path - Process error reports when automatic reporting is enabled (file watch) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
<n>:<timestamp> linux-pc systemd[1]: apport-autoreport.timer - Process error reports when automatic reporting is enabled (timer based) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
```

`-c` gives the number of matching lines and nothing else. `-n` prefixes each match with the line
number it came from in the original file, so `<n>` above is a position in `/var/log/syslog`, not a
count.

The `WSL` lines are why `-i` matters: they carry `ERROR` in capitals, and without `-i` the pattern
`error` would miss every one of them. The `systemd[1]` lines would match with or without `-i`,
because the word "error" is already lowercase in "Process error reports" — and they show that
`grep` matches text, not meaning, since those lines are not failures at all.

**Verify.** `grep -i -c "error" /var/log/syslog` and the number of lines that `grep -i "error"
/var/log/syslog` prints are the same number, because `-c` counts what the plain command shows.
Run the two commands back to back: the log keeps growing, so a new line that arrives between the
two runs makes the counts differ by one.

**Continue** to [§3](#3-following-a-live-file) to watch a log grow while you work.

---

## 3. Following a live file

_Estimated: 12 minutes._

**Understand.** Logs are how the behavior of a system is observed, and they grow while you watch.
`tail -f` prints the last lines of a file and then keeps the file open, printing each new line as
it is written, until you stop it.

`tail -f` follows the *file descriptor* — the open connection to the file — not the name. When a
log is rotated, the old file is renamed and a new file takes its name; `tail -f` keeps reading the
old, renamed file, so new lines stop appearing. `tail -F` follows the *name* and reopens the file,
so it keeps up across a rotation. GNU's `tail(1)` spells the difference out: "With --follow (-f),
tail defaults to following the file descriptor, which means that even if a tail'ed file is renamed,
tail will continue to track its end… Use --follow=name in that case. That causes tail to track the
named file in a way that accommodates renaming, removal and creation." `-F` is `--follow=name
--retry` in a single flag.

To stop following, press **Ctrl+C**. That sends an interrupt to the running command and the shell
prompt comes back. `tail -f` does not need a second terminal to start, but you do need one to make
something happen while it runs.

**Try.**

Before you follow the log, check that your user can read it. The mode and the owner tell you who
can:

```bash
ls -l /var/log/syslog
groups
```

```text
$ ls -l /var/log/syslog
-rw-r----- 1 syslog adm <size> <date> <time> /var/log/syslog
$ groups
trainee adm cdrom sudo dip plugdev users
```

The mode `-rw-r-----` says the owner (`syslog`) can read and write it, the group (`adm`) can read
it, and everyone else cannot. Your own user account reaches the file through the `adm` group, which
is why `groups` listing `adm` matters. If it does not, ask your trainer for access, or follow a log
your own user can read.

Open a second terminal tab. In the first one:

```bash
tail -f /var/log/syslog
```

`tail -f` prints the last ten lines and then waits. In the second tab, run something that writes to
the log:

```bash
sudo apt update
```

```text
$ sudo apt update
[sudo: authenticate] Password:
Hit:1 http://security.ubuntu.com/ubuntu resolute-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu resolute InRelease
Hit:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease
<count> packages can be upgraded. Run 'apt list --upgradable' to see them.
```

`sudo` runs one command as root and asks for your own password; M5 covers it in full. The lines
that appear in the `tail -f` window are not `apt`'s own — they come from the services `apt update`
starts:

```text
<timestamp> linux-pc systemd[1]: Starting apt-news.service - Update APT News...
<timestamp> linux-pc systemd[1]: Starting esm-cache.service - Update the local ESM caches...
<timestamp> linux-pc systemd[1]: apt-news.service: Deactivated successfully.
<timestamp> linux-pc systemd[1]: Finished apt-news.service - Update APT News.
<timestamp> linux-pc dbus-daemon[<pid>]: [system] Activating via systemd: service name='org.freedesktop.PackageKit' unit='packagekit.service' requested by ':1.<n>' (uid=0 pid=<pid> comm="/usr/bin/gdbus call --system --dest org.freedeskto" label="kernel")
<timestamp> linux-pc systemd[1]: Starting packagekit.service - PackageKit Daemon...
<timestamp> linux-pc PackageKit: daemon start
<timestamp> linux-pc dbus-daemon[<pid>]: [system] Successfully activated service 'org.freedesktop.PackageKit'
<timestamp> linux-pc systemd[1]: Started packagekit.service - PackageKit Daemon.
<timestamp> linux-pc systemd[1]: esm-cache.service: Deactivated successfully.
<timestamp> linux-pc systemd[1]: Finished esm-cache.service - Update the local ESM caches.
```

You can also add a line of your own with `logger`, which writes one message into the system log,
tagged with your username:

```bash
logger "M4 practice: watching syslog with tail -f"
```

```text
<timestamp> linux-pc trainee: M4 practice: watching syslog with tail -f
```

Press **Ctrl+C** in the `tail -f` window to stop following.

**Verify.** Lines appeared in the `tail -f` window that were not there before you ran `sudo apt
update` — that is the evidence that a file was followed as it grew. The `logger` line carries your
username, so you can pick it out of the noise. After Ctrl+C, your prompt is back and the window
stops filling.

**Continue** to [§4](#4-editing-in-the-terminal) to change a file from the terminal.

---

## 4. Editing in the terminal

_Estimated: 13 minutes._

**Understand.** A server has no GUI, so configuration gets edited in a terminal editor. Two are on
this image. The lab's own editing step uses **nano**, and so does the fourth item in the
definition of done; `vim` is here because it is the editor you will meet on production servers,
and because the brief's best practice is about it.

**nano** is the forgiving one. `nano file` opens the file, or starts a new one if it does not
exist. You type, and the shortcuts are listed at the bottom of the screen as you go, so there is
nothing to memorize. `^` means the Control key:

| Keys | What it does |
| --- | --- |
| `Ctrl+O` | Write the file out — save it |
| `Enter` | Confirm the file name, after `Ctrl+O` |
| `Ctrl+X` | Exit the editor |

**vim** is the one that is everywhere, and the one the brief's best practice is about. It has two
modes, and that is what makes it hard at first:

| Keys | What it does |
| --- | --- |
| `i` | Enter insert mode — now your keystrokes become text |
| `Esc` | Leave insert mode and go back to normal mode |
| `:wq` | Write the file and quit (Enter confirms) |
| `:q` | Quit. If you have unsaved changes, it refuses and shows `E37: No write since last change (add ! to override)` |
| `:q!` | Quit without saving, discarding the changes — the `!` overrides that refusal |

Both editors work the same way from the outside: open, change, save, exit, then confirm the result
from the shell with `cat`. `man nano` and `man vim` are the reference pages; vim's own help lives
inside the program (`:help`), and `vimtutor` is an interactive introduction installed on this
image.

To show what changed, copy the file before you edit it, then compare the two with `diff -u`. The
`---` line names the old file, the `+++` line the new one, and a `@@` line says where the change
is. A line starting with `-` was removed, a line starting with `+` was added, and a line starting
with a space is unchanged context.

**Try.**

Practice on a file the lab does not use, so the lab's own files stay untouched for
[§5](#5-worked-example-the-m4-lab), Steps 4–5:

```bash
vim ~/linux-course/m4-editor-practice.txt
```

`vim` opens the file in normal mode. Press `i` to enter insert mode, type one line of your own,
press `Esc` to go back to normal mode, then type `:wq` and press Enter to write the file and
quit. That is the whole sequence the brief's best practice is about — `Esc` and `:wq`.

Check the result from the shell:

```bash
cat ~/linux-course/m4-editor-practice.txt
```

```text
$ cat ~/linux-course/m4-editor-practice.txt
M4 vim practice: typed in insert mode, saved with :wq.
```

`cat` shows what is actually in the file, spaces and all. If a line starts with spaces you did not
type, the editor kept them — that happens when text is pasted in, because the paste brings its
indentation with it. Type the line instead of pasting it, or delete the leading spaces in the
editor.

`nano` works the same way from the outside — open, type, save with `Ctrl+O`, confirm with
`Enter`, exit with `Ctrl+X` — and it lists those shortcuts at the bottom of the screen as you go.
The lab's own editing step uses `nano` on `notes.txt`; see [§5](#5-worked-example-the-m4-lab),
Steps 4–5. The `diff -u` exercise that shows a changed file is in the deliverable steps of the
same section, under "Showing what changed."

**Verify.** `cat` of the practice file shows the line you typed, which means `:wq` wrote it out.
The lab's `notes.txt` and the `diff -u` output are separate evidence, gathered in
[§5](#5-worked-example-the-m4-lab), Steps 4–5 and "Showing what changed."

**Continue** to [§5](#5-worked-example-the-m4-lab) to run the whole lab end to end.

---

## 5. Worked example: the M4 lab

_Estimated: 25 minutes._

**Understand.** The lab views, follows, searches, and edits a real log file on your own system,
and ends with the brief's deliverable: a filtered log excerpt, produced by a search tool rather
than by an editor's find function.

### Step 1 — View a system file in full, then a page at a time

```bash
cat /etc/os-release
less /etc/os-release
```

`cat` prints all thirteen lines of `/etc/os-release` at once and leaves them on the screen. `less`
opens the same file in its own full-screen view; press `q` and the screen goes back to how it
looked before, with none of the file left on it.

*Capture:* the `cat` output, and a sentence on how the two behaved differently and when you would
choose each.

### Step 2 — Follow the system log as it grows

In the first terminal:

```bash
tail -f /var/log/syslog
```

In a second tab, run the trigger:

```bash
sudo apt update
```

```text
$ sudo apt update
[sudo: authenticate] Password:
Hit:1 http://security.ubuntu.com/ubuntu resolute-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu resolute InRelease
Hit:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease
<count> packages can be upgraded. Run 'apt list --upgradable' to see them.
```

New lines appear in the `tail -f` window as `apt update` starts the services it depends on:

```text
<timestamp> linux-pc systemd[1]: Starting apt-news.service - Update APT News...
<timestamp> linux-pc systemd[1]: Starting esm-cache.service - Update the local ESM caches...
<timestamp> linux-pc systemd[1]: apt-news.service: Deactivated successfully.
<timestamp> linux-pc systemd[1]: Finished apt-news.service - Update APT News.
<timestamp> linux-pc systemd[1]: Starting packagekit.service - PackageKit Daemon...
<timestamp> linux-pc PackageKit: daemon start
<timestamp> linux-pc systemd[1]: Started packagekit.service - PackageKit Daemon.
<timestamp> linux-pc systemd[1]: esm-cache.service: Deactivated successfully.
<timestamp> linux-pc systemd[1]: Finished esm-cache.service - Update the local ESM caches.
```

`apt` does not write a line of its own to `/var/log/syslog`. `sudo` records the command it ran in
a different log:

```bash
grep apt /var/log/auth.log | tail -3
```

```text
$ grep apt /var/log/auth.log | tail -3
<timestamp> linux-pc sudo: trainee : TTY=/dev/pts/<n> ; PWD=/home/trainee ; USER=root ; COMMAND=/usr/bin/apt update
```

Searching for `apt` finds the `sudo` line for the `apt update` command. Searching for `sudo`
instead would also match the PAM session lines that surround it, so the command's own line may
not be the last one you see.

Stop following with **Ctrl+C**.

*Capture:* the pasted `tail -f` output showing the new lines, and the Ctrl+C that stopped it.

### Step 3 — Search the log for a case-insensitive pattern

```bash
grep -i "error" /var/log/syslog
```

```text
$ grep -i "error" /var/log/syslog
<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
<timestamp> linux-pc kernel: WSL (<pid> - Relay(<pid>)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
<timestamp> linux-pc systemd[1]: apport-autoreport.path - Process error reports when automatic reporting is enabled (file watch) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
<timestamp> linux-pc systemd[1]: apport-autoreport.timer - Process error reports when automatic reporting is enabled (timer based) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
```

The five lines above are only the first five of a longer output — your log has more matches than
that. Every line printed contains the pattern in some case — that is what `-i` does. Read them
anyway: the `apport-autoreport` lines match the word "error" inside "Process error reports" and
are not failures.

*Capture:* the `grep` output with the command that produced it. Paste every line, not just these
five: the definition of done in [`brief.md`](./brief.md) rules out trimming.

### Step 4 — Create and edit a file in nano

```bash
nano ~/linux-course/m3/data/raw/notes.txt
```

Add two lines of text. Save with `Ctrl+O`, confirm with `Enter`, and exit with `Ctrl+X`.

*Capture:* a note of the keys you used.

### Step 5 — Confirm the file from the command line

```bash
cat ~/linux-course/m3/data/raw/notes.txt
```

```text
$ cat ~/linux-course/m3/data/raw/notes.txt
M4 practice note: written in nano.
Saved with Ctrl+O and closed with Ctrl+X.
```

*Capture:* the `cat` output, which is the evidence that the file exists with the two lines in it.

### Deliverable steps from tasks.md

**The top and bottom of the log, with `head` and `tail`.** Size the log before you open it, then
look at each end:

```bash
wc -l /var/log/syslog
head -5 /var/log/syslog
tail -5 /var/log/syslog
```

```text
$ wc -l /var/log/syslog
<count> /var/log/syslog

$ head -5 /var/log/syslog
<timestamp> linux-pc rsyslogd: [origin software="rsyslogd" swVersion="8.2512.0" x-pid="<pid>" x-info="https://www.rsyslog.com"] rsyslogd was HUPed
<timestamp> linux-pc kernel: message repeated 3 times: [ mini_init (<pid>): drop_caches: 3]
<timestamp> linux-pc kernel: bash: bash: potentially unexpected fatal signal 8.
<timestamp> linux-pc kernel: CPU: 6 UID: 1000 PID: <pid> Comm: bash Not tainted <version>-microsoft-standard-WSL2 #1 PREEMPT(none)
<timestamp> linux-pc kernel: RIP: 0033:0x7ddb8e245ebb

$ tail -5 /var/log/syslog
<timestamp> linux-pc CRON[<pid>]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
<timestamp> linux-pc chronyd[<pid>]: Selected source <address> (2.ntp.ubuntu.com)
<timestamp> linux-pc systemd[1]: Starting motd-news.service - Message of the Day...
<timestamp> linux-pc systemd[1]: motd-news.service: Deactivated successfully.
<timestamp> linux-pc systemd[1]: Finished motd-news.service - Message of the Day.
```

Every line has the same layout: a timestamp, the hostname, the program that wrote it, and the
message. `head` shows the oldest lines still in the file, `tail` the newest. The kernel release on
WSL2 carries a `-microsoft-standard-WSL2` suffix, because Microsoft builds the kernel rather than
the distribution.

*Capture:* the line count and both ends of the log.

**Saving the filtered excerpt to a file.** The brief's deliverable is the excerpt, produced by a
search tool rather than by hand:

```bash
grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
```

```text
$ wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
<count> /home/trainee/linux-course/m3/data/processed/syslog-errors.txt
```

The count from `wc -l` is the same as the count from `grep -i -c "error" /var/log/syslog`. Run the
two commands back to back, because the log keeps growing: a new line that arrives between the two
runs makes the counts differ by one.

*Capture:* the redirect command and the count.

**Showing what changed.** Copy the file, edit it, and diff the two. This uses the `README.md` from
M3's example scheme, which is a one-line file, so the diff is one added line against one
unchanged line:

```bash
cp ~/linux-course/m3/docs/README.md ~/linux-course/m3/docs/README.md.bak
nano ~/linux-course/m3/docs/README.md
diff -u ~/linux-course/m3/docs/README.md.bak ~/linux-course/m3/docs/README.md
```

If your M3 tree has no `README.md`, use `notes.txt` instead — copy it to `notes.txt.bak`, edit
`notes.txt`, and diff the two the same way. Its hunk header reads `@@ -1,2 +1,3 @@`, because
`notes.txt` starts with two lines:

```bash
cp ~/linux-course/m3/data/raw/notes.txt ~/linux-course/m3/data/raw/notes.txt.bak
nano ~/linux-course/m3/data/raw/notes.txt
diff -u ~/linux-course/m3/data/raw/notes.txt.bak ~/linux-course/m3/data/raw/notes.txt
```

```text
$ diff -u ~/linux-course/m3/docs/README.md.bak ~/linux-course/m3/docs/README.md
--- /home/trainee/linux-course/m3/docs/README.md.bak	<timestamp>
+++ /home/trainee/linux-course/m3/docs/README.md	<timestamp>
@@ -1 +1,2 @@
 Raw order files land in data/raw/orders.
+data/raw/notes.txt holds the M4 practice notes.
```

The hunk header `@@ -1 +1,2 @@` says the change is at the top of the file: one line before, two
after. The single line starting with `+` is the line you added.

*Capture:* the `diff` output.

**Write-up.** Fill every heading of [`write-up-template.md`](./write-up-template.md) in your own
`write-up.md`: "What I built," "Why it's built this way," "How to build it," "Concepts worth
explaining," "What tripped me up," "Checkpoint evidence," and "What I'd do differently." Under
"Checkpoint evidence," paste the filtered excerpt, the command that produced it, and the diff.
Finish with the final self-review: re-read your write-up against the definition of done in
[`brief.md`](./brief.md).

### Clean-up

The `tail -f` is already stopped — Ctrl+C in Step 2 did that. The practice files stay under
`~/linux-course/`, as the brief says: the editor practice file `~/linux-course/m4-editor-practice.txt`
from [§4](#4-editing-in-the-terminal), and the lab's own `notes.txt` under
`~/linux-course/m3/data/raw/`. What the clean-up does **not** cover: the saved excerpt in
`~/linux-course/m3/data/processed/` and the backup copy `~/linux-course/m3/docs/README.md.bak`
(or `~/linux-course/m3/data/raw/notes.txt.bak`, if you used the `notes.txt` fallback) both stay
on disk, because they are your evidence. Remove them only when you no longer need them for the
write-up.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Map your notes to the definition of done in [`brief.md`](./brief.md):

- [ ] DoD-01: The trainee's notes describe how `cat` and `less` behaved differently on
      `/etc/os-release` and when each would be chosen (Lab step 1). *Verify via
      [§5](#5-worked-example-the-m4-lab), Step 1.*
- [ ] DoD-02: The pasted `tail -f` output shows the new line produced by the trainee's own
      `sudo apt update`, demonstrating that a file was followed as it grew and then stopped (Lab
      step 2). *Verify via [§5](#5-worked-example-the-m4-lab), Step 2.*
- [ ] DoD-03: The `grep -i "error" /var/log/syslog` output contains only lines that match the
      pattern, with no manual trimming (Lab step 3). *Verify via
      [§5](#5-worked-example-the-m4-lab), Step 3; the saved excerpt in the deliverable steps is
      the same command, and its line count matches `grep -c`.*
- [ ] DoD-04: `notes.txt` exists with the two lines written in `nano`, and the pasted `cat` output
      confirms it from the command line (Lab steps 4–5). *Verify via
      [§5](#5-worked-example-the-m4-lab), Steps 4–5.*

When all four are ticked, transfer the evidence into your `write-up.md` and continue to M5.

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- Use `less` rather than `cat` for anything longer than a screenful
  ([§1](#1-viewing-a-file)).
- Use `tail -f` to watch a log during an active process
  ([§3](#3-following-a-live-file)).
- Remember `Esc` and `:wq` when first learning `vim`
  ([§4](#4-editing-in-the-terminal)).
- Search logs with `grep -i`, since log messages are inconsistently capitalized
  ([§2](#2-searching-contents)).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `cat: /home/trainee/linux-course/m3/data/raw/notes.txt: No such file or directory`, the same message from `grep` with a `grep: ` prefix, the bare path followed by `: No such file or directory` from `less`, or `tail: cannot open '/home/trainee/linux-course/m3/data/raw/notes.txt' for reading: No such file or directory` followed by `tail: no files remaining` | The path is wrong — the file is not at that path, or not there yet. The four commands report the one problem in their own words | Check the path with `ls -l ~/linux-course/m3/data/raw/` |
| `ls -l /var/log/syslog` shows `-rw-r----- 1 syslog adm`, and `groups` does not list `adm` | The log is owned by `syslog`, and is readable by the `adm` group and by root only. Your user account is not in that group | Confirm with `ls -l /var/log/syslog` and `groups`. Ask your trainer for access, or follow a log your own user can read |
| `grep` prints nothing and returns to the prompt | No line matches the pattern as you typed it. Log messages are capitalized inconsistently, so `error` misses `ERROR` and `Errors` | Check the pattern, or add `-i`: `grep -i "error" /var/log/syslog` ([§2](#2-searching-contents)) |
| `tail -f` keeps running, but no new lines appear after the log is rotated | `-f` follows the file descriptor, so it keeps reading the old, renamed file while the new lines go to the new file under the same name | Use `tail -F`, which follows the name and reopens the file ([§3](#3-following-a-live-file)) |
| `sudo apt update` adds no line of its own to `/var/log/syslog` | `sudo` logs the command it ran to `/var/log/auth.log` instead. The new lines in `syslog` come from the services `apt update` starts | Look for the `apt-news`, `esm-cache`, and `packagekit` lines, and check `grep apt /var/log/auth.log \| tail -3` for `sudo`'s own record ([§5](#5-worked-example-the-m4-lab), Step 2) |
| The lines in the file you edited start with spaces you did not type | Text pasted into the editor brought its indentation with it | Type the line instead of pasting it, or remove the leading spaces in the editor. `cat` shows the spaces exactly as they are in the file ([§4](#4-editing-in-the-terminal)) |
| `nano` prints `[ Directory '/home/trainee/linux-course/m3/data/raw' does not exist ]` | The folder the file goes in does not exist — usually because the M3 tree was never built | Create it with `mkdir -p ~/linux-course/m3/data/raw` ([§4](#4-editing-in-the-terminal)) |

---

## 8. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference. On this image `cat`, `head`, `tail`, and `wc` come from the uutils (`rust-`)
implementation of coreutils, so their local pages are the uutils ones and differ from the GNU
coreutils pages linked below. `cp`, `less`, `grep`, `diff`, `logger`, `nano`, and `vim` are the
GNU or upstream pages.

- `man cat` — the uutils (`rust-cat`) page on this image. Concatenating files onto standard output.
- `man head` — the uutils (`rust-head`) page on this image. The `-n` option that sets how many
  lines to print.
- `man tail` — the uutils (`rust-tail`) page on this image. The `-f` option that follows a growing
  file, and `-F`, which on this image is `--follow=name --retry`. The page documents `-F` but not
  the rotation behavior behind it; GNU's `tail(1)`, linked below, is the page that explains that
  `-f` follows the file descriptor and `--follow=name` follows the name.
- `man wc` — the uutils (`rust-wc`) page on this image. The `-l` option that counts lines.
- `man cp` — the GNU coreutils page. Copying a file, which is how the backup is made before an
  edit.
- `man less` — the GNU less page. The pager commands: Space, `b`, `/`, `n`, and `q`.
- `man grep` — the GNU grep page. The `-i`, `-n`, `-c`, `-v`, and `-F` options.
- `man nano` — the GNU nano page. Its EDITING section explains the `^` prefix for Control keys.
- `man diff` — the GNU diffutils page. The `-u` unified format and the `@@` hunk header.
- `man logger` — the util-linux page. Writing a message into the system log.
- `man vim` — the Vim page. Vim's own help is inside the program (`:help`); `vimtutor` is an
  interactive introduction, and `man vim` points at both.

Online copies of the same pages:

- [`cat(1)`](https://man7.org/linux/man-pages/man1/cat.1.html) — GNU coreutils' page; it differs
  from the local uutils one.
- [`head(1)`](https://man7.org/linux/man-pages/man1/head.1.html) — GNU coreutils' page; it differs
  from the local uutils one.
- [`tail(1)`](https://man7.org/linux/man-pages/man1/tail.1.html) — GNU coreutils' page; it differs
  from the local uutils one, and it is the page that explains the rotation behavior of `-f` and
  `--follow=name`.
- [`wc(1)`](https://man7.org/linux/man-pages/man1/wc.1.html) — GNU coreutils' page; it differs from
  the local uutils one.
- [`cp(1)`](https://man7.org/linux/man-pages/man1/cp.1.html)
- [`less(1)`](https://man7.org/linux/man-pages/man1/less.1.html)
- [`grep(1)`](https://man7.org/linux/man-pages/man1/grep.1.html)
- [`diff(1)`](https://man7.org/linux/man-pages/man1/diff.1.html)
- [`logger(1)`](https://man7.org/linux/man-pages/man1/logger.1.html)
- [nano's documentation](https://www.nano-editor.org/docs.php) — the editor's own site; there is no
  `nano(1)` page on man7.org.
- [Vim's help](https://vimhelp.org/) — the help files that `man vim` points at; there is no
  `vim(1)` page on man7.org.

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — the template your `write-up.md` is made from.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md)
  — where M4 sits in the curriculum (Layer 2, Week 2, with M3).

Curriculum resources for the Filesystem layer (M3–M4), by name:

- *Linux Filesystem Explained*
- *File System Structure Explained* (video)
- *Basic Linux Navigation and File Management*
- *Viewing and Editing Files from the Terminal*
