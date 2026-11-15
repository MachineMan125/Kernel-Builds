# Kernel-Builds
Online Compile for Various Kernels.

## Automatic linux-tkg builds

`build-linux-tkg-debian.yml` also runs by itself once a day. Each scheduled
run looks up the newest stable release of the kernel series named in
`AUTO_SERIES` and builds it, with the `AUTO_SCHEDULER` scheduler, for every
machine in `AUTO_MACHINES` that does not have it yet. The packages are
published as a GitHub Release named `linux-<version>-<scheduler>`, with one
`linux-image` and `linux-headers` package and a `SHA256SUMS-<machine>.txt`
per machine. Those three settings are at the top of the workflow's `env:`
block.

- A machine without `configs/<machine>.config` is skipped with a warning.
- If one machine's build fails, the others are still published and the
  failed one is retried on the next scheduled run.
- Runs started by hand work as before and publish nothing.
- GitHub only runs the schedule from the default branch, and pauses it
  after 60 days without commits (re-enable it on the Actions page).

## MuQSS test builds

`build-linux-tkg-muqss-test.yml` is started by hand only and never publishes a
Release; packages are uploaded as run artifacts (kept 30 days) and carry `-mqt`
in their name, so they never mix with the scheduled `-muqss-` kernels. It uses
the same pinned linux-tkg and the same checks as the main workflow, always with
MuQSS (7.2 / 7.3), and adds these experiments:

| Input | Choices | Notes |
| --- | --- | --- |
| `muqss_runqueue` | mc (default), smt, none, mc-llc, smp, all | Compiled-in layout; any kernel can still override it at boot with `rqshare=` |
| `psi` | boot (default), off, on | `boot` = compiled in, enabled only with the `psi=1` boot parameter; `off` = linux-tkg's workaround |
| `iotime` | on (default), off | `CONFIG_MUQSS_IOTIME`; adds `/proc/<pid>/iotime` |
| `rr_interval` | default (6), 2, 3, 4, 6, 8 | Compiled-in timeslice in ms |
| `yield_type` | default (1), 0, 1, 2 | Compiled-in `sched_yield` behaviour |
| `iso_cpu` | default (70), 25, 50, 70, 90 | Compiled-in SCHED_ISO CPU share |

The last three remain adjustable at runtime in `/proc/sys/kernel/`. Useful boot
parameters on these kernels: `rqshare=mc` / `rqshare=smt`, `psi=1`,
`schedstats=enable`.
