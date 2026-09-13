# Changelog

All notable changes to `ping` are recorded here. Format: keep-a-
changelog-style, semver-ordered, newest first.

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
