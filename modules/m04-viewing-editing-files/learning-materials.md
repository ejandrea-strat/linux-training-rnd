# M4 Learning Materials — Viewing & Editing Files

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, comfortable with the M1–M3
> material (shell, pipes, redirection, `man`, the `~/linux-course` directory layout). You need
> the Linux environment from M0 (WSL2, a VM, or Linux-in-a-container), a terminal, and — for
> part of the lab — a second terminal tab or window.
>
> **Scope note.** Every example output below is **illustrative** unless it is explicitly labeled
> as coming from the author's observation notes
> ([`knowledge/observations/m04-viewing-editing-files.md`](../../knowledge/observations/m04-viewing-editing-files.md)).
> The labeled outputs were captured on WSL2 on a specific machine; values such as hostname
> (`CANATOY-1063`), username (`ejandrea`), timestamps, PIDs, log line counts, and file paths will
> differ on yours. Never copy an example as evidence — run the command and paste your own output.
> Tools with no captured output in the notes (`less`, `nano`) are described in words only.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes roughly **95–100 minutes**. M4
> shares Week 2 of the course with M3 (5–6 estimated hours in total for the filesystem/data
> layer).

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

_Estimated: 15 minutes._

**Understand.** A small set of tools covers nearly every day-to-day viewing case, and which one
fits depends on whether the file is short, long, or still growing:

| Tool | What it does | Choose it when |
| --- | --- | --- |
| `cat` | Prints the whole file to the screen in one go | The file is a screenful or less, and you want the text to stay on screen (to read or copy) |
| `less` | Opens the file in a full-screen pager | The file is longer than a screenful; you need to scroll or search inside it |
| `head` | Prints the first 10 lines (change with `-n`) | You want the start of a file — e.g. a log's oldest entries |
| `tail` | Prints the last 10 lines (change with `-n`) | You want the end of a file — e.g. a log's newest entries |
| `tail -f` | Prints the last lines, then keeps printing new lines as they are written | The file is still growing — see [§3](#3-following-a-live-file) |

Inside `less`: Space moves forward a page, `b` moves back, `/word` searches, `n` jumps to the
next match, and `q` quits — it is the same viewer `man` uses, which you already met in M2. The
author's notes report that `less /etc/os-release` opened the file in a full-screen viewer and
that pressing `q` restored the screen with none of the file left on it; there is no captured
`less` screen output, so the guide does not show one. See `man less`, `man head`, and
`man tail`.

**Try.**

```bash
cat /etc/os-release          # whole file, stays on screen
less /etc/os-release        # page at a time; press q to exit
wc -l /var/log/syslog       # how many lines is "long"?
head -5 /var/log/syslog     # the oldest lines still in the file
tail -5 /var/log/syslog     # the newest lines
```

Output from the author's observation notes:

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
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

```text
$ wc -l /var/log/syslog
1675 /var/log/syslog
$ head -5 /var/log/syslog
2026-10-20T00:14:19.879621+08:00 CANATOY-1063 rsyslogd: [origin software="rsyslogd" swVersion="8.2512.0" x-pid="490" x-info="https://www.rsyslog.com"] rsyslogd was HUPed
2026-10-20T00:14:11.099267+08:00 CANATOY-1063 kernel: message repeated 3 times: [ mini_init (144): drop_caches: 3]
2026-10-20T00:14:51.827769+08:00 CANATOY-1063 kernel: bash: bash: potentially unexpected fatal signal 8.
2026-10-20T00:14:51.827818+08:00 CANATOY-1063 kernel: CPU: 6 UID: 1000 PID: 1094 Comm: bash Not tainted 6.18.54.1-microsoft-standard-WSL2 #1 PREEMPT(none)
2026-10-20T00:14:51.829110+08:00 CANATOY-1063 kernel: RIP: 0033:0x7ddb8e245ebb
$ tail -5 /var/log/syslog
2026-10-20T20:17:32.809052+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory
2026-10-20T20:21:33.066436+08:00 CANATOY-1063 wsl-pro-service[43235]: message repeated 4 times: [ #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory]
2026-10-20T20:21:33.066569+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Exiting after <nil>: check if the Windows agent is installed and running.
2026-10-20T20:21:33.068558+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Deactivated successfully.
2026-10-20T20:21:33.068663+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Consumed 317ms CPU time over 31min 4.576s wall clock time, 76.1M memory peak.
```

Reading these together: `cat` printed all 13 lines of `/etc/os-release` at once and the text
stayed on screen. The syslog is 1,675 lines — far too much for `cat`. `head -5` shows the oldest
lines still in the file (from 00:14; the first line, `rsyslogd was HUPed`, is what rsyslog logs
when it reopens its files after log rotation, so the file only goes back to then). `tail -5`
shows the newest lines (20:17–20:21, shortly before the author ran it). Every syslog line has
the same layout: a timestamp, the hostname (`CANATOY-1063` on the author's machine — use your
own hostname when reading your output), the program that wrote it, and the message.

**Verify.** Your `cat /etc/os-release` output stays on screen after the command finishes; your
`wc -l` prints a line count; `head -5` and `tail -5` print 5 lines each, with the `tail`
timestamps later than the `head` timestamps. You exited `less` with `q` and got your prompt and
screen back.

**Continue** to [§2](#2-searching-contents) to pull specific lines out of a file that is too long
to read.

---

## 2. Searching contents

_Estimated: 15 minutes._

**Understand.** `grep` prints every line that contains a match for the pattern. The pattern is a
regular expression by default (you met this idea in M2), so `.` matches any character unless you
pass `-F` to treat the pattern as plain text. The options you will use most:

| Option | Effect |
| --- | --- |
| `-i` | Ignore upper/lower case — `error`, `Error`, and `ERROR` all match |
| `-n` | Put the line number in front of each match |
| `-v` | Invert the match: print the lines that do **not** match |
| `-c` | Print only the count of matching lines, not the lines |
| `-F` | Treat the pattern as a fixed string, not a regular expression |

`-i` matters for logs because log messages are inconsistently capitalized — that is why the
brief recommends `grep -i` for log searching. `grep` matches text, not meaning: a line matches
whether or not it reports an actual problem. See `man grep`.

**Try.**

```bash
grep -i -c "error" /var/log/syslog          # how many lines match?
grep -i -n "error" /var/log/syslog | head -5   # first few matches, with line numbers
grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
```

(Output redirected with `>` is the M2 skill; the destination directory here is the author's M3
project layout — use the path that fits yours.)

Output from the author's observation notes:

```text
$ grep -i -c "error" /var/log/syslog
39

$ grep -i -n "error" /var/log/syslog | head -5
14:2026-10-20T00:16:56.255896+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
15:2026-10-20T00:16:56.255992+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
16:2026-10-20T00:16:56.256005+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
129:2026-10-20T09:58:17.441698+08:00 CANATOY-1063 systemd[1]: apport-autoreport.path - Process error reports when automatic reporting is enabled (file watch) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
130:2026-10-20T09:58:17.441700+08:00 CANATOY-1063 systemd[1]: apport-autoreport.timer - Process error reports when automatic reporting is enabled (timer based) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport

$ grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
$ wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
39 /home/ejandrea/linux-course/m3/data/processed/syslog-errors.txt
```

Reading these: 39 of the log's 1,675 lines matched, and the saved file has the same 39 lines —
the count agreeing is how you know nothing was trimmed by hand. Because of `-i`, the matches
include `ERROR`, `error`, and `Errors`. Note lines 129–130: they only contain the word in
"Process error reports" — `grep` matched the text, not what the line means. Not every match is a
real error; judging that is a separate step. Producing the filtered excerpt for the deliverable,
and seeing the rest of the matches, is [§5](#5-worked-example-the-m4-lab), Step 4.

**Verify.** `-c` prints just a number; `-n` output starts each line with `N:` where `N` is the
line's position in the file; your saved excerpt's `wc -l` equals the `-c` count.

**Continue** to [§3](#3-following-a-live-file) to watch a file while it is being written.

---

## 3. Following a live file

_Estimated: 15 minutes._

**Understand.** Logs are how you observe the behavior of a system, and they grow while you watch.
`tail -f file` prints the last lines, then keeps the file open and prints each new line as it is
written. Two details worth knowing:

- **`-f` versus `-F`.** `tail -f` follows that file by name; `tail -F` also keeps following if
  the file is replaced, which happens when logs are rotated. See `man tail`.
- **Stopping.** Ctrl+C stops `tail -f`. It sends an interrupt to the running command, and your
  prompt comes back — the same way it stops any other foreground command.

To produce a new line on purpose while following the system log, `logger "message"` writes a line
into the system log from your own shell. See `man logger`.

**Try.** In one terminal:

```bash
tail -f /var/log/syslog
```

Leave it running. In a separate terminal or tab, generate activity — the lab suggests
`sudo apt update`, and `logger "your own test message"` adds a line you can recognize
uniquely. Then return to the first terminal and watch the new lines arrive. Stop with Ctrl+C.

The author's full captured sequence, including both terminals, is shown with the rest of the lab
evidence in [§5](#5-worked-example-the-m4-lab), Step 3 — including one finding to expect: the
`sudo` command's own log line does **not** appear in `/var/log/syslog`. The author confirmed
where it went:

Output from the author's observation notes:

```text
$ grep apt /var/log/auth.log | tail -3
2026-10-20T20:32:03.338130+08:00 CANATOY-1063 sudo: ejandrea : TTY=/dev/pts/4 ; PWD=/home/ejandrea/projects/linux-training-rnd ; USER=root ; COMMAND=/usr/bin/apt update
```

Authentication-related events such as `sudo` go to `/var/log/auth.log`, not `/var/log/syslog`;
what appeared in `syslog` were the services that `apt update` set off (more in
[§5](#5-worked-example-the-m4-lab), Step 3).

**Verify.** New lines appear in the `tail -f` terminal without you re-running the command there;
after Ctrl+C the prompt returns and the terminal accepts commands again.

**Continue** to [§4](#4-editing-in-the-terminal) to change a file with no GUI in sight.

---

## 4. Editing in the terminal

_Estimated: 15 minutes._

**Understand.** Configuration on a server is routinely edited with no GUI available, so terminal
editors are core skills. The lab uses `nano`; the brief's best practice also names `vim`'s
escape hatch. The workflow to learn is the same shape in both: **open → change → save → exit →
show what changed**.

**`nano`.** `nano <file>` opens the file, or starts a new one if it does not exist. Type to make
changes. Ctrl+O saves (press Enter to confirm the file name); Ctrl+X exits. The bottom two lines
of the screen list the shortcuts, where `^` means Ctrl. `nano`'s behavior is described from the
author's notes; there is no captured `nano` screen in them, so none is shown here. See
`man nano`.

**`vim` (the escape hatch).** If you open a file in `vim` by accident: press `i` to start
typing (insert mode), `Esc` to stop typing, then `:wq` and Enter to write and quit. That
sequence is the brief's best practice for first-time `vim` users.

**Showing what changed — `diff`.** To prove an edit, keep a copy from before it, then compare:

```bash
cp file file.bak        # the "before" copy
# ... edit file ...
diff -u file.bak file   # unified diff: what changed
```

In unified diff output, the `---` and `+++` lines name the two files and their timestamps; a
`@@` line locates the change; lines starting with `-` were removed and lines starting with `+`
were added; lines starting with a space are unchanged context. See `man diff`. The author's real
`cp` → `nano` → `diff -u` sequence on their M3 project README is worked through in
[§5](#5-worked-example-the-m4-lab), Step 6.

**Try.**

```bash
nano ~/linux-course/m3/data/raw/notes.txt     # or a practice path of your choice
```

Add two lines of text, save with Ctrl+O (Enter to confirm), exit with Ctrl+X. Then — without
reopening the editor — confirm from the command line:

```bash
cat ~/linux-course/m3/data/raw/notes.txt
```

Output from the author's observation notes:

```text
$ nano ~/linux-course/m3/data/raw/notes.txt
$ cat ~/linux-course/m3/data/raw/notes.txt
   M4 practice note: written in nano.
   Saved with Ctrl+O and closed with Ctrl+X.
```

Keys the author used in `nano`: Ctrl+O, then Enter to confirm the file name, and Ctrl+X to exit.
Both lines start with three spaces, because the example lines were copied with their indentation
— and `cat` shows exactly what is in the file, spaces included. (The lab places `notes.txt` under
`data/raw`; the author notes this does not match their own M3 convention of `data/raw` for
incoming data files, but kept the lab's path because DoD-04 checks it.)

**Verify.** Your `cat` prints your file's contents without opening the editor again, and the text
matches what you typed — including any leading spaces you did not intend to keep.

**Continue** to [§5](#5-worked-example-the-m4-lab) to produce the lab's evidence end to end.

---

## 5. Worked example: the M4 lab

> **Purpose.** The lab steps from [`brief.md`](./brief.md), plus the diff capture required by
> [`tasks.md`](./tasks.md) and the write-up's checkpoint evidence, with exactly what to capture.
> The outputs shown are the author's; yours will differ in hostname, timestamps, counts, and
> content. Record your own.

_Estimated: 25–30 minutes._

### Step 1 — View a file whole, then a page at a time

**Understand.** The point is the contrast: `cat` leaves the file on screen; `less` takes over
the screen and clears it away on quit.

**Try.**

```bash
cat /etc/os-release
less /etc/os-release      # press q to exit
```

Raw output of the `cat` command from the author's observation notes (WSL2, Ubuntu 26.04.1 LTS):

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
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

The author's account of the `less` half, in words: it opened the file in a full-screen viewer,
and when `q` was pressed, the screen went back to how it looked before, with none of the file
left on it. There is no captured `less` screen, so paste your own description of what you saw.

**Capture.** In your notes: the full `cat` output, prompt included, and a sentence on how `less`
differed — and when you would pick each (see the tool table in [§1](#1-viewing-a-file)).

### Step 2 — Top and bottom of the log

**Understand.** `tasks.md` asks you to view your log from the top and the bottom and note which
tool fit best. `wc -l` sizes the problem; `head` and `tail` sample each end.

**Try.**

```bash
wc -l /var/log/syslog
head -5 /var/log/syslog
tail -5 /var/log/syslog
```

Raw output from the author's observation notes:

```text
$ wc -l /var/log/syslog
1675 /var/log/syslog
$ head -5 /var/log/syslog
2026-10-20T00:14:19.879621+08:00 CANATOY-1063 rsyslogd: [origin software="rsyslogd" swVersion="8.2512.0" x-pid="490" x-info="https://www.rsyslog.com"] rsyslogd was HUPed
2026-10-20T00:14:11.099267+08:00 CANATOY-1063 kernel: message repeated 3 times: [ mini_init (144): drop_caches: 3]
2026-10-20T00:14:51.827769+08:00 CANATOY-1063 kernel: bash: bash: potentially unexpected fatal signal 8.
2026-10-20T00:14:51.827818+08:00 CANATOY-1063 kernel: CPU: 6 UID: 1000 PID: 1094 Comm: bash Not tainted 6.18.54.1-microsoft-standard-WSL2 #1 PREEMPT(none)
2026-10-20T00:14:51.829110+08:00 CANATOY-1063 kernel: RIP: 0033:0x7ddb8e245ebb
$ tail -5 /var/log/syslog
2026-10-20T20:17:32.809052+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory
2026-10-20T20:21:33.066436+08:00 CANATOY-1063 wsl-pro-service[43235]: message repeated 4 times: [ #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory]
2026-10-20T20:21:33.066569+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Exiting after <nil>: check if the Windows agent is installed and running.
2026-10-20T20:21:33.068558+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Deactivated successfully.
2026-10-20T20:21:33.068663+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Consumed 317ms CPU time over 31min 4.576s wall clock time, 76.1M memory peak.
```

**Capture.** The line count, plus the `head` and `tail` output with their prompts. Note in your
own words which tool fit best for which end of the job.

### Step 3 — Follow the log as it grows (lab step 2)

**Understand.** Two terminals: one follows, one generates. The evidence is new lines arriving in
the follower that were not there when it started.

**Try.** Terminal A:

```bash
tail -f /var/log/syslog
```

Leave it running. Terminal B:

```bash
sudo apt update
logger "M4 test: following syslog with tail -f"
```

Watch Terminal A. Then stop with Ctrl+C.

Raw output from the author's observation notes — Terminal A (WSL2, hostname `CANATOY-1063`;
`tail -f` first prints the last 10 lines already in the file, so those ten — 20:17 to 20:31 —
were pre-existing, and the rest arrived while following):

```text
$ tail -f /var/log/syslog
2026-10-20T20:17:01.729041+08:00 CANATOY-1063 CRON[43561]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
2026-10-20T20:17:32.809052+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory
2026-10-20T20:21:33.066436+08:00 CANATOY-1063 wsl-pro-service[43235]: message repeated 4 times: [ #033[33mWARNING#033[0m Daemon: could not connect to Windows Agent: could not get address: could not read agent port file "/mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address": open /mnt/c/Users/ejandrea.canatoy_str.CANATOY-1063/.ubuntupro/.address: no such file or directory]
2026-10-20T20:21:33.066569+08:00 CANATOY-1063 wsl-pro-service[43235]: #033[33mWARNING#033[0m Exiting after <nil>: check if the Windows agent is installed and running.
2026-10-20T20:21:33.068558+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Deactivated successfully.
2026-10-20T20:21:33.068663+08:00 CANATOY-1063 systemd[1]: wsl-pro.service: Consumed 317ms CPU time over 31min 4.576s wall clock time, 76.1M memory peak.
2026-10-20T20:31:07.687343+08:00 CANATOY-1063 kernel: message repeated 7 times: [ mini_init (139): drop_caches: 3]
2026-10-20T20:31:48.484230+08:00 CANATOY-1063 kernel: WSL (45288 - Relay(45291)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T20:31:48.484449+08:00 CANATOY-1063 kernel: WSL (45288 - Relay(45291)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T20:31:48.484457+08:00 CANATOY-1063 kernel: WSL (45288 - Relay(45291)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T20:32:03.413203+08:00 CANATOY-1063 systemd[1]: Starting apt-news.service - Update APT News...
2026-10-20T20:32:03.449315+08:00 CANATOY-1063 systemd[1]: Starting esm-cache.service - Update the local ESM caches...
2026-10-20T20:32:05.469667+08:00 CANATOY-1063 systemd[1]: apt-news.service: Deactivated successfully.
2026-10-20T20:32:05.471408+08:00 CANATOY-1063 systemd[1]: Finished apt-news.service - Update APT News.
2026-10-20T20:32:05.696120+08:00 CANATOY-1063 dbus-daemon[151]: [system] Activating via systemd: service name='org.freedesktop.PackageKit' unit='packagekit.service' requested by ':1.16' (uid=0 pid=45829 comm="/usr/bin/gdbus call --system --dest org.freedeskto" label="kernel")
2026-10-20T20:32:05.701566+08:00 CANATOY-1063 systemd[1]: Starting packagekit.service - PackageKit Daemon...
2026-10-20T20:32:05.746686+08:00 CANATOY-1063 PackageKit: daemon start
2026-10-20T20:32:05.820581+08:00 CANATOY-1063 dbus-daemon[151]: [system] Successfully activated service 'org.freedesktop.PackageKit'
2026-10-20T20:32:05.821163+08:00 CANATOY-1063 systemd[1]: Started packagekit.service - PackageKit Daemon.
2026-10-20T20:32:06.600370+08:00 CANATOY-1063 systemd[1]: esm-cache.service: Deactivated successfully.
2026-10-20T20:32:06.601459+08:00 CANATOY-1063 systemd[1]: Finished esm-cache.service - Update the local ESM caches.
2026-10-20T20:32:12.157867+08:00 CANATOY-1063 ejandrea: M4 test: following syslog with tail -f
```

Raw output from the author's observation notes — Terminal B:

```text
$ sudo apt update
[sudo: authenticate] Password:
Hit:1 http://security.ubuntu.com/ubuntu resolute-security InRelease
Hit:2 http://archive.ubuntu.com/ubuntu resolute InRelease
Hit:3 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease
18 packages can be upgraded. Run 'apt list --upgradable' to see them.
$ logger "M4 test: following syslog with tail -f"
```

Reading the capture: `tail -f` printed the last 10 lines already in the file — 20:17 to 20:31 —
then kept waiting. After those ten pre-existing lines, new lines arrive from 20:32:03 onward. The
ones from 20:32:03 — systemd starting `apt-news.service` and `esm-cache.service`, then the
PackageKit daemon — are services that `sudo apt update` set off. The last line, at 20:32:12, is
the `logger` message, tagged with the author's username. The `sudo` command's own line is not in
`syslog`; the author confirmed it was logged in `/var/log/auth.log` instead (shown in
[§3](#3-following-a-live-file)). Then Ctrl+C stopped the follow and the prompt came back.

**Capture.** The pasted `tail -f` output showing at least one line that appeared *after* you
started following, attributable to your own action in the other terminal — plus proof you
stopped it (the prompt returning after Ctrl+C). DoD-02 checks this.

### Step 4 — Search the log (lab step 3)

**Understand.** Produce the module's deliverable: a filtered excerpt containing only matching
lines, made with `grep`, not by hand-trimming in an editor.

**Try.**

```bash
grep -i -c "error" /var/log/syslog
grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
cat ~/linux-course/m3/data/processed/syslog-errors.txt
```

Output from the author's observation notes — the count:

```text
$ grep -i -c "error" /var/log/syslog
39
```

Output from the author's observation notes — saving the excerpt and verifying it:

```text
$ grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
$ wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
39 /home/ejandrea/linux-course/m3/data/processed/syslog-errors.txt
```

The full excerpt, from the author's observation notes:

```text
$ cat ~/linux-course/m3/data/processed/syslog-errors.txt
2026-10-20T00:16:56.255896+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T00:16:56.255992+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T00:16:56.256005+08:00 CANATOY-1063 kernel: WSL (30361 - Relay(30362)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T09:58:17.441698+08:00 CANATOY-1063 systemd[1]: apport-autoreport.path - Process error reports when automatic reporting is enabled (file watch) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
2026-10-20T09:58:17.441700+08:00 CANATOY-1063 systemd[1]: apport-autoreport.timer - Process error reports when automatic reporting is enabled (timer based) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
2026-10-20T09:58:17.442641+08:00 CANATOY-1063 kernel: RAS: Correctable Errors collector initialized.
2026-10-20T09:58:20.354156+08:00 CANATOY-1063 kernel: WSL (156) ERROR: CheckConnection: getaddrinfo() failed: -5
2026-10-20T10:08:06.265110+08:00 CANATOY-1063 systemd[1]: apport-autoreport.path - Process error reports when automatic reporting is enabled (file watch) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
2026-10-20T10:08:06.265112+08:00 CANATOY-1063 systemd[1]: apport-autoreport.timer - Process error reports when automatic reporting is enabled (timer based) skipped, unmet condition check ConditionPathExists=/var/lib/apport/autoreport
2026-10-20T10:08:06.266013+08:00 CANATOY-1063 kernel: RAS: Correctable Errors collector initialized.
2026-10-20T10:08:52.230843+08:00 CANATOY-1063 kernel: WSL (155) ERROR: CheckConnection: getaddrinfo() failed: -5
2026-10-20T10:10:19.373024+08:00 CANATOY-1063 kernel: WSL (1649 - Relay(1650)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T10:10:19.373196+08:00 CANATOY-1063 kernel: WSL (1649 - Relay(1650)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T10:10:19.373202+08:00 CANATOY-1063 kernel: WSL (1652 - Relay(1653)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T10:10:19.373203+08:00 CANATOY-1063 kernel: WSL (1649 - Relay(1650)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T10:10:19.373214+08:00 CANATOY-1063 kernel: WSL (1652 - Relay(1653)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T10:10:19.376244+08:00 CANATOY-1063 kernel: WSL (1652 - Relay(1653)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T10:13:29.328696+08:00 CANATOY-1063 ubuntu-insights[4614]: 2026/10/20 10:13:29 WARN failed to run wsl.exe -v error="exit status 1"
2026-10-20T10:13:29.856393+08:00 CANATOY-1063 ubuntu-insights[4614]: 2026/10/20 10:13:29 ERROR failed to collect insights: insights write failed: report already exists for this period
2026-10-20T11:48:35.792384+08:00 CANATOY-1063 kernel: WSL (161) ERROR: No buffer space available @telemetry.cpp:190 (StartTelemetryAgent)
2026-10-20T16:36:16.408542+08:00 CANATOY-1063 kernel: WSL (32754 - Relay(32757)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T16:36:16.412198+08:00 CANATOY-1063 kernel: WSL (32754 - Relay(32757)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T16:36:16.412272+08:00 CANATOY-1063 kernel: WSL (32754 - Relay(32757)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T16:36:45.268493+08:00 CANATOY-1063 kernel: WSL (33388 - Relay(33389)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T16:36:45.268603+08:00 CANATOY-1063 kernel: WSL (33388 - Relay(33389)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T16:36:45.268609+08:00 CANATOY-1063 kernel: WSL (33388 - Relay(33389)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T16:36:45.280129+08:00 CANATOY-1063 kernel: WSL (33391 - Relay(33393)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T16:36:45.280156+08:00 CANATOY-1063 kernel: WSL (33391 - Relay(33393)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T16:36:45.280157+08:00 CANATOY-1063 kernel: WSL (33391 - Relay(33393)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T16:36:54.868801+08:00 CANATOY-1063 kernel: WSL (33687 - Relay(33688)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T16:36:54.868895+08:00 CANATOY-1063 kernel: WSL (33687 - Relay(33688)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T16:36:54.868905+08:00 CANATOY-1063 kernel: WSL (33687 - Relay(33688)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T17:25:02.540164+08:00 CANATOY-1063 kernel: WSL (41103 - Relay(41104)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T17:25:02.540396+08:00 CANATOY-1063 kernel: WSL (41107 - Relay(41109)) ERROR: ReceiveMessage:523: ExpectedMessage LxInitMessageExitStatus, but socket Control was closed
2026-10-20T17:25:02.540402+08:00 CANATOY-1063 kernel: WSL (41103 - Relay(41104)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T17:25:02.540520+08:00 CANATOY-1063 kernel: WSL (41107 - Relay(41109)) ERROR: InitCreateProcessUtilityVm:2129: Failed to write exit status 5
2026-10-20T17:25:02.544226+08:00 CANATOY-1063 kernel: WSL (41103 - Relay(41104)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T17:25:02.544270+08:00 CANATOY-1063 kernel: WSL (41107 - Relay(41109)) ERROR: InitCreateProcessUtilityVm:2149: waitpid failed 5
2026-10-20T19:55:57.199284+08:00 CANATOY-1063 kernel: WSL (155) ERROR: CheckConnection: getaddrinfo() failed: -5
```

Reading the excerpt: every line matches, and nothing was trimmed by hand — the `-c` count (39)
equals the file's line count. Because of `-i`, it caught `ERROR`, `error`, and `Errors` alike.
Not every match is a real error: the `apport-autoreport` lines only contain the word in "Process
error reports", the `RAS: Correctable Errors collector initialized.` lines are startup messages,
and one `ubuntu-insights` line is a `WARN` that only mentions `error="exit status 1"`. Most
matches (31 of the 39 on the author's machine) are WSL's own kernel messages. The excerpt is
also a snapshot: it stops at 19:55, so the three WSL `ERROR` lines from 20:31 — which appeared
in the `tail -f` capture in Step 3 — came after it was saved.

**Capture.** The command you used, the `-c` count, and the full `cat` of your excerpt file. The
excerpt must contain only matching lines, produced by the tool. DoD-03 checks this.

### Step 5 — Create and edit a file in nano (lab steps 4–5)

**Understand.** Two lines written in the editor, confirmed from the command line — proving the
round trip without any GUI.

**Try.**

```bash
nano ~/linux-course/m3/data/raw/notes.txt
# type two lines; Ctrl+O, Enter to save; Ctrl+X to exit
cat ~/linux-course/m3/data/raw/notes.txt
```

Raw output from the author's observation notes:

```text
$ nano ~/linux-course/m3/data/raw/notes.txt
$ cat ~/linux-course/m3/data/raw/notes.txt
   M4 practice note: written in nano.
   Saved with Ctrl+O and closed with Ctrl+X.
```

Both lines start with three spaces because the example lines were copied with their indentation —
`cat` shows exactly what is in the file. Use your own practice text; the path is the lab's, kept
because DoD-04 checks it.

**Capture.** The pasted `cat` output showing your two lines, prompt included. DoD-04 checks this.

### Step 6 — Show what changed with diff (from tasks.md)

**Understand.** `tasks.md` requires at least one terminal edit with a visible diff, and the
write-up's checkpoint evidence asks for it. The pattern is: copy first, edit, then `diff -u`.

**Try.** The author used their M3 project README as the file to edit:

```bash
cp ~/linux-course/m3/docs/README.md ~/linux-course/m3/docs/README.md.bak
nano ~/linux-course/m3/docs/README.md
# make one small edit; Ctrl+O, Enter; Ctrl+X
diff -u ~/linux-course/m3/docs/README.md.bak ~/linux-course/m3/docs/README.md
```

Raw output from the author's observation notes:

```text
$ diff -u ~/linux-course/m3/docs/README.md.bak ~/linux-course/m3/docs/README.md
--- /home/ejandrea/linux-course/m3/docs/README.md.bak	2026-10-20 20:30:43.046544138 +0800
+++ /home/ejandrea/linux-course/m3/docs/README.md	2026-10-20 20:38:13.848760908 +0800
@@ -1,4 +1,5 @@
 # Orders data
+   data/raw/notes.txt holds my M4 nano practice notes.
 
 - `data/raw/orders/`: order files exactly as they arrive, one per day (`orders_YYYY-MM-DD.csv`). Never edited.
 - `data/processed/orders/`: cleaned copies made by `scripts/clean_orders.sh` (`orders_clean_YYYY-MM-DD.csv`).
```

Reading the diff: the `---` and `+++` lines name the two files and when each was last changed.
`@@ -1,4 +1,5 @@` says the change is near the top: 4 lines before, 5 after. The one line starting
with `+` is the added line; lines starting with a space are unchanged context. The added line
kept the three spaces copied by accident during this edit, and `diff` shows them too.

**Capture.** The `diff -u` output showing your added (or removed) line against the backup. Edit
a file of your own — the author's README is an example, not a requirement.

**Clean-up.** Stop `tail -f` with Ctrl+C if it is still running (Step 3). The practice file
(`notes.txt`) and your backup/diff artifacts stay under `~/linux-course/`.

**Continue** to [§6](#6-completion-checklist) to confirm the deliverable is complete.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] **DoD-01 — `cat` versus `less`, and when to choose each.** Your notes describe how `cat`
      and `less` behaved differently on `/etc/os-release`, and when each would be chosen
      (Lab step 1). *Verify via [§5](#5-worked-example-the-m4-lab), Step 1.*
- [ ] **DoD-02 — Following a growing file.** The pasted `tail -f` output shows the new line
      produced by your own `sudo apt update`, demonstrating the file was followed as it grew
      and then stopped with Ctrl+C (Lab step 2).
      *Verify via [§5](#5-worked-example-the-m4-lab), Step 3.*
- [ ] **DoD-03 — The filtered excerpt.** The `grep -i "error" /var/log/syslog` output
      contains only lines that match, with no manual trimming —
      the `-c` count and the saved file's `wc -l` agree (Lab step 3).
      *Verify via [§5](#5-worked-example-the-m4-lab), Step 4.*
- [ ] **DoD-04 — `notes.txt`, confirmed from the command line.** `notes.txt` exists with the
      two lines written in `nano`, and the pasted `cat` output confirms it (Lab steps 4–5).
      *Verify via [§5](#5-worked-example-the-m4-lab), Step 5.*

When all four are ticked, transfer the evidence — including the diff that
[`tasks.md`](./tasks.md) asks for ([§5](#5-worked-example-the-m4-lab), Step 6) — into your
write-up ([`write-up-template.md`](./write-up-template.md)) and continue to M5.

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Use `less` rather than `cat` for anything longer than a screenful.** `cat` on a 1,675-line
  log scrolls past before you can read it; `less` lets you page and search
  ([§1](#1-viewing-a-file)).
- **Use `tail -f` to watch a log during an active process.** Running a command in a second
  terminal and watching the lines arrive is how you confirm what it did
  ([§3](#3-following-a-live-file)).
- **Remember `Esc` and `:wq` when first learning `vim`.** Insert with `i`, leave insert mode
  with `Esc`, then `:wq` and Enter to write and quit ([§4](#4-editing-in-the-terminal)).
- **Search logs with `grep -i`, since log messages are inconsistently capitalized.** The
  author's own excerpt caught `ERROR`, `error`, and `Errors` only because of `-i`
  ([§2](#2-searching-contents)).

### Troubleshooting

| Symptom | Likely cause | Recommended fix |
| --- | --- | --- |
| The `sudo` line you expected after `sudo apt update` never appears in `/var/log/syslog`, even though other `apt`-related lines do | `sudo` logs its commands to `/var/log/auth.log`, not `/var/log/syslog`. The author verified this with `grep apt /var/log/auth.log \| tail -3` ([§3](#3-following-a-live-file)). | Check `/var/log/auth.log` for the `sudo` line; judge the `tail -f` evidence by the service lines `apt update` set off (systemd units, PackageKit) instead. |
| Your `notes.txt` (or diff) shows leading spaces you do not remember typing | Text copied from an example — including the example's indentation — lands in the file verbatim; `cat` and `diff` show exactly what is on disk. The author's own notes.txt lines and README edit each kept three stray spaces ([§5](#5-worked-example-the-m4-lab), Steps 5–6). | Retype the line instead of pasting, or delete the leading spaces in the editor before saving. `cat` after saving is the ground truth. |
| `grep -i "error"` returns lines that are not errors — startup messages, or the word "error" inside unrelated phrases | `grep` matches text, not meaning. In the author's excerpt, `apport-autoreport` lines matched "Process error reports", and a `RAS: Correctable Errors collector initialized.` line is a normal startup message ([§2](#2-searching-contents)). | Expect some false positives; narrow the pattern (add context words or use `-w`) and review matches rather than assuming every hit is a failure. |

---

## 8. References

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man cat` — concatenate files and print.
- `man less` — the pager: paging keys, `/` search, `q` quit.
- `man head` — output the first part of files (`-n` to change the count).
- `man tail` — output the last part of files; `-f` to follow, `-F` to follow across renames.
- `man grep` — print lines matching a pattern; `-i`, `-n`, `-v`, `-c`, `-F`.
- `man nano` — the terminal editor: Ctrl+O save, Ctrl+X exit.
- `man diff` — compare files line by line; `-u` for unified output.
- `man logger` — make entries in the system log.

Online copies of the same pages (confirmed live):

- [`cat(1)`](https://man7.org/linux/man-pages/man1/cat.1.html)
- [`less(1)`](https://man7.org/linux/man-pages/man1/less.1.html)
- [`head(1)`](https://man7.org/linux/man-pages/man1/head.1.html)
- [`tail(1)`](https://man7.org/linux/man-pages/man1/tail.1.html)
- [`grep(1)`](https://man7.org/linux/man-pages/man1/grep.1.html)
- [`nano(1)`](https://manpages.debian.org/bookworm/nano/nano.1.en.html) (Debian mirror; man7.org
  does not host nano)
- [`diff(1)`](https://man7.org/linux/man-pages/man1/diff.1.html)
- [`logger(1)`](https://man7.org/linux/man-pages/man1/logger.1.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module, including the diff capture.
- [`write-up-template.md`](./write-up-template.md) — where the lab evidence goes.
- [`knowledge/observations/m04-viewing-editing-files.md`](../../knowledge/observations/m04-viewing-editing-files.md)
  — the author's captured output, used as the labeled examples in this guide.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) —
  where M4 sits in the curriculum (Filesystem/data layer, Week 2, with M3).
- Curriculum resource for this stage: *Viewing and Editing Files from the Terminal* (listed for
  the M3–M4 layer in the architecture doc).
