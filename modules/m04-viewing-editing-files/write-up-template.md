# Viewing & Editing Files Write-up

## What I built

A filtered log excerpt: `~/linux-course/m3/data/processed/syslog-errors.txt`, the 39 lines of `/var/log/syslog` that match `error` in any case, made with `grep -i "error" /var/log/syslog` and a redirect. I also followed the log with `tail -f` while `sudo apt update` ran in a second terminal, wrote `notes.txt` in `nano`, and edited my M3 README in `nano`, with `diff -u` against a backup copy to show the change. My setup is the same as in M1 to M3: WSL2 on Windows, Ubuntu 26.04.1 LTS, bash. The [observation notes](../../knowledge/observations/m04-viewing-editing-files.md) also cover `cat` vs. `less`, and the top and bottom of the log with `head` and `tail`.

I also used EPAV to turn my notes into [`learning-materials.md`](./learning-materials.md), a study guide that teaches M4 to the next trainee.

## Why it's built this way (key decisions)

**Log and pattern:** `/var/log/syslog` is a real, busy log on this system, 1,675 lines on the day of the lab, and it's the same log the lab follows with `tail -f`, so one file covers viewing, following and searching. The pattern is the lab's own, and `error` is the first thing to search for when something breaks. `-i` matters because log messages aren't capitalized consistently: it caught `ERROR`, `error` and `Errors` alike. The excerpt goes in the M3 tree's `data/processed`, since a filtered copy of a raw log is processed data.

**Editor:** `nano`, because the lab uses it and its shortcuts are listed at the bottom of the screen, so there was nothing to memorize. That saved time compared with `vim`, where saving takes `Esc` and `:wq`. The cost showed up once: example text pasted into `nano` kept its leading spaces, which `cat` and `diff` then showed.

## How to build it (teach it to the next trainee)

1. Size the file before you open it: `wc -l /var/log/syslog`. Mine had 1,675 lines, so `cat` was out.
2. Look at the ends with `head -5` and `tail -5`, and use `less` for anything in between: Space and `b` to page, `/word` to search, `q` to quit.
3. Follow the log while something happens: `tail -f /var/log/syslog` in one terminal, the action in another, like `sudo apt update`, or `logger "test"` for a line you can spot. New lines show up on their own, and Ctrl+C stops it.
4. Count the matches, then save them: `grep -i -c "error" /var/log/syslog`, then `grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt`. If `wc -l` on the file gives the same number as `-c`, nothing was trimmed.
5. Read the matches before trusting them. `grep` matches text, so some lines only contain the word.
6. Copy a file before you edit it: `cp README.md README.md.bak`. Edit it in `nano`, save with Ctrl+O and Enter, exit with Ctrl+X, then check it with `cat` and show the change with `diff -u README.md.bak README.md`.

**If a command's line doesn't show up in `syslog`:** check `/var/log/auth.log`. That's where `sudo` logs on Ubuntu.

## Concepts worth explaining

**Streaming vs. paginated viewing:** `cat` streams the whole file to the screen, and it stays there. That's fine for the 13 lines of `/etc/os-release`. `less` pages through a file in its own full-screen view, and when I quit, the screen went back to how it was. For a 1,675-line log, `less`, `head` or `tail` work, while `cat` just scrolls past.

**Pattern search:** `grep` keeps the lines that contain the pattern and drops the rest, without changing the file. With `-i`, `error` matched `ERROR`, `error` and `Errors`. But it matches text, not meaning: 7 of my 39 matches weren't errors, like "Process error reports" in an `apport` line. A saved excerpt is also a snapshot. Mine stopped at 19:55, and three new WSL errors arrived at 20:31, after I saved it.

## What tripped me up

- `sudo`'s own line never showed up in `syslog`. On Ubuntu, `sudo` logs to `/var/log/auth.log`. What `tail -f` showed were the services `apt update` started, like `apt-news` and `esm-cache`.
- Not every `error` match was an error. The `apport-autoreport` lines matched "Process error reports", and `RAS: Correctable Errors collector initialized.` is a startup message.
- My saved excerpt was out of date about a minute later. The three WSL errors from 20:31 aren't in it.
- I pasted the example lines with their indentation, so both lines in `notes.txt` and my README edit start with three spaces. `cat` and `diff` showed them exactly.
- The lab puts `notes.txt` in `data/raw`, which doesn't fit my M3 scheme, where `data/raw` holds data files as they arrive.

## Checkpoint evidence

**The command and its count:**
```text
$ grep -i "error" /var/log/syslog > ~/linux-course/m3/data/processed/syslog-errors.txt
$ wc -l ~/linux-course/m3/data/processed/syslog-errors.txt
39 /home/ejandrea/linux-course/m3/data/processed/syslog-errors.txt
```

**The filtered excerpt:**
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

**The diff from the terminal edit:**
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

## What I'd do differently

I'd type the practice lines in `nano` instead of pasting them, so they wouldn't carry three extra spaces. I'd also read through the `grep` matches before saving the excerpt, and narrow the pattern if the false positives get in the way. And I'd save the excerpt again after following the log, so it includes the newest lines.