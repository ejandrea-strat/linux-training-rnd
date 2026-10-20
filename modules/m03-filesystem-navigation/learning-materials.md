# M3 Learning Materials — Filesystem Navigation

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals, working in the course's Linux
> environment (WSL2, a virtual machine, or Linux in a container). You finished M1 and M2, so the
> shell, the terminal, `ls`, and pipes are familiar; this module builds on them.
>
> **Scope note.** Every example output below is **illustrative** unless it is explicitly labeled as
> coming from the author's observation notes
> ([`knowledge/observations/m03-filesystem-navigation.md`](../../knowledge/observations/m03-filesystem-navigation.md)).
> The labeled outputs were captured on WSL2 on a specific machine; values such as your username,
> home directory, file counts, and dates will differ on yours. Never copy an example as evidence —
> run the command and paste your own output.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes roughly **70–85 minutes**. M3 shares
> Week 2 of the course with M4 (5–6 estimated hours in total).

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
that root. There are no drive letters. On WSL2 the Windows drives appear as folders inside the
tree: `C:` is mounted at `/mnt/c`.

The **Filesystem Hierarchy Standard (FHS)** is a shared agreement about *what lives where* in that
tree. The directories this module cares about:

| Directory | What it holds | Examples |
| --- | --- | --- |
| `/etc` | System-wide configuration files, mostly plain text | `/etc/os-release`, `/etc/wsl.conf` |
| `/var` | Data that changes while the system runs — logs, caches | `/var/log` |
| `/home` | One folder per user, your personal files live here | `/home/<username>`; root's own home is `/root` instead |
| `/usr` | Installed programs and their files: commands in `/usr/bin`, libraries in `/usr/lib`, man pages under `/usr/share` | `/usr/bin/ls` |

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

Output from the author's observation notes:

```text
$ ls /
bin   dev  home  lib    lost+found  mnt  proc  run   snap  sys  usr
boot  etc  init  lib64  media       opt  root  sbin  srv   tmp  var

$ ls -ld /bin
lrwxrwxrwx 1 root root 7 Apr 20  2026 /bin -> usr/bin

$ ls /mnt
c  wsl  wslg
```

Read the listing: `/etc`, `/var`, `/home`, and `/usr` are all there, plus `/root` (root's home),
and `/proc` and `/dev` from M1. In `ls -ld /bin`, the `l` at the start of the mode means *link*,
and `-> usr/bin` shows where it points. On WSL2, `/mnt/c` is the Windows C: drive — Windows drives
show up as folders inside the tree, not as letters.

> **Use your own machine.** Your top-level listing may differ slightly (for example, `/snap` on
> Ubuntu, or no `/mnt/wsl` outside WSL2). That is expected; the four FHS directories above are
> what matters.

**Verify.** `ls /` shows a single tree with `/etc`, `/var`, `/home`, and `/usr` in it;
`ls -ld /bin` either shows a link (`l...` and `->`) or a plain directory, depending on the distro.

**Continue** to [§2](#2-absolute-vs-relative-paths) to learn how paths name a place in that tree —
and how to tell whether a path is anchored or relative.

---

## 2. Absolute vs. relative paths

_Estimated: 12 minutes._

**Understand.** A path names a location in the tree. Two kinds:

| Kind | Starts with | Means | Example |
| --- | --- | --- | --- |
| **Absolute path** | `/` | The same place from anywhere | `/var/log` |
| **Relative path** | anything else | Starts from where you are now (`$PWD`) | `data/raw`, `../processed` |

Knowing which kind you are typing is what prevents getting lost. An absolute path is the right
choice for jumping somewhere unrelated (`/var/log`); a relative path is shorter when the target is
close by.

Three special names worth knowing:

- **`.`** — the folder you are in right now.
- **`..`** — the folder one level up.
- **`~`** — your home folder (`$HOME`). `~` is not part of a relative path: the shell swaps it in
  *before* the command runs (this is *tilde expansion*, described under "Tilde Expansion" in
  `man bash`), turning `~/linux-course` into the absolute path `/home/<your username>/linux-course`.
  Because the shell does the swap, `~` does not work inside quotes: `cd "~"` looks for a folder
  literally named `~`. Running `cd` with no argument also goes home.

Every folder contains `.` and `..`, which is why `ls -a` (M2) always shows them.

**Try.** From your home directory, move to the system log folder and back, confirming each location:

```bash
cd ~
pwd
cd /var/log
pwd
cd ~
pwd
```

Output from the author's observation notes:

```text
$ cd ~
$ pwd
/home/ejandrea
$ cd /var/log
$ pwd
/var/log
$ cd ~
$ pwd
/home/ejandrea
```

> **Use your own values.** The author's home is `/home/ejandrea`. Yours is `/home/<your
> username>` — `pwd` after `cd ~` will print it. Substitute your own path everywhere.

**Verify.** Your first and third `pwd` print the same home path; the second prints `/var/log`.

**Continue** to [§3](#3-building-structures-in-one-step) to create nested folders of your own — in
a single command.

---

## 3. Building structures in one step

_Estimated: 10 minutes._

**Understand.** `mkdir` creates a folder. To create a *nested* path like `a/b/c`, the parent
folders must exist. The option that creates every missing folder along the way is **`-p`**
(`--parents`):

```bash
mkdir -p a/b/c      # creates a, a/b, and a/b/c in one command
```

Two properties of `-p` worth knowing (see `man mkdir`):

- **Without `-p`**, `mkdir a/b/c` fails with "No such file or directory" if `a/b` does not exist
  yet, and you would have to create each level one at a time.
- **With `-p`**, an already-existing folder is *not* an error, so running the same command twice is
  safe. One `mkdir -p` can also take several paths at once.

**Try.** Build a nested practice tree under your home directory:

```bash
mkdir -p ~/linux-course/m3/data/raw
mkdir -p ~/linux-course/m3/data/processed
```

Both commands print nothing when they work — `mkdir -p` is silent on success. The first command
made every missing folder in `m3/data/raw` in one go, without a single `cd`.

The author went one step further, matching the brief's deliverable (a structure that is
*populated*, not an empty shell) by adding sibling folders and a couple of sample files:

```bash
mkdir -p ~/linux-course/m3/data/raw/orders ~/linux-course/m3/data/processed/orders ~/linux-course/m3/scripts ~/linux-course/m3/docs
echo "order_id,amount" > ~/linux-course/m3/data/raw/orders/orders_2026-10-20.csv
echo "order_id,amount" > ~/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv
echo "Raw order files land in data/raw/orders." > ~/linux-course/m3/docs/README.md
```

(The `echo ... > file` pattern is M2's redirection; it writes a small file in one step.)

The author's scheme follows how data moves: raw files land in `data/raw`, cleaned copies go to
`data/processed`, and `scripts` and `docs` are for the code and the notes about the data. Design your own scheme
before you build — a deliberate organization is the point, not a flat dump.

**Verify.** `ls ~/linux-course/m3` shows `data`, plus whatever siblings you chose. Re-running any
of the `mkdir -p` commands succeeds silently instead of complaining.

**Continue** to [§4](#4-searching-instead-of-browsing) to find files in a tree without walking it
by hand.

---

## 4. Searching instead of browsing

_Estimated: 13 minutes._

**Understand.** When you do not know a file's exact location, search instead of clicking through
folders. Two tools, two different engines:

| Tool | How it searches | Trade-off |
| --- | --- | --- |
| `find` | Walks the folders **live**, every time it runs | Always up to date, slower on big trees |
| `locate` | Queries a **saved database** of file names, updated about once a day | Much faster, but can miss files created since the last update; it might not be installed at all |

`find` takes a starting point and tests, for example a name pattern:

```bash
find /etc -name "*.conf"
```

`-name` matches against the file's **base name** with shell-style patterns: `*` means any
characters, `?` means exactly one character. Unlike `grep`'s regular expressions (M2), the `.` here
is a **plain dot** — a name like `dconf` does not match `"*.conf"`; the pattern matches names
ending in the literal characters `.conf`. Use `-iname` if you want upper/lower case ignored.

One trap: **quote the pattern** (`"*.conf"`). Quotes stop the *shell* from expanding the `*`
before `find` ever sees it. Full details in `man find`.

**Try.**

```bash
find /etc -name "*.conf" | head -10     # a sample of the results
find /etc -name "*.conf" | wc -l        # how many, in total
```

Output from the author's observation notes:

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

$ find /etc -name "*.conf" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
127
```

Several things to notice:

- **127 entries** — the author's count from their own machine. Yours will differ; run the command
  and record your own number.
- `find` searches **subfolders**, like `/etc/ssh/ssh_config.d/` and `/etc/rsyslog.d/`. M2's
  `ls -l /etc | grep ".conf"` only ever listed the top level, so it found far fewer (31 in the
  author's M2 run).
- The results are **not alphabetical**: `find` lists files in the order it walks the folders,
  unlike the sorted output of `ls`.
- The `Permission denied` lines are folders the author's user cannot open. They are **errors**
  (standard error), so they appeared on screen but were not counted in the 127.
- The `head -10` run showed only two of the four errors because `find` stopped once `head` had its
  ten lines.

**Verify.** Both commands run on your machine; `wc -l` prints a number; at least one result path
contains a subfolder of `/etc`, proving the search went deeper than the top level.

**Continue** to [§5](#5-worked-example-the-m3-lab) to produce the lab evidence.

---

## 5. Worked example: the M3 lab

> **Purpose.** The five lab steps from [`brief.md`](./brief.md), with the exact evidence to
> capture. The outputs shown are the author's; yours will differ in username, home path, and
> counts. Record your own.

_Estimated: 20–25 minutes._

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

Output from the author's observation notes:

```text
$ cd ~
$ pwd
/home/ejandrea
$ cd /var/log
$ pwd
/var/log
$ cd ~
$ pwd
/home/ejandrea
```

**Capture.** Paste all three `pwd` outputs into your notes. They show the starting location
(`~` → the author's `/home/ejandrea`), the move to `/var/log`, and the return home. Use *your*
home path, not the author's. *Verify via [§2](#2-absolute-vs-relative-paths).*

### Step 2 — Build the nested tree with `mkdir -p`

**Understand.** The lab's minimum is two `mkdir -p` commands — `~/linux-course/m3/data/raw` and
`~/linux-course/m3/data/processed` — creating the tree in one step, without stepping into each
level. The author added an `orders` level inside both, so the tree is three folders deep under
`m3` (`data`, then `raw` or `processed`, then `orders`), which meets DoD-02's three-level
requirement.

**Try.**

```bash
mkdir -p ~/linux-course/m3/data/raw
mkdir -p ~/linux-course/m3/data/processed
```

Silent output — success. The author also populated the tree (extra folders plus three small files
via `echo >`), which the brief's deliverable asks for. See [§3](#3-building-structures-in-one-step)
for those commands and the organizational scheme behind them.

**Capture.** Record the `mkdir -p` commands you ran. No output to paste — the proof is the
listing in step 3. *Verify via [§3](#3-building-structures-in-one-step).*

### Step 3 — List the tree, including hidden files

**Understand.** `ls -la` lists everything in a folder, including entries starting with `.`.

**Try.**

```bash
ls -la ~/linux-course/m3
```

Output from the author's observation notes:

```text
$ ls -la ~/linux-course/m3
total 20
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 data
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 docs
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 scripts
```

`ls -la` shows one level: `data`, `docs`, `scripts`, plus `.` (`m3` itself) and `..`
(`linux-course`, the folder above). It does **not** show what is inside `data`. To see the whole
tree in one listing, add `-R` (recursive):

```bash
ls -laR ~/linux-course/m3
```

Output from the author's observation notes:

```text
$ ls -laR ~/linux-course/m3
/home/ejandrea/linux-course/m3:
total 20
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 data
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 docs
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 scripts

/home/ejandrea/linux-course/m3/data:
total 16
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 processed
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 raw

/home/ejandrea/linux-course/m3/data/processed:
total 12
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 orders

/home/ejandrea/linux-course/m3/data/processed/orders:
total 12
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 ..
-rw-r--r-- 1 ejandrea ejandrea   16 Oct 20 16:27 orders_clean_2026-10-20.csv

/home/ejandrea/linux-course/m3/data/raw:
total 12
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 orders

/home/ejandrea/linux-course/m3/data/raw/orders:
total 12
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 3 ejandrea ejandrea 4096 Oct 20 16:27 ..
-rw-r--r-- 1 ejandrea ejandrea   16 Oct 20 16:27 orders_2026-10-20.csv

/home/ejandrea/linux-course/m3/docs:
total 12
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 ..
-rw-r--r-- 1 ejandrea ejandrea   41 Oct 20 16:27 README.md

/home/ejandrea/linux-course/m3/scripts:
total 8
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 ..
```

Every folder in the listing has both `.` and `..` — and `..` always points one level up: in
`data/raw/orders`, `..` is `raw`. That is the structure proving three levels of depth (`data`,
then `raw` or `processed`, then `orders`) with a CSV file at the bottom of each side.

**Capture.** Paste **both** listings into your notes: `ls -la ~/linux-course/m3`, because that is
the command the lab asks for, and `ls -laR ~/linux-course/m3`, because plain `ls -la` shows only
one level and cannot show the three levels of depth that DoD-02 requires — the recursive listing
is the one that shows them. *Verify via [§3](#3-building-structures-in-one-step).*

### Step 4 — Search `/etc` for `.conf` files

**Understand.** This step proves search beats browsing: `ls -l /etc | grep ".conf"` (M2) only sees
the top level; `find` sees the whole subtree.

**Try.**

```bash
find /etc -name "*.conf" | wc -l
```

Output from the author's observation notes:

```text
$ find /etc -name "*.conf" | wc -l
find: ‘/etc/credstore.encrypted’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/ssl/private’: Permission denied
find: ‘/etc/credstore’: Permission denied
127
```

**Capture.** Record **how many results your own machine returned** (127 on the author's WSL2
system — yours will differ), taken from your own `wc -l` output, along with the command itself.
The `Permission denied` lines are errors on standard error; they are not part of the count.
*Verify via [§4](#4-searching-instead-of-browsing).*

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

Output from the author's observation notes:

```text
$ cd ~/linux-course/m3/data/raw
$ ls ../processed
orders
$ cd ../processed
$ pwd
/home/ejandrea/linux-course/m3/data/processed
```

**Capture.** Paste the `ls ../processed` command and its output. Then, **without running a
command**, state the absolute path of `processed`: `~` is `/home/<your username>`, and `processed`
sits at `linux-course/m3/data/processed` under it, so the full path is
`/home/<your username>/linux-course/m3/data/processed`. The author worked it out the same way, and
the `pwd` at the end of the block confirmed it: `/home/ejandrea/linux-course/m3/data/processed`.
*Verify via [§2](#2-absolute-vs-relative-paths).*

### Deliverable evidence — find a file in your tree by search

**Understand.** The brief's deliverable asks for proof that a file *in your own structure* was
located with a search tool, not by visual browsing. Step 4 searched a system folder; this closes
the loop on your tree.

**Try.**

```bash
find ~/linux-course/m3 -name "*.csv"
```

Output from the author's observation notes:

```text
$ find ~/linux-course/m3 -name "*.csv"
/home/ejandrea/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv
/home/ejandrea/linux-course/m3/data/raw/orders/orders_2026-10-20.csv
```

Both CSV files were found by name without opening a single folder. The printed paths are full
paths starting with `/home/...` because the shell expanded `~` before `find` ran.

**Capture.** Paste the command and your own results into your notes. This is the search evidence
for the brief's deliverable only — DoD-03's count comes from Step 4.

**Clean-up.** Nothing to clean up. The practice tree stays under `~/linux-course/`.

---

## 6. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] **DoD-01 — `pwd` before and after the move.** Your pasted `pwd` output shows the location
      before and after `cd /var/log`, and that `cd ~` returned to the home directory. *Verify via
      [§5](#5-worked-example-the-m3-lab), Step 1.*
- [ ] **DoD-02 — A three-level tree from `mkdir -p`.** The tree under `~/linux-course/m3` is at
      least three levels deep and was created with `mkdir -p`, without stepping into each new
      level one at a time. Your pasted `ls -la` output shows it (the `ls -laR` listing from
      [§5](#5-worked-example-the-m3-lab) Step 3 is the one that shows all three levels). *Verify
      via [§5](#5-worked-example-the-m3-lab), Steps 2–3.*
- [ ] **DoD-03 — A count from your own `find`.** Your notes record how many results
      `find /etc -name "*.conf"` returned, taken from your own output, and show that files were
      located by search rather than by visual browsing. *Verify via
      [§5](#5-worked-example-the-m3-lab), Step 4.*
- [ ] **DoD-04 — A relative path, and the absolute path by inspection.** `ls ../processed` ran
      from `~/linux-course/m3/data/raw` using a relative path, and you can state the absolute
      path of `processed` without running a command to check. *Verify via
      [§5](#5-worked-example-the-m3-lab), Step 5.*

When all four are ticked, transfer the evidence into your write-up
([`write-up-template.md`](./write-up-template.md)) and continue to M4. The brief's deliverable
also needs the search that found a file in your own tree — see the Deliverable evidence part of
[§5](#5-worked-example-the-m3-lab).

---

## 7. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Use `pwd` liberally when learning to navigate.** Confirm after every `cd` until relative
  paths feel automatic ([§2](#2-absolute-vs-relative-paths)).
- **Prefer `mkdir -p` when creating nested paths.** One command, no stepping, safe to re-run
  ([§3](#3-building-structures-in-one-step)).
- **Keep practice files in the home directory**, not in `/etc`, `/usr`, or other system
  directories. System directories belong to the distro; your exercises belong under `~`
  ([§1](#1-the-filesystem-hierarchy-standard-fhs)).
- **Use `find` to locate files by name or attribute, and `grep` for file contents.** Different
  tools, different jobs ([§4](#4-searching-instead-of-browsing)).

### Troubleshooting

| Symptom | Likely cause | Recommended fix |
| --- | --- | --- |
| `find` prints `Permission denied` lines mixed with results | Some folders under the search path belong to other users or root, and your user cannot open them. The messages are errors on standard error, not results ([§4](#4-searching-instead-of-browsing)). | Expected on `/etc`. Ignore the lines, or filter them out (`2>/dev/null`). When piping to `wc -l`, remember errors are not counted. |
| Piping `find` to `head` shows fewer error lines than piping to `wc -l` | `head` stops reading after its quota of lines, so `find` exits early and never reaches the folders that would have produced the remaining errors ([§4](#4-searching-instead-of-browsing)). | Expected behavior. Use an unpiped `find` (or `wc -l`) when you want the complete picture. |
| `ls -la ~/linux-course/m3` does not show the folders three levels down | `ls` without `-R` lists **one level** only ([§5](#5-worked-example-the-m3-lab), Step 3). | Add `-R`: `ls -laR ~/linux-course/m3` lists every folder recursively. |
| `find` results are not in alphabetical order | `find` prints in the order it walks the folders, not sorted like `ls` ([§4](#4-searching-instead-of-browsing)). | Expected. Pipe through `sort` if you need sorted output: `find /etc -name "*.conf" \| sort`. |

---

## 8. References

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man hier` — the Filesystem Hierarchy Standard as your distro implements it; what each
  top-level directory is for.
- `man mkdir` — creating directories, including `-p` / `--parents`.
- `man find` — tests such as `-name` and `-iname`, and the shell-style pattern syntax.
- `man locate` — the file-name database search. Note: `locate` **may not be installed** on your
  system (the lab uses `find`, which needs no database).
- `man bash` — the `cd` builtin, and "Tilde Expansion" for how `~` is substituted before a
  command runs.

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — where the lab evidence goes.
- [`knowledge/observations/m03-filesystem-navigation.md`](../../knowledge/observations/m03-filesystem-navigation.md)
  — the author's captured output, used as the labeled examples in this guide.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) —
  where M3 sits in the curriculum (Filesystem layer, Week 2, with M4).

Curriculum resources for the Filesystem layer (M3–M4), by name:

- *Linux Filesystem Explained*
- *File System Structure Explained* (video)
- *Basic Linux Navigation and File Management*
- *Viewing and Editing Files from the Terminal*
