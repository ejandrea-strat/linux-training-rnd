# Users, Permissions, Ownership Write-up

## What I built

An access-restricted shared folder, `/srv/m5-reports`, for a Q3 orders report that a group of analysts can read but not change. I own the folder and the report, and both belong to the `m5-analysts` group. The folder is 750 and the report is 640. To prove the scheme, I made two test users: `m5-analyst`, who is in the group, and `m5-outsider`, who isn't. Running as each one with `sudo -u`, the analyst could list the folder and read the report but couldn't create a file, and the outsider couldn't list or read anything. My setup is the same as in M1 to M4: WSL2 on Windows, Ubuntu 26.04.1 LTS, bash. The [lab file](../../knowledge/resources/m05-permissions-ownership-lab.md) also covers my own account and groups, `chmod 600` and `chmod u+x` on practice files, a folder with read but no execute, and what `sudo` asks for when reading `/etc/shadow`.

I also used EPAV to write [`learning-materials.md`](./learning-materials.md), a study guide that teaches M5 to the next trainee.

## Why it's built this way (key decisions)

**Scheme and identities:** The folder holds a draft report that one person writes and a team reads, so the owner gets everything, the analysts' group gets read-only access, and everyone else gets nothing. On the folder, that's 750: the group's `r-x` lets the analysts list it and go in, and leaving out `w` keeps them from creating, renaming, or deleting files there. On the report, 640 lets the group read it and nobody but the owner change it. `m5-analyst` is the allowed identity because it's in `m5-analysts`. `m5-outsider` is the restricted one because it's neither the owner nor in the group, so it gets the `---` for everyone else. These are the least permissive modes the scheme needs, and no step uses 777. The folder is in `/srv`, the FHS folder for data a system serves, because `/home/ejandrea` is already 750, so the test users couldn't have reached a folder inside it.

**Ownership vs. permission bits:** Ownership decides which set of bits applies to someone, and the bits decide what that set allows. The group ownership is what ties the group's bits to the analysts, so `chown "$USER":m5-analysts` on the folder and `chgrp m5-analysts` on the report mattered as much as the modes. They took separate commands: `chown` and `chgrp` set the owner and group, and `chmod` set the bits, without changing each other. The difference showed in who could run them. `chgrp` needed `sudo`, because the report's owner isn't in `m5-analysts`, and an owner can only give a file to a group they're in. `chmod 640` didn't, because an owner can always change a file's bits. Where the bits sit mattered too: the analyst's `touch` was stopped by the folder's group bits, and the outsider's `cat` was stopped at the folder, before the report's own bits came into play.

## How to build it (teach it to the next trainee)

1. Decide what the folder is for and who gets what. Mine holds a report I write and analysts read, so the owner gets everything, the group gets read-only, and everyone else gets nothing. Write down what each identity should be able to do before you run anything.
2. Create the identities and check them: `sudo groupadd m5-analysts`, `sudo useradd -m -s /bin/bash -G m5-analysts m5-analyst`, and `sudo useradd -m -s /bin/bash m5-outsider`. Then run `id m5-analyst`, `id m5-outsider`, and `getent group m5-analysts`. Only the allowed user should show the group.
3. Pick a place the test users can reach. Run `ls -ld ~` first. Mine is `drwxr-x---`, so other users can't get into my home, and a test folder there would fail for the wrong reason. I used `/srv/m5-reports`.
4. Set the ownership, then the bits: `sudo mkdir -p /srv/m5-reports`, `sudo chown "$USER":m5-analysts /srv/m5-reports`, then `chmod 750 /srv/m5-reports`. Once `chown` has made the folder yours, `chmod` doesn't need `sudo`.
5. Add a file and give it the group: write it with `echo "..." > /srv/m5-reports/q3-summary.txt`, then run `sudo chgrp m5-analysts /srv/m5-reports/q3-summary.txt` (with `sudo`, since you're not in the group) and `chmod 640 /srv/m5-reports/q3-summary.txt`.
6. Check the state with `ls -ld /srv/m5-reports` and `ls -l /srv/m5-reports`, and read each permission string as owner, group, and other.
7. Prove both outcomes with `sudo -u`, which runs a command as that user instead of root:
   - Allowed: `sudo -u m5-analyst ls -l /srv/m5-reports` and `sudo -u m5-analyst cat /srv/m5-reports/q3-summary.txt` should work.
   - Denied: `sudo -u m5-analyst touch /srv/m5-reports/new-file.txt`, `sudo -u m5-outsider ls /srv/m5-reports`, and `sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt` should each fail with "Permission denied".

   Paste both the successes and the failures. A scheme that's only configured isn't proven.

**If an allowed user is denied:** check the owner and group with `ls -ld` and the user's groups with `id` before changing any bits, and remember that every folder on the path needs execute for that user. Don't reach for `sudo` or 777 to make the error go away.

## Concepts worth explaining

**The identity model:** Every process runs as a user with a set of groups, and Linux checks that identity against a file's owner and group to pick one set of bits: the owner's if you own the file, the group's if you're in its group, and everyone else's otherwise. My account is `ejandrea`, UID 1000, with `sudo` among my groups, which is what lets me use `sudo`. `m5-analyst` has `m5-analysts` as an extra group, so it got the folder's group bits, `r-x`, and `m5-outsider` got the `---` for everyone else. The accounts are in `/etc/passwd`, the groups in `/etc/group`, and the password hashes in `/etc/shadow`, which only root and the `shadow` group can read.

**When elevated privilege is warranted:** `sudo` runs one command as root, after asking for my own password, not root's. Root isn't limited by read and write bits, so `sudo` would make any "Permission denied" go away, and that's exactly why it's the wrong fix for one: the wrong owner or mode would still be there. In this module, it was warranted for things the system protects: creating the users and the group, making a folder in `/srv`, giving that folder to me, and reading `/etc/shadow`. It wasn't needed for `chmod` on files and folders I own. `sudo -u` was the other useful side: it runs a command as a test user, without root's bypass, which is what made the tests real.

## What tripped me up

- With read but no execute, `ls xtest` still listed `a.txt`, but `cd xtest` was denied, even though I own the folder.
- `chgrp` needed `sudo` even though I own the report, because I'm not in `m5-analysts`.
- The `sudo` prompt, `[sudo: authenticate] Password:`, doesn't say whose password it wants. On Ubuntu 26.04, `sudo` is sudo-rs, and root's `*` in `/etc/shadow` showed it couldn't be a root password: root has none.
- `m5-analyst` got UID 1001 but GID 1002, because `groupadd` had already given 1001 to `m5-analysts`.
- None of the `sudo` commands in the shared-folder setup asked for my password again, because `sudo` still had it from the step before.

## Checkpoint evidence

**The identities:**
```text
$ id m5-analyst
uid=1001(m5-analyst) gid=1002(m5-analyst) groups=1002(m5-analyst),1001(m5-analysts)
$ id m5-outsider
uid=1002(m5-outsider) gid=1003(m5-outsider) groups=1003(m5-outsider)
$ getent group m5-analysts
m5-analysts:x:1001:m5-analyst
```

**The folder's ownership and permissions:**
```text
$ ls -ld /srv/m5-reports
drwxr-x--- 2 ejandrea m5-analysts 4096 Oct 21 21:55 /srv/m5-reports
$ ls -l /srv/m5-reports
total 4
-rw-r----- 1 ejandrea m5-analysts 42 Oct 21 21:55 q3-summary.txt
```

**The authorized attempt, as `m5-analyst`:**
```text
$ sudo -u m5-analyst ls -l /srv/m5-reports
total 4
-rw-r----- 1 ejandrea m5-analysts 42 Oct 21 21:55 q3-summary.txt
$ sudo -u m5-analyst cat /srv/m5-reports/q3-summary.txt
Q3 orders summary: draft for the analysts
```

**The denied attempts, the analyst writing and the outsider reading:**
```text
$ sudo -u m5-analyst touch /srv/m5-reports/new-file.txt
touch: cannot touch '/srv/m5-reports/new-file.txt': Permission denied
$ sudo -u m5-outsider ls /srv/m5-reports
ls: cannot open directory '/srv/m5-reports': Permission denied
$ sudo -u m5-outsider cat /srv/m5-reports/q3-summary.txt
cat: /srv/m5-reports/q3-summary.txt: Permission denied
```

## What I'd do differently

I'd drop the `sudo` from `chmod 750`. After `chown`, the folder was mine, so a plain `chmod` would have worked, and this module is about using `sudo` only where it's needed. I'd also set the setgid bit on the folder with `sudo chmod g+s /srv/m5-reports`, which makes new files inside it take the folder's group, so a new report wouldn't need `sudo chgrp` every time. That one needs `sudo`: because I'm not in `m5-analysts`, a plain `chmod g+s` leaves the bit off without any error.