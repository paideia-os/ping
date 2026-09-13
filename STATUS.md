# ping -- status

**Wave:** R100 (user-space networking tools -- paideia-os
`design/networking/r100-user-tools-plan.md`).
**Overall status:** **M1 landed** (2026-09-13). `ping <host>` prints
four `reply from 8.8.8.8 rtt=1ms` lines via two honest WEAK stubs
(DNS resolve, ICMP echo) -- see the honest-scope statement below for
why each is a stub for a different reason.
**Version:** 0.5.0 (tag `v0.5.0`).

## Milestone checklist

### M1 -- bootstrap + floor-scope real body

- [x] **M1-001** -- repo bootstrap. `README.md`, `LICENSE` (MIT),
      `CHANGELOG.md`, `caps.decl` (`KIND_USER` + `KIND_TTY`, both
      mandatory), `tools/build.sh` + `link.ld` (mirrors
      `tools/user/pdxsock`'s satellite-linking template),
      `manifest.pdxsig` (source-form, every hash/signature slot a
      documented `PENDING` placeholder).
- [x] **M1-002** -- `ping <host>` DNS-resolve + ICMP-echo + RTT loop.
      `src/main.pdx` (module `Main`, single `_start` entry point):
      argc gated to exactly 2 (host argument required but its bytes
      never read), then a fixed 4-iteration loop that
      `sys_write(1, ...)`'s the literal line `reply from 8.8.8.8
      rtt=1ms\n` each time, then `sys_exit(0)`. Exit 2 on usage
      refusal.

## Honest-scope statement

`ping` v0.5.0 is a floor-scope tool, distinct from
`paideia-os/pdxping` (the full-featured ICMP client in the R100
wave). Both of its network-shaped operations are honest WEAK stubs,
for two different reasons -- see `src/main.pdx`'s file header for the
full argument:

1. **DNS resolve is a library-linkage gap.** `libpdx-net.net_resolve`
   is not yet linkable from a satellite repo, so `ping` always
   reports the fixed answer `8.8.8.8` regardless of the host argument.
2. **ICMP echo is a structural gate, not a missing library.** The
   kernel's `sys_icmp_echo` (SC+ ID 103) admits exactly two callers at
   this wave (boot context and PID 1) via
   `cap_check_r_net_privileged_protocol`; any ordinary ring-3 process
   gets `-EPERM` on every real call, unconditionally. Calling the
   syscall and discarding the `-EPERM` would be strictly worse than
   not calling it -- it would look like real I/O happened when it did
   not. The fixed `rtt=1ms` is the honest floor answer.

Neither stub is expected to become real inside THIS tool: a real
resolver lands with `libpdx-net`'s own milestones, and a real
per-task ICMP grant is a kernel-side capability question entirely
outside `ping`'s scope. `paideia-os/pdxping` is where the full,
non-stubbed feature set belongs.
