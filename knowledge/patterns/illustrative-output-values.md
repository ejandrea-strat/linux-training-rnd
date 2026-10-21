# Pattern — Placeholder policy for illustrative command output in a teaching guide

**Discovered in:** M1 — Linux Fundamentals learning materials rewrite, 2026-10-22
**Context:** A study guide whose command outputs are illustrative examples rather than captured
evidence, written from an author's lab file.

## Problem

A guide's examples get read as evidence. Two failure modes pull in opposite directions:

- An example that carries the author's real hostname, PIDs, memory totals, or timestamps looks like
  captured output, and readers copy it as evidence or stop trusting the guide.
- Blanket-placeholdering breaks examples the surrounding prose depends on. A guide that explains
  "PID 1 is `/sbin/init`" cannot show `<pid>` in that row and stay coherent.

## Pattern

1. **Classify every value in an example before writing it.** Two classes:
   - **Environment contract** — fixed for the course image or the course's platform: PID 1 and PID 2
     in a process listing, the distribution's own `/etc/os-release` keys, the `-microsoft-standard-WSL2`
     kernel suffix under WSL2, the `uutils coreutils` implementation of `uname` on the course image.
   - **Machine-specific** — hostname, user account PIDs, CPU model, memory and swap totals, process
     counts, timestamps, build dates.
2. **Keep contract values literal; placeholder-ize machine-specific ones.** Use `<version>`,
   `<pid>`, `<ppid>`, `<date>`, `<count>`, `<cpu model>`, `<n>` — one placeholder per kind of value,
   reused consistently.
3. **Never invent a realistic-looking value.** A made-up `73.4g` or `938636` reads as real output.
   A placeholder is honest about being a placeholder.
4. **Take contract values from the author's lab file, then confirm them on the course image.** The lab
   file supplies the values; `uname --version`, `man -w 5 proc`, `cat /etc/wsl.conf`, and
   `cat /etc/os-release` confirm them. When the two disagree, the lab file wins and the guide says
   so.
5. **Make the example and the prose agree.** If the text says "PID 1 is `/sbin/init`", the example
   shows PID 1 and `/sbin/init`. A mismatch teaches the wrong thing, and no amount of placeholder
   discipline fixes it.
6. **For a distribution-specific file, show that distribution's real keys and name the distribution.**
   A synthesized generic `/etc/os-release` is wrong in detail — wrong key names, wrong URLs. Show
   Ubuntu's actual file, say it is Ubuntu's set of keys, and mark only the point release as
   variable.
7. **Every literal value in an example must be checkable by the reader** on the course environment,
   from the lab file or from a command they can run.
8. **Match the layout the reader will actually see, and check which implementation produced it.**
   On the course image `ls` is the uutils (`rust-ls`) build, not GNU coreutils, and it differs from
   GNU in ways an example will contradict:
   - It sizes the owner, group, and size columns to the widest value in the listing, so GNU's fixed
     padding must not be copied. Placeholders change the width anyway — keep the field *order*
     right (mode, links, owner, group, size, date, name) and let the padding go.
   - It prints in columns when stdout is a terminal and one entry per line when piped. Write the
     terminal form for anything a reader types at a prompt.
   - `man ls` resolves to the uutils page, not the GNU coreutils one. Say so where the page is
     cited.
9. **Man page output wraps to the terminal width.** A captured `man` page differs from a lab file's
   capture only in wrapping. Capture at the default 80 columns, and prefer a piped capture
   (`man <cmd> | head -15`) so the example is reproducible instead of dependent on the pager.
10. **An interactive example must show what the keypress produced, not only what was typed.** For
    tab completion, the typed prefix alone leaves the reader guessing; show the completed line, then
    the result of running it. A checkpoint that asks "what did Tab do with this partial path?"
    cannot be answered from the prefix.

## Why it works

The guide stays honest — nothing in it can be mistaken for captured evidence — while staying
verifiable and internally consistent. A reader can compare an example to their own output, see the
same shape, and know exactly which fields are fixed and which are theirs to fill in.

## Applies to

Any study guide or documentation deliverable whose command outputs are illustrative rather than
recorded. Complements the build-to-teach pattern in `documentation-deliverable-modules.md`: that one
separates deliverable from evidence, this one decides what an illustrative example may contain.
