# M6 Lab — Package Management

What I learned from researching M6, then the lab as I ran it on WSL2 with Ubuntu and bash: each command, its real output copied exactly, and what it shows. I pasted each block of commands at once, so I've put each command's output under it here. Block C's install log is long, so it's shown as excerpts, each a run of lines copied as printed.

## What I learned from research

1. **System-level package management:**
   - A repository is a server that holds packages, plus a signed index of what's in it. On Ubuntu 24.04 and later, `apt`'s repositories are listed in `/etc/apt/sources.list.d/ubuntu.sources`, with the server (`URIs`), the release (`Suites`), the sections (`Components`, like `main` and `universe`), and the key that checks the index (`Signed-By`).
   - `sudo apt update` downloads the latest index from each repository. It doesn't install or upgrade anything. `apt install` then picks the version from that index, which is why the brief says to update first: an old index can point to versions the server no longer has.
   - Each package lists the packages it needs (`Depends`). Dependency resolution is `apt` working out the whole set, including what those packages need in turn, and installing the missing ones. It marks them as installed automatically, so `apt autoremove` can remove them later once nothing needs them.
   - `apt install` writes into folders owned by root, like `/usr/bin` and `/usr/lib/jvm`, and takes a lock under `/var/lib/dpkg`, so it needs `sudo`. That's elevated privilege that's warranted in M5's terms: the task changes the system itself. `apt policy`, `apt show`, and `apt list` only read, so they don't need `sudo`. See `man apt`, `man apt-cache`, `man sources.list`, and `man dpkg`.

2. **Language-level package management:**
   - `pip` installs Python packages from PyPI, Python's own package index, not from Ubuntu's repositories. It tracks what it installed in the Python environment it belongs to, in that environment's `site-packages` folder. `apt` knows nothing about those packages, and `pip` knows nothing about `apt`'s.
   - `pip` resolves dependencies too, but `pip uninstall` removes only the package you name and leaves its dependencies behind. There's no `pip` version of `apt autoremove`.
   - A virtual environment, made with `python3 -m venv .venv`, is a folder with its own `python`, its own `pip`, and its own `site-packages`. `source .venv/bin/activate` puts its `bin` folder first on `PATH`, so `python` and `pip` mean that environment's until `deactivate`. Packages installed there don't touch the system Python, and the folder belongs to me, so no `sudo` is needed. On Ubuntu, `python3 -m venv` needs the `python3-venv` package, which `apt` installs.
   - Ubuntu's own tools run on the system Python and depend on the library versions `apt` put in `/usr/lib/python3/dist-packages`. A `pip install` there could replace one of them with a different version and break those tools without `apt` knowing. So the system Python has an `EXTERNALLY-MANAGED` marker file (PEP 668), and `pip` refuses to install into it and points to a virtual environment instead. See `python3 -m venv --help`, `pip help install`, and `pip help uninstall`.

3. **Verifying an install:**
   - "Ran without error" only means the installer finished. To verify, ask the installed program for its version and the package manager for what it installed: `java -version` and `javac -version`, plus `apt list --installed` and `apt policy default-jdk`, for the JDK, and `pip show requests` plus a real `import` in the same Python, for the Python package.
   - `which java` shows which file runs when I type `java`, `readlink -f` follows the links to the real file, and `dpkg -S <file>` names the package that installed that file.
   - When `apt list`'s output goes into a pipe, it warns that it "does not have a stable CLI interface". The warning goes to the screen, not into the pipe, so it doesn't change what `grep` keeps. See `man apt`, `man dpkg`, `man update-alternatives`, and `pip help show`.

## Before installing anything

```text
$ mkdir -p ~/linux-course/m6
$ cd ~/linux-course/m6
$ java -version
Command 'java' not found, but can be installed with:
sudo apt install openjdk-25-jre-headless       # version 25.0.3+9-2~26.04.2, or
sudo apt install default-jre                   # version 2:1.25-77
sudo apt install openjdk-11-jre-headless       # version 11.0.31+11-1ubuntu1~26.04.2
sudo apt install openjdk-17-crac-jre-headless  # version 17.0.19+10-0ubuntu1~26.04.1
sudo apt install openjdk-17-jre-headless       # version 17.0.19+10-1~26.04.2
sudo apt install openjdk-21-crac-jre-headless  # version 21.0.11+10-0ubuntu1~26.04.1
sudo apt install openjdk-21-jre-headless       # version 21.0.11+10-1~26.04.2
sudo apt install openjdk-25-crac-jre-headless  # version 25.0.3+9-0ubuntu1~26.04.1
sudo apt install openjdk-26-jre-headless       # version 26.0.1+8-2~26.04.2
sudo apt install openjdk-8-jre-headless        # version 8u492-ga~us2-0ubuntu1~26.04.1
$ python3 --version
Python 3.14.4
$ pip --version
Command 'pip' not found, but can be installed with:
sudo apt install python3-pip
$ pip install requests
Command 'pip' not found, but can be installed with:
sudo apt install python3-pip
$ ls /usr/lib/python3*/EXTERNALLY-MANAGED
/usr/lib/python3.14/EXTERNALLY-MANAGED
```

**Observation:** Shows that there was no Java at all yet, and Ubuntu's command-not-found helper listed the packages that would provide it, from OpenJDK 8 to 26. `default-jre` is version `2:1.25-77`, so the default points to Java 25. Python 3.14.4 is installed, but `pip` isn't, so the brief's `pip install requests` couldn't even run. And the system Python has the `EXTERNALLY-MANAGED` marker, so even with `pip` installed, a plain `pip install` into it would be refused.

```text
$ cat /etc/apt/sources.list.d/ubuntu.sources
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.

## Ubuntu distribution repository
##
## The following settings can be adjusted to configure which packages to use from Ubuntu.
## Mirror your choices (except for URIs and Suites) in the security section below to
## ensure timely security updates.
##
## Types: Append deb-src to enable the fetching of source package.
## URIs: A URL to the repository (you may add multiple URLs)
## Suites: The following additional suites can be configured
##   <name>-updates   - Major bug fix updates produced after the final release of the
##                      distribution.
##   <name>-backports - software from this repository may not have been tested as
##                      extensively as that contained in the main release, although it includes
##                      newer versions of some applications which may provide useful features.
##                      Also, please note that software in backports WILL NOT receive any review
##                      or updates from the Ubuntu security team.
## Components: Aside from main, the following components can be added to the list
##   restricted  - Software that may not be under a free license, or protected by patents.
##   universe    - Community maintained packages. Software in this repository receives maintenance
##                 from volunteers in the Ubuntu community, or a 10 year security maintenance
##                 commitment from Canonical when an Ubuntu Pro subscription is attached.
##   multiverse  - Community maintained of restricted. Software from this repository is
##                 ENTIRELY UNSUPPORTED by the Ubuntu team, and may not be under a free
##                 licence. Please satisfy yourself as to your rights to use the software.
##                 Also, please note that software in multiverse WILL NOT receive any
##                 review or updates from the Ubuntu security team.
##
## See the sources.list(5) manual page for further settings.
Types: deb
URIs: http://archive.ubuntu.com/ubuntu/
Suites: resolute resolute-updates resolute-backports
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

## Ubuntu security updates. Aside from URIs and Suites,
## this should mirror your choices in the previous section.
Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: resolute-security
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

**Observation:** Shows the two repositories `apt` uses. The first is `archive.ubuntu.com` for the `resolute` release (26.04) and its `-updates` and `-backports` suites, and the second is `security.ubuntu.com` for `resolute-security`. Both include the `main`, `universe`, `restricted`, and `multiverse` components, and both are checked with the same Ubuntu archive key in `Signed-By`.

```text
$ apt policy default-jdk
default-jdk:
  Installed: (none)
  Candidate: 2:1.25-77
  Version table:
     2:1.25-77 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
$ apt show default-jdk
Package: default-jdk
Version: 2:1.25-77
Priority: optional
Section: devel
Source: java-common (0.77)
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian Java Maintainers <pkg-java-maintainers@lists.alioth.debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 6144 B
Provides: java-sdk (= 25)
Depends: default-jre (= 2:1.25-77), default-jdk-headless (= 2:1.25-77), openjdk-25-jdk
Homepage: https://wiki.debian.org/Java/
Cnf-Extra-Commands: javac,javadoc,jar
Download-Size: 882 B
APT-Sources: http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
Description: Standard Java or Java compatible Development Kit
 This dependency package points to the Java runtime, or Java compatible
 development kit recommended for this architecture, which is
 openjdk-25-jdk for amd64.
```

**Observation:** Shows that `default-jdk` wasn't installed (`Installed: (none)`), and that the version `apt` would install is `2:1.25-77` from `resolute/main`. `apt show` shows it's a tiny package, 6144 B installed, because it only points to the real JDK: its `Depends` line is `default-jre`, `default-jdk-headless`, and `openjdk-25-jdk`. Neither command needed `sudo`, because they only read.

```text
$ apt install default-jdk
Error: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
Error: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
```

**Observation:** Shows that `apt install` without `sudo` stopped before doing anything: it couldn't open the dpkg lock in `/var/lib/dpkg`, which only root can write to, and it asked `are you root?`. Installing system-wide software needs root, the case M5 called warranted.

## Updating the package list

```text
$ sudo apt update
[sudo: authenticate] Password:
Hit:1 http://archive.ubuntu.com/ubuntu resolute InRelease
Get:2 http://archive.ubuntu.com/ubuntu resolute-updates InRelease [137 kB]
Get:3 http://security.ubuntu.com/ubuntu resolute-security InRelease [137 kB]
Get:4 http://archive.ubuntu.com/ubuntu resolute-backports InRelease [137 kB]
Get:5 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 Packages [744 kB]
Get:6 http://archive.ubuntu.com/ubuntu resolute-updates/main Translation-en [176 kB]
Get:7 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 Components [110 kB]
Get:8 http://archive.ubuntu.com/ubuntu resolute-updates/universe amd64 Packages [334 kB]
Get:9 http://archive.ubuntu.com/ubuntu resolute-updates/universe amd64 Components [201 kB]
Get:10 http://archive.ubuntu.com/ubuntu resolute-backports/universe amd64 Components [1056 B]
Get:11 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Packages [606 kB]
Get:12 http://security.ubuntu.com/ubuntu resolute-security/main amd64 Components [46.8 kB]
Get:13 http://security.ubuntu.com/ubuntu resolute-security/universe amd64 Packages [205 kB]
Get:14 http://security.ubuntu.com/ubuntu resolute-security/universe amd64 Components [55.5 kB]
Fetched 2892 kB in 8s (362 kB/s)
20 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

**Observation:** Shows that `sudo` asked for my password with `[sudo: authenticate] Password:`, and `apt update` then checked each repository's index. `Hit` means the index hadn't changed since the last update, and `Get` means it downloaded a newer one, 2892 kB in all. It didn't install or upgrade anything: the last line only counts 20 packages that could be upgraded.

## Installing the JDK

The start of the output, the plan and the prompt:

```text
$ sudo apt install default-jdk
Installing:
  default-jdk

Installing dependencies:
  alsa-topology-conf       libgif7         libxaw7        luit
  alsa-ucm-conf            libice-dev      libxcb-shape0  openjdk-25-jdk
  ca-certificates-java     libice6         libxcb1-dev    openjdk-25-jdk-headless
  default-jdk-headless     libjpeg-turbo8  libxdmcp-dev   openjdk-25-jre
  default-jre              libjpeg8        libxft2        openjdk-25-jre-headless
  default-jre-headless     libnspr4        libxkbfile1    uuid-dev
  fonts-dejavu-extra       libnss3         libxmu6        x11-utils
  java-common              libpcsclite1    libxpm4        x11proto-dev
  libasound2-data          libsm-dev       libxt-dev      xorg-sgml-doctools
  libasound2t64            libsm6          libxt6t64      xtrans-dev
  libatk-wrapper-java      libx11-dev      libxv1
  libatk-wrapper-java-jni  libxau-dev      libxxf86dga1

Suggested packages:
  alsa-utils          libsm-doc   openjdk-25-demo    fonts-ipafont-gothic  fonts-indic
  libasound2-plugins  libx11-doc  openjdk-25-source  fonts-ipafont-mincho  mesa-utils
  libice-doc          libxcb-doc  visualvm           fonts-wqy-microhei
  pcscd               libxt-doc   libnss-mdns        | fonts-wqy-zenhei

Summary:
  Upgrading: 0, Installing: 47, Removing: 0, Not Upgrading: 20
  Download size: 158 MB
  Space needed: 376 MB / 1023 GB available

Continue? [Y/n] y
```

**Observation:** Shows dependency resolution. I asked for one package, `default-jdk`, and `apt` worked out 46 more that it needs, including the real JDK (`openjdk-25-jdk` and `openjdk-25-jdk-headless`), the runtime (`openjdk-25-jre` and `openjdk-25-jre-headless`), and the X11 and sound (ALSA) libraries the desktop parts of Java use: 47 packages, a 158 MB download. It didn't ask for my password again, because `sudo` still had it from the update. The prompt is `Continue? [Y/n]`.

Two of the 47 downloads:

```text
Get:12 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 openjdk-25-jre-headless amd64 25.0.4.1+1-1~26.04.4 [59.0 MB]
Get:19 http://archive.ubuntu.com/ubuntu resolute-updates/main amd64 openjdk-25-jdk amd64 25.0.4.1+1-1~26.04.4 [1782 kB]
```

**Observation:** Shows that OpenJDK 25.0.4.1 came from `resolute-updates`. Before the update, in the first block, the helper offered `25.0.3+9-2~26.04.2`. With the old index, `apt` would have gone for the older version, which is why the brief updates first.

Two parts of the setup:

```text
Setting up ca-certificates-java (20260311) ...
No JRE found. Skipping Java certificates setup.
```

```text
Setting up openjdk-25-jre-headless:amd64 (25.0.4.1+1-1~26.04.4) ...
update-alternatives: using /usr/lib/jvm/java-25-openjdk-amd64/bin/java to provide /usr/bin/java (java) in auto mode
```

**Observation:** Shows that the packages were set up in an order: when `ca-certificates-java` was set up there was no JRE yet, so it skipped its step. Setting up `openjdk-25-jre-headless` made `/usr/bin/java` point to the JDK's `java` through `update-alternatives`, in auto mode.

Later, when the JRE was set up:

```text
Processing triggers for ca-certificates-java (20260311) ...
Adding debian:ACCVRAIZ1.pem
Adding debian:AC_RAIZ_FNMT-RCM.pem
```

**Observation:** Shows that once the JRE was there, `ca-certificates-java` ran its step after all, as a trigger. It printed 120 `Adding debian:` lines in all, one per certificate, and only the first two are shown here. That's the trust store Java uses for HTTPS.

The end of the output:

```text
Setting up default-jre (2:1.25-77) ...
Setting up default-jdk-headless (2:1.25-77) ...
Setting up openjdk-25-jdk:amd64 (25.0.4.1+1-1~26.04.4) ...
update-alternatives: using /usr/lib/jvm/java-25-openjdk-amd64/bin/jconsole to provide /usr/bin/jconsole (jconsole) in auto mode
Setting up default-jdk (2:1.25-77) ...
```

**Observation:** Shows that `default-jdk` itself was set up last, after everything it depends on.

## Verifying the JDK

```text
$ java -version
openjdk version "25.0.4.1" 2026-08-18
OpenJDK Runtime Environment (build 25.0.4.1+1-1-26.04.4-Ubuntu)
OpenJDK 64-Bit Server VM (build 25.0.4.1+1-1-26.04.4-Ubuntu, mixed mode, sharing)
$ javac -version
javac 25.0.4.1
$ which java
/usr/bin/java
$ readlink -f "$(which java)"
/usr/lib/jvm/java-25-openjdk-amd64/bin/java
$ dpkg -S "$(readlink -f "$(which java)")"
openjdk-25-jre-headless:amd64: /usr/lib/jvm/java-25-openjdk-amd64/bin/java
```

**Observation:** Shows that `java -version` reports OpenJDK 25.0.4.1, the same version `apt` downloaded, and `javac` is there too, so this is a full JDK, not only a runtime. `which java` gives `/usr/bin/java`, but that's a link: `readlink -f` follows it to `/usr/lib/jvm/java-25-openjdk-amd64/bin/java`, and `dpkg -S` says that file belongs to `openjdk-25-jre-headless`, one of the dependencies, not to `default-jdk`.

```text
$ apt list --installed | grep jdk

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

default-jdk-headless/resolute,now 2:1.25-77 amd64 [installed,automatic]
default-jdk/resolute,now 2:1.25-77 amd64 [installed]
openjdk-25-jdk-headless/resolute-updates,resolute-security,now 25.0.4.1+1-1~26.04.4 amd64 [installed,automatic]
openjdk-25-jdk/resolute-updates,resolute-security,now 25.0.4.1+1-1~26.04.4 amd64 [installed,automatic]
openjdk-25-jre-headless/resolute-updates,resolute-security,now 25.0.4.1+1-1~26.04.4 amd64 [installed,automatic]
openjdk-25-jre/resolute-updates,resolute-security,now 25.0.4.1+1-1~26.04.4 amd64 [installed,automatic]
$ apt policy default-jdk
default-jdk:
  Installed: 2:1.25-77
  Candidate: 2:1.25-77
  Version table:
 *** 2:1.25-77 500
        500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages
        100 /var/lib/dpkg/status
$ ls -ld /usr/lib/jvm /usr/lib/jvm/*
drwxr-xr-x 4 root root 4096 Oct 22 15:57 /usr/lib/jvm
lrwxrwxrwx 1 root root   25 Mar  7  2026 /usr/lib/jvm/default-java -> java-1.25.0-openjdk-amd64
lrwxrwxrwx 1 root root   21 Sep  4 14:19 /usr/lib/jvm/java-1.25.0-openjdk-amd64 -> java-25-openjdk-amd64
drwxr-xr-x 9 root root 4096 Oct 22 15:57 /usr/lib/jvm/java-25-openjdk-amd64
drwxr-xr-x 2 root root 4096 Oct 22 15:57 /usr/lib/jvm/openjdk-25
```

**Observation:** Shows that `apt` tracks the JDK. `default-jdk` is `[installed]`, because I asked for it, and the other five JDK packages are `[installed,automatic]`, because dependency resolution brought them in. The warning printed on the screen but not into the pipe, so `grep` still got only package lines. `apt policy` now shows `Installed: 2:1.25-77`, with `***` on the installed version and `/var/lib/dpkg/status` as the record of it. In `/usr/lib/jvm`, `default-java` links to `java-1.25.0-openjdk-amd64`, which links to the real folder, `java-25-openjdk-amd64`.

## The virtual environment tool

```text
$ sudo apt install python3-venv
Installing:
  python3-venv

Installing dependencies:
  python3-pip-whl  python3-setuptools-whl  python3.14-venv

Summary:
  Upgrading: 0, Installing: 4, Removing: 0, Not Upgrading: 20
  Download size: 2610 kB
  Space needed: 2958 kB / 1023 GB available

Continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu resolute/universe amd64 python3-pip-whl all 25.1.1+dfsg-1ubuntu2 [1430 kB]
Get:2 http://archive.ubuntu.com/ubuntu resolute/universe amd64 python3-setuptools-whl all 78.1.1-0.1build1 [1174 kB]
Get:3 http://archive.ubuntu.com/ubuntu resolute-updates/universe amd64 python3.14-venv amd64 3.14.4-1ubuntu0.2 [5356 B]
Get:4 http://archive.ubuntu.com/ubuntu resolute/universe amd64 python3-venv amd64 3.14.3-0ubuntu2 [1032 B]
Fetched 2610 kB in 8s (313 kB/s)
Selecting previously unselected package python3-pip-whl.
(Reading database ... 45132 files and directories currently installed.)
Preparing to unpack .../python3-pip-whl_25.1.1+dfsg-1ubuntu2_all.deb ...
Unpacking python3-pip-whl (25.1.1+dfsg-1ubuntu2) ...
Selecting previously unselected package python3-setuptools-whl.
Preparing to unpack .../python3-setuptools-whl_78.1.1-0.1build1_all.deb ...
Unpacking python3-setuptools-whl (78.1.1-0.1build1) ...
Selecting previously unselected package python3.14-venv.
Preparing to unpack .../python3.14-venv_3.14.4-1ubuntu0.2_amd64.deb ...
Unpacking python3.14-venv (3.14.4-1ubuntu0.2) ...
Selecting previously unselected package python3-venv.
Preparing to unpack .../python3-venv_3.14.3-0ubuntu2_amd64.deb ...
Unpacking python3-venv (3.14.3-0ubuntu2) ...
Setting up python3-setuptools-whl (78.1.1-0.1build1) ...
Setting up python3-pip-whl (25.1.1+dfsg-1ubuntu2) ...
Setting up python3.14-venv (3.14.4-1ubuntu0.2) ...
```

**Observation:** Shows that `python3-venv` comes from `universe` and pulled in three dependencies. One is `python3-pip-whl`, a copy of `pip` that new virtual environments start with. So the system still has no `pip` command, but every virtual environment gets its own.

## requests in a virtual environment

```text
$ cd ~/linux-course/m6
$ python3 -m venv .venv
$ source .venv/bin/activate
$ which python pip
/home/ejandrea/linux-course/m6/.venv/bin/python
/home/ejandrea/linux-course/m6/.venv/bin/pip
$ pip --version
pip 25.1.1 from /home/ejandrea/linux-course/m6/.venv/lib/python3.14/site-packages/pip (python 3.14)
$ pip install requests
Collecting requests
  Downloading requests-2.34.2-py3-none-any.whl.metadata (4.8 kB)
Collecting charset_normalizer<4,>=2 (from requests)
  Downloading charset_normalizer-3.5.2-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (46 kB)
Collecting idna<4,>=2.5 (from requests)
  Downloading idna-3.20-py3-none-any.whl.metadata (7.2 kB)
Collecting urllib3<3,>=1.26 (from requests)
  Downloading urllib3-2.8.0-py3-none-any.whl.metadata (7.4 kB)
Collecting certifi>=2023.5.7 (from requests)
  Downloading certifi-2026.10.19-py3-none-any.whl.metadata (2.5 kB)
Downloading requests-2.34.2-py3-none-any.whl (73 kB)
Downloading charset_normalizer-3.5.2-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (255 kB)
Downloading idna-3.20-py3-none-any.whl (69 kB)
Downloading urllib3-2.8.0-py3-none-any.whl (135 kB)
Downloading certifi-2026.10.19-py3-none-any.whl (136 kB)
Installing collected packages: urllib3, idna, charset_normalizer, certifi, requests
Successfully installed certifi-2026.10.19 charset_normalizer-3.5.2 idna-3.20 requests-2.34.2 urllib3-2.8.0
```

**Observation:** Shows that after `source .venv/bin/activate`, `python` and `pip` were the ones inside `.venv`, and `pip` is version 25.1.1, the copy from `python3-pip-whl`. `pip install requests` downloaded from PyPI and did its own dependency resolution: `requests` needs `charset_normalizer`, `idna`, `urllib3`, and `certifi`, so it installed five packages. No `sudo` was needed, because `.venv` is my own folder.

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
$ ls -ld /usr/lib/python3/dist-packages .venv/lib/python3*/site-packages
drwxr-xr-x  14 ejandrea ejandrea  4096 Oct 22 16:01 .venv/lib/python3.14/site-packages
drwxr-xr-x 153 root     root     12288 Aug 27 22:12 /usr/lib/python3/dist-packages
$ deactivate
$ python3 -c "import requests; print(requests.__version__, requests.__file__)"
2.32.5 /usr/lib/python3/dist-packages/requests/__init__.py
$ apt list --installed 2>/dev/null | grep python3-requests
python3-requests/resolute-updates,resolute-security,now 2.32.5+dfsg-1ubuntu1.1 all [installed,automatic]
```

**Observation:** Shows that `pip show` reports `requests` 2.34.2 in the virtual environment's `site-packages`, and the `import` in the same Python loaded that copy. After `deactivate`, the system Python loaded a different copy: `requests` 2.32.5 in `/usr/lib/python3/dist-packages`, which `apt` had already installed as `python3-requests`, a dependency of something else (`[installed,automatic]`). So there are two copies, from two package managers that don't know about each other: mine, 2.34.2, owned by me in `.venv`, and Ubuntu's, 2.32.5, owned by root. A `pip install` into the system Python could have replaced Ubuntu's copy under the tools that need it, which is what the `EXTERNALLY-MANAGED` marker prevents.

## Which manager installed what

`apt` installed the JDK, because a JDK is system-wide software: it goes into `/usr/lib/jvm`, and `/usr/bin/java` points to it for every user. `apt` tracks it, resolves its dependencies, and gets its security updates. `pip` installed `requests`, because it's a library for one project. It went into that project's virtual environment, at the version from PyPI, without touching the system Python or needing `sudo`. Each was installed by the manager for its layer, and each was verified with that manager's own command: `apt list` and `apt policy` for the JDK, and `pip show` for `requests`.

## Removing requests

```text
$ source ~/linux-course/m6/.venv/bin/activate
$ pip uninstall requests
Found existing installation: requests 2.34.2
Uninstalling requests-2.34.2:
  Would remove:
    /home/ejandrea/linux-course/m6/.venv/lib/python3.14/site-packages/requests-2.34.2.dist-info/*
    /home/ejandrea/linux-course/m6/.venv/lib/python3.14/site-packages/requests/*
Proceed (Y/n)? y
  Successfully uninstalled requests-2.34.2
```

**Observation:** Shows that `pip` found the installed `requests` 2.34.2, listed exactly what it would remove (its `dist-info` folder and the `requests` package folder), and removed them after I answered `y`. I didn't delete anything by hand.

## After the removal

```text
$ pip show requests
WARNING: Package(s) not found: requests
$ python -c "import requests"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
    import requests
ModuleNotFoundError: No module named 'requests'
$ pip list
Package            Version
------------------ ----------
certifi            2026.10.19
charset-normalizer 3.5.2
idna               3.20
pip                25.1.1
urllib3            2.8.0
$ deactivate
```

**Observation:** Shows that `requests` is gone from the virtual environment: `pip show` can't find it, and the `import` fails with `ModuleNotFoundError`. `pip list` shows that its four dependencies are still there, because `pip uninstall` removes only the package you name. It only removed files under `.venv`, as its `Would remove` list showed, so Ubuntu's copy in `/usr/lib/python3/dist-packages` was never part of it.

## Succeeding silently but still broken (DoD-05)

An install can succeed without errors and still be broken when what runs isn't what was installed. For the JDK, that would be another `java` earlier on `PATH`, or `update-alternatives` pointing `/usr/bin/java` to a different JDK: `apt` would still report OpenJDK 25 as installed, but `java -version` would show another version. For Python, it would be installing into one environment and running with another. In this lab, after `deactivate`, the system Python still imported a `requests`, but it was Ubuntu's 2.32.5, not the 2.34.2 I installed, so a script needing 2.34 would get the wrong copy with no error at install time. The version checks catch both: `java -version`, `readlink -f`, and `dpkg -S` show what actually runs and where it came from, and `pip show` plus an `import` in the same Python show which copy that Python loads.

The JDK and `.venv` stay installed. The brief's clean-up only covers `requests`, which `pip uninstall` removed.
