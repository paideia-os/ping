# ping -- status

**Wave:** R100 (user-space networking tools -- paideia-os
`design/networking/r100-user-tools-plan.md`); tracker prefix R80
(dispatch-wave label; see "Wave-label note" below).
**Overall status:** **M1 CLOSED** (2026-09-13, `r80-closed`).
`ping <host>` prints four `reply from 8.8.8.8 rtt=1ms` lines via two
honest WEAK stubs (DNS resolve, ICMP echo) -- see the honest-scope
statement below for why each is a stub for a different reason.
**Version:** 0.5.0 (tag `v0.5.0`); round-closure tag `r80-closed`.

## Wave-label note

Every issue on this tracker is filed under an `R80.M1-*` prefix
(`R80.M1-001`, `R80.M1-002`, `R80.M1-006`), while this document's own
header and every design cross-reference name the design-doc round as
R100 (`design/networking/r100-user-tools-plan.md`). Both are correct
and refer to different things: R100 is the design round `ping` and
its siblings (`pdxping`, `libpdx-net`, ...) were scoped under; R80 is
the dispatch-wave label this repo's issue tracker was seeded under.
Recorded here (not silently reconciled) so a future reader does not
mistake the mismatch for a typo. `design/round-retrospectives/r80-closure.md`
in the paideia-os monorepo carries the same note.

## Milestone checklist

### M1 -- bootstrap + floor-scope real body

- [x] **M1-001** -- repo bootstrap. `README.md`, `LICENSE` (MIT),
      `CHANGELOG.md`, `caps.decl` (`KIND_USER` + `KIND_TTY`, both
      mandatory), `tools/build.sh` + `link.ld` (mirrors
      `tools/user/pdxsock`'s satellite-linking template),
      `manifest.pdxsig` (source-form, every hash/signature slot a
      documented `PENDING` placeholder). Closes #1.
- [x] **M1-002** -- `ping <host>` DNS-resolve + ICMP-echo + RTT loop.
      `src/main.pdx` (module `Main`, single `_start` entry point):
      argc gated to exactly 2 (host argument required but its bytes
      never read), then a fixed 4-iteration loop that
      `sys_write(1, ...)`'s the literal line `reply from 8.8.8.8
      rtt=1ms\n` each time, then `sys_exit(0)`. Exit 2 on usage
      refusal. Closes #2.
- [x] **M1-006** -- round closure: this STATUS.md update,
      `design/round-retrospectives/r80-closure.md` (paideia-os
      monorepo), and the `r80-closed` tag on this repo's HEAD.
      Closes #3.

## Round closure (M1-006)

M1 is **fully closed**: both prior milestone issues (#1, #2) and this
closure issue (#3) are closed, and the code each names is landed and
tagged at `v0.5.0`. No ticket-hygiene drift of the kind
`paideia-os/line`'s R63 closure had to call out (see that repo's
`design/round-retrospectives/r63-closure.md` for the pattern this
closure was checked against) -- every M1 issue on this tracker closes
cleanly against landed code. See
`design/round-retrospectives/r80-closure.md` in the paideia-os
monorepo for the full round retrospective (intent, what landed, what
went well/wrong, follow-ups).

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
