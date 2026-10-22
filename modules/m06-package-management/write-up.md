# Package Management Write-up

## What I built

A Java Development Kit installed system-wide with `apt`, and the Python package `requests` installed with `pip` in a virtual environment, `~/linux-course/m6/.venv`. `sudo apt install default-jdk` brought in OpenJDK 25.0.4.1 from Ubuntu's `resolute-updates` repository, plus 46 dependencies. `pip install requests` brought in `requests` 2.34.2 from PyPI, plus its four dependencies. I verified the JDK with `java -version`, `javac -version`, `apt list --installed`, and `apt policy`, and followed `/usr/bin/java` to the package that owns it with `readlink -f` and `dpkg -S`. I verified `requests` with `pip show` and an `import` in the same Python. Then I removed `requests` with `pip uninstall` and checked it was gone. My setup is the same as in M1 to M5: WSL2 on Windows, Ubuntu 26.04.1 LTS, bash. The [lab file](../../knowledge/resources/m06-package-management-lab.md) has every command and its output, including the repositories `apt` uses, the dependency list, and the two copies of `requests` on the system.

I also used EPAV to write [`learning-materials.md`](./learning-materials.md), a study guide that teaches M6 to the next trainee.

## Why it's built this way (key decisions)

**Why `requests`:** It's the standard Python library for HTTP calls, which is how a data pipeline pulls from an API, so it's a realistic project library. It has dependencies of its own, which shows `pip`'s dependency resolution, and Ubuntu already ships its own copy as `python3-requests`, which shows how system and project packages stay apart.

**Why two managers:** The JDK is system-wide software, so `apt` installs it into root-owned folders, tracks it, and keeps it updated. `requests` is a library for one project, so `pip` installs it into that project's virtual environment. The brief's plain `pip install requests` couldn't run: `pip` isn't installed on Ubuntu 26.04, and the system Python is marked `EXTERNALLY-MANAGED`, so `pip` would refuse to install into it anyway. A virtual environment is what the marker points to, and it keeps the project's version, 2.34.2, apart from Ubuntu's 2.32.5, without `sudo`.

**What made each install trustworthy, not just "ran without error":** Each check asks something the installer's exit doesn't answer. For the JDK, `java -version` reports 25.0.4.1, the same version `apt` downloaded. `readlink -f` and `dpkg -S` show that the `java` on `PATH` is the file from `openjdk-25-jre-headless`, not some other Java, and `apt list` and `apt policy` show that `apt` recorded the install. For `requests`, `pip show` names the version and the folder, and the `import` in the same Python loaded that copy, from that folder. The same `import` from the system Python loaded Ubuntu's 2.32.5, so the check only counts when it runs in the environment the package went into.

## How to build it (teach it to the next trainee)

1. Look before installing anything: run `java -version`, `python3 --version`, and `pip --version`. On Ubuntu 26.04 there's no Java, Python 3.14 is there, and `pip` isn't.
2. Check where `apt` gets packages: `cat /etc/apt/sources.list.d/ubuntu.sources` lists the repositories, and `apt policy default-jdk` and `apt show default-jdk` show the version it would install and what it depends on. These only read, so they don't need `sudo`.
3. Update the index, then install: `sudo apt update`, then `sudo apt install default-jdk`. Read the plan before answering `Continue? [Y/n]`: it lists every dependency `apt` will bring in. Without `sudo`, `apt install` fails on the dpkg lock, because installing changes the system.
4. Verify the JDK:
   - `java -version` and `javac -version` should both report the version `apt` downloaded.
   - `readlink -f "$(which java)"` shows the real file behind `/usr/bin/java`, and `dpkg -S` on that path names the package it came from.
   - `apt list --installed | grep jdk` and `apt policy default-jdk` show what `apt` recorded.
5. Get the virtual environment tool with `sudo apt install python3-venv`. It also brings the `pip` that new environments start with.
6. Make and use a virtual environment: `cd ~/linux-course/m6`, `python3 -m venv .venv`, and `source .venv/bin/activate`. Then check `which python pip`: both should be inside `.venv`.
7. Install and verify the Python package: `pip install requests`, then `pip show requests` and `python -c "import requests; print(requests.__version__, requests.__file__)"`. The version and folder from both should match, and the folder should be inside `.venv`.
8. Remove it with the manager that installed it: `pip uninstall requests`, then check `pip show requests` and the `import` again. Both should fail. `pip list` will still show the dependencies, because `pip uninstall` only removes the package you name. Run `deactivate` when you're done.

**If a check doesn't match:** look at which program or which Python is running before reinstalling. `which`, `readlink -f`, and `requests.__file__` show what's actually being used.

## Concepts worth explaining

**System vs. language-level package management:** `apt` manages the system: it installs from Ubuntu's repositories into root-owned folders like `/usr/lib/jvm` and `/usr/lib/python3/dist-packages`, and Ubuntu's own tools depend on what it puts there. `pip` manages Python packages: it installs from PyPI into one Python environment. The two don't know about each other. On this system, `apt` had already installed `requests` 2.32.5 as `python3-requests`, a dependency of something else, and `pip` installed 2.34.2 in `.venv`. If `pip` could install into the system Python, it could replace Ubuntu's copy under the tools that need it, and `apt` wouldn't know. That's why the system Python is marked `EXTERNALLY-MANAGED`, and why project libraries go in a virtual environment.

**Dependency resolution:** A package lists what it needs, and the manager works out the whole set and installs what's missing. I asked `apt` for one package, `default-jdk`, which is only 6144 B because it just points to the real JDK, and `apt` installed 47, including `openjdk-25-jdk`, the runtime, and the X11 and sound libraries. It marked the 46 dependencies as installed automatically, so `apt autoremove` can clean them up later. `pip` did the same for `requests`, installing `certifi`, `charset_normalizer`, `idna`, and `urllib3`, but it doesn't track them that way: `pip uninstall requests` left all four behind.

## What tripped me up

- `pip` wasn't installed, so the brief's `pip install requests` gave `Command 'pip' not found`. And even with `pip` installed, the system Python's `EXTERNALLY-MANAGED` marker would have stopped it.
- `python3 -m venv` needed `python3-venv` from `apt` first, so the Python part of the lab started with a system install.
- The system Python already had a `requests`, Ubuntu's 2.32.5. After `deactivate`, the `import` still worked, just with a different copy from the one I installed.
- `apt list` printed `WARNING: apt does not have a stable CLI interface` when its output went into `grep`. It goes to the screen, not into the pipe, so the results were still only package lines.
- `ca-certificates-java` printed `No JRE found. Skipping Java certificates setup.` during the install, which looked like a failure. It only ran before the JRE was set up, and it added the certificates later, as a trigger.
- `pip uninstall requests` left its four dependencies in `.venv`.

## Checkpoint evidence

**The JDK:**
```text
$ java -version
openjdk version "25.0.4.1" 2026-08-18
OpenJDK Runtime Environment (build 25.0.4.1+1-1-26.04.4-Ubuntu)
OpenJDK 64-Bit Server VM (build 25.0.4.1+1-1-26.04.4-Ubuntu, mixed mode, sharing)
$ javac -version
javac 25.0.4.1
$ dpkg -S "$(readlink -f "$(which java)")"
openjdk-25-jre-headless:amd64: /usr/lib/jvm/java-25-openjdk-amd64/bin/java
$ apt policy default-jdk
default-jdk:
  Installed: 2:1.25-77
  Candidate: 2:1.25-77
  Version table:
 *** 2:1.25-77 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
        100 /var/lib/dpkg/status
```

**The Python package, in the virtual environment:**
```text
$ pip show requests
Name: requests
Version: 2.34.2
Summary: Python HTTP for Humans.
Home-page:
Author:
Author-email: Kenneth Reitz <me@kennethreitz.org>
License: Apache-2.0
Location: /home/ejandrea/linux-course/m6/.venv/lib/python3.14/site-packages
Requires: certifi, charset_normalizer, idna, urllib3
Required-by:
$ python -c "import requests; print(requests.__version__, requests.__file__)"
2.34.2 /home/ejandrea/linux-course/m6/.venv/lib/python3.14/site-packages/requests/__init__.py
```

**After `pip uninstall requests`:**
```text
$ pip show requests
WARNING: Package(s) not found: requests
$ python -c "import requests"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
```

## What I'd do differently

I'd check what the system already has before installing a Python package, with `python3 -c "import requests"` and `apt list --installed | grep python3-`, so a second copy doesn't come as a surprise. I'd also keep the project's libraries in a `requirements.txt` and install with `pip install -r requirements.txt`, so the virtual environment can be rebuilt exactly. And after `pip uninstall`, I'd check `pip list` for the dependencies left behind, or remove the whole `.venv` when the project is done, since `pip` has nothing like `apt autoremove`.
