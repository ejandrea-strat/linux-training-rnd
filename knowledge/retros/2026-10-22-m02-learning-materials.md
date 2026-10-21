# Retro — 2026-10-22 — M2 learning materials rewrite

**Module:** M2 — CLI Basics
**Artifact:** `modules/m02-cli-basics/learning-materials.md` (rewrite, 724 lines)
**Cycle:** EPAV E → P → A → V

## Backlog items

| # | Item | Where | Why it is out of scope here | Proposed fix |
| --- | --- | --- | --- | --- |
| B-1 | `knowledge/patterns/brief.md` holds a Terraform-course brief (AWS, Terraform, tfenv, VS Code) that belongs to a different course, and the graph indexes it as a Brief-pattern node | `knowledge/patterns/brief.md` | It is another course's reference artifact, not a pattern for this repo, and this cycle's contract allows writing the M2 guide, one pattern addition, and this retro | Move it to `knowledge/reference/` or drop it, so a `graphify query` about briefs cannot return Terraform content in the Linux training repo |
| B-2 | The M1 retro's B-1 asked for the M2 guide's two dead `knowledge/observations/m02-cli-basics.md` links to be repointed at `knowledge/resources/m02-cli-basics-lab.md` | `knowledge/retros/2026-10-22-m01-learning-materials.md` | Superseded — the M2 guide now drops the link entirely, so it never points at the lab file | Update the M1 retro's B-1 row to say the links were removed, not repointed |
| B-3 | The capstone cohort document links `./mock-data/` and three CSV files that resolve to nothing | `modules/capstone-end-to-end-pipeline-cohort.md` | Another stage's document | Add the mock data or drop the links — carried over unchanged from the M1 retro |

## Notes

- The only source of real output is the lab file `knowledge/resources/m02-cli-basics-lab.md`. The
  guide does not link it and does not present any example as captured output; every example is
  labeled illustrative and checked against a live run on the course image.
- Environment facts confirmed on the course image during validation: `ls` is uutils coreutils
  0.10.0 (`man ls` resolves to the `rust-ls` page); `grep` is GNU grep 3.12; bash is 5.3.9;
  `man grep | head -15` and `grep --help | head -10` in the guide are byte-identical to the live
  output at 80 columns.
- The lab file's `man grep | head -15` capture is wrapped wider than 80 columns, so the guide uses
  the 80-column form; the text is identical, only the wrapping differs.
- `ls -la ~` on the image: the home directory is `drwxr-x---` (mode 750), owned by the user, whose
  primary group is its own private group (`USERGROUPS_ENAB yes` in `/etc/login.defs`), and `..` is
  `/home`, owned by `root`. A guide that calls mode 750 "owner-only" is wrong — the group can read
  and enter.
- uutils `ls` sizes its owner, group, and size columns to the widest value in the listing, and
  prints in columns only when stdout is a terminal. Both behaviors are now recorded in
  `knowledge/patterns/illustrative-output-values.md`.
- The lab file's tab-completion line `ls /etc/sys` is the partial path typed before the keypress,
  not a path that is expected to exist. The guide presents it that way and nothing more.

## Applied in this cycle

- `knowledge/patterns/illustrative-output-values.md` gained items 8–10: the uutils `ls` layout
  differences, man page wrapping, and showing what an interactive keypress produced.
