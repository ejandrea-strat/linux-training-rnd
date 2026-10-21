# CLI Basics Write-up

## What I built

`conf-list.txt`, a redirected file listing of the entries in `/etc` that match ".conf". One command made it, using both a pipe and a redirect: `ls -l /etc | grep ".conf" > conf-list.txt`. Before that, I made `practice.txt` with two lines, the first written with `>` and the second with `>>`. My setup is the same as in M1 (WSL2 on Windows, Ubuntu 26.04.1 LTS, bash), with Warp as my terminal and the plain Ubuntu terminal to compare Tab completion. The [lab file](../../knowledge/resources/m02-cli-basics-lab.md) also covers `man grep`, `grep --help`, Tab completion, and short vs. long flags.

I also used EPAV to write [`learning-materials.md`](./learning-materials.md), a study guide that teaches M2 to the next trainee.

## Why it's built this way (key decisions)

**Command and data:** the brief leaves both open, as long as they're real and not a toy example. `/etc` is where the system keeps its configuration files, so filtering it for `.conf` works on real files. It's also the lab's pipeline, so the deliverable only needed `> conf-list.txt` added at the end.

**Overwrite vs. append:**
- In `practice.txt`, line one uses `>` so the file starts fresh, and line two uses `>>` so it goes after line one instead of replacing it.
- With `>` on both lines, the second command would erase line one, and only "line two" would be left.
- With `>>` on both, the file would look the same the first time, but running the commands again would add a second "line one" and "line two" under the first two. Starting with `>` is what makes the commands give the same file every time they run.
- `conf-list.txt` uses `>` for the same reason: each run replaces the list. With `>>`, every run would add another full copy of the list to the end.

**Research questions that shaped the pipeline:**
- Reading a pipeline left to right set the order: `ls -l /etc` produces the listing, `grep ".conf"` keeps the lines that match, and `>` saves what's left to a file.
- What `>` and `>>` each do, and what's lost with the wrong one, decided which operator each file got.

## How to build it (teach it to the next trainee)

1. Run the first command on its own and look at what it prints: `ls -l /etc`. Know what goes into the pipe before you add anything to it.
2. Add a filter with a pipe: `ls -l /etc | grep ".conf"`. Read it left to right: `ls -l /etc` lists everything in `/etc`, then `grep` keeps only the lines that match.
3. Check the result for anything that doesn't belong. Mine had `dconf` and `netconfig`, which don't have ".conf" in their names. In grep, `.` means any character, so use `grep -F ".conf"` if you only want names that really contain ".conf".
4. When the output on the screen is what you want, add the redirect at the end: `ls -l /etc | grep ".conf" > conf-list.txt`. Nothing prints this time because the output went into the file, so run `cat conf-list.txt` to check it. I kept the lab's `grep ".conf"` for mine, so `dconf` and `netconfig` are still in it.
5. Pick `>` or `>>` on purpose. `>` gives you a fresh file on every run, which is what a listing like `conf-list.txt` needs. `>>` adds to the end, which is what you want for a log or any file whose old lines you need to keep.

**When a command surprises you:** check `man` or `--help` before guessing. `grep --help` is where `-F` shows up.

## Concepts worth explaining

**Pipes vs. redirection:** both change where output goes. A pipe sends it to another command, and a redirect sends it to a file instead of the screen. My deliverable uses both: the pipe hands the `ls -l /etc` listing to `grep`, and the redirect puts what `grep` kept into `conf-list.txt`. Only standard output (fd 1) goes through either one, so errors (fd 2) still show on the screen.

**How built-in help changes how I work:** I don't have to memorize flags or look them up online, because the answers are already on the machine. When `grep ".conf"` matched `dconf` and `netconfig`, grep's built-in help explained why: the pattern is a regular expression by default, so `.` means any character, and `-F` (`--fixed-strings`) matches plain text instead. With `-F`, those two were gone. `man grep` is the full manual I can scroll and search, and `grep --help` is the short summary. Tab helps the same way with paths: one Tab turned `cd ~/linux-cou` into `cd ~/linux-course/`.

## What tripped me up

- I thought `>>` was the one that overwrites. It's the other way around: `>` overwrites and `>>` appends.
- `grep ".conf"` matched `dconf` and `netconfig`, even though neither has ".conf" in its name.
- I thought I could move around a man page with nano commands. `man` opens a viewer, not an editor, so it's Space and `b` to move between pages, `/` to search, `n` for the next match, and `q` to quit.
- I thought Tab would fill in the first match alphabetically when there's more than one. It doesn't, and my two terminals didn't even behave the same. In Warp, the list showed up on the first Tab. In the plain Ubuntu terminal, the first Tab only rang the bell and the second one showed the list. Warp has its own completion menu, so that difference comes from the terminal, not bash (the terminal vs. shell split from M1).
- I thought an argument was any value passed to a command or flag. An argument is what the command acts on, like `~/linux-course/m2` in `ls -a ~/linux-course/m2`. A value like `auto` in `--color=auto` belongs to the flag.

## Checkpoint evidence

**`practice.txt`, with `>` for line one and `>>` for line two:**
```text
$ mkdir -p ~/linux-course/m2
$ cd ~/linux-course/m2
$ echo "line one" > practice.txt
$ echo "line two" >> practice.txt
$ cat practice.txt
line one
line two
```

**The redirected file listing, `conf-list.txt`:**
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

## What I'd do differently

I'd check `grep --help` before running the pipeline instead of after. I would have known `.` is a wildcard and added `-F`, so `conf-list.txt` would only have names that really contain ".conf". I'd also write down which terminal I'm in from the start, since Warp and plain bash don't handle Tab the same way.