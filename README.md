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
