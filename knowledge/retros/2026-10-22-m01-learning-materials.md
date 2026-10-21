# Retro — 2026-10-22 — M1 learning materials rewrite

**Module:** M1 — Linux Fundamentals
**Artifact:** `modules/m01-linux-fundamentals/learning-materials.md` (rewrite, 649 lines)
**Cycle:** EPAV E → P → A → V

## Backlog items

| # | Item | Where | Why it is out of scope here | Proposed fix |
| --- | --- | --- | --- | --- |
| B-1 | Two links point to `knowledge/observations/m02-cli-basics.md`, a path removed in commit `324a99b` | `modules/m02-cli-basics/learning-materials.md`, lines 14 and 678 | Another module's guide; this cycle's contract allows writing only the M1 guide | Repoint both at `../../knowledge/resources/m02-cli-basics-lab.md` in the M2 cycle |
| B-2 | `./mock-data/` and three CSV links resolve to nothing | `modules/capstone-end-to-end-pipeline-cohort.md` | Cohort document from another stage | Add the mock data or drop the links |
| B-3 | graphify node line numbers for the M1 guide are stale | `graphify-out/graph.json` | The graph is rebuilt on commit, not on edit | Rebuild the graph after the commit |

## Notes

- The only source of real output is the lab file `knowledge/resources/m01-linux-fundamentals-lab.md`.
  The `knowledge/observations/` directory was removed in commit `324a99b`; the M2 guide still links to
  the old path, which is item B-1.
- Environment values confirmed on the course image during validation: `uname --version` prints
  `uname (uutils coreutils) 0.10.0`; `man 5 proc` lists `proc_cpuinfo(5)` and `proc_pid(5)` in
  SEE ALSO and documents neither section itself; `/etc/wsl.conf` carries `[boot] systemd=true`;
  `uname -a` ends in `-microsoft-standard-WSL2`; `/etc/os-release` is Ubuntu's file, with
  `BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"` and `UBUNTU_CODENAME=resolute`.
- The author's lab file shows PID 1 as `/sbin/init` and PID 2 as `/init`. Those two rows are part of
  the environment contract and stay literal in the guide; every other PID is a placeholder.
