# M2 Lab — CLI Basics

The M2 lab as I ran it on WSL2 with Ubuntu and bash: each command, its real output copied exactly, and what it shows.

## Redirection: > and >>

```text
$ mkdir -p ~/linux-course/m2
$ cd ~/linux-course/m2
$ echo "line one" > practice.txt
$ echo "line two" >> practice.txt
$ cat practice.txt
line one
line two
```

**Observation:** Shows that both lines are in practice.txt. Line one was written with `>`, which created the file, and line two with `>>`, which added it at the end. If I had used `>` for both, the second command would have erased line one, and the file would only have "line two".

**Expected vs actual:** I expected `>>` to be the one that overwrites, but it's the other way around: `>` overwrites and `>>` appends. So choosing `>` by mistake is what loses the file's contents, while choosing `>>` by mistake only leaves the old lines in. That's why line one was still there after the second `echo`.

## Pipes

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

**Observation:** Shows the entries in /etc that match ".conf", like `adduser.conf` and `ca-certificates.conf`. Read left to right, `ls -l /etc` lists everything in /etc, then `grep ".conf"` keeps only the lines that match. `dconf` and `netconfig` matched too, even without ".conf" in their names, because in grep `.` means any character.

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

**Observation:** Shows the same list, but this time it went into conf-list.txt instead of the screen. This is the redirected file listing the brief asks for: a pipe and a redirect in one command.

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

**Observation:** With `-F`, grep treats `.conf` as plain text, so `dconf` and `netconfig` are gone. `manpath.config` stays because it really contains ".conf".

## Built-in help

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

**Observation:** Shows that `man grep` and `grep --help` both explain grep's options. `man grep` opens a full manual I can scroll and search, while `grep --help` just prints a short summary.

Something I didn't know: grep treats the pattern as a regular expression by default, and `-F` (`--fixed-strings`) makes it match plain text instead.

**Expected vs actual:** I expected to move around a man page with nano commands, but `man` opens a viewer, not an editor. Space and `b` move between pages, `/` searches, `n` jumps to the next match, and `q` quits.

## Tab completion

Typing `cd ~/linux-cou` and pressing Tab turned it into `cd ~/linux-course/`.

```text
$ ls /etc/sys    (Tab pressed twice)
sysctl.d/ systemd/
```

**Observation:** Shows that Tab finished the path by itself, because only one folder starts with "linux-cou". With `ls /etc/sys`, more than one entry matched, and the two terminals behaved differently. In Warp, the list showed up on the first Tab, because Warp has its own completion menu. In the plain Ubuntu terminal, the first Tab only rang the bell and the second Tab listed them, which matches bash's default from my research. This shows the terminal emulator and the shell really are separate programs, like in M1.

**Expected vs actual:** I expected Tab to fill in the first match alphabetically when there's more than one, but neither Warp nor bash picks one for me. Plain bash rings the bell on the first Tab and lists the matches on the second, while Warp shows its own list on the first Tab.

## Short vs long flags

```text
$ ls -a ~/linux-course/m2
.  ..  conf-list.txt  practice.txt

$ ls --all ~/linux-course/m2
.  ..  conf-list.txt  practice.txt
```

**Observation:** Shows that `ls -a` and `ls --all` print the same list, including `.` and `..`, which plain `ls` hides. The short flag and the long flag are two names for the same option.

**Expected vs actual:** I expected the only difference between short and long flags to be how they're typed. That part was right, since `ls -a` and `ls --all` print the same list, but short flags can also be combined, like `-la`, and long flags can't. I also expected an argument to be any value passed to a flag or command, but an argument is what the command acts on, like `~/linux-course/m2` here. A value given to a flag, like `auto` in `--color=auto`, belongs to the flag.
