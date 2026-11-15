# Task 4: Traps

This branch implements the assembly, backtrace, and alarm exercises in the
[2026 MIT traps lab](https://pdos.csail.mit.edu/6.828/2026/labs/traps.html).
It starts from MIT's `traps` starter commit
`cdb46aaccca2b22dc7422667f186969d3b31b1fc` and is independent of `task-3`.

[answers-traps.txt](answers-traps.txt) contains the written answers, including
addresses verified against the generated assembly and the endianness example.

## Build and test in WSL

Use `~/xv6-task-4` as the checkout directory in Ubuntu.
Keep the build on Linux's filesystem in a path without spaces.

```sh
cd ~/xv6-task-4
make grade
python3 grade-lab-extra
make qemu
```

Inside xv6, run `bttest`, `alarmtest`, `alarmcheck`, `call`, `endian`, or
`usertests -q`. Exit QEMU with Ctrl-A followed by X. For another machine:

```sh
git clone --branch task-4 https://github.com/iHuzaifakhan/xv6-repository.git xv6-task-4
cd xv6-task-4
make grade
```

## Implementation details

- The frame-pointer helper and bounded stack walk support backtraces from
  `sys_pause` and `panic`.
- `sigalarm` validates the interval and executable user handler address.
  Address zero is permitted; zero intervals disable future alarms.
- Timer interrupts save the full trapframe, redirect the user program
  counter, and prevent nested handlers. `sigreturn` restores all registers,
  including the interrupted `a0`, before rearming the timer. Calls outside
  a handler return an error.
- Handlers can disable or reconfigure alarms before returning. Forked
  processes start with alarms disabled. Successful exec discards handlers
  and saved contexts referring to the old image; failed exec retains them.
- The saved trapframe is embedded in the process structure, so no additional
  dynamic allocation or mapped user page is needed for alarm state.
- ELF load headers separate executable kernel text from writable data.
  The small `forktest` binary uses the standard user linker layout. These
  avoid the starter's RWX-segment linker warnings with current binutils.

## Validation

Tests use Ubuntu WSL, GCC 15.2.0, QEMU 10.2.1, the default three virtual CPUs,
and 128 MiB RAM. MIT's grading script and `user/alarmtest.c` are unchanged.
The official grader checks the three backtrace addresses with `addr2line`,
runs all four alarm tests, and runs `usertests -q`.

`validation/grade-summary.txt` records the official grading results.
`validation/extra-checks.txt` records invalid arguments, reconfiguration and
disabling from handlers, fork state, exec from an active handler, the
backtrace output, the assembly examples, and the original `forktest`.
`call` prints `12 13`; `endian` prints `HE110 World`.

To inspect the generated example and resolve a backtrace address:

```sh
make user/_call
less user/call.asm
riscv64-linux-gnu-addr2line -e kernel/kernel 0xADDRESS
```

`time.txt` records one hour, rounding up the automated implementation and
validation time for this lab; it is not a claim about prior student effort.
