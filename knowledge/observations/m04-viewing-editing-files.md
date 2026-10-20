# M4 Observation Notes

## Expectations before investigating

1. **How do `cat`, `less`, `head`, `tail`, and `tail -f` differ, and when is each the right choice?**
   I think `cat` prints the whole file at once, so it's fine for short files like `/etc/os-release` from M1. `less` shows one page at a time and lets me scroll and search, like `man` in M2. `head` and `tail` show the first or last few lines, and `tail -f` keeps printing new lines as they're added, which fits a log.

2. **What does `grep` match?**
   I think `grep` prints every line that contains the pattern. From M2, the pattern is a regular expression by default, so `.` means any character unless I use `-F`.

3. **What do `grep`'s options for case, line numbers, and inverting a match do?**
   I think `-i` ignores upper and lower case, `-n` puts the line number in front of each match, and `-v` prints the lines that don't match instead.

4. **How do you follow a file as it grows?**
   I think `tail -f` does it: it shows the last lines, then waits and prints each new line as it's written.

5. **How do you stop following it?**
   I think Ctrl+C stops `tail -f` and gives me the prompt back, the same way it stops other commands.

6. **How do you open, change, save, and exit in nano?**
   I think `nano <file>` opens the file, or creates it if it doesn't exist. I type to change it, Ctrl+O saves, and Ctrl+X exits. The shortcuts are listed at the bottom of the screen.

7. **How do you show what changed in a file?**
   I think `diff` compares two files and prints the lines that are different, so I'd need a copy of the file from before the edit.

## What I learned from research

1. **Viewing a file:**
   - `cat` prints the whole file to the screen in one go. That's fine for a short file, but a long log just scrolls past.
   - `less` opens the file in a pager: Space and `b` move a page, `/word` searches, `n` jumps to the next match, and `q` quits. It's the same viewer `man` uses.
   - `head` prints the first 10 lines and `tail` the last 10. `-n 5` changes how many.
   - `tail -f` prints the last lines, then keeps the file open and prints new lines as they're written.
   - Rule of thumb: `cat` for a screenful or less, `less` for anything longer, `head` or `tail` for the start or the end, and `tail -f` for a file that's still growing. See `man less`, `man head` and `man tail`.

2. **Searching contents:**
   - `grep` prints every line that contains a match for the pattern. The pattern is a regular expression by default (M2), and `-F` makes it plain text.
   - `-i` ignores case, so `error`, `Error` and `ERROR` all match. Log messages aren't capitalized consistently, which is why the brief recommends `grep -i` for logs.
   - `-n` puts the line number in front of each match, `-v` inverts the match and prints the lines that don't match, and `-c` prints only the count. See `man grep`.

3. **Following a live file:**
   - `tail -f file` follows a file as it grows. `tail -F` also keeps following if the file is replaced, which happens when logs are rotated.
   - Ctrl+C stops it. It sends an interrupt to the running command, and the prompt comes back.
   - `logger "message"` writes a line into the system log, which is an easy way to make a new line appear while following `/var/log/syslog`. See `man tail` and `man logger`.

4. **Editing in the terminal:**
   - `nano file` opens the file, or starts a new one if it doesn't exist. Type to change it. Ctrl+O saves it (press Enter to confirm the file name), and Ctrl+X exits. The bottom two lines list the shortcuts, where `^` means Ctrl.
   - In `vim`, press `i` to start typing, `Esc` to stop, then `:wq` and Enter to save and quit. That's the brief's best practice for first using `vim`.
   - To show what changed, copy the file before editing (`cp file file.bak`), then run `diff -u file.bak file`. Lines starting with `-` were removed and lines starting with `+` were added. See `man nano` and `man diff`.

## Viewing a file

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

`less /etc/os-release`: it opened the file in a full-screen viewer, and when I pressed `q`, the screen went back to how it looked before, with none of the file left on it.

**Observation:** Shows that `cat` printed all 13 lines of `/etc/os-release` at once, the same file and output as in M1, and the text stayed on the screen. `less` showed the same file in a viewer and cleared it away when I quit. So `cat` fits a short file I want to keep on screen or copy, and `less` fits anything longer than a screenful, where I need to scroll or search.

## Top and bottom of the log

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

**Observation:** Shows that `/var/log/syslog` is 1,675 lines long, far too much to read with `cat`. `head -5` shows the oldest lines, from 00:14 today. The first one, `rsyslogd was HUPed`, is what rsyslog logs when it reopens its files after the log is rotated, so this file only goes back to then. `tail -5` shows the newest lines, from 20:17 to 20:21, shortly before I ran it. Every line has the same layout: a timestamp, the hostname `CANATOY-1063`, the program that wrote it, and the message.

## Following the log

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

In the other terminal:

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

Afterwards, checking where `sudo` logged the command:

```text
$ grep apt /var/log/auth.log | tail -3
2026-10-20T20:32:03.338130+08:00 CANATOY-1063 sudo: ejandrea : TTY=/dev/pts/4 ; PWD=/home/ejandrea/projects/linux-training-rnd ; USER=root ; COMMAND=/usr/bin/apt update
```

**Observation:** Shows that `tail -f` first printed the last 10 lines of the log, from 20:17 to 20:31, then kept waiting. When I ran `sudo apt update` in the other terminal, new lines arrived from 20:32:03: systemd starting `apt-news.service` and `esm-cache.service`, and the PackageKit daemon starting up. Those come from services that `apt update` sets off. `sudo`'s own line isn't in `syslog`: the `auth.log` check shows it logged the command there, at 20:32:03, just before the first new line. The last line, at 20:32:12, is my `logger` message, tagged with my username, `ejandrea`. Then I stopped `tail -f` with Ctrl+C, and the prompt came back.

## Searching the log

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

The full excerpt:

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

**Observation:** Shows that `grep -i "error"` matched 39 of the log's 1,675 lines, and the saved file has the same 39 lines, all of them matches, so nothing was trimmed by hand. `-c` gave the count without printing the lines, and `-n` added each match's line number in the log (14, 15, 16, 129 and 130). Because of `-i`, it caught `ERROR`, `error` and `Errors` alike. Not every match is a real error, though: the four `apport-autoreport` lines only contain the word in "Process error reports", the two `RAS: Correctable Errors collector initialized.` lines are startup messages, and one `ubuntu-insights` line is a `WARN` that only mentions `error="exit status 1"`. `grep` matches the text, not what the line means. Most of the matches, 31 of the 39, are WSL's own kernel messages, like `WSL (156) ERROR: CheckConnection: getaddrinfo() failed: -5`. The excerpt is also a snapshot: it stops at 19:55, so the three WSL `ERROR` lines from 20:31, which showed up in my `tail -f` output, came after I saved it.

## Editing in nano

```text
$ nano ~/linux-course/m3/data/raw/notes.txt
$ cat ~/linux-course/m3/data/raw/notes.txt
   M4 practice note: written in nano.
   Saved with Ctrl+O and closed with Ctrl+X.
```

Keys I used in nano: Ctrl+O, then Enter to confirm the file name, and Ctrl+X to exit.

**Observation:** Shows that `notes.txt` has both lines, confirmed with `cat` without opening `nano` again. Both lines start with three spaces, because I copied the example lines with their indentation, and `cat` shows exactly what's in the file. The lab puts `notes.txt` in `data/raw`, which doesn't fit my M3 scheme, where `data/raw` holds data files as they arrive. I kept the lab's path because DoD-04 checks it.

## Showing what changed

```text
$ cp ~/linux-course/m3/docs/README.md ~/linux-course/m3/docs/README.md.bak
$ nano ~/linux-course/m3/docs/README.md
$ diff -u ~/linux-course/m3/docs/README.md.bak ~/linux-course/m3/docs/README.md
--- /home/ejandrea/linux-course/m3/docs/README.md.bak	2026-10-20 20:30:43.046544138 +0800
+++ /home/ejandrea/linux-course/m3/docs/README.md	2026-10-20 20:38:13.848760908 +0800
@@ -1,4 +1,5 @@
 # Orders data
+   data/raw/notes.txt holds my M4 nano practice notes.
 
 - `data/raw/orders/`: order files exactly as they arrive, one per day (`orders_YYYY-MM-DD.csv`). Never edited.
 - `data/processed/orders/`: cleaned copies made by `scripts/clean_orders.sh` (`orders_clean_YYYY-MM-DD.csv`).
```

**Observation:** Shows that `diff -u` compared the copy from before the edit with the file after it. The `---` and `+++` lines name the two files and when each was last changed, and `@@ -1,4 +1,5 @@` says the change is near the top: 4 lines before, 5 after. The one line starting with `+` is the line I added, and the lines starting with a space are unchanged context. The added line kept the three spaces I copied by accident, and `diff` shows them too.