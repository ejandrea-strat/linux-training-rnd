# M3 Lab — Filesystem Navigation

What I learned from researching M3, then the lab as I ran it on WSL2 with Ubuntu and bash: each command, its real output copied exactly, and what it shows.

## What I learned from research

1. **The FHS:**
   - Linux has one tree that starts at `/`, with no drive letters. Other disks show up as folders inside it, like the Windows C: drive at `/mnt/c` on WSL2.
   - `/etc` holds system-wide config files, mostly plain text (e.g. `/etc/os-release`, `/etc/wsl.conf`).
   - `/var` holds data that changes while the system runs, like logs in `/var/log` and caches.
   - `/home` has one folder per user, like `/home/<username>`. Root's home is `/root` instead.
   - `/usr` holds installed programs and their files: commands in `/usr/bin`, libraries in `/usr/lib`, and man pages under `/usr/share`. On Ubuntu, `/bin` is a link to `/usr/bin`, so the `/usr/bin/ls` and `/bin/ls` in M1's `type -a ls` are the same file.
   - It's a convention because the kernel only follows paths and doesn't check what a folder is for. The layout works because distros and programs agree to use it. `man hier` describes it.

2. **Absolute vs. relative paths:**
   - An absolute path starts with `/` and means the same place from anywhere (`/var/log`). A relative path starts from the current folder, so `data/raw` depends on where `pwd` says I am.
   - `.` is the current folder and `..` is the folder one level up. Every folder has both, which is why `ls -a` shows them.
   - `~` is my home folder (`$HOME`). The shell swaps it in before the command runs, so it doesn't work inside quotes: `cd "~"` fails. `cd` alone also goes home. This is under "Tilde Expansion" in `man bash`.

3. **Building structures in one step:**
   - `mkdir -p` (`--parents`) makes every missing folder in the path, so `mkdir -p a/b/c` creates all three levels in one command.
   - Without `-p`, `mkdir a/b/c` fails with "No such file or directory" if `a/b` doesn't exist yet.
   - With `-p`, a folder that already exists isn't an error, so running the same command twice is safe. One `mkdir -p` can also take several paths. See `man mkdir`.

4. **Searching instead of browsing:**
   - `find` searches the folders live every time it runs, starting where I tell it: `find /etc -name "*.conf"`. It's always up to date but slower on big trees.
   - `locate` searches a saved database of file names that gets updated about once a day, so it's faster but can miss new files. It might not be installed.
   - `-name` takes a shell-style pattern: `*` means any characters and `?` means one character. Unlike grep in M2, `.` is a plain dot here. `-iname` ignores upper and lower case.
   - Quote the pattern (`"*.conf"`) so the shell doesn't expand the `*` before `find` sees it. See `man find`.

## Top-level folders

```text
$ ls /
bin   dev  home  lib    lost+found  mnt  proc  run   snap  sys  usr
boot  etc  init  lib64  media       opt  root  sbin  srv   tmp  var

$ ls -ld /bin
lrwxrwxrwx 1 root root 7 Apr 20  2026 /bin -> usr/bin

$ ls /mnt
c  wsl  wslg
```

**Observation:** Shows the top of the one tree that starts at `/`. The four folders from my research are all here: `/etc`, `/var`, `/home` and `/usr`, plus `/root`, root's own home, and `/proc` and `/dev` from M1. `/bin` isn't a separate folder: `ls -ld` shows it's a link (the `l` at the start and `-> usr/bin`), so the `/bin/ls` and `/usr/bin/ls` from M1's `type -a ls` are the same file. `/mnt` has `c`, the Windows C: drive, so Windows drives show up as folders inside the tree instead of drive letters. Some entries are only here because of WSL: `/init` is the WSL init from M1, and `/mnt/wsl` is where M2's `resolv.conf` link pointed.

## Moving around

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

**Observation:** Shows that I started in my home folder, `/home/ejandrea`. `cd /var/log` moved me to the system's log folder, and `cd ~` brought me back to `/home/ejandrea`. `pwd` confirmed each move.

**Expected vs actual:** I expected `~` to expand to `/home/<my username>`, and it did: `cd ~` put me back in `/home/ejandrea`.

## Building the tree

```text
$ mkdir -p ~/linux-course/m3/data/raw
$ mkdir -p ~/linux-course/m3/data/processed
$ mkdir -p ~/linux-course/m3/data/raw/orders ~/linux-course/m3/data/processed/orders ~/linux-course/m3/scripts ~/linux-course/m3/docs
$ echo "order_id,amount" > ~/linux-course/m3/data/raw/orders/orders_2026-10-20.csv
$ echo "order_id,amount" > ~/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv
$ echo "Raw order files land in data/raw/orders." > ~/linux-course/m3/docs/README.md
```

**Observation:** Shows that none of these commands printed anything, because `mkdir -p` and a redirected `echo` are silent when they work. The first command made every missing folder in `m3/data/raw` in one go, without a `cd` into each level. The third `mkdir -p` made four folders at once. My scheme follows how data moves: files land in `data/raw`, cleaned copies go to `data/processed`, and `scripts` and `docs` are for the code and the notes about the data.

## Listing the tree

```text
$ ls -la ~/linux-course/m3
total 20
drwxr-xr-x 5 ejandrea ejandrea 4096 Oct 20 16:27 .
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 ..
drwxr-xr-x 4 ejandrea ejandrea 4096 Oct 20 16:27 data
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 docs
drwxr-xr-x 2 ejandrea ejandrea 4096 Oct 20 16:27 scripts

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

**Observation:** Shows that `ls -la` only lists one level: `data`, `docs` and `scripts`, plus `.` (`m3` itself) and `..` (`linux-course`). `ls -laR` goes into every folder, so it shows the whole tree, three folders deep under `m3` (`data`, then `raw` or `processed`, then `orders`), with a CSV file at the bottom of each side.

**Expected vs actual:** I expected `.` to be the current folder and `..` the folder above it. That matches: every folder in the `ls -laR` listing has both, and in `data/raw/orders`, `..` is `raw`.

## Searching /etc

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

**Observation:** Shows that `find` returned 127 entries ending in `.conf` in `/etc`, far more than the 31 lines from M2's `ls -l /etc | grep ".conf"`. `find` also searches subfolders, like `/etc/ssh/ssh_config.d/` and `/etc/rsyslog.d/`, while `ls -l /etc` only listed the top level. The results aren't in alphabetical order, because `find` lists files in the order it walks the folders instead of sorting them like `ls`. The `Permission denied` lines are folders my user can't open. They're errors (fd 2), so they showed on the screen but weren't counted in the 127. The `head -10` run only printed two of the four errors, since `find` stopped once `head` had its 10 lines.

## Finding a file in my tree

```text
$ find ~/linux-course/m3 -name "*.csv"
/home/ejandrea/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv
/home/ejandrea/linux-course/m3/data/raw/orders/orders_2026-10-20.csv
```

**Observation:** Shows that `find` located both CSV files in my tree by name, without browsing into any folder. It printed full paths starting with `/home/ejandrea`, because the shell expanded `~` before `find` ran.

## Relative paths

```text
$ cd ~/linux-course/m3/data/raw
$ ls ../processed
orders
$ cd ../processed
$ pwd
/home/ejandrea/linux-course/m3/data/processed
```

The absolute path of `processed`, worked out without running a command: `~` is `/home/ejandrea`, and `processed` is at `linux-course/m3/data/processed` under it, so the full path is `/home/ejandrea/linux-course/m3/data/processed`. The `pwd` at the end of the block matched it.

**Observation:** Shows that from `data/raw`, `..` pointed to `data`, so `../processed` reached the `processed` folder next to `raw` and listed its `orders` folder. `cd ../processed` and `pwd` confirmed the absolute path: `/home/ejandrea/linux-course/m3/data/processed`. An absolute path is better for jumping somewhere unrelated, like `/var/log`. A relative path is shorter when the target is close, like `../processed` next to `raw`.
