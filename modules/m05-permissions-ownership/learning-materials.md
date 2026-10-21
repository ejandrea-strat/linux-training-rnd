# M5 Learning Materials — Users, Permissions, Ownership

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals. No prior exposure to accounts,
> groups, or permission bits is assumed, and every command is explained before it is used. You
> need the Linux environment set up in M0 (WSL2, a VM, or Linux-in-a-container), a terminal, and
> an account that can reach `sudo`.
>
> **Scope note.** Every example output below is **illustrative** unless it is explicitly labeled as
> coming from the author's observation notes
> ([`knowledge/observations/m05-permissions-ownership.md`](../../knowledge/observations/m05-permissions-ownership.md)).
> The labeled outputs were captured on one WSL2 Ubuntu machine; values such as usernames, UIDs,
> GIDs, dates, and file sizes will differ on yours. Never copy an example as evidence — run the
> command and paste your own output.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **115 minutes** — 15 + 20 + 15 + 15
> for the four concept sections, 35 for the worked example, and 5 each for the checklist, the
> best practices, and the references. M5 shares **Week 3** of the course with M6 (package
> management), and the course architecture treats M5 as a dependency of **both M6 and M13**
> (security and secrets hygiene) — which is why permissions come back later in the course.

## Contents

1. [Users and groups](#1-users-and-groups)
2. [Permission bits](#2-permission-bits)
3. [Ownership vs. permission](#3-ownership-vs-permission)
4. [Elevated privilege](#4-elevated-privilege)
5. [Worked example: the M5 lab](#5-worked-example-the-m5-lab)
6. [Completion checklist](#6-completion-checklist)
7. [Best practices and troubleshooting](#7-best-practices-and-troubleshooting)
8. [References](#8-references)

---

## 1. Users and groups

_Estimated: 15 minutes._

**Understand.** Every file and every process belongs to a user and a group. When something asks
for access, Linux checks that identity — not "who is logged in", not "which script asked" —
against the file's owner and group. Four terms carry the whole model:

| Term | What it is |
| --- | --- |
| **UID** | The numeric user ID. Root is 0. It is the identity the kernel checks. |
| **GID** | The numeric group ID. |
| **Primary group** | The one group named by the GID field in `/etc/passwd`. |
| **Supplementary groups** | Extra groups the account belongs to, listed in `/etc/group`. |

Three commands answer "who am I?": `whoami` prints the username, `id` prints the UID, the primary
GID, and every group with both numbers and names, and `groups` prints just the group names.

The accounts themselves live in three files:

| File | One line per entry | Readable by |
| --- | --- | --- |
| `/etc/passwd` | `name:x:UID:GID:GECOS:home:shell` | everyone (`rw-r--r--`) |
| `/etc/group` | `name:x:GID:members` | everyone (`rw-r--r--`) |
| `/etc/shadow` | password hash plus aging fields | root and the `shadow` group only (`rw-r-----`) |

The `x` in `/etc/passwd` is the point: the hash is not there. A user's **primary** group comes
from the GID in `/etc/passwd`, so that account does not have to appear in the member list of its
own group. `getent passwd <name>` and `getent group <name>` look an entry up the way the system
does, which also covers accounts that do not come from these files at all.

> **Use your own values.** The examples below carry the author's account name `ejandrea`, UID
> 1000, and GID 1000. Read your own `id` output instead.

**Try.**

```bash
id
groups
umask
ls -ld ~
```

Output from the author's observation notes:

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

Reading that: the UID is 1000 and the primary group is the account's own, GID 1000. `id` and
`groups` list the same seven groups, with `id` adding the numbers. `sudo` is number 27 among
them — that membership is what allows the account to use `sudo`. The `umask` of `0022` is what
keeps new files and folders from being writable by the group or by everyone else
([§2](#2-permission-bits)). The home folder is `drwxr-x---`.

Now look at where the accounts and groups are defined:

```bash
grep "^$USER:" /etc/passwd
grep "$USER" /etc/group
ls -l /etc/passwd /etc/group /etc/shadow
cat /etc/shadow
```

Output from the author's observation notes:

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

Reading that: the `/etc/passwd` line carries the UID and GID, an empty comment field, the home
folder, and the shell, with `x` where the password would be. The account appears in six group
member lists, matching the extra groups `id` reported. The last line, `ejandrea:x:1000:`, is the
account's own group with an empty member list — it matched the grep only because the group
carries the same name. The `ls -l` output shows the two readable files as `rw-r--r--` and
`/etc/shadow` as `rw-r-----` owned by `root:shadow`, which is why the plain `cat` was denied.

**Verify.** `id` prints a UID, a primary GID, and a list of groups; `groups` prints the same
names without the numbers. Your own `/etc/passwd` line shows `x` in the password field, and
`cat /etc/shadow` returns `Permission denied`.

**Continue** to [§2](#2-permission-bits) to read the bits those identities are checked against.

---

## 2. Permission bits

_Estimated: 20 minutes._

**Understand.** `ls -l` prints a fixed set of fields. Read them left to right:

| Field | Example | What it is |
| --- | --- | --- |
| Type | `-` | `-` regular file, `d` directory, `l` symbolic link |
| Owner bits | `rw-` | read, write, execute for the **owner** |
| Group bits | `r--` | read, write, execute for the **group** |
| Other bits | `r--` | read, write, execute for **everyone else** |
| Link count | `1` | how many hard links point at the file |
| Owner | `ejandrea` | user name, or the UID if there is no name |
| Group | `ejandrea` | group name, or the GID if there is no name |
| Size | `0` | bytes |
| Time | `Oct 21 21:54` | when the contents were last modified (`ls -lc` shows the inode change time instead) |
| Name | `shared-report.txt` | the name as you typed it |

The nine characters after the type are **three separate three-character sets** — owner, group,
other — in that order, each in the order read, write, execute. A letter means the permission is
on; a `-` means it is off. Only one set ever applies to you: if you own the file, the owner bits
apply and the other two are ignored.

**Octal** is the same information as digits. Each set of three becomes one digit, made by adding
read (4), write (2), and execute (1):

| Symbolic | Value | Octal digit |
| --- | --- | --- |
| `rwx` | 4 + 2 + 1 | 7 |
| `rw-` | 4 + 2 | 6 |
| `r-x` | 4 + 1 | 5 |
| `r--` | 4 | 4 |
| `---` | 0 | 0 |

So `640` reads `rw-r-----`. An octal mode sets **all nine bits at
once**.

**Symbolic** modes name who and what they change: `u` for the owner, `g` for the group, `o` for
others, `a` for all, then an operator — `+` adds, `-` removes, `=` sets exactly — then the bits.
`chmod u+x run.sh` adds execute for the owner and leaves every other bit as it was. That is the
whole difference between the two notations.

**New files and folders** start from a fixed base and lose whatever the **umask** takes away: a
new file starts from `rw-rw-rw-` and a new folder from `rwxrwxrwx`. With the `0022` umask from
[§1](#1-users-and-groups), a new file lands on 644 (`rw-r--r--`) and a new folder on 755
(`rwxr-xr-x`).

> **Use your own values.** The examples below carry the author's account name, group name, and
> dates. Read the shape of the output, then run it yourself and paste your own.

**Try.**

```bash
mkdir -p ~/linux-course/m5
cd ~/linux-course/m5
touch shared-report.txt
ls -l shared-report.txt
```

Output from the author's observation notes:

```text
$ ls -l shared-report.txt
-rw-r--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 shared-report.txt
```

Reading it left to right: `-` is a regular file; `rw-` is the owner (the author), `r--` is the
group, `r--` is everyone else; one hard link; owner `ejandrea`; group `ejandrea`; 0 bytes,
because `touch` made an empty file; the time it was last modified; the name. The three sets are
6, 4, and 4, so the mode is 644 — the `rw-rw-rw-` a new file starts from, minus the group's and
everyone else's write, which the `0022` umask takes away.

Now a directory and a file inside it, which is the shape this module builds toward. The next
block is a preview of the shared folder from [§5](#5-worked-example-the-m5-lab), Step 6 — that
folder does not exist on your machine yet, and you will create it there.

Output from the author's observation notes:

```text
$ ls -ld /srv/m5-reports
drwxr-x--- 2 ejandrea m5-analysts 4096 Oct 21 21:55 /srv/m5-reports
$ ls -l /srv/m5-reports
total 4
-rw-r----- 1 ejandrea m5-analysts 42 Oct 21 21:55 q3-summary.txt
```

The report is `-rw-r-----`: 6, 4, and 0 — so 640. The folder's mode is yours to decode in
[§5](#5-worked-example-the-m5-lab), Step 5. `ls -ld` is used for the directory because `ls -l` on a
directory lists its contents instead of the directory itself.

**Verify.** Your `ls -l` output splits cleanly into a type character, three three-character
sets, and then owner, group, size, time, and name. You can state the octal value of a permission
string you did not write yourself, and you know why a brand-new file is 644 rather than 666.

**Continue** to [§3](#3-ownership-vs-permission) to separate the bits from the identity they
belong to.

---

## 3. Ownership vs. permission

_Estimated: 15 minutes._

**Understand.** Two separate settings hang off every file, and they are changed by different
commands:

| Command | Changes | Leaves alone |
| --- | --- | --- |
| `chown user:group file` | the owner, and the group too when you add `:group` | the permission bits |
| `chgrp group file` | the group only | the owner and the bits |
| `chmod mode file` | the permission bits only | the owner and the group |

Changing the owner leaves the bits alone, and changing the bits leaves the owner alone. Two
limits on who may do what: only root can give a file to a different owner, and a file's owner
can change its group **only to a group the owner belongs to** — while the owner can always
change the file's own bits.

On a **directory**, the same three letters mean different things:

| Bit | On a file | On a directory |
| --- | --- | --- |
| `r` | read the contents | list the names in it |
| `w` | change the contents | create, delete, and rename entries in it |
| `x` | run it | **search**: go into it and reach what's inside by name |

Without execute on a folder you cannot `cd` into it or open anything inside it, even a file
whose own bits would allow it. With read but no execute you can see the names but not their
details. And whether you can delete or rename a file depends on the **folder's** write and
execute bits, not on the file's own bits — except in folders with the sticky bit, like `/tmp`.

**Try.** Watch the directory execute bit do its own job:

```bash
mkdir -p xtest
touch xtest/a.txt
chmod 644 xtest
ls -ld xtest
ls xtest
cd xtest
chmod 755 xtest
```

Output from the author's observation notes:

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

Reading it: after `chmod 644` the folder is `drw-r--r--` — `d` for a directory, and **no `x` for
anyone, the owner included**. `ls xtest` still listed `a.txt`, because read on a folder lets you
list the names in it. But `cd xtest` was denied even though the author owns the folder, because
going in needs execute. The same bit means different things on the two objects: on `run.sh`, `x`
means the file can be run; on `xtest`, it decides whether the owner can go in at all. The final
`chmod 755` gave no error, so execute is back.

Now ownership and permissions as separate commands on the shared folder. The next block is a
preview of that work from [§5](#5-worked-example-the-m5-lab), Step 6 — the folder does not exist
on your machine yet.

Output from the author's observation notes:

```text
$ sudo chown "$USER":m5-analysts /srv/m5-reports
$ sudo chmod 750 /srv/m5-reports
$ echo "Q3 orders summary: draft for the analysts" > /srv/m5-reports/q3-summary.txt
$ sudo chgrp m5-analysts /srv/m5-reports/q3-summary.txt
$ chmod 640 /srv/m5-reports/q3-summary.txt
```

Three different settings, set by three different commands: `chown` set the folder's owner and
group, `chmod` set its bits, and `chgrp` set the report's group. The last `chmod` needed no
`sudo` at all, because the author owns the report and an owner may always change their own
file's bits. The `chgrp` did need it, for the reason above: the author is not in `m5-analysts`,
and an owner can only hand a file to a group they belong to.

**Verify.** You can name, for any file, which command changes its owner and which changes its
bits, and say why one of them needed `sudo` and the other did not. You can explain why the owner
of `xtest` could not `cd` into it.

**Continue** to [§4](#4-elevated-privilege) to look at the one tool that bypasses all of this.

---

## 4. Elevated privilege

_Estimated: 15 minutes._

**Understand.** `sudo` runs **one** command as another user. `sudo command` runs it as root;
`sudo -u <user> command` runs it as that user instead — which is how you test a permission
scheme from the outside. Before running anything, `sudo` consults the **sudoers policy** to see
whether your account is allowed (on Ubuntu, members of the `sudo` group are), and then asks for
**your own** password, not root's.

After that, `sudo` does not ask again for a while in the same session; the policy sets how long.
`sudo -k` makes it forget, so the next `sudo` asks again. `sudo -v` asks now if it needs to, so
the commands after it do not stop for the password. `sudo` also logs each command it runs — in
M4, `sudo apt update` showed up in `/var/log/auth.log`.

Root is not limited by read and write bits. That is why `sudo` makes a "Permission denied" go
away, and also why it is the wrong fix for one: the wrong owner or the wrong mode is still there
afterwards.

Elevated privilege is warranted when the task itself changes or reads something the system
protects: installing packages, editing files in `/etc`, adding users and groups, reading
`/etc/shadow`, or creating a folder outside your home, like `/srv/m5-reports`. Use `sudo` for
the one command that needs it, not `sudo su`, which leaves a whole root shell open — the man7
`sudo(8)` page is explicit that in a `sudo su` shell the later commands are not subject to the
sudoers policy at all.

> **One caveat about that manual page.** The `sudo(8)` page on man7 documents the original
> `sudo`. The author's notes show Ubuntu 26.04 running **sudo-rs**, a rewrite in Rust that reads
> the same `/etc/sudoers`, so the policy wording above still applies — but your prompt text and
> your version string will differ from the classic `sudo`.

**Try.** Watch what `sudo` actually asks for, with the hash redacted before it reaches the
screen:

```bash
sudo -k
sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
```

Output from the author's observation notes:

```text
$ sudo -k
$ sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
[sudo: authenticate] Password:
root:*:20692:0:99999:7:::
ejandrea:[redacted]:20738:0:99999:7:::
```

Reading it: `sudo -k` made `sudo` forget the cached password, so it asked with
`[sudo: authenticate] Password:`, and the author typed their own. The prompt does not name an
account — but on this machine root's line shows it could not have been a root password: root's
field holds `*` where a hash would be, so no root password works here. With `sudo`, `cat` ran
as root, which can read `/etc/shadow`, while the same `cat` without `sudo` was denied earlier.
The `sed` replaced the hash with `[redacted]` before it reached the screen, and `grep` kept only
root's line and the author's. The `20738` after the password field is the day the password was
last changed, counted from Jan 1, 1970 — Oct 12, 2026.

Keep that redaction in your own pipeline. **Never paste a real hash into your notes.**

Now run a command as another identity. The next block is a preview of the denied attempts from
the shared folder in [§5](#5-worked-example-the-m5-lab), Step 6 — the test accounts and the
folder do not exist on your machine yet.

Output from the author's observation notes:

```text
$ sudo -u m5-analyst touch /srv/m5-reports/new-file.txt
touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied
$ sudo -u m5-outsider ls /srv/m5-reports
ls: cannot open directory '/srv/m5-reports': Permission denied
$ sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt
cat: /srv/m5-reports/q3-summary.txt: Permission denied
```

All three failures are the design working, not a mistake. The analyst is in the folder's group,
whose bits are `r-x`, and creating a file needs **write** on the directory. The outsider is
neither owner nor in the group, so it gets the other bits, `---`. And the folder is what stops
the outsider's `cat`: without execute on `/srv/m5-reports`, nothing inside it can be reached by
name — and the report's own `---` for everyone else would have stopped that read anyway.

**Verify.** `sudo -k` makes the next `sudo` prompt again; the prompt asks for your own password;
and `sudo -u <user> …` runs the command as that user instead of as root, so it is denied exactly
what that identity is denied.

**Continue** to [§5](#5-worked-example-the-m5-lab) to run the whole lab and capture the evidence.

---

## 5. Worked example: the M5 lab

> **Purpose.** The lab steps from [`brief.md`](./brief.md), plus the module's deliverable, with
> the exact evidence to capture. The outputs shown are the author's; yours will differ in
> usernames, UIDs, GIDs, dates, and sizes. Record your own.

_Estimated: 35 minutes._

### Step 1 — See the default permissions

**Understand.** The lab starts by observing what you already have, before changing anything.

**Try.**

```bash
mkdir -p ~/linux-course/m5
cd ~/linux-course/m5
touch shared-report.txt
ls -l shared-report.txt
```

Output from the author's observation notes:

```text
$ mkdir -p ~/linux-course/m5
$ cd ~/linux-course/m5
$ touch shared-report.txt
$ ls -l shared-report.txt
-rw-r--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 shared-report.txt
```

**Capture.** Paste the `ls -l` line verbatim, with the `$ ls -l shared-report.txt` prompt next to
it, and write in your own words what the owner, group, and other positions each say. That
reading is the point of the step, not the paste.

### Step 2 — Restrict the file to its owner

**Understand.** One octal mode, nine bits set at once.

**Try.**

```bash
chmod 600 shared-report.txt
ls -l shared-report.txt
```

Output from the author's observation notes:

```text
$ chmod 600 shared-report.txt
$ ls -l shared-report.txt
-rw------- 1 ejandrea ejandrea 0 Oct 21 21:54 shared-report.txt
```

**Capture.** The `ls -l` output **before and after** the change. The owner, group, size, and date
stayed the same — only the bits moved — and that is worth a sentence in your notes.

### Step 3 — Make a script executable by its owner only

**Understand.** A symbolic mode that names one thing and leaves the rest alone.

**Try.**

```bash
touch run.sh
ls -l run.sh
chmod u+x run.sh
ls -l run.sh
```

Output from the author's observation notes:

```text
$ touch run.sh
$ ls -l run.sh
-rw-r--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 run.sh
$ chmod u+x run.sh
$ ls -l run.sh
-rwxr--r-- 1 ejandrea ejandrea 0 Oct 21 21:54 run.sh
```

**Capture.** Both `ls -l run.sh` lines, and a sentence saying what `u+x` changed and what it
left alone: the owner's `rw` stayed, the group and everyone else kept their `r--`, and only one
`x` was added. The result is 744 — the same as `chmod 744` would give, but `chmod 744` sets all
nine bits at once.

### Step 4 — Watch what `sudo` asks for

**Understand.** The prompt, the password it wants, and a redacted result.

**Try.**

```bash
sudo -k
sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
```

Output from the author's observation notes:

```text
$ sudo -k
$ sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
[sudo: authenticate] Password:
root:*:20692:0:99999:7:::
ejandrea:[redacted]:20738:0:99999:7:::
```

**Capture.** The prompt line and the redacted output. Confirm in your notes that the password
asked for was **your own**: it is the password you typed that let the command run, and a wrong
password would have failed the prompt. On the author's machine the output also showed root's
password field as `*`, which is how that machine could tell no root password was in play —
yours may show `!` or a hash instead, so judge by which password you typed. Keep the redaction
in the pipeline — **never paste a hash**.

### Step 5 — Say what `750` means, before you check it

**Understand.** `750` is three sets of three bits — one for the owner, one for the group, one for
everyone else. The octal table in [§2](#2-permission-bits) turns each digit into a permission
string. Do not decode them yet: write your own explanation first, then check it.

**Try.** In your own words, in your notes, **before reading any further**: state what the owner
can do, what the group can do, and what everyone else can do under `750`. Write the mode as the
`ls -l` permission string too.

**Capture.** Your own-words explanation, in your notes, written first and checked second.

**Verify.** Three checks, in this order:

1. Check your explanation against the octal table in [§2](#2-permission-bits) — 7, 5, 0 and the
   `rwx` / `r-x` / `---` they add up to.
2. The author's own-words answer, for comparison once you have written yours:

   > With 750, the owner can read, write, and execute, the group can read and execute but not
   > write, and everyone else can do nothing.
   >
   > Each digit is one set of bits, in the order owner, group, everyone else, and it adds up read
   > (4), write (2), and execute (1): 7 is 4+2+1, or `rwx`; 5 is 4+1, or `r-x`; and 0 is `---`.
   > So `ls -l` shows 750 as `rwxr-x---`. On a folder like `/srv/m5-reports`, the owner can list
   > it, go into it, and create or delete files in it. The group can list it and go into it, and
   > open the files inside as far as each file's own bits allow, but can't create, rename, or
   > delete anything there. Everyone else can't list it or go into it, so nothing inside is
   > reachable for them.

3. After Step 6, check your explanation a second time against **your own** `ls -ld` output for the
   folder you built, and say whether it matches. The author's notes record the same second check:
   `ls -ld` showed `/srv/m5-reports` as `drwxr-x---`; as the owner the author could create the
   report in it; `m5-analyst`, in the group, could list it and read the report but not create a
   file; and `m5-outsider`, as everyone else, could not list it or read anything in it.

### Step 6 — Build the shared directory and prove it

**Understand.** This is the module's deliverable: a directory with a permission and ownership
scheme **you** designed, where access is deliberately restricted for one identity and open for
another, with both outcomes demonstrated and not just configured.

**Try.** Before any command, settle the design — the steps from the module's task list:

- Decide what your shared directory is for, and which identity gets access and which doesn't.
- Confirm with your trainer what counts as a real test of the restriction, if you are unsure.
- Make sure at least one additional user or group is available to test with.

The author's scheme: a Q3 report folder for a group of analysts, with a second account kept
outside that group to prove the restriction. Now build it.

```bash
cd /
sudo groupadd m5-analysts
sudo useradd -m -s /bin/bash -G m5-analysts m5-analyst
sudo useradd -m -s /bin/bash m5-outsider
id m5-analyst
id m5-outsider
getent group m5-analysts
sudo mkdir -p /srv/m5-reports
sudo chown "$USER":m5-analysts /srv/m5-reports
chmod 750 /srv/m5-reports
echo "Q3 orders summary: draft for the analysts" > /srv/m5-reports/q3-summary.txt
sudo chgrp m5-analysts /srv/m5-reports/q3-summary.txt
chmod 640 /srv/m5-reports/q3-summary.txt
ls -ld /srv/m5-reports
ls -l /srv/m5-reports
```

The author's run used `sudo chmod 750`, which didn't need `sudo`: `chown` had already made the
folder theirs ([§7](#7-best-practices-and-troubleshooting)).

Output from the author's observation notes:

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

Reading it: the author owns the folder and its group is `m5-analysts`; `m5-analyst` has that
group as an extra group and `m5-outsider` does not; `getent` lists `m5-analyst` as the group's
only member. The folder is `drwxr-x---` (750) and the report is `-rw-r-----` (640). The analyst
account got UID 1001 but GID 1002, because `groupadd` had already given 1001 to `m5-analysts` —
a user's UID and GID do not have to match.

Now prove it, as each identity:

```bash
sudo -u m5-analyst ls -l /srv/m5-reports
sudo -u m5-analyst cat /srv/m5-reports/q3-summary.txt
sudo -u m5-analyst touch /srv/m5-reports/new-file.txt
sudo -u m5-outsider ls /srv/m5-reports
sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt
```

Output from the author's observation notes:

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

**Capture.** All five commands and their outcomes. A configuration is not a proof: your notes
need one identity that **succeeds** and one identity that **fails**, with the error messages
pasted.

Check your Step 5 explanation against your own `ls -ld` output now
(`drwxr-x---` is 750 — owner `rwx`, group `r-x`, others `---`), and say whether it holds.

Note the least-permissive rule at work: 750 on the folder and 640 on the report are the least
the scheme needs, since the analysts only have to list and read. No step used `777`.

**Clean-up.** Nothing to clean up. The practice files stay under `~/linux-course/`. Step 6's test
accounts (`m5-analyst`, `m5-outsider`), the `m5-analysts` group, and `/srv/m5-reports` sit
outside `~/linux-course/`, so they stay until your trainer has reviewed the module.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] **DoD-01 — The permission change, shown.** The trainee's pasted `ls -l shared-report.txt`
      output before and after `chmod 600` shows the change, and the notes read the permission
      string for the owner, group, and other positions (Lab steps 1–2). *Verify via
      [§5](#5-worked-example-the-m5-lab), Steps 1–2.*
- [ ] **DoD-02 — The execute bit, shown.** The pasted `ls -l run.sh` output shows the execute bit
      set for the owner only, and the notes state what `u+x` changed and what it left alone (Lab
      step 3). *Verify via [§5](#5-worked-example-the-m5-lab), Step 3.*
- [ ] **DoD-03 — The `sudo` prompt, shown.** The notes show that `sudo cat /etc/shadow` prompted
      for the trainee's own password rather than a root password, with the password hashes
      redacted (Lab step 4). *Verify via [§5](#5-worked-example-the-m5-lab), Step 4.*
- [ ] **DoD-04 — The `750` explanation, written first.** The trainee's own-words explanation of
      `750` was written before it was checked against the Concepts section, and states what owner,
      group, and other can each do (Lab step 5). *Verify via
      [§5](#5-worked-example-the-m5-lab), Step 5.*
- [ ] **DoD-05 — No blanket `777`.** No step in the notes uses a blanket `777` "open everything"
      shortcut; every mode set is the least permissive one that still works. *Verify via
      [§5](#5-worked-example-the-m5-lab), Step 6 and [§7](#7-best-practices-and-troubleshooting).*

Where DoD-04 says "the Concepts section", the reference is the permission-bits material in
[§2](#2-permission-bits) and the checks in [§5](#5-worked-example-the-m5-lab), Step 5.

When all five are ticked, transfer the evidence into your write-up
([`write-up-template.md`](./write-up-template.md)) and continue to M6.

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Use `sudo <command>` for the one command that needs it, not `sudo su`.** A `sudo su` shell
  is outside the sudoers policy, so every later command in it runs unchecked. Watch for the
  `sudo` that was never needed: in the shared-folder build the author ran `sudo chmod 750` on a
  folder that `chown` had already made theirs, so a plain `chmod 750` would have done it
  ([§3](#3-ownership-vs-permission), [§5](#5-worked-example-the-m5-lab)).
- **Default to the least permissive setting that still works.** 750 on the folder and 640 on the
  report were the least the scheme needed, because the analysts only have to list and read
  ([§5](#5-worked-example-the-m5-lab)).
- **Never treat `777` as a quick fix for a permission error.** It removes the restriction
  instead of fixing the cause. The modes set in the author's notes were 600 and `u+x` on the
  practice files, 644 and then 755 on `xtest`, 750 on the shared folder, and 640 on the report
  ([§2](#2-permission-bits), [§5](#5-worked-example-the-m5-lab)).
- **Check both ownership (`chown`) and permission bits (`chmod`) when debugging "permission
  denied".** The outsider's `cat` of the report was blocked by the *folder's* missing execute
  bit, not by the report's own bits — and the analyst's denied `touch` was about the folder's
  write bit, not the file's ([§3](#3-ownership-vs-permission),
  [§4](#4-elevated-privilege)).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `cat: /etc/shadow: Permission denied` | `/etc/shadow` is `rw-r-----` owned by `root:shadow`, and a normal account is not in the `shadow` group. | Run the one read as root: `sudo cat /etc/shadow`, and keep the hash redacted in anything you paste. |
| `ls xtest` lists `a.txt`, but `cd xtest` returns `bash: cd: xtest: Permission denied` — even though you own the folder | The folder has read but no execute ("search") bit for anyone, the owner included. Read lists the names; execute is what lets you go in. | Give the folder execute as well. `chmod 755 xtest` restored it in the author's notes with no error. |
| `touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied` | The identity is in the folder's group, but the folder's group bits are `r-x`. Creating a file needs **write** on the *directory*, not on the file. | Either add write to the group's directory bits, or design the scheme so that identity only reads — as the author's does. |
| As the outsider, `ls` reports `cannot open directory` and `cat` reports `Permission denied` | The outsider is neither owner nor in the group, so it gets the "other" bits, `---`. The folder is what stops the `cat`: without execute on the folder, nothing inside can be reached by name. | Nothing to fix — that is the restriction working. When a read fails, check the *folder's* mode, not only the file's. |
| `sudo` does not ask for your password again | After the first prompt, `sudo` does not ask again for a while in the same session; the policy sets how long. | Run `sudo -k` to make `sudo` forget, so the next command asks again. `sudo -v` asks now if it needs to, so the commands after it do not stop for the password. |
| The prompt reads `[sudo: authenticate] Password:` and does not name an account | On Ubuntu 26.04 `sudo` is sudo-rs, a Rust rewrite that reads the same `/etc/sudoers`; the classic prompt text is not what you will see. | Type **your own** password — the one that lets the command run. On the author's machine root's entry in `/etc/shadow` held `*`, so no root password could have been typed there; yours may show `!` or a hash, so judge by which password you typed. |
| `id m5-analyst` shows `uid=1001` but `gid=1002`, which looks mismatched | `groupadd` had already taken GID 1001 for `m5-analysts` before the account was created, so the account's own group got the next free GID. | Nothing to fix. A user's UID and GID are independent numbers and do not have to match. |

---

## 8. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man id` — print real and effective user and group IDs.
- `man 5 passwd` — the `/etc/passwd` file format and every field.
- `man 5 group` — the `/etc/group` file format and the member list.
- `man 5 shadow` — the `/etc/shadow` fields, including the last-changed day count.
- `man chmod` — symbolic and octal modes, and the special bits.
- `man chown` — change file owner and group; the `OWNER[:GROUP]` syntax.
- `man chgrp` — change group ownership.
- `man ls` — the long listing and every column it prints.
- `man sudo` — the security policy and credential caching. On Ubuntu 26.04 this opens the
  sudo-rs page, which has no SECURITY NOTES section; the point about `sudo su` comes from the
  `sudo(8)` page below.
- `man 8 useradd` — creating accounts, including `-m` and `-G`.
- `man 8 groupadd` — creating groups.
- `man 7 path_resolution` — how the kernel resolves a path, directory by directory.
- `help umask` — the mask that decides new files' and folders' permissions.

Online copies of the same pages (confirmed live):

- [`chmod(1)`](https://man7.org/linux/man-pages/man1/chmod.1.html)
- [`chown(1)`](https://man7.org/linux/man-pages/man1/chown.1.html)
- [`chgrp(1)`](https://man7.org/linux/man-pages/man1/chgrp.1.html)
- [`id(1)`](https://man7.org/linux/man-pages/man1/id.1.html)
- [`sudo(8)`](https://man7.org/linux/man-pages/man8/sudo.8.html) — documents the original `sudo`;
  Ubuntu 26.04 runs sudo-rs, which reads the same `/etc/sudoers` but prompts differently
- [`useradd(8)`](https://man7.org/linux/man-pages/man8/useradd.8.html)
- [`groupadd(8)`](https://man7.org/linux/man-pages/man8/groupadd.8.html)
- [`passwd(5)`](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [`group(5)`](https://man7.org/linux/man-pages/man5/group.5.html)
- [`shadow(5)`](https://man7.org/linux/man-pages/man5/shadow.5.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module, including the shared-directory
  design steps.
- [`write-up-template.md`](./write-up-template.md) — where the lab evidence goes.
- [`knowledge/observations/m05-permissions-ownership.md`](../../knowledge/observations/m05-permissions-ownership.md)
  — the author's captured output, used as the labeled examples in this guide.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) —
  where M5 sits in the curriculum (Access & Software layer, Week 3, with M6).
- Curriculum resource for this stage: [Linux File Permissions Explained](https://www.redhat.com/en/blog/linux-file-permissions-explained)
  (Red Hat), and *User and File System Privileges* (listed for the Access & Software layer in the
  architecture doc).
