# Filesystem Navigation Write-up

## What I built

A project folder at `~/linux-course/m3`, laid out like a small data pipeline: `data/raw/orders` for order files as they arrive, `data/processed/orders` for cleaned copies, plus `scripts` and `docs`. I made it with `mkdir -p`, without a `cd` into each level, and added a CSV file with a header row on each side and a README in `docs`. To locate a file in it, I used `find ~/linux-course/m3 -name "*.csv"`, which found both CSV files by name. My setup is the same as in M1 and M2: WSL2 on Windows, Ubuntu 26.04.1 LTS, bash. The [observation notes](../../knowledge/observations/m03-filesystem-navigation.md) also cover the top-level folders under `/`, moving with absolute and relative paths, and searching `/etc` with `find`.

I also used EPAV to turn my notes into [`learning-materials.md`](./learning-materials.md), a study guide that teaches M3 to the next trainee.

## Why it's built this way (key decisions)

**Scheme:** the structure follows how data moves through a pipeline. Files land in `data/raw/orders` and are never edited, cleaned copies go to `data/processed/orders`, and `scripts` and `docs` are for the code and the notes about the data. Keeping raw and processed apart means a mistake while cleaning never touches the original file. The lab's `data/raw` and `data/processed` already pointed at this split. A flat dump would put raw and cleaned files in one folder, with only their names to tell them apart.

**Depth and tradeoff:** the tree is three folders deep under `m3`: `data`, then `raw` or `processed`, then `orders`. The `orders` level leaves room for other datasets later, like `data/raw/customers`, without mixing them with orders. Dates go in the file names (`orders_2026-10-20.csv`) instead of another folder level, which keeps the tree at three levels. The tradeoff is longer paths, like `~/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv`, which is where Tab completion and `find` help.

## How to build it (teach it to the next trainee)

1. Decide the scheme before creating anything. Write down what each folder is for, like raw files vs. cleaned ones.
2. Build the whole tree in one command with `mkdir -p`: `mkdir -p ~/linux-course/m3/data/raw/orders ~/linux-course/m3/data/processed/orders ~/linux-course/m3/scripts ~/linux-course/m3/docs`. `-p` makes every missing folder in each path, so there's no `cd` into each level, and running it again is safe.
3. Add files where they belong, for example with M2's redirection: `echo "order_id,amount" > ~/linux-course/m3/data/raw/orders/orders_2026-10-20.csv`.
4. Check the tree with `ls -laR ~/linux-course/m3`. Plain `ls -la` only shows the top level, so add `-R` to see every folder.
5. Find a file by name instead of browsing: `find ~/linux-course/m3 -name "*.csv"`. Quote the pattern so the shell doesn't expand the `*` before `find` sees it.
6. Inside the tree, use relative paths for short moves: from `data/raw`, `ls ../processed` reaches the folder next to it. Use an absolute path, like `/var/log`, to jump somewhere unrelated.

**To make sure you're not lost:** work out the absolute path in your head first, then confirm it with `pwd`. From `data/raw`, `processed` is `~/linux-course/m3/data/processed`.

## Concepts worth explaining

**Absolute vs. relative paths:** an absolute path starts with `/`, so it means the same place no matter where I am, like `/var/log`. A relative path starts from the folder I'm in. From `data/raw`, `..` is `data`, so `../processed` is the folder next to `raw`. `~` looks like a shortcut inside a path, but the shell turns it into my home folder, `/home/ejandrea`, before the command runs. That's why `find ~/linux-course/m3` printed full paths starting with `/home/ejandrea`.

**The FHS:** Linux has one tree that starts at `/`, with no drive letters. My `ls /` showed `/etc`, `/var`, `/home` and `/usr` at the top, and even the Windows C: drive is just a folder, `/mnt/c`. It's a convention, not a kernel rule: the kernel only follows paths, and the layout works because distros and programs agree to use it. On Ubuntu, `/bin` is only a link to `/usr/bin`, so the `/bin/ls` and `/usr/bin/ls` from M1 are the same file.

## What tripped me up

- `ls -la ~/linux-course/m3` only listed `data`, `docs` and `scripts`, so my three-level tree looked flat until I added `-R`.
- `find /etc -name "*.conf"` printed `Permission denied` lines between the results. They're folders my user can't open, and since they're errors, `wc -l` didn't count them in the 127.
- The `head -10` run only showed two of the four `Permission denied` lines, because `find` stopped once `head` had its 10 lines.
- `find` doesn't sort its results like `ls` does. It lists them in the order it walks the folders.

## Checkpoint evidence

**The structure, as a recursive listing:**
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

**The search that located a file in it:**
```text
$ find ~/linux-course/m3 -name "*.csv"
/home/ejandrea/linux-course/m3/data/processed/orders/orders_clean_2026-10-20.csv
/home/ejandrea/linux-course/m3/data/raw/orders/orders_2026-10-20.csv
```

## What I'd do differently

I'd put real rows in the CSV files from the start instead of just a header, and add a script to `scripts`, so the tree shows the raw-to-processed flow and not only the folders. I'd also run `ls -laR` right after building the tree, since plain `ls -la` hides everything below the top level.