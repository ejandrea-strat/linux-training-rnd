# M3 Learning Materials — Filesystem Navigation

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
> you use any example as evidence.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes **87 minutes** — 12 + 12 + 10 + 13
> for the four concept sections, 25 for the worked example, and 5 each for the checklist, the best
> practices, and the references. M3 shares **Week 2** of the course with M4, in **Layer 2
> (Filesystem & Data Handling)**. Commands assume **bash** on **WSL2 Ubuntu 26.04**, the course
> environment; on a Virtual Machine or in a container the same commands report your own
> environment's values.
>
> **Prerequisites.** M1's `head` and `wc -l`, and M2's pipes plus the fact that errors (file
> descriptor 2) do not travel through a pipe — [§4](#4-searching-instead-of-browsing) relies on both.
> The shell, the terminal, `ls`, and `man` from M1 and M2 are assumed throughout.

## Contents

1. [The Filesystem Hierarchy Standard (FHS)](#1-the-filesystem-hierarchy-standard-fhs)
2. [Absolute vs. relative paths](#2-absolute-vs-relative-paths)
3. [Building structures in one step](#3-building-structures-in-one-step)
4. [Searching instead of browsing](#4-searching-instead-of-browsing)
5. [Worked example: the M3 lab](#5-worked-example-the-m3-lab)
6. [Completion checklist](#6-completion-checklist)
7. [Best practices and troubleshooting](#7-best-practices-and-troubleshooting)
8. [References](#8-references)

---

## 1. The Filesystem Hierarchy Standard (FHS)

_Estimated: 12 minutes._

**Understand.** Linux has **one tree** of files and folders. It starts at a single root, `/`, and
everything — your home folder, installed programs, logs, even other disks — hangs somewhere under
that root. There are no drive letters. On WSL2 the Windows drives appear as folders inside the tree:
`C:` is mounted at `/mnt/c`.

The **Filesystem Hierarchy Standard (FHS)** is a shared agreement about *what lives where* in that
tree. The directories this module cares about:

| Directory | What it holds | Examples |
| --- | --- | --- |
| `/etc` | Configuration files local to the machine, mostly plain text | `/etc/os-release`, `/etc/wsl.conf` |
| `/var` | Files that change in size while the system runs — spool files, logs, caches | `/var/log` |
| `/home` | One folder per user; your personal files live here | `/home/<username>`; root's own home is `/root` instead |
| `/usr` | Shareable, read-only program files: commands in `/usr/bin`, libraries in `/usr/lib`, man pages under `/usr/share` | `/usr/bin/ls` |

On Ubuntu, `/bin` is not a separate folder — it is a **link** to `/usr/bin`. That is why M1's
`type -a ls` showed both `/usr/bin/ls` and `/bin/ls`: same file, two names.

Why is the FHS a **convention** and not a rule? The kernel only follows paths; it never checks what
a folder is "for". A program would still run from a different folder as long as it knew where to
look. The layout works because distros and programs agree to use it — `man hier` describes the
agreement.

**Try.**

```bash
ls /                       # the top of the one tree
ls -ld /bin                # is /bin a folder or a link?
ls /mnt                    # other disks/OSes as folders (WSL2 shows c, wsl, wslg)
```

An illustrative example of what those commands print on the course image:

```text
$ ls /
bin   dev  home  lib    lost+found  mnt  proc  run   snap  sys  usr
boot  etc  init  lib64  media       opt  root  sbin  srv   tmp  var

$ ls -ld /bin
lrwxrwxrwx 1 root root 7 <date> /bin -> usr/bin

$ ls /mnt
c  wsl  wslg
```

Read the listing: `/etc`, `/var`, `/home`, and `/usr` are all there, plus `/root` (root's home),
and `/proc` and `/dev` from M1. In `ls -ld /bin`, the `l` at the start of the mode means *link*, and
`-> usr/bin` shows where it points. On WSL2, `/mnt/c` is the Windows C: drive — Windows drives show
up as folders inside the tree, not as letters.

> **Your machine may differ.** Your top-level listing may show a different set (for example, no
> `/init` outside WSL2). That is expected; the four FHS directories above are what matters.

**Verify.** `ls /` shows a single tree with `/etc`, `/var`, `/home`, and `/usr` in it;
`ls -ld /bin` either shows a link (`l...` and `->`) or a plain directory, depending on the distro.

**Continue** to [§2](#2-absolute-vs-relative-paths) to learn how paths name a place in that tree —
and how to tell whether a path is anchored or relative.

---

## 2. Absolute vs. relative paths

_Estimated: 12 minutes._

**Understand.** A path names a location in the tree. Two kinds:

| Kind | Starts with | Means | Better when |
| --- | --- | --- | --- |
| **Absolute path** | `/` | The same place from anywhere | You are jumping somewhere unrelated, like `/var/log` |
| **Relative path** | anything else | Starts from the current folder (what `pwd` prints) | The target is close, like `..` in the Try below, or the tree should stay portable |

Knowing which kind you are typing is what prevents getting lost. An absolute path is the right
choice for jumping somewhere unrelated; a relative path is shorter when the target is close by.

Three special names worth knowing:

- **`.`** — the folder you are in right now.
- **`..`** — the folder one level up.
- **`~`** — your home folder. `~` is not part of a relative path: the shell swaps it in *before* the
  command runs (this is *tilde expansion*, described under "Tilde Expansion" in `man bash`), turning
  `~/linux-course` into the absolute path `/home/<your username>/linux-course`. Because the shell
  does the swap, `~` does not work inside quotes: `cd "~"` looks for a folder literally named `~`.
  Running `cd` with no argument also goes home.

Every folder contains `.` and `..`, which is why `ls -a` (M2) always shows them.

**Try.** Start in the system log folder and move with `..` and with relative names, confirming each
location:

```bash
cd /var/log
pwd          # /var/log
cd ..        # up one level
pwd          # /var
cd log       # back down, relative to where you are
pwd          # /var/log
cd ../..     # up two levels
pwd          # /
cd           # no argument: home
pwd          # /home/trainee
```

An illustrative example of what those commands print:

```text
$ cd /var/log
$ pwd
/var/log
$ cd ..
$ pwd
/var
$ cd log
$ pwd
/var/log
$ cd ../..
$ pwd
/
$ cd
$ pwd
/home/trainee
```

`cd ..` worked because `..` is the folder one level up from `/var/log`, which is `/var`. `cd log`
then went back down by a **relative** name: `log` exists inside `/var`, so no leading `/` was
needed. `cd ../..` repeated `..` twice to reach `/`, and `cd` with no argument returned to your home
folder.

The same `~` shortcut fails when it is quoted. An illustrative example:

```text
$ cd "~"
bash: cd: ~: No such file or directory
```

**Verify.** Your five `pwd` runs print `/var/log`, `/var`, `/var/log`, `/`, and your home path, in
that order; only the last one is home. You can say why `cd log` worked only because you were already
in `/var`.

**Continue** to [§3](#3-building-structures-in-one-step) to create nested folders of your own — in a
single command.

---

## 3. Building structures in one step

_Estimated: 10 minutes._

**Understand.** `mkdir` creates a folder. To create a *nested* path like `a/b/c`, the parent folders
must exist first. The option that creates every missing folder along the way is **`-p`**
(`--parents`):

```bash
mkdir -p a/b/c      # creates a, a/b, and a/b/c in one command
```

Two properties of `-p` worth knowing (see `man mkdir`):

- **Without `-p`**, `mkdir a/b/c` fails if `a/b` does not exist yet, and you would have to create
  each level one at a time.
- **With `-p`**, an already-existing folder is *not* an error, so running the same command twice is
  safe. One `mkdir -p` can also take several paths at once.

On this course image `man mkdir` opens the uutils (`rust-mkdir`) page; its `-p, --parents` entry
says "make parent directories as needed".

**Try.** Try the brief's first path without `-p`:

```bash
mkdir ~/linux-course/m3/data/raw
```

An illustrative example of what it prints:

```text
$ mkdir ~/linux-course/m3/data/raw
mkdir: No such file or directory
```

The command created nothing, because `m3/data` did not exist yet.
[§5](#5-worked-example-the-m3-lab) Step 2 creates the same path with `-p`, which makes every missing
level in one command.

**Verify.** The command fails with `mkdir: No such file or directory`, and `ls ~/linux-course` shows
no `m3` folder.

**Continue** to [§4](#4-searching-instead-of-browsing) to find files in a tree without walking it by
hand.

---

## 4. Searching instead of browsing

_Estimated: 13 minutes._

**Understand.** When you do not know a file's exact location, search instead of clicking through
folders. Two tools, two different engines:

| Tool | How it searches | Trade-off |
| --- | --- | --- |
| `find` | Walks the folders **live**, every time it runs | Always up to date, slower on big trees |
| `locate` | Queries a **saved database** of file names, updated about once a day | Much faster, but can miss files created since the last update |

`locate` is **not installed** on this course image and has no local man page, so this guide does not
run it. The idea is still worth knowing: its speed comes from a database that is only refreshed
periodically, while `find` sees the tree as it is right now.

`find` takes a starting point and a test. For a name pattern:

```bash
find /etc -name "*.conf"
```

`-name` matches against the file's **base name** with shell-style patterns: `*` means any
characters, `?` means exactly one character. Unlike `grep`'s regular expressions (M2), the `.` here
is a **plain dot** — a name like `dconf` does not match `"*.conf"`; the pattern matches names ending
in the literal characters `.conf`. Use `-iname` if you want upper and lower case treated the same.

`find` can also match by **attribute**. `-type f` keeps regular files, `-type d` keeps directories,
and either can be combined with a name test. So `find ~/linux-course/m3 -type f` lists every file in
the module tree — a preview, because [§5](#5-worked-example-the-m3-lab) Step 2 builds that tree.
For searching **inside** files, `grep` is the tool (M2): `find` finds names and attributes, `grep`
finds contents.

One trap: **quote the pattern** (`"*.conf"`). Quotes stop the *shell* from expanding the `*` before
`find` ever sees it. Full details in `man find`.

**Try.**

```bash
find /etc -name "*.conf" | head -10     # a sample of the results
find /etc -name "*.conf" | wc -l        # how many, in total
```

An illustrative example of what the first command prints:

```text
$ find /etc -name "*.conf" | head -10
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
/etc/PackageKit/PackageKit.conf
/etc/PackageKit/Vendor.conf
/etc/ca-certificates.conf
/etc/locale.conf
/etc/resolv.conf
/etc/ssh/ssh_config.d/20-systemd-ssh-proxy.conf
/etc/rsyslog.d/21-cloudinit.conf
/etc/rsyslog.d/50-default.conf
/etc/logrotate.conf
/etc/wsl.conf
```

An illustrative example of the count:

```text
$ find /etc -name "*.conf" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
<count>
```

Several things to notice:

- **`<count>` entries** is your own number; it differs between machines. Run the command and record
  yours.
- `find` searches **subfolders**, like `/etc/ssh/ssh_config.d/` and `/etc/rsyslog.d/`. M2's
  `ls -l /etc | grep ".conf"` only ever listed the top level, so it found far fewer.
- The results are **not alphabetical**: `find` lists files in the order it walks the folders, unlike
  the sorted output of `ls`.
- The `Permission denied` lines are folders your user cannot open. They are **errors** on standard
  error (file descriptor 2), so they appear on screen but do not travel through the pipe to `wc`,
  and they are not part of the count.
- The `head -10` run shows **fewer** error lines than the `wc -l` run: `head` closes the pipe once it
  has its ten lines, so `find` stops early and never reaches the folders that would have produced the
  rest.

Case matters in a name pattern. `-name` is case-sensitive, `-iname` is not. An illustrative
comparison:

```text
$ find /etc -name "*.CONF" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
0

$ find /etc -iname "*.CONF" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
<count>
```

The uppercase pattern matches no names, because no name in `/etc` ends in uppercase `.CONF`.
`-iname` treats upper and lower case as the same, so it finds them. The `.CONF` pattern
is a demonstration only; the lab uses `*.conf`.

**Verify.** Both `find` commands run on your machine; `wc -l` prints a number; at least one result
path contains a subfolder of `/etc`, proving the search went deeper than the top level. The `-iname`
run finds the `.conf` files that the `-name "*.CONF"` run misses.

**Continue** to [§5](#5-worked-example-the-m3-lab) to produce the lab evidence.

---

## 5. Worked example: the M3 lab

_Estimated: 25 minutes._

**Understand.** The five lab steps come from the Lab section of [`brief.md`](./brief.md); the
deliverable steps come from the checklist in [`tasks.md`](./tasks.md). Each part gives the commands,
an illustrative example of what they print, and the evidence to capture in your own notes.

### Step 1 — Move around and confirm with `pwd`

**Understand.** `pwd` prints the folder you are in. Run it before and after a `cd` so the move is
proven, not assumed.

**Try.**

```bash
cd ~
pwd
cd /var/log
pwd
cd ~
pwd
```

An illustrative example of what those commands print:

```text
$ cd ~
$ pwd
/home/trainee
$ cd /var/log
$ pwd
/var/log
$ cd ~
$ pwd
/home/trainee
```

**Capture.** Paste all three `pwd` outputs into your notes. They show the starting location
(`~` → `/home/trainee`), the move to `/var/log`, and the return home. Use *your* home path.
*Verify via [§2](#2-absolute-vs-relative-paths).*

### Step 2 — Choose the scheme, build the tree, fill it

**Understand.** [`tasks.md`](./tasks.md) says to decide the organizational scheme before creating
anything, and to check with the trainer if the design is borderline. The lab's minimum is two
`mkdir -p` commands, but the brief's deliverable asks for a *populated* structure that reflects a
deliberate scheme rather than a flat dump. This example follows how data moves: raw files land in
`data/raw`, cleaned copies go to `data/processed`, and `scripts` and `docs` hold the code and the
notes about the data. Design your own scheme before you build.

**Try.**

```bash
mkdir -p ~/linux-course/m3/data/raw
mkdir -p ~/linux-course/m3/data/processed
mkdir -p ~/linux-course/m3/data/raw/orders ~/linux-course/m3/data/processed/orders ~/linux-course/m3/scripts ~/linux-course/m3/docs
echo "order_id,amount" > ~/linux-course/m3/data/raw/orders/orders_2026-01-15.csv
echo "order_id,amount" > ~/linux-course/m3/data/processed/orders/orders_clean_2026-01-15.csv
echo "Raw order files land in data/raw/orders." > ~/linux-course/m3/docs/README.md
```

The first two commands are the brief's; the rest fill the scheme. None of them print anything when
they work — `mkdir -p` and a redirected `echo` are silent on success. The third `mkdir -p` made four
folders at once. Any date works; the file names are your choice.

`echo ... > file` is M2's redirection: it writes a small file in one step.

**Capture.** Record the commands you ran and the scheme you chose. There is no output to paste; the
proof is the listing in Step 3. If you asked the trainer whether the design is borderline, record
the answer. *Verify via [§3](#3-building-structures-in-one-step).*

### Step 3 — List the tree, including hidden files

**Understand.** `ls -la` lists everything in a folder, including entries starting with `.`. It lists
**one level**; to see the three levels under `m3` that DoD-02 asks for, follow it with `ls -laR`
(recursive).

**Try.**

```bash
ls -la ~/linux-course/m3
ls -laR ~/linux-course/m3
```

An illustrative example of the first command:

```text
$ ls -la ~/linux-course/m3
total <n>
drwxr-xr-x 5 trainee trainee 4096 <date> .
drwxr-xr-x 4 trainee trainee 4096 <date> ..
drwxr-xr-x 4 trainee trainee 4096 <date> data
drwxr-xr-x 2 trainee trainee 4096 <date> docs
drwxr-xr-x 2 trainee trainee 4096 <date> scripts
```

An illustrative example of the recursive listing:

```text
$ ls -laR ~/linux-course/m3
/home/trainee/linux-course/m3:
total <n>
drwxr-xr-x 5 trainee trainee 4096 <date> .
drwxr-xr-x 4 trainee trainee 4096 <date> ..
drwxr-xr-x 4 trainee trainee 4096 <date> data
drwxr-xr-x 2 trainee trainee 4096 <date> docs
drwxr-xr-x 2 trainee trainee 4096 <date> scripts

/home/trainee/linux-course/m3/data:
total <n>
drwxr-xr-x 4 trainee trainee 4096 <date> .
drwxr-xr-x 5 trainee trainee 4096 <date> ..
drwxr-xr-x 3 trainee trainee 4096 <date> processed
drwxr-xr-x 3 trainee trainee 4096 <date> raw

/home/trainee/linux-course/m3/data/processed:
total <n>
drwxr-xr-x 3 trainee trainee 4096 <date> .
drwxr-xr-x 4 trainee trainee 4096 <date> ..
drwxr-xr-x 2 trainee trainee 4096 <date> orders

/home/trainee/linux-course/m3/data/processed/orders:
total <n>
drwxr-xr-x 2 trainee trainee 4096 <date> .
drwxr-xr-x 3 trainee trainee 4096 <date> ..
-rw-r--r-- 1 trainee trainee   16 <date> orders_clean_2026-01-15.csv

/home/trainee/linux-course/m3/data/raw:
total <n>
drwxr-xr-x 3 trainee trainee 4096 <date> .
drwxr-xr-x 4 trainee trainee 4096 <date> ..
drwxr-xr-x 2 trainee trainee 4096 <date> orders

/home/trainee/linux-course/m3/data/raw/orders:
total <n>
drwxr-xr-x 2 trainee trainee 4096 <date> .
drwxr-xr-x 3 trainee trainee 4096 <date> ..
-rw-r--r-- 1 trainee trainee   16 <date> orders_2026-01-15.csv

/home/trainee/linux-course/m3/docs:
total <n>
drwxr-xr-x 2 trainee trainee 4096 <date> .
drwxr-xr-x 5 trainee trainee 4096 <date> ..
-rw-r--r-- 1 trainee trainee   41 <date> README.md

/home/trainee/linux-course/m3/scripts:
total <n>
drwxr-xr-x 2 trainee trainee 4096 <date> .
drwxr-xr-x 5 trainee trainee 4096 <date> ..
```

Every folder in the listing has both `.` and `..`, and `..` always points one level up: in
`data/raw/orders`, `..` is `raw`. That is the structure proving three levels of depth (`data`, then
`raw` or `processed`, then `orders`) with a CSV file at the bottom of each side.

**Capture.** Paste **both** listings into your notes: `ls -la ~/linux-course/m3`, because that is
the command the lab asks for, and `ls -laR ~/linux-course/m3`, because plain `ls -la` shows only one
level and cannot show the three levels of depth that DoD-02 requires — the recursive listing is the
one that shows them. *Verify via [§3](#3-building-structures-in-one-step).*

### Step 4 — Search `/etc` for `.conf` files

**Understand.** This step proves search beats browsing: `ls -l /etc | grep ".conf"` (M2) only sees
the top level; `find` sees the whole subtree.

**Try.**

```bash
find /etc -name "*.conf" | wc -l
```

An illustrative example:

```text
$ find /etc -name "*.conf" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
<count>
```

**Capture.** Record **how many results your own machine returned**, taken from your own `wc -l`
output, along with the command itself. DoD-03 needs your own count. The `Permission denied` lines are
errors on standard error; they are not part of the count. *Verify via
[§4](#4-searching-instead-of-browsing).*

### Step 5 — Reference a sibling with a relative path

**Understand.** From inside `data/raw`, the `processed` folder is a *sibling*: go up one level with
`..`, then down into `processed`.

**Try.**

```bash
cd ~/linux-course/m3/data/raw
ls ../processed
cd ../processed
pwd
```

An illustrative example:

```text
$ cd ~/linux-course/m3/data/raw
$ ls ../processed
orders
$ cd ../processed
$ pwd
/home/trainee/linux-course/m3/data/processed
```

**Capture.** Paste the `ls ../processed` command and its output. Then, **without running a
command**, state the absolute path of `processed`: `~` is `/home/<your username>` (`/home/trainee` in
the example), and `processed` sits at `linux-course/m3/data/processed` under it, so the full path is
`/home/<your username>/linux-course/m3/data/processed`. The `pwd` at the end of the block confirms
it — but write your answer before you check. *Verify via [§2](#2-absolute-vs-relative-paths).*

### Deliverable steps from tasks.md

The lab produces the evidence; the deliverable is the populated tree plus the search evidence and
the write-up. The checklist in [`tasks.md`](./tasks.md) is the order of work.

**Setup.** Decide the organizational scheme before creating anything, and check with the trainer if
the design is borderline. Step 2 is where the scheme is chosen and built, so record your decision
there rather than repeating the commands here.

**Build — top-level folders.** From [§1](#1-the-filesystem-hierarchy-standard-fhs)'s `ls /`, note
what lives in each top-level FHS folder you explored, at least `/etc`, `/var`, `/home`, and `/usr`.
*Capture:* one line per folder.

**Build — path choice.** Using Steps 1 and 5, state when an absolute path is better and when a
relative path is better. *Capture:* one sentence per kind, with the Step 1 and Step 5 examples.

**Build — structure and search.** Step 2 builds and fills the tree; the search below locates a file
in it. Neither is repeated here.

**Verify.** Confirm the tree is at least three levels deep and was created in one step; state any
file's absolute path without running a command; confirm the file was found by search, not by visual
browsing.

**Write-up.** Fill every heading of [`write-up-template.md`](./write-up-template.md) in your own
`write-up.md`: "What I built," "Why it's built this way," "How to build it," "Concepts worth
explaining," "What tripped me up," "Checkpoint evidence," and "What I'd do differently." Under
"Checkpoint evidence," paste the tree listing and the search command from below. Finish with the
final self-review: re-read your write-up against the definition of done in [`brief.md`](./brief.md).

### Deliverable evidence — find a file in your tree by search

**Understand.** The brief's deliverable asks for proof that a file *in your own structure* was
located with a search tool, not by visual browsing. Step 4 searched a system folder; this closes the
loop on your tree.

**Try.**

```bash
find ~/linux-course/m3 -type f
find ~/linux-course/m3 -name "*.csv"
```

An illustrative example:

```text
$ find ~/linux-course/m3 -type f
/home/trainee/linux-course/m3/data/processed/orders/orders_clean_2026-01-15.csv
/home/trainee/linux-course/m3/data/raw/orders/orders_2026-01-15.csv
/home/trainee/linux-course/m3/docs/README.md

$ find ~/linux-course/m3 -name "*.csv"
/home/trainee/linux-course/m3/data/processed/orders/orders_clean_2026-01-15.csv
/home/trainee/linux-course/m3/data/raw/orders/orders_2026-01-15.csv
```

The first command lists every **regular file** in the tree — the `-type f` attribute test from
[§4](#4-searching-instead-of-browsing). The second finds the CSV files by name without opening a
single folder. The printed paths are full paths starting with `/home/...` because the shell expanded
`~` before `find` ran. `find` lists in the order it walks the folders, so your order may differ.

**Capture.** Paste both commands and your own results into your notes. The `find ... -type f` output
is the attribute evidence; the `find ... -name "*.csv"` output is the search evidence for the
brief's deliverable. DoD-03's count still comes from Step 4.

**Clean-up.** Nothing to clean up. The practice tree stays under `~/linux-course/` on purpose, and
nothing else in this guide leaves files behind.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Map your notes to the definition of done in [`brief.md`](./brief.md):

- [ ] DoD-01: The trainee's pasted `pwd` output shows the location before and after `cd /var/log`,
      and that `cd ~` returned to the home directory (Lab step 1). *Verify via
      [§5](#5-worked-example-the-m3-lab), Step 1.*
- [ ] DoD-02: The tree under `~/linux-course/m3` is at least three levels deep and was created with
      `mkdir -p`, without stepping into each new level one at a time. The pasted `ls -la` output shows
      it (Lab steps 2–3). *Verify via [§5](#5-worked-example-the-m3-lab), Steps 2–3; the `ls -laR`
      listing is the one that shows all three levels.*
- [ ] DoD-03: The notes record how many results `find /etc -name "*.conf"` returned, taken from the
      trainee's own output, and show that files were located by search rather than by visual browsing
      (Lab step 4). *Verify via [§5](#5-worked-example-the-m3-lab), Step 4.*
- [ ] DoD-04: `ls ../processed` ran from `~/linux-course/m3/data/raw` using a relative path, and the
      trainee can state the absolute path of `processed` without running a command to check (Lab
      step 5). *Verify via [§5](#5-worked-example-the-m3-lab), Step 5.*

When all four are ticked, transfer the evidence into your `write-up.md` and continue to M4.

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- Use `pwd` liberally when learning to navigate ([§2](#2-absolute-vs-relative-paths)).
- Prefer `mkdir -p` when creating nested paths ([§3](#3-building-structures-in-one-step)).
- Keep practice files in the home directory, not in `/etc`, `/usr`, or other system directories
  ([§1](#1-the-filesystem-hierarchy-standard-fhs)).
- Use `find` to locate files by name or attribute, and `grep` for file contents
  ([§4](#4-searching-instead-of-browsing)).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `find /etc ...` prints `Permission denied` lines | The folder is a system folder your user cannot read; the message is an error on standard error, not a result | Expected; these are system folders the lab doesn't need. Ignore them, or add `2>/dev/null`, which sends errors (file descriptor 2) to `/dev/null`, where they're discarded. |
| `mkdir ~/linux-course/m3/data/raw` prints `mkdir: No such file or directory` | The parent folders do not exist and `-p` was not used | Use `mkdir -p ~/linux-course/m3/data/raw` ([§5](#5-worked-example-the-m3-lab), Step 2) |
| `cd "~"` prints `bash: cd: ~: No such file or directory` | Quotes prevent tilde expansion, so the shell looks for a folder named `~` | Use `cd ~` or plain `cd` ([§2](#2-absolute-vs-relative-paths)) |
| `locate <name>` prints `bash: locate: command not found` | `locate` is not installed on this image | Use `find`; it needs no database ([§4](#4-searching-instead-of-browsing)) |
| `find /etc -name "*.CONF" \| wc -l` prints the `Permission denied` lines and `0` | The `-name` test is case-sensitive, and the files end in lowercase `.conf` | Use `-iname "*.CONF"` ([§4](#4-searching-instead-of-browsing)) |
| `ls -la ~/linux-course/m3` does not show the folders three levels down | `ls` without `-R` lists one level only | Add `-R` ([§5](#5-worked-example-the-m3-lab), Step 3) |
| `find` results are not in alphabetical order | `find` prints in the order it walks the folders | Expected; pipe through `sort` if you need sorted output |
| Piping `find` to `head` shows fewer `Permission denied` lines than piping to `wc -l` | `head` closes the pipe after its quota, so `find` stops early | Expected; use an unpiped run or `wc -l` for the full picture |

---

## 8. References

_Estimated: 5 minutes._

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man hier` — the Filesystem Hierarchy Standard as your distro implements it; what each top-level
  directory is for.
- `man mkdir` — creating directories, including `-p` / `--parents`. On this course image the local
  page is the uutils (`rust-mkdir`) page, not the GNU coreutils one.
- `man ls` — the listing flags used in this module: `-a`, `-l`, and `-R` for a recursive listing. On
  this course image the local page is the uutils (`rust-ls`) page, not the GNU coreutils one.
- `man find` — tests such as `-name`, `-iname`, and `-type`, and the shell-style pattern syntax.
- `man bash` — the `cd` builtin, and "Tilde Expansion" for how `~` is substituted before a command
  runs.
- `locate` — not installed on this course image and has no local man page. The guide explains it in
  [§4](#4-searching-instead-of-browsing) without running it.

Online copies of the same pages:

- [`hier(7)`](https://man7.org/linux/man-pages/man7/hier.7.html)
- [`mkdir(1)`](https://man7.org/linux/man-pages/man1/mkdir.1.html) — GNU's page; it differs from the
  local uutils page.
- [`ls(1)`](https://man7.org/linux/man-pages/man1/ls.1.html) — GNU's page; it differs from the local
  uutils page.
- [`find(1)`](https://man7.org/linux/man-pages/man1/find.1.html)
- [`bash(1)`](https://man7.org/linux/man-pages/man1/bash.1.html#EXPANSION), opened at the EXPANSION
  section, where Tilde Expansion is

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — the template your `write-up.md` is made from.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md)
  — where M3 sits in the curriculum (Layer 2, Week 2, with M4).

Curriculum resources for the Filesystem layer (M3–M4), by name:

- *Linux Filesystem Explained*
- *File System Structure Explained* (video)
- *Basic Linux Navigation and File Management*
- *Viewing and Editing Files from the Terminal*
