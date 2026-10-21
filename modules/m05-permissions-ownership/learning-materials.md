# M5 Learning Materials — Users, Permissions, Ownership

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own `write-up.md`, made from
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, working in the course's Linux
> environment (WSL2, a virtual machine, or Linux in a container).
>
> **Scope note.** Every command output in this guide is **illustrative** — it shows the shape of
> what a command prints, not a record of a completed lab. Values such as your username, home path,
> dates, and file counts differ on your machine. Run each command and paste your own output before
> you use any example as evidence. No password hash appears anywhere in this guide.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **115 minutes** — 15 + 20 + 15 + 15
> for the four concept sections, 35 for the worked example, and 5 each for the checklist, the best
> practices, and the references. M5 shares **Week 3** of the course with M6, in **Layer 3 (Access &
> Software)**. Commands assume **bash** on **WSL2 Ubuntu 26.04**, the course environment; on a
> Virtual Machine or in a container the same commands report your own environment's values.
>
> **Prerequisites.** M4 (viewing and editing files): you need `ls -l`, `cat`, and redirection to read
> what you create here. M2–M3 (command syntax and filesystem navigation): you need to move around a
> directory and create files in it.

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

**Understand.** Every file has an owner (a user) and a group. Every process runs as a user with a
set of groups. When a process asks to open a file, Linux compares that identity with the file's
owner and group and picks exactly one of three permission sets. `man 7 path_resolution` states the
rule:

> The first group of three is used when the effective user ID of the calling process equals the owner
> ID of the file. The second group of three is used when the group ID of the file either equals the
> effective group ID of the calling process, or is one of the supplementary group IDs of the calling
> process (as set by setgroups(2)). When neither holds, the third group is used.

The three sets are mutually exclusive. You never collect the owner's rights *and* the group's
rights: the first match wins, and "other" is what everybody else gets. That is why a file can be
wide open to its group and still closed to the rest of the system, and why a file can be closed to
its group and still open to everybody else.

Linux keeps the identities in three text databases:

| File | One line per | Line format | Who can read it |
| --- | --- | --- | --- |
| `/etc/passwd` | account | `name:x:UID:GID:comment:home:shell` | everyone |
| `/etc/group` | group | `name:x:GID:member_list` | everyone |
| `/etc/shadow` | account, password data | `name:hash:last_change:min:max:warn:inactive:expire:reserved` | root and the `shadow` group |

The `x` in a `/etc/passwd` line is not a password. It is a placeholder saying the real hash lives in
`/etc/shadow`, so the account list can stay world-readable while the hashes stay locked down.

Your **primary group** is the GID field in your `/etc/passwd` line, which is why you are not listed
as a member of it in `/etc/group`. Every other group you belong to appears in a member list there.
`id` and `groups` show both kinds: the primary group first, then the supplementary ones.

`getent passwd <name>` and `getent group <name>` query the same databases the system itself uses, so
they also find accounts that come from somewhere other than these two files.

**Try.** Everything here already exists on your system. Nothing needs to be created first.

```bash
whoami
id
groups
umask
ls -ld /
grep "^$USER:" /etc/passwd
grep "$USER" /etc/group
ls -l /etc/passwd /etc/group /etc/shadow
cat /etc/shadow
```

*Illustrative example — not captured output:*

```text
trainee
```

```text
uid=1000(trainee) gid=1000(trainee) groups=1000(trainee),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users)
```

```text
trainee adm cdrom sudo dip plugdev users
```

```text
0022
```

```text
drwxr-xr-x <links> root root 4096 <date> /
```

```text
trainee:x:1000:1000::/home/trainee:/bin/bash
```

```text
adm:x:4:syslog,trainee
cdrom:x:24:trainee
sudo:x:27:trainee
dip:x:30:trainee
plugdev:x:46:trainee
users:x:100:trainee
trainee:x:1000:
```

```text
-rw-r--r-- 1 root root    <size> <date> /etc/group
-rw-r--r-- 1 root root    <size> <date> /etc/passwd
-rw-r----- 1 root shadow  <size> <date> /etc/shadow
```

```text
cat: /etc/shadow: Permission denied
```

The refusal on the last command is the whole point of `/etc/shadow` having a group of its own: you
are not in `shadow`, so the `---` for everybody else applies.

**Verify.**

- `id` and `groups` report the same memberships; `id` adds the numeric IDs.
- The long listing has the fields this module reads everywhere else: the type and the nine
  permission characters, the link count, the owner, the group, the size, and the path.
- Your own group line in `/etc/group` — the last one, named after you — has an empty member list.
  That is your primary group, inherited from the GID in your `/etc/passwd` line, so it is not listed
  there.
- Every other group you belong to shows up in a member list with your name in it.
- `/etc/passwd` and `/etc/group` are `rw-r--r--`, so anyone can read them. `/etc/shadow` is
  `rw-r-----`, owned by `root:shadow`, and you are not in `shadow`, so `cat` was refused.
- Your `umask` is `0022`. Nothing in this section used it, but it decides the mode of every file
  you create from here on.

**Continue** to [§2](#2-permission-bits) to read the permission string that belongs to each of
these identities.

---

## 2. Permission bits

_Estimated: 20 minutes._

**Understand.** `ls -l` prints one line per file. Every field on that line means something:

| Field | Example | What it is |
| --- | --- | --- |
| first character | `-` or `d` | type: `-` regular file, `d` directory, `l` symbolic link |
| next nine characters | `rw-r--r--` | permissions: three sets of three — owner, group, other — each read, write, execute |
| link count | `1` | hard links to the file; for a directory, 2 plus the number of subdirectories |
| owner | `root` | the user that owns the file |
| group | `shadow` | the group that owns the file |
| size | `4096` | bytes; for a directory, the space its list of entries takes, normally 4096 on ext4 |
| time | `Oct 21 21:55` | when the contents last changed |
| name | `/etc/shadow` | the name, or the path you asked for |

A letter means the permission is on and a `-` means it is off. There is no separate access-control
system on top of these nine bits: if a rule is not in the bits, nothing grants it.

**Octal notation.** Each set of three is one digit, made by adding the value of each permission:
read is 4, write is 2, execute is 1.

| Symbolic | Bits | Octal digit |
| --- | --- | --- |
| `rwx` | 4+2+1 | 7 |
| `rw-` | 4+2+0 | 6 |
| `r-x` | 4+0+1 | 5 |
| `r--` | 4+0+0 | 4 |
| `---` | 0 | 0 |

A mode is three of those digits, owner first, then group, then other. `640` is owner `rw-` (6),
group `r--` (4), other `---` (0), which is the string `rw-r-----`.

**Symbolic notation.** `chmod [who][operator][bits] file`. The *who* is `u` for the owner, `g` for
the group, `o` for others, `a` for all. The *operator* is `+` to add, `-` to remove, `=` to set
exactly. The *bits* are `r`, `w`, `x`. `chmod u+x` adds execute to the owner's set and leaves every
other bit exactly as it was. `chmod 640 file` sets all nine bits at once, and anything you leave out
of the mode is cleared.

**Where the defaults come from.** A new file starts from `rw-rw-rw-` and a new directory from
`rwxrwxrwx`, and the bits named in your `umask` are taken away. With a `umask` of `0022` a new file
comes out `644` (`rw-r--r--`) and a new directory `755` (`rwxr-xr-x`). `umask` is a bash builtin, so
`help umask` documents it.

`stat -c '%a %A %n' file` prints the octal mode, the symbolic mode, and the name, which is a
convenient way to read a mode without changing anything.

**Preview.** You set modes yourself in [§5](#5-worked-example-the-m5-lab): `chmod 600` on a file you
create in Step 2, and `chmod u+x` on a script in Step 3. Every mode set in this module is the least
one that still does the job.

**Try.** Read the modes of files that already exist, and convert them to octal by hand.

```bash
stat -c '%a %A %n' /etc/passwd /etc/group /etc/shadow
stat -c '%a %A %U %G %n' /etc/shadow
```

*Illustrative example — not captured output:*

```text
644 -rw-r--r-- /etc/passwd
644 -rw-r--r-- /etc/group
640 -rw-r----- /etc/shadow
```

```text
640 -rw-r----- root shadow /etc/shadow
```

**Verify.**

- `/etc/passwd` is `644`: the owner can read and write it, the group can read it, everybody else can
  read it. Nothing about it is secret, which is why it holds every account on the system.
- `/etc/shadow` is `640`: owner read and write, group read, everybody else nothing. The `640` on the
  second line is the same mode, with the owner and the group spelled out.
- Both lines report the same mode twice, once in each notation, so you can check one against the
  other.

**Continue** to [§3](#3-ownership-vs-permission) to separate the two settings that decide who gets
those bits.

---

## 3. Ownership vs. permission

_Estimated: 15 minutes._

**Understand.** Who owns a file and what the owner is allowed to do with it are two separate
settings, and changing one does not change the other.

- `chown user:group file` changes the owner, and the group too when you write the `user:group`
  form. `chgrp group file` changes only the group.
- `chmod` changes only the permission bits. The owner and the group stay exactly as they were.
- `chown` on a file you own leaves the permission bits alone. `chmod` on a file you own leaves the
  owner and group alone.

Who is allowed to change what is set out in `man 2 chown`:

> Only a privileged process (Linux: one with the CAP_CHOWN capability) may change the owner of a
> file. The owner of a file may change the group of the file to any group of which that owner is a
> member. A privileged process (Linux: with CAP_CHOWN) may change the group arbitrarily.

In plain terms: an owner can always change the bits on their own file, and can hand that file to a
group they belong to. Giving a file to a different owner needs root. That is the difference between
"the file is mine" and "the file is mine and here is what I allow".

**A directory's bits mean something else.** The same three letters, three different jobs:

| Bit on a directory | What it lets you do |
| --- | --- |
| read (`r`) | list the names of the entries inside (`ls`) |
| write (`w`) | create, delete, and rename entries inside (`touch`, `rm`, `mv`) |
| execute (`x`) | search: enter the directory (`cd`) and reach the entries inside by name (`cat dir/file`) |

Without the execute bit you cannot enter the directory or open anything inside it, even a file whose
own bits would allow it. With read but no execute you can see the names but not their details.
And because the directory is what holds the entries, deleting a file depends on the **directory's**
write and execute bits, not on the file's own bits — with one exception, the sticky bit, which is
what keeps one user in a shared folder like `/tmp` from removing another user's files.

**Preview.** In [§5](#5-worked-example-the-m5-lab) you use `chown`, `chgrp`, and `chmod` on a shared
folder and on a report inside it, and the three of them each change something different.

**Try.** Make your own folder with read but no execute, and see the two halves of the read bit.

```bash
mkdir -p ~/linux-course/m5/xtest
touch ~/linux-course/m5/xtest/a.txt
chmod 644 ~/linux-course/m5/xtest
ls -ld ~/linux-course/m5/xtest
ls ~/linux-course/m5/xtest
cd ~/linux-course/m5/xtest
chmod 755 ~/linux-course/m5/xtest
ls -ld ~/linux-course/m5/xtest
```

The `cd` is expected to fail. The shell stays where it was, so the last two commands still run and
the folder is left usable.

*Illustrative example — not captured output:*

```text
drw-r--r-- 2 trainee trainee 4096 <date> /home/trainee/linux-course/m5/xtest
```

```text
a.txt
```

```text
bash: cd: /home/trainee/linux-course/m5/xtest: Permission denied
```

```text
drwxr-xr-x 2 trainee trainee 4096 <date> /home/trainee/linux-course/m5/xtest
```

**Verify.**

- `chmod 644` on a **directory** did not produce `rw-r--r--`. It produced `drw-r--r--`: the same
  four letters, minus the `x` for everybody, including you. The `d` in front is the type, not a
  permission.
- `ls` still listed `a.txt`, because read on a directory is the permission to see the names inside.
- `cd` was refused, because search permission is the permission to go in. The same `x` bit that
  makes a script runnable makes a folder enterable.
- After `chmod 755`, the folder is `drwxr-xr-x` and `cd` works. The link count stayed at 2 because
  the folder contains a file, not a subdirectory.

**Continue** to [§4](#4-elevated-privilege) to meet the escape hatch that makes all of this
disappear, and why that is usually the wrong answer.

---

## 4. Elevated privilege

_Estimated: 15 minutes._

**Understand.** `sudo command` runs one command with elevated privileges. What happens between
typing it and getting output:

1. It checks the sudoers policy to see whether your account is allowed to run that command. On
   Ubuntu the usual grant is membership in the `sudo` group, so the policy is about *your groups*,
   not about your name.
2. It asks for **your own password**, not root's. The prompt names no account at all:
   `[sudo: authenticate] Password:`.
3. It caches the credential, so later `sudo` commands in the same session do not ask again. `sudo -k`
   makes it forget the cached credential, so the next command asks again. `sudo -v` refreshes it
   now, so the commands after it do not stop.
4. It logs the command it runs.

`sudo -u <user> command` runs the command as another user instead of as root. That is how you test
a restriction from the inside rather than reasoning about it, and
[§5](#5-worked-example-the-m5-lab) uses it to prove the restriction from both sides.

**Why `sudo` is the wrong fix for a permission error.** Root bypasses the permission model.
`man 7 path_resolution` says so: "On a traditional UNIX system, the superuser (root, user ID 0) is
all-powerful, and bypasses all permissions restrictions when accessing files." So `sudo` makes
"Permission denied" go away without fixing the owner or the mode that were wrong. The wrong owner
and the wrong bits are still there after the command finishes, and the next person without `sudo`
hits the same wall.

**When elevated privilege is genuinely warranted**, because the task itself changes or reads
something the system protects:

- installing packages
- editing files under `/etc`
- creating system accounts and groups
- reading `/etc/shadow`
- creating a folder outside your own home directory, such as `/srv` — the shared folder you build in
  [§5](#5-worked-example-the-m5-lab)

The habit worth resisting is reaching for `sudo` the moment something fails, instead of reading the
owner and the bits to find out why.

**Preview.** [§5](#5-worked-example-the-m5-lab) uses `sudo` for exactly these reasons, one command
at a time, and uses `sudo -u` to check the restriction from the point of view of the other identity.

**Try.** Watch the prompt, and read `/etc/shadow` without letting a hash reach the screen.

```bash
sudo -k
sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
```

*Illustrative example — not captured output:*

```text
[sudo: authenticate] Password:
root:*:<days>:0:99999:7:::
trainee:[redacted]:<days>:0:99999:7:::
```

The pipeline is doing three jobs. `sudo cat` reads the file as root, which is allowed. `sed` replaces
every password hash with `[redacted]` before anything is printed, so no hash reaches your screen and
none ends up in your write-up. `grep` keeps only root's line and yours, so the output stays short
enough to read.

The two lines also answer the question the prompt raises. Root's password field is `*`, not a hash.
A `*` in `/etc/shadow` means no password ever matches that account, so there is no root password to
type — the password you typed was your own. The number after the password field is the day the
password was last changed, counted from January 1, 1970; `man 5 shadow` calls it the last-change
field.

**Verify.**

- The prompt appeared, asked for a password, and named no account.
- Root's line shows `*` where a hash would be, so the password you typed could not have been root's.
- Your own line shows `[redacted]` where your hash was, and a day count after it.
- `sudo -k` before the command is what made the prompt appear at all; without it, a cached
  credential would have let the command run silently.

**Continue** to [§5](#5-worked-example-the-m5-lab) to run the lab end to end, then build the shared
directory and prove the restriction from both sides.

---

## 5. Worked example: the M5 lab

_Estimated: 35 minutes._

Work through the steps in order. Run every command in your own environment and keep your own
output — the write-up needs the real thing, not the examples in this guide.

### Step 1 — Create a practice file and read its default permissions

```bash
mkdir -p ~/linux-course/m5
cd ~/linux-course/m5
touch shared-report.txt
ls -l shared-report.txt
```

*Illustrative example — not captured output:*

```text
-rw-r--r-- 1 trainee trainee 0 <date> shared-report.txt
```

`644` is not a coincidence. The file was created as `rw-rw-rw-` and your `umask` of `0022` took
away the group write and the other write.

**Capture in your write-up:** the `ls -l shared-report.txt` output, and the mode it shows.

### Step 2 — Restrict the file to its owner

```bash
chmod 600 shared-report.txt
ls -l shared-report.txt
```

*Illustrative example — not captured output:*

```text
-rw------- 1 trainee trainee 0 <date> shared-report.txt
```

Only the permissions changed. The owner, the group, the size, and the date are the same as in Step 1
— that is the proof that `chmod` changes the bits and nothing else.

**Capture in your write-up:** both `ls -l` outputs, side by side, plus a sentence of your own
reading the three sets: owner `rw-`, group `---`, other `---`.

### Step 3 — Make a script executable by its owner only

```bash
touch run.sh
ls -l run.sh
chmod u+x run.sh
ls -l run.sh
```

*Illustrative example — not captured output:*

```text
-rw-r--r-- 1 trainee trainee 0 <date> run.sh
```

```text
-rwxr--r-- 1 trainee trainee 0 <date> run.sh
```

`u+x` added execute to the owner's set and nothing else. The owner's `rw` stayed, the group's and
everybody else's `r--` stayed. A symbolic mode changes only the bits it names; an octal mode would
have set all nine at once.

**Capture in your write-up:** the `ls -l run.sh` output before and after, and a sentence naming
what `u+x` changed (the owner's execute bit) and what it left alone (every other bit).

### Step 4 — Watch what `sudo` asks for

```bash
sudo -k
sudo cat /etc/shadow | sed -E 's/^([^:]+):\$[^:]*:/\1:[redacted]:/' | grep -E "^(root|$USER):"
```

*Illustrative example — not captured output:*

```text
[sudo: authenticate] Password:
root:*:<days>:0:99999:7:::
trainee:[redacted]:<days>:0:99999:7:::
```

Two things go in the write-up: the prompt text, and the fact that the password was yours. The
evidence that it was yours and not root's is in the output — root's password field is `*`, so no
root password exists to be typed. [§4](#4-elevated-privilege) explains the whole sequence.

**Capture in your write-up:** the prompt, with the hash redacted. Never paste a hash.

### Step 5 — Say what `750` means, before you check it

In your own words, write down what `750` says about the owner, about the group, and about
everybody else: what each of the three can do.

Write it down first, then check it against the digit table in [§2](#2-permission-bits), the concept
section for permission bits.

**Capture in your write-up:** your own-words explanation, written before you looked it up.

### Deliverable — build the shared directory and prove it

This part runs after Step 5 on purpose: the mode you just wrote down is the mode you are about to
set. It needs a second identity, so it is also where the two test accounts are created.

**Create the group and the two test accounts.**

```bash
getent group m5-analysts
sudo groupadd m5-analysts
sudo useradd -m -s /bin/bash -G m5-analysts m5-analyst
sudo useradd -m -s /bin/bash m5-outsider
id m5-analyst
id m5-outsider
getent group m5-analysts
```

*Illustrative example — not captured output:*

```text
uid=<uid>(m5-analyst) gid=<gid>(m5-analyst) groups=<gid>(m5-analyst),<gid>(m5-analysts)
```

```text
uid=<uid>(m5-outsider) gid=<gid>(m5-outsider) groups=<gid>(m5-outsider)
```

```text
m5-analysts:x:<gid>:m5-analyst
```

The first account carries `m5-analysts` as an extra group; the second carries no group but its own.
If a name is already taken on your system, `getent` will find it and the `groupadd` or `useradd`
that follows will report the name is taken and change nothing — check the memberships with `id` and
carry on.

**Create the folder and set the scheme.**

```bash
ls -ld /srv
sudo mkdir -p /srv/m5-reports
sudo chown "$USER":m5-analysts /srv/m5-reports
chmod 750 /srv/m5-reports
ls -ld /srv
ls -ld /srv/m5-reports
```

*Illustrative example — not captured output:*

```text
drwxr-xr-x 2 root root 4096 <date> /srv
```

```text
drwxr-xr-x 3 root root 4096 <date> /srv
```

```text
drwxr-x--- 2 trainee m5-analysts 4096 <date> /srv/m5-reports
```

Three separate settings, three separate commands. `chown` set the owner and the group, `chmod` set
the bits, and the link count on `/srv` went from 2 to 3 because it now holds one subdirectory. The
`chmod 750` needs no `sudo` of its own: `chown` has already made the folder yours, and an owner can
always change the bits on their own file.

`750` is the mode you wrote down in Step 5. On a **directory**, it means you can list the folder,
go into it, and create or delete files in it. The group's read and execute let it list the folder,
go into it, and reach the files inside by name, but with no write it cannot add, rename, or remove
anything there. Everybody else cannot list the folder or go into it, so nothing inside is reachable
for them at all.

**Add the report and give it its own group.**

```bash
echo "Q3 orders summary: draft for the analysts" > /srv/m5-reports/q3-summary.txt
sudo chgrp m5-analysts /srv/m5-reports/q3-summary.txt
chmod 640 /srv/m5-reports/q3-summary.txt
ls -l /srv/m5-reports
```

*Illustrative example — not captured output:*

```text
total 4
-rw-r----- 1 trainee m5-analysts 42 <date> q3-summary.txt
```

The report is `640`: you can read and change it, the group can read it, everybody else gets nothing.
`chgrp` needed `sudo` because you own the report but are not a member of `m5-analysts`, and an
owner may only hand a file to a group they belong to. `chmod 640` needed no `sudo`, because you own
the report.

**Prove it, from both sides.**

```bash
sudo -u m5-analyst ls -l /srv/m5-reports
sudo -u m5-analyst cat /srv/m5-reports/q3-summary.txt
sudo -u m5-analyst touch /srv/m5-reports/new-file.txt
sudo -u m5-outsider ls /srv/m5-reports
sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt
```

*Illustrative example — not captured output:*

```text
total 4
-rw-r----- 1 trainee m5-analysts 42 <date> q3-summary.txt
```

```text
Q3 orders summary: draft for the analysts
```

```text
touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied
```

```text
ls: cannot open directory '/srv/m5-reports': Permission denied
```

```text
cat: /srv/m5-reports/q3-summary.txt: Permission denied
```

`sudo -u <user>` runs the command as that user, so you see what the user sees. The analyst is in the
folder's group, so it got the group's `r-x`: it could list the folder and read the report, but it
could not create a file, because creating one needs write on the folder and the group has none. The
outsider is neither the owner nor in the group, so it got the `---` for everybody else: it could not
list the folder, and it could not reach the report either. The folder is what stopped that last
`cat` — without execute on `/srv/m5-reports` nothing inside is reachable by name — and the report's
own `---` for everybody else would have stopped it as well.

Both outcomes are demonstrated, not just configured. That is the deliverable.

**Capture in your write-up:** the mode and ownership of the folder and of the report, the denied
attempt, and the authorized attempt.

### Clean-up

The brief's clean-up for this module is short: the practice files stay under `~/linux-course/`.
Nothing from Steps 1 to 5 needs to be deleted, and the folder you made in
[§3](#3-ownership-vs-permission) stays there too.

What that clean-up does not cover is the deliverable, which leaves three things behind: the two
test accounts with their home folders, the `m5-analysts` group, and the `/srv/m5-reports` folder.
They stay in place while you need them, because your write-up's checkpoint evidence points at them.
When you are finished with them, remove them yourself:

```bash
sudo userdel -r m5-analyst
sudo userdel -r m5-outsider
sudo groupdel m5-analysts
sudo rm -r /srv/m5-reports
```

The `-r` on `userdel` takes each account's home folder with it. The group is not yours, and `/srv`
belongs to root even though the folder inside it is yours, so those removals need `sudo`. There is no
`-f` on the `rm` on purpose: `-f` only hides mistakes, and under `sudo` a typo in the path does real
damage.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Every item below is one of M5's definition-of-done items, in the brief's own words. Check each one
against your own notes.

- [ ] DoD-01: The trainee's pasted `ls -l shared-report.txt` output before and after `chmod 600`
  shows the change, and the notes read the permission string for the owner, group, and other
  positions (Lab steps 1–2). Verify via [§5](#5-worked-example-the-m5-lab) and
  [§2](#2-permission-bits).
- [ ] DoD-02: The pasted `ls -l run.sh` output shows the execute bit set for the owner only, and the
  notes state what `u+x` changed and what it left alone (Lab step 3). Verify via
  [§5](#5-worked-example-the-m5-lab) and [§2](#2-permission-bits).
- [ ] DoD-03: The notes show that `sudo cat /etc/shadow` prompted for the trainee's own password
  rather than a root password, with the password hashes redacted (Lab step 4). Verify via
  [§4](#4-elevated-privilege) and [§5](#5-worked-example-the-m5-lab).
- [ ] DoD-04: The trainee's own-words explanation of `750` was written before it was checked against
  the Concepts section, and states what owner, group, and other can each do (Lab step 5). Verify via
  [§5](#5-worked-example-the-m5-lab) and [§2](#2-permission-bits).
- [ ] DoD-05: No step in the notes uses a blanket `777` "open everything" shortcut; every mode set is
  the least permissive one that still works. Verify via
  [§5](#5-worked-example-the-m5-lab) and [§2](#2-permission-bits).

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- Use `sudo <command>` for the one command that needs it, not `sudo su` — see
  [§4](#4-elevated-privilege).
- Default to the least permissive setting that still works — see
  [§5](#5-worked-example-the-m5-lab) and [§2](#2-permission-bits).
- Never treat `777` as a quick fix for a permission error — see
  [§5](#5-worked-example-the-m5-lab) and [§2](#2-permission-bits).
- Check both ownership (`chown`) and permission bits (`chmod`) when debugging "permission denied" —
  see [§3](#3-ownership-vs-permission) and [§5](#5-worked-example-the-m5-lab).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `cat: /etc/shadow: Permission denied` | `/etc/shadow` is `rw-r-----`, owned by `root:shadow`, and you are not in `shadow`. | Read it with `sudo` for that one command, and redact the hash with `sed` before it reaches the screen. |
| `touch: cannot touch '/etc/m5-probe.txt': Permission denied` | `/etc` belongs to root, so only root can create a file there. | Do the work in your own directory. Use `sudo` for the one command that genuinely has to land in `/etc`. |
| `ls: cannot open directory '/root': Permission denied` | `/root` is root's home and is closed to everybody else. | Leave other accounts' home directories alone. Use `sudo -u <user>` for an account you are entitled to check. |
| `bash: cd: /home/trainee/linux-course/m5/xtest: Permission denied` | The directory has no execute (search) bit, so it cannot be entered. | Add execute on the directory — `chmod 755` — not on the files inside it. |
| `chown: changing ownership of 'shared-report.txt': Operation not permitted (os error 1)` | Only a privileged process may hand a file to a new owner (`man 2 chown`). | Only root can do that, so this one command has to run as root: `sudo chown <user>:<group> <file>`. Then set the bits with a plain `chmod` as the new owner. |
| `chgrp: changing group of 'shared-report.txt': Operation not permitted (os error 1)` | An owner may only give a file to a group they belong to. | Have root run that one `chgrp`, as the deliverable does: `sudo chgrp <group> <file>`. Then set the bits with a plain `chmod`. |
| `touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied` | The folder's group has `r-x` and no `w`, so the group cannot create files in it. | Give the group write only if the group really has to create files. Otherwise have the owner create the file. |
| `ls: cannot open directory '/srv/m5-reports': Permission denied` | The folder grants "other" nothing, and this account is neither the owner nor in the group. The denial is the restriction working as designed. | Nothing to fix unless this account is supposed to have access. If it is, add it to `m5-analysts` and log in again before you retest. |
| `cat: /srv/m5-reports/q3-summary.txt: Permission denied` | The outsider gets the folder's "other" bits, so the path closes before the report's own bits come into it. The denial is the restriction working as designed. | Nothing to fix unless this account is supposed to have access. If it is, add it to `m5-analysts` and log in again before you retest. |

The `(os error 1)` at the end of the `chown` and `chgrp` messages is the course image's coreutils
(uutils 0.10.0) reporting the system error number. GNU coreutils prints the same two messages
without that suffix.

---

## 8. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference. On this image `ls`, `id`, `stat`, `chmod`, `chown`, and `chgrp` come from the uutils
(`rust-`) implementation of coreutils, so their local pages are the uutils ones and differ from the
GNU coreutils pages linked below. `sudo`, `sudoers`, and `visudo` are sudo-rs pages;
`passwd`, `shadow`, `useradd`, and `groupadd` are shadow-utils pages, and `group` is a Linux
man-pages page.

- `man ls` — the uutils (`rust-ls`) page on this image. The `-l` long listing whose fields this
  module reads.
- `man id` — the uutils (`rust-id`) page on this image. Printing the user ID, the primary group,
  and every supplementary group.
- `man stat` — the uutils (`rust-stat`) page on this image. The `%a` (octal mode), `%A` (symbolic
  mode), `%U`, and `%G` format specifiers.
- `man chmod` — the uutils (`rust-chmod`) page on this image. The symbolic and numeric modes, and
  the `umask` relationship.
- `man chown` — the uutils (`rust-chown`) page on this image. Changing the owner, and the group
  with the `user:group` form.
- `man chgrp` — the uutils (`rust-chgrp`) page on this image. Changing the group only.
- `man 2 chown` — the kernel-level rule about who may change an owner and who may change a group.
- `man sudo` — the sudo-rs 0.2.13 page on this image. The policy check, the password prompt, the
  cached credential, and `sudo -u`. The page has no SECURITY NOTES section; the original sudo's
  `sudo(8)`, linked below, does.
- `man 5 passwd` — the account database: the seven fields of a `/etc/passwd` line.
- `man 5 group` — the group database: the four fields of a `/etc/group` line.
- `man 5 shadow` — the password database: the nine fields of a `/etc/shadow` line, and what the
  last-change day count means.
- `man 7 path_resolution` — how a pathname is resolved, and how the three permission groups are
  chosen.
- `man 8 useradd` — creating the test accounts: `-m` for a home folder, `-G` for an extra group,
  `-s` for the shell.
- `man 8 groupadd` — creating the `m5-analysts` group.
- `man 5 sudoers` — the policy `sudo` reads before it runs anything.
- `man 8 visudo` — editing that policy safely.
- `help umask` — a bash builtin, not a man page.

Online copies of the same pages:

- [`chmod(1)`](https://man7.org/linux/man-pages/man1/chmod.1.html) — GNU's page; it differs from the
  local uutils page, and it is the page that documents the set-user-ID and set-group-ID rules.
- [`chown(1)`](https://man7.org/linux/man-pages/man1/chown.1.html) — GNU's page; it differs from the
  local uutils page.
- [`chgrp(1)`](https://man7.org/linux/man-pages/man1/chgrp.1.html) — GNU's page; it differs from the
  local uutils page.
- [`id(1)`](https://man7.org/linux/man-pages/man1/id.1.html) — GNU's page; it differs from the local
  uutils page.
- [`stat(1)`](https://man7.org/linux/man-pages/man1/stat.1.html) — GNU's page; it differs from the
  local uutils page.
- [`sudo(8)`](https://man7.org/linux/man-pages/man8/sudo.8.html) — the original sudo's page, which is
  not GNU's; it differs from the local sudo-rs page, and it has the SECURITY NOTES section the
  local one lacks.
- [`passwd(5)`](https://man7.org/linux/man-pages/man5/passwd.5.html) — the Linux man-pages version;
  the local page is shadow-utils'.
- [`group(5)`](https://man7.org/linux/man-pages/man5/group.5.html) — the Linux man-pages version,
  which is also the local page.
- [`shadow(5)`](https://man7.org/linux/man-pages/man5/shadow.5.html) — the Linux man-pages version;
  the local page is shadow-utils'.
- [`path_resolution(7)`](https://man7.org/linux/man-pages/man7/path_resolution.7.html) — the Linux
  man-pages version, which is also the local page.
- [`useradd(8)`](https://man7.org/linux/man-pages/man8/useradd.8.html) — the Linux man-pages
  version; the local page is shadow-utils'.
- [`groupadd(8)`](https://man7.org/linux/man-pages/man8/groupadd.8.html) — the Linux man-pages
  version; the local page is shadow-utils'.
- [Linux file permissions explained](https://www.redhat.com/en/blog/linux-file-permissions-explained)
  — Red Hat, Scott McBrien, January 10, 2023.

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — the template your `write-up.md` is made from.
- Course architecture:
  [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) — where M5 sits in the
  curriculum (Layer 3, Week 3, with M6).

Curriculum resources for the Access & Software layer (M5), by name:

- *Linux File Permissions Explained* (Red Hat)
- *User and File System Privileges*
