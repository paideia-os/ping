# Changelog

All notable changes to `ping` are recorded here. Format: keep-a-
changelog-style, semver-ordered, newest first.

## [1.0.0] - 2026-09-14 - Wave mu-04: real ICMP echo (ping#3)

### Changed
- **Real `sys_icmp_echo` (SC+ 103) call, replacing the v0.5.0 fixed
  `rtt=1ms` WEAK stub.** Each of the four attempts now: builds a real
  4-byte dst IPv4 buffer + an 8-byte payload, brackets a real
  `sys_icmp_echo` call with two real `sys_clock_read_ns` (SC+ 66)
  reads, and reports the outcome honestly:
  - success (kernel returned a real `rtt_ns`): `reply from 8.8.8.8
    rtt=<ms>ms` on fd 1, `<ms>` a genuine runtime `div`-by-1000000
    conversion.
  - `-EPERM` (the syscall's `R_NET_PRIVILEGED_PROTOCOL` gate admits
    only boot context / PID 1 -- this ring-3 process is neither, so
    this is the expected outcome on every real system today): `ping:
    icmp echo denied (EPERM) attempt_ns=<n>` on fd 2, `<n>` the real
    ping-side wall-clock bracket of the actual attempt.
  - `-EINVAL`: a defensive, symmetrically-handled branch (this tool's
    own argument construction never triggers it).
- DNS resolution is unchanged: still a fixed `8.8.8.8` WEAK stub, for
  the unrelated reason that `libpdx-net.net_resolve` is not yet
  linkable from a satellite repo.

### Known gaps
- `sys_icmp_echo`'s kernel-side admission gate means this binary's
  real call is expected to return `-EPERM` on every deployment until a
  future round widens that gate or grants ring-3 callers a real
  per-task ICMP capability -- no code change needed in this repo when
  that happens, since the real call + argument shape already exist.

## [0.5.0] - 2026-09-13

First tagged release. Closes ping#1 (M1-001 repo bootstrap) and
ping#2 (M1-002 DNS-resolve + ICMP-echo + RTT loop real body).

### Added
- **Repo bootstrap (ping#1, M1-001).** `README.md`, `LICENSE` (MIT),
  `caps.decl` (`KIND_USER` + `KIND_TTY`, both mandatory -- no network
  capability kind declared, since neither network-shaped operation
  makes a real syscall at this milestone; see `caps.decl`'s own
  header note), `tools/build.sh` + `link.ld` (mirrors
  `tools/user/pdxsock`'s satellite-linking template, `paideia-as
  >= 0.36.0`), `manifest.pdxsig` (source-form dual-sign manifest,
  every hash/signature slot a documented `PENDING` placeholder).

- **`ping <host>` DNS-resolve + ICMP-echo + RTT loop (ping#2,
  M1-002).** `src/main.pdx` (module `Main`, single `_start` entry
  point): argc gated to exactly 2 (a host argument is required but
  its bytes are never read at this floor scope), then a fixed
  4-iteration loop writes the literal line `reply from 8.8.8.8
  rtt=1ms\n` to fd 1 each time, then `sys_exit(0)`. Usage refusal
  (argc != 2) writes a diagnostic to fd 2 and exits 2.

  Both network-shaped fields are honest WEAK stubs, for two distinct
  reasons documented at length in `src/main.pdx`'s file header and
  `README.md`: DNS resolve is a library-linkage gap (`libpdx-net`
  is not yet satellite-linkable), while ICMP echo is a structural gate
  (the kernel's real `sys_icmp_echo`, SC+ ID 103, refuses every caller
  except boot context and PID 1 via
  `cap_check_r_net_privileged_protocol` -- an ordinary ring-3 `ping`
  process would get `-EPERM` on every real call, so the honest choice
  is not to make the call at all).
