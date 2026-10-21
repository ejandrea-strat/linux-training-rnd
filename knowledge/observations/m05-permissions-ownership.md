# M5 Observation Notes

## Expectations before investigating

1. **How do you see which user and groups the current account has?**
   I think `whoami` prints my username, and `id` or `groups` lists the groups I'm in. I expect `sudo` to be one of them, since that's what lets me use `sudo`.

2. **Where are accounts and groups defined?**
   I think accounts are listed in `/etc/passwd` and groups in `/etc/group`, and the password hashes are kept apart in `/etc/shadow`, which a normal user can't read.

3. **How do you read the output of `ls -l`?**
   I think the first character says whether it's a file or a folder, and the next nine are three sets of `rwx` for the owner, the group, and everyone else, with `-` where a permission is off. Then come the owner, the group, the size, the date, and the name.

4. **How do the symbolic and octal notations map to each other?**
   I think each octal digit is one of those three sets, with read, write, and execute counting as 4, 2, and 1, so `6` would be `rw-`. Symbolic modes like `u+x` change only the part they name.

5. **What do `chown` and `chmod` each change?**
   I think `chown` changes who owns a file and `chmod` changes its permissions, and changing one leaves the other as it was.

6. **Why does a directory's execute bit behave differently from a file's?**
   I think execute on a folder means I can go into it, and read means I can list it. So a folder with read but no execute might show its file names but not let me `cd` into it.

7. **What does `sudo` actually do?**
   I think `sudo` runs one command as root, asks for my own password rather than a root password, and doesn't ask again for a few minutes after that.

8. **When is elevated privilege genuinely warranted?**
   I think it's for changes to the system itself, like installing packages, editing files in `/etc`, or adding users, and not for getting past a "Permission denied" on my own files.

## What 750 means (Lab step 5)

With 750, the owner can read, write, and execute, the group can read and execute but not write, and everyone else can do nothing.

Each digit is one set of bits, in the order owner, group, everyone else, and it adds up read (4), write (2), and execute (1): 7 is 4+2+1, or `rwx`; 5 is 4+1, or `r-x`; and 0 is `---`. So `ls -l` shows 750 as `rwxr-x---`. On a folder like `/srv/m5-reports`, the owner can list it, go into it, and create or delete files in it. The group can list it and go into it, and open the files inside as far as each file's own bits allow, but can't create, rename, or delete anything there. Everyone else can't list it or go into it, so nothing inside is reachable for them.

**Checked after the lab:** It matches. `ls -ld` shows `/srv/m5-reports` as `drwxr-x---`. As the owner, I could create the report in it. `m5-analyst`, in the group, could list it and read the report but not create a file, and `m5-outsider`, as everyone else, couldn't list it or read anything in it.

## What I learned from research

1. **Users and groups:**
   - Every process runs as a user with a set of groups. When it asks for access to a file, Linux checks that identity against the file's owner and group.
   - `whoami` prints the current username. `id` prints the user ID (UID), the primary group (GID), and every group the account is in, as numbers and names. `groups` prints just the group names.
   - Accounts are defined in `/etc/passwd`, one line each: `name:x:UID:GID:comment:home:shell`. The `x` means the password hash is kept in `/etc/shadow` instead, which only root and the `shadow` group can read.
   - Groups are defined in `/etc/group`, one line each: `name:x:GID:members`. The members list has the users who are in the group as an extra group. A user's primary group comes from the GID in `/etc/passwd`, so the user doesn't need to be listed there.
   - `getent passwd <name>` and `getent group <name>` look an entry up the way the system does, which also covers accounts that don't come from these files. See `man id`, `man 5 passwd`, `man 5 group`, and `man 5 shadow`.

2. **Permission bits:**
   - In `ls -l`, the first character is the type: `-` for a file, `d` for a directory, `l` for a symbolic link. The next nine characters are three sets of three, for the owner, the group, and everyone else ("other"), each in the order read, write, execute. A letter means that permission is on, and `-` means it's off.
   - After the permissions come the number of links, the owner, the group, the size in bytes, the time its contents were last modified, and the name.
   - In octal, each set is one digit, made by adding read (4), write (2), and execute (1). So `rwx` is 7, `rw-` is 6, `r-x` is 5, `r--` is 4, and `---` is 0, and `chmod 640` sets `rw-r-----`.
   - Symbolic modes name who (`u` for the owner, `g` for the group, `o` for others, `a` for all), an operator (`+` adds, `-` removes, `=` sets exactly), and the bits. `chmod u+x run.sh` adds execute for the owner and leaves every other bit as it was. An octal mode sets all nine bits at once.
   - New files start from `rw-rw-rw-` and new folders from `rwxrwxrwx`, minus the bits in the `umask`. With a `umask` of `0022`, a new file gets `644` (`rw-r--r--`) and a new folder `755` (`rwxr-xr-x`). See `man chmod`, `man ls`, and `help umask`.

3. **Ownership vs. permission:**
   - `chown user:group file` changes the owner, and the group too when you add `:group`. `chgrp group file` changes only the group. `chmod` changes only the permission bits. Changing the owner leaves the bits alone, and changing the bits leaves the owner alone.
   - Only root can give a file to a different owner. A file's owner can change its group, but only to a group the owner is in, and can change its permission bits.
   - On a directory, the bits mean something else. Read lets you list the names in it. Write lets you create, delete, and rename entries in it. Execute, called "search" for directories, lets you go into it and reach what's inside by name.
   - Without execute on a folder, you can't `cd` into it or open anything inside, even a file whose own bits would allow it. With read but no execute, you can see the names but not their details.
   - Whether you can delete or rename a file depends on the folder's write and execute bits, not on the file's own bits, except in folders with the sticky bit, like `/tmp`. See `man chown`, `man chgrp`, `man chmod`, and `man 7 path_resolution`.

4. **Elevated privilege:**
   - `sudo command` runs one command as root, or as another user with `sudo -u <user> command`. It first checks the sudoers policy to see whether your account is allowed (on Ubuntu, members of the `sudo` group are), then asks for your own password, not root's.
   - After that, it doesn't ask again for a while in the same session; the policy sets how long. `sudo -k` makes it forget, so the next `sudo` asks again. `sudo -v` asks now if it needs to, so the commands after it don't stop for the password.
   - `sudo` logs each command it runs. In M4, `sudo apt update` showed up in `/var/log/auth.log`.
   - On Ubuntu 26.04, `sudo` is sudo-rs, a rewrite in Rust that reads the same `/etc/sudoers`. Its password prompt is `[sudo: authenticate] Password:`, as in M4.
   - Root isn't limited by read and write bits. That's why `sudo` makes a "Permission denied" go away, and also why it's the wrong fix for one: the wrong owner or mode is still there.
   - Elevated privilege is warranted when the task itself changes or reads something the system protects: installing packages, editing files in `/etc`, adding users and groups, reading `/etc/shadow`, or creating a folder outside your home, like `/srv/m5-reports`. Use `sudo` for the one command that needs it, not `sudo su`, which leaves a whole root shell open. See `man sudo`.

## Who I am on this system

```text
$ id
uid=1000(ejandrea) gid=1000(ejandrea) groups=1000(ejandrea),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users)
$ groups
ejandrea adm cdrom sudo dip plugdev users
$ umask
0022
$ ls -ld ~
drwxr-x--- 12 ejandrea ejandrea 4096 Oct 21 15:49 /home/ejandrea
```

**Observation:** Shows that I'm `ejandrea`, UID 1000, and my primary group is also `ejandrea`, GID 1000. `id` and `groups` list the same seven groups, and `id` gives their numbers too. One of them is `sudo` (27), which is what lets me use `sudo`. My `umask` is `0022`, so new files and folders don't get write permission for the group or everyone else. My home folder is `drwxr-x---`, which is 750, the same mode the shared folder gets later. Anyone who isn't me and isn't in the `ejandrea` group can't go into it, so the test users couldn't have reached a folder inside my home. That's why the shared folder is in `/srv`.

## Where accounts and groups are defined

```text
$ grep "^$USER:" /etc/passwd
ejandrea:x:1000:1000::/home/ejandrea:/bin/bash
$ grep "$USER" /etc/group
adm:x:4:syslog,ejandrea
cdrom:x:24:ejandrea
sudo:x:27:ejandrea
dip:x:30:ejandrea
plugdev:x:46:ejandrea
users:x:100:ejandrea
ejandrea:x:1000:
$ ls -l /etc/passwd /etc/group /etc/shadow
-rw-r--r-- 1 root root    784 Oct 12 13:09 /etc/group
-rw-r--r-- 1 root root   1384 Oct 12 13:09 /etc/passwd
-rw-r----- 1 root shadow  780 Oct 12 13:09 /etc/shadow
$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

**Observation:** Shows that my line in `/etc/passwd` has my UID and GID (1000), an empty comment field, my home folder, and my shell, with `x` where the password would be. In `/etc/group`, I'm listed as a member of six groups, the same ones `id` showed besides my own. The last line is my own group, `ejandrea`, with an empty member list: it's my primary group through the GID in `/etc/passwd`, so I don't need to be listed. `grep "$USER"` only matched it because the group has my name. Everyone can read `/etc/passwd` and `/etc/group` (`rw-r--r--`), but `/etc/shadow` is `rw-r-----`, owned by root and the `shadow` group, with nothing for everyone else. I'm not in `shadow`, so `cat /etc/shadow` was denied.

## Default permissions and chmod 600

```text
$ mkdir -p ~/linux-course/m5
$ cd ~/linux-course/m5
$ touch shared-report.txt
$ ls -l shared-report.txt
-rw-r--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 shared-report.txt
$ chmod 600 shared-report.txt
$ ls -l shared-report.txt
-rw------- 1 ejandrea ejandrea 0 Oct 21 21:54 shared-report.txt
```

**Observation:** Shows that the new file started as `-rw-r--r--`, which is 644: the first `-` because it's a regular file, then `rw-` for the owner (me), `r--` for the group (`ejandrea`), and `r--` for everyone else. That's the `rw-rw-rw-` a new file starts from, without the group's and everyone else's write, which my `umask` of `0022` takes away. After `chmod 600`, it's `-rw-------`: the owner still has `rw-`, and the group and everyone else have `---`, so only I can read or change it. The owner, group, size (0, since `touch` made it empty), and date stayed the same. Only the permissions changed.

## Making run.sh executable

```text
$ touch run.sh
$ ls -l run.sh
-rw-r--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 run.sh
$ chmod u+x run.sh
$ ls -l run.sh
-rwxr--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 run.sh
```

**Observation:** Shows that `run.sh` also started as `-rw-r--r--`. `chmod u+x` added `x` to the owner's set only, so it became `-rwxr--r--`, which is 744. The owner's `rw` and the group's and everyone else's `r--` stayed as they were, because a symbolic mode changes only the bits it names. `chmod 744` would end up the same here, but it sets all nine bits at once instead of adding one.

## A folder with read but no execute

```text
$ mkdir -p xtest
$ touch xtest/a.txt
$ chmod 644 xtest
$ ls -ld xtest
drw-r--r-- 2 ejandrea ejandrea 4096 Oct 21 21:54 xtest
$ ls xtest
a.txt
$ cd xtest
bash: cd: xtest: Permission denied
$ chmod 755 xtest
```

**Observation:** Shows that after `chmod 644`, `xtest` was `drw-r--r--`: `d` for a directory, and no `x` for anyone, me included. `ls xtest` still listed `a.txt`, because read on a folder lets you list the names in it. But `cd xtest` was denied even though I own the folder, because going into a folder needs execute. So the same bit means different things: on `run.sh`, `x` means it can be run, and on `xtest`, it decides whether I can go in at all. The last `chmod 755` gave no error, so `xtest` has execute back.

## sudo and /etc/shadow

```text
$ sudo -k
$ sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
[sudo: authenticate] Password:
root:*:20692:0:99999:7:::
ejandrea:[redacted]:20738:0:99999:7:::
```

**Observation:** Shows that after `sudo -k` made `sudo` forget my password, it asked for one with `[sudo: authenticate] Password:`, and I typed my own. The prompt doesn't name an account, but root's line shows it couldn't have been a root password: root has `*` where a hash would be, so no password works for root. With `sudo`, `cat` ran as root, which can read `/etc/shadow`, while the same `cat` without `sudo` was denied earlier. `sed` replaced my hash with `[redacted]` before it reached the screen, and `grep` kept only root's line and mine. The `20738` after my password field is the day my password was last changed, counted from Jan 1, 1970. That's Oct 12, 2026, the same date `ls -l` showed for `/etc/shadow` earlier.

## The shared folder

```text
$ cd /
$ sudo groupadd m5-analysts
$ sudo useradd -m -s /bin/bash -G m5-analysts m5-analyst
$ sudo useradd -m -s /bin/bash m5-outsider
$ id m5-analyst
uid=1001(m5-analyst) gid=1002(m5-analyst) groups=1002(m5-analyst),1001(m5-analysts)
$ id m5-outsider
uid=1002(m5-outsider) gid=1003(m5-outsider) groups=1003(m5-outsider)
$ getent group m5-analysts
m5-analysts:x:1001:m5-analyst
$ sudo mkdir -p /srv/m5-reports
$ sudo chown "$USER":m5-analysts /srv/m5-reports
$ sudo chmod 750 /srv/m5-reports
$ echo "Q3 orders summary: draft for the analysts" > /srv/m5-reports/q3-summary.txt
$ sudo chgrp m5-analysts /srv/m5-reports/q3-summary.txt
$ chmod 640 /srv/m5-reports/q3-summary.txt
$ ls -ld /srv/m5-reports
drwxr-x--- 2 ejandrea m5-analysts 4096 Oct 21 21:55 /srv/m5-reports
$ ls -l /srv/m5-reports
total 4
-rw-r----- 1 ejandrea m5-analysts 42 Oct 21 21:55 q3-summary.txt
```

**Observation:** Shows the scheme. `/srv/m5-reports` holds the Q3 report for the analysts: I own it, and its group is `m5-analysts`. `id` shows that `m5-analyst` has `m5-analysts` as an extra group and `m5-outsider` doesn't, and `getent` lists `m5-analyst` as the group's only member. The folder is `drwxr-x---`, 750: I can list it, go in, and add or delete files, the analysts can list it and go in, and everyone else gets nothing. The report is `-rw-r-----`, 640: I can read and change it, the analysts can only read it, and everyone else gets nothing. It's 42 bytes, the 41 characters of the line plus the newline `echo` adds. `m5-analyst` got UID 1001 but GID 1002, because `groupadd` had already given 1001 to `m5-analysts`, so a user's UID and GID don't have to match.

Ownership and permissions took separate commands: `chown` set the folder's owner and group, `chmod` set its bits, and `chgrp` set the report's group. `mkdir` needed `sudo` because `/srv` is a system folder, and `chown` needed it because only root can give a folder to a new owner. `chgrp` needed `sudo` even though I own the report, because I'm not in `m5-analysts`, and an owner can only give a file to a group they're in. `chmod 640` didn't need it, because a file's owner can change its bits. `sudo chmod 750` didn't need it either: after `chown`, the folder was mine, so a plain `chmod 750` would have worked. None of the `sudo` lines asked for my password, because `sudo` still had it from the step before.

## Proving it: one allowed, one denied

```text
$ sudo -u m5-analyst ls -l /srv/m5-reports
total 4
-rw-r----- 1 ejandrea m5-analysts 42 Oct 21 21:55 q3-summary.txt
$ sudo -u m5-analyst cat /srv/m5-reports/q3-summary.txt
Q3 orders summary: draft for the analysts
$ sudo -u m5-analyst touch /srv/m5-reports/new-file.txt
touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied
$ sudo -u m5-outsider ls /srv/m5-reports
ls: cannot open directory '/srv/m5-reports': Permission denied
$ sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt
cat: /srv/m5-reports/q3-summary.txt: Permission denied
```

**Observation:** Shows both outcomes, with `sudo -u` running each command as the test user instead of root. `m5-analyst` is in the folder's group, so it got the group's `r-x`: it could list the folder, and with the report's group `r--`, read the report. Its `touch` was denied, because creating a file needs write on the folder, and the group has none. `m5-outsider` isn't the owner or in the group, so it got everyone else's `---`: `ls` couldn't open the folder, and `cat` couldn't reach the report. The folder is what stopped that `cat`: without execute on `/srv/m5-reports`, nothing inside can be reached, and the report's own `---` for everyone else would have stopped it too.

No step used `777`. The modes set were 600 and `u+x` on the practice files, 644 and then 755 on `xtest`, 750 on the shared folder, and 640 on the report. 750 and 640 are the least the scheme needs, since the analysts only have to list and read.