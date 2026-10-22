# M6 Learning Materials — Package Management

> **Purpose.** A self-contained study guide for everything in [`brief.md`](./brief.md). It teaches the material but does **not** replace the lab. Work the lab on your own machine and record what really happened in your own `write-up.md`, made from [`write-up-template.md`](./write-up-template.md).  
> **Audience.** Data engineers who are new to Linux internals, working in the course's Linux environment (WSL2, a virtual machine, or Linux in a container).  
> **Scope note.** Every command output in this guide is **illustrative** — it shows the shape of what a command prints, not a record of a completed lab. Values such as your username, host, dates, and versions differ on your machine. Run each command and paste your own output before you use any example as evidence.  
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**: read the explanation, run the commands on your own system, confirm your output matches the *shape* of the examples, then move on. The guided path takes **90 minutes** total — 15 + 15 + 10 for the three concept sections, 35 for the worked example, and 5 each for the completion checklist, the best practices and troubleshooting, and the references. M6 is **Week 3** in **Layer 3 (Access & Software)**, shared with M5. Commands assume **bash** on **WSL2 Ubuntu 26.04**.  
> **Prerequisites.** M5 (Users, Permissions, Ownership): you need to know why elevated privilege is sometimes needed and how to read ownership/permissions. M2–M3 (CLI Basics, Filesystem Navigation): you need to run commands, read output, and move around directories.

## Contents

1. [System-level package management](#1-system-level-package-management)
2. [Language-level package management](#2-language-level-package-management)
3. [Verifying an install](#3-verifying-an-install)
4. [Worked example: the lab](#4-worked-example-the-lab)
5. [Completion checklist](#5-completion-checklist)
6. [Best practices and troubleshooting](#6-best-practices-and-troubleshooting)
7. [References](#7-references)

---

## 1. System-level package management

_Estimated: 15 minutes._

### Understand.

A **package** is a bundled piece of software plus metadata (version, description, dependencies). A **package manager** installs, updates, tracks, and removes packages in a controlled way. On Ubuntu, the system package manager is `apt` (front-end) working with `dpkg` (the low-level installer).

**Research answers (system level):**
- **What a repository is:** A repository is a server that holds packages and a signed index of what's available. On Ubuntu 26.04, `apt` reads repository definitions from `/etc/apt/sources.list.d/ubuntu.sources` (deb822 style in this release). Each entry gives the `URIs`, `Suites` (like `resolute`, `resolute-updates`, `resolute-security`), `Components` (`main`, `universe`, `restricted`, `multiverse`), and `Signed-By` (the key used to verify the index).
- **What dependency resolution does:** `apt` figures out the full set of packages needed (direct and indirect), checks what is already installed, and installs what's missing. Some dependencies are marked as automatically installed; `apt autoremove` can remove them later if nothing depends on them.
- **What installing system-wide software requires from M5's permission model:** Installing system-wide writes to system directories (like `/usr/`, `/usr/lib/jvm/`) and uses the dpkg lock under `/var/lib/dpkg/`. That is why installing needs sudo. See [M5 §4](../m05-permissions-ownership/learning-materials.md#4-elevated-privilege) for when elevated privilege is warranted. Use `sudo` only where it's needed (for example `sudo apt update` and `sudo apt install`).

`sudo apt update` downloads the latest index from each configured repository. It does **not** install or upgrade packages. After updating, `apt` can pick the correct version from that index. `sudo apt install <package>` then resolves and installs the package plus anything it depends on.

### Try.

Look at what `apt` would do without changing anything. These commands read only.

```bash
cat /etc/apt/sources.list.d/ubuntu.sources
apt policy default-jdk
apt show default-jdk
```

*Illustrative example — not captured output:*

```text
## Ubuntu distribution repository
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: resolute resolute-updates resolute-backports
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

## Ubuntu security updates.
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: resolute-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

```text
default-jdk:
  Installed: (none)
  Candidate: <version>
  Version table:
     <version> 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
```

```text
Package: default-jdk
Version: <version>
Priority: optional
Section: devel
Source: java-common
Depends: default-jre (= <version>), default-jdk-headless (= <version>), openjdk-25-jdk
Description: Standard Java or Java compatible Development Kit
```

Now see what happens if you try to install without `sudo`:

```bash
apt install default-jdk
```

*Illustrative example — not captured output:*

```text
Error: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
Error: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

The actual install (`sudo apt update`, `sudo apt install default-jdk`) is done in [§4](#4-worked-example-the-lab), Step 1.

### Verify.

- A repository gives packages and a signed index; `apt update` refreshes that index without installing.
- `default-jdk` is small because it only depends on the real JDK packages (its Depends line in `apt show`). Step 1 in [§4](#4-worked-example-the-lab) shows the full set `apt` resolves.
- Without `sudo`, `apt install` cannot acquire the dpkg lock.
- Installing needs sudo because it changes the system. See [M5 §4](../m05-permissions-ownership/learning-materials.md#4-elevated-privilege).

### Continue.

Move to [§2](#2-language-level-package-management) to see how language package managers differ and why the system Python is treated differently.

---

## 2. Language-level package management

_Estimated: 15 minutes._

### Understand.

Language ecosystems often have their own package managers (for example `pip` for Python). These install packages from their own index (like PyPI) into a specific Python environment, not into Ubuntu's system package directories managed by `apt`. `apt` knows nothing about them and they know nothing about `apt`.

**Research answers (language level):**
- **How it differs from the system manager:** The system manager (`apt`) manages system software from Ubuntu repositories into system directories and tracks installs with `dpkg`. A language manager (`pip`) manages libraries from a language index (like PyPI) into a specific Python environment (`site-packages`). They are separate and do not share tracking.
- **Why installing a library globally into the system's Python can cause problems:** Ubuntu's system Python is used by system tools. On Ubuntu 26.04, `/usr/lib/python3.14/EXTERNALLY-MANAGED` is present (PEP 668). If `pip` could replace packages in the system site-packages, it could break those tools. For that reason, installing into the system Python is discouraged and `pip` will refuse in that context; the marker points to using a virtual environment instead.

A **virtual environment** (`python3 -m venv`) creates a folder (for example `.venv/`) with its own `python`, its own `pip`, and its own `site-packages`. When activated with `source .venv/bin/activate`, that environment's `bin` directory comes first on `PATH`, so `python` and `pip` run from it. Packages go there and belong to that environment; no `sudo` is needed. Installing `python3-venv` with `apt` provides the venv module on this system.

**Note about the brief:** the brief shows `pip install requests` directly. On this environment, `pip` is not installed by default and the system Python is externally managed, so that command will not run as written. The correct fix is to create and use a virtual environment.

### Try.

Check the environment first:

```bash
python3 --version
pip --version
ls /usr/lib/python3*/EXTERNALLY-MANAGED
```

*Illustrative example — not captured output:*

```text
Python <version>
Command 'pip' not found, but can be installed with:
sudo apt install python3-pip
/usr/lib/python3.14/EXTERNALLY-MANAGED
```

See the marker content:

```bash
cat /usr/lib/python3.14/EXTERNALLY-MANAGED
```

*Illustrative example — not captured output (trimmed):*

```text
[externally-managed]
Error=To install Python packages system-wide, try apt install
 python3-xyz, where xyz is the package you are trying to
 install.

 If you wish to install a non-Debian-packaged Python package,
 create a virtual environment using python3 -m venv path/to/venv.
 Then use path/to/venv/bin/python and path/to/venv/bin/pip.
```

The virtual environment setup and installing `requests` are done in [§4](#4-worked-example-the-lab), Step 4.

### Verify.

- `pip` installs from PyPI into the active Python environment, not system directories managed by `apt`.
- The EXTERNALLY-MANAGED marker + missing `pip` mean the brief's direct command won't run; use a virtual environment.
- `pip` resolves dependencies too, and `pip uninstall` removes only what you name — both are shown in [§4](#4-worked-example-the-lab), Steps 4 and 5.
- Ubuntu may already ship a system copy of a popular library (`python3-requests`) in `/usr/lib/python3/dist-packages`; [§3](#3-verifying-an-install) shows how to find it.

### Continue.

Move to [§3](#3-verifying-an-install) to learn how to verify an install and how it can still be broken.

---

## 3. Verifying an install

_Estimated: 10 minutes._

### Understand.

"Ran without error" is not proof. You need to ask: **which version is installed, and where did it come from?** Use the package manager's records and the program's own version check.

**Research answer (verifying):**
- **How to ask the command line which version of each component is installed and where it came from:** For a system package, use the program's version check (`java -version`), check what runs on `PATH` (`which`, `readlink -f`), find the owning package (`dpkg -S`), and ask `apt` for its records (`apt list --installed`, `apt policy`). For a language package in a virtual environment, use `pip show <pkg>` and run an `import` in the same Python (`python -c "import ...; print(...__version__, ...__file__)"`).

**For a system package (via `apt`):**
- The program's version check tells you what actually runs
- `which` and `readlink -f` show the real file on `PATH`
- `dpkg -S <file>` shows which package owns that file
- `apt list --installed | grep <pkg>` shows what `apt` has recorded (the warning goes to stderr, not into the pipe)
- `apt policy <pkg>` shows installed/candidate/version table. Packages brought in as dependencies may show `[installed,automatic]`, while the package you explicitly asked for may show `[installed]`.

**For a language package (via `pip` in a venv):**
- `pip show <pkg>` shows name, version, location, requires
- `python -c "import <pkg>; ..."` run in the **same Python** (inside an activated venv) shows which copy loads

**How an install can succeed but still be broken:** Even if an installer reports success, what actually runs might be wrong. Examples:
- **PATH order:** another program earlier on `PATH` shadows the one you just installed.
- **Alternatives:** `update-alternatives` points the link to a different program.
- **Different Pythons:** you installed into one Python environment but are running another (system Python vs venv). The two copies of `requests` show this — in the venv you get one version/location; outside the venv the system Python may load a different copy from `/usr/lib/python3/dist-packages`.

The fix is to verify: check `which`/`readlink -f`, check the package manager's records, and check the import/version in the same environment you're using.

### Try.

Check a package that's already installed on the system (no new installs here):

```bash
python3 --version
which python3
readlink -f "$(which python3)"
dpkg -S "$(readlink -f "$(which python3)")"
apt policy python3
```

*Illustrative example — not captured output:*

```text
Python <version>
/usr/bin/python3
/usr/bin/python3.14
python3.14-minimal: /usr/bin/python3.14
python3:
  Installed: <version>
  Candidate: <version>
  Version table:
 *** <version> 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
        100 /var/lib/dpkg/status
```

Before creating any venv, see what the system Python has:

```bash
python3 -c "import requests; print(requests.__version__, requests.__file__)"
apt list --installed 2>/dev/null | grep python3-requests
```

*Illustrative example — not captured output:*

```text
<version> /usr/lib/python3/dist-packages/requests/__init__.py
python3-requests/resolute-updates,resolute-security,now <version> all [installed,automatic]
```

The JDK and `requests` in a venv are verified in [§4](#4-worked-example-the-lab), Steps 2-4.

### Verify.

- Use both the program's version and the package manager's records.
- `[installed]` means you asked for it; `[installed,automatic]` means it was pulled in as a dependency.
- The system copy of `requests` is the "succeeds but still broken" case: an import can succeed while loading a different copy than the one you installed. Step 4 in [§4](#4-worked-example-the-lab) compares it with the copy you install.

### Continue.

Move to [§4](#4-worked-example-the-lab) to work through all lab steps and the deliverable steps in order.

---

## 4. Worked example: the lab

_Estimated: 35 minutes._

Work through in order. Run every command on your own machine. The examples are illustrative — replace them with your own output when capturing evidence.

### Step 1: Update the package list and install a JDK

**Goal:** Update the index, install a JDK with `apt`, and note the repository and dependencies it pulled in.

```bash
sudo apt update
sudo apt install default-jdk
```

*Illustrative example — not captured output (trimmed):*

```text
Hit:1 http://archive.ubuntu.com/ubuntu resolute InRelease
Get:2 http://archive.ubuntu.com/ubuntu resolute-updates InRelease [<size>]
Get:3 http://security.ubuntu.com/ubuntu resolute-security InRelease [<size>]
Get:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease [<size>]
Fetched <size> in <n>s (<rate>)
<count> packages can be upgraded. Run 'apt list --upgradable' to see them.
Installing:
  default-jdk

Installing dependencies:
  alsa-topology-conf  libxaw7  luit  default-jdk-headless  default-jre  openjdk-25-jdk  openjdk-25-jdk-headless  openjdk-25-jre  openjdk-25-jre-headless  ...
Summary:
  Upgrading: 0, Installing: <count>, Removing: 0, Not Upgrading: <count>
  Download size: <size> MB
  Space needed: <size> MB
Continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 openjdk-25-jre-headless amd64 <version> [<size> MB]
Get:2 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 openjdk-25-jdk amd64 <version> [<size> kB]
```

**Notes to capture:** which repository the JDK came from (look at the `Get:` lines — for example `resolute-updates`), and the dependencies `apt` pulled in (the full list under "Installing dependencies:").

### Step 2: Verify the JDK

```bash
java -version
javac -version
which java
readlink -f "$(which java)"
dpkg -S "$(readlink -f "$(which java)")"
```

*Illustrative example — not captured output:*

```text
openjdk version "<version>" <date>
...
javac <version>
/usr/bin/java
/usr/lib/jvm/java-25-openjdk-amd64/bin/java
openjdk-25-jre-headless:amd64: /usr/lib/jvm/java-25-openjdk-amd64/bin/java
```

**Capture:** `java -version` output.

### Step 3: Confirm the package is tracked by the package manager

```bash
apt list --installed | grep jdk
apt policy default-jdk
```

*Illustrative example — not captured output:*

```text

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

default-jdk-headless/resolute,now <version> amd64 [installed,automatic]
default-jdk/resolute,now <version> amd64 [installed]
openjdk-25-jdk-headless/resolute-updates,... amd64 [installed,automatic]
openjdk-25-jdk/resolute-updates,... amd64 [installed,automatic]
openjdk-25-jre-headless/resolute-updates,... amd64 [installed,automatic]
openjdk-25-jre/resolute-updates,... amd64 [installed,automatic]
default-jdk:
  Installed: <version>
  Candidate: <version>
  Version table:
 *** <version> 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
        100 /var/lib/dpkg/status
```

**Capture:** the `apt list --installed | grep jdk` output. Note `[installed]` vs `[installed,automatic]`.

### Step 4: Install a Python package with pip in a virtual environment and verify

**First, set up the environment (required on this system):**

```bash
sudo apt install python3-venv
mkdir -p ~/linux-course/m6
cd ~/linux-course/m6
python3 -m venv .venv
source .venv/bin/activate
which python pip
pip --version
```

*Illustrative example — not captured output:*

```text
/home/trainee/linux-course/m6/.venv/bin/python
/home/trainee/linux-course/m6/.venv/bin/pip
pip <version> from /home/trainee/linux-course/m6/.venv/lib/python3.14/site-packages/pip (python 3.14)
```

**Now install and verify `requests`:**

```bash
pip install requests
pip show requests
python -c "import requests; print(requests.__version__, requests.__file__)"
deactivate
python3 -c "import requests; print(requests.__version__, requests.__file__)"
source ~/linux-course/m6/.venv/bin/activate
```

*Illustrative example — not captured output:*

```text
Successfully installed certifi-<version> charset_normalizer-<version> idna-<version> requests-<version> urllib3-<version>
Name: requests
Version: <version>
Location: /home/trainee/linux-course/m6/.venv/lib/python3.14/site-packages
Requires: certifi, charset_normalizer, idna, urllib3
<version> /home/trainee/linux-course/m6/.venv/lib/python3.14/site-packages/requests/__init__.py
<version> /usr/lib/python3/dist-packages/requests/__init__.py
```

**Capture:** `pip show requests` output and the two `import` results (venv copy and system copy). Step 5 starts with the venv active.

### Step 5: Remove the pip package cleanly

```bash
pip uninstall requests
pip show requests
python -c "import requests"
pip list
deactivate
```

*Illustrative example — not captured output:*

```text
Found existing installation: requests <version>
Uninstalling requests-<version>:
  Would remove:
    /home/trainee/linux-course/m6/.venv/lib/python3.14/site-packages/requests-<version>.dist-info/*
    /home/trainee/linux-course/m6/.venv/lib/python3.14/site-packages/requests/*
Proceed (Y/n)? y
  Successfully uninstalled requests-<version>
WARNING: Package(s) not found: requests
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
Package            Version
------------------ ----------
certifi            <version>
charset-normalizer <version>
idna               <version>
pip                <version>
urllib3            <version>
```

**Capture:** `pip uninstall requests` output.

### Deliverable steps (from tasks.md)

**Note which manager installed what (required):** In your notes, state that **`apt` installed the JDK because it's system-wide software** (installed into system directories, tracked by `apt`, brought in dependencies), and **`pip` installed `requests` into the virtual environment because it's a project library** (installed from PyPI into `.venv`, no `sudo` needed, isolated from the system Python). Do this in your own words.

**Write your own explanation:** Using your version checks from Steps 2 and 4, including the two copies of `requests` from Step 4, write in your own words what would need to be true for an install to succeed silently but still be broken.

**Clean-up:** The brief's clean-up is only `pip uninstall requests` (done in Step 5). **What the clean-up does not cover:** the JDK installed by `apt` stays, and the `.venv` directory stays. `pip uninstall` only removed `requests`; its dependencies (`certifi`, `charset_normalizer`, `idna`, `urllib3`) remain in the venv (as shown by `pip list`). Contrast this with `apt autoremove` (which can remove automatically installed deps), but do not run `autoremove` here.

**What to capture in your write-up (minimum):** `java -version`, `apt list --installed | grep jdk`, `pip show requests` (in venv), `pip uninstall requests` output. Also include your notes about repository/dependencies for the JDK and your own explanation for DoD-05.

---

## 5. Completion checklist

_Estimated: 5 minutes._

- [ ] **DoD-01: The trainee's pasted `java -version` output shows the actual JDK version that `apt`
  installed, taken from the command line and not assumed from the install log (Lab steps 1–2).** Verify via [§4](#4-worked-example-the-lab).
- [ ] **DoD-02: The pasted `apt list --installed | grep jdk` output shows that the JDK is tracked by the
  system package manager (Lab step 3).** Verify via [§4](#4-worked-example-the-lab).
- [ ] **DoD-03: The pasted `pip show requests` output shows the Python package and its version, verified
  separately from `apt`, and the notes state which package manager installed which piece and why that
  was the right one for that layer (Lab steps 1 and 4).** Verify via [§4](#4-worked-example-the-lab).
- [ ] **DoD-04: The pasted `pip uninstall requests` output shows that the package was removed through the
  package manager and not by deleting files (Lab step 5).** Verify via [§4](#4-worked-example-the-lab).
- [ ] **DoD-05: Using the version checks from steps 2 and 4, the trainee can explain what would need to be
  true for an install to succeed silently but still be broken.** Verify via [§3](#3-verifying-an-install) and [§4](#4-worked-example-the-lab).

---

## 6. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- Run `sudo apt update` before `sudo apt install` — see [§1](#1-system-level-package-management).
- Use the system package manager for system-wide software and `pip` or `conda` in a virtual environment for project-specific Python libraries — see [§2](#2-language-level-package-management).
- Verify an install with a version check, not by the absence of an error — see [§3](#3-verifying-an-install).
- Remove packages with the package manager, never by deleting files by hand — see [§4](#4-worked-example-the-lab).

### Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `Error: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)` followed by `Error: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?` | Trying to install system packages without elevated privilege. | Use `sudo apt install ...`. For why elevated privilege is needed, see [M5 §4](../m05-permissions-ownership/learning-materials.md#4-elevated-privilege). |
| `Command 'pip' not found, but can be installed with:` followed by `sudo apt install python3-pip` | `pip` not installed on this system. The system Python is externally managed. | Don't install `python3-pip` system-wide or use `--break-system-packages`. Install `python3-venv`, create `python3 -m venv .venv`, `source .venv/bin/activate`, use `pip` from the venv. |
| `apt list --installed \| grep ...` shows a warning on screen | `apt list` does not have a stable CLI interface. The warning goes to stderr. | Ignore the warning on stderr; filter with `grep` on stdout. It doesn't affect the package lines. |
| `python3 -c "import requests"` still works after `deactivate`, but it loads a different version | You installed in the venv but ran with the system Python; Ubuntu has `python3-requests` in `/usr/lib/python3/dist-packages`. | Always activate the venv, or call the venv's `python` by its full path. Check with `which python` inside the venv and with `requests.__file__`. |
| `java -version` shows different version than expected | Another `java` is earlier on `PATH`, or `update-alternatives` points elsewhere. | Check `which java`, `readlink -f "$(which java)"`, `dpkg -S` on that path, and `apt policy`. |
| `pip uninstall` left packages behind | `pip uninstall` removes only the named package, not its dependencies. | This is normal for pip. If you want to remove everything, remove the whole `.venv` directory when done. Contrast with `apt autoremove` (not run here). |

---

## 7. References

_Estimated: 5 minutes._

### Man pages (offline, primary)
- `man apt` (8) — high-level interface; `update`, `install`, `list`, `policy`
- `man apt-cache` (8) — `show`, `policy` details
- `man 5 sources.list` — repository configuration (deb822 style on this release)
- `man dpkg` (1) — `-S` (search for file owner), low-level installer
- `man update-alternatives` (1) — managing alternative programs

### Help text (no dedicated man page required)
- `python3 -m venv --help` — creating virtual environments
- `pip help install`, `pip help uninstall`, `pip help show` — installing, removing, and showing package info

### Module files
- [`brief.md`](./brief.md) — objective, scope, lab, definition of done, best practices
- [`tasks.md`](./tasks.md) — checklist for this module
- [`write-up-template.md`](./write-up-template.md) — template for `write-up.md`

### Course architecture
- [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) — M6 is Week 3, Layer 3 (Access & Software), after M5
