# ping

Minimal `ping <host>` -- DNS resolve + ICMP echo + RTT loop, printed as
four `reply from <ip> rtt=<n>ms` lines.

This is deliberately the FLOOR-scope tool, distinct from
`paideia-os/pdxping` (the full-featured ICMP client in the R100 wave).
`ping` exists to give the shell tree a tiny, dependency-free ICMP-shaped
command; `pdxping` is where sequence numbers, real RTT timing, packet
loss statistics, and the rest of a real ping implementation live.

## Scope (M1, v0.5.0)

- DNS resolution is a **WEAK stub**: `libpdx-net.net_resolve` is not
  yet linkable from a satellite repo, so `ping` always reports the
  fixed answer `8.8.8.8` regardless of the host argument. This is not
  a bug -- it is the documented floor behaviour until a real resolver
  is wired in.
- ICMP echo is a **WEAK stub** for a structural reason, not just a
  library-availability one: the kernel's `sys_icmp_echo` (SC+ ID 103,
  `src/kernel/core/syscall/handlers/sys_icmp_echo.pdx`) gates on
  `R_NET_PRIVILEGED_PROTOCOL`, which as of this wave admits exactly
  two callers -- boot context and PID 1 (init). Any ordinary ring-3
  process (this tool included) gets `-EPERM` on every real call. Doing
  the honest thing at v0.5.0 means not making that call at all and
  instead reporting a fixed `rtt=1ms` -- calling the syscall and
  discarding the `-EPERM` would be strictly worse (it would look like
  real I/O happened when it did not).
- Exactly 4 echoes, no packet-loss accounting, no sequence numbers, no
  `-c`/`-i`/`-t` flags.

## Spec

Full design lives in the paideia-os monorepo at
[`design/networking/r100-user-tools-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/networking/r100-user-tools-plan.md).

## License

MIT.
