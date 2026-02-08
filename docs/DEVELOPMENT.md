# SyscallGuard Development Guide


This document describes the development workflow, repo structure, and how to run demos.


---


## Repo Layout


```text
syscall-guard/
├── ebpf/                 # eBPF programs (kernel space)
│   ├── syscall_guard.bpf.c
│   └── vmlinux.h         # generated per machine
├── user/                 # user-space loader + analysis + policy
│   ├── main.c
│   ├── policy.c
│   └── logging.c
├── include/              # shared headers for user-space + config
│   ├── common.h
│   ├── policy.h
│   └── logging.h
├── scripts/              # training + analysis + helper scripts
│   ├── train_baseline.py
│   ├── analyze_logs.py
│   └── run_demo.sh
├── examples/             # small programs to generate syscall patterns
│   ├── normal_io.c
│   ├── net_connect.c
│   └── spawn_proc.c
├── configs/              # policies + settings
│   ├── policy.json
│   └── settings.json
├── docs/                 # project docs
│   ├── SETUP.md
│   ├── DEVELOPMENT.md
│   └── ARCHITECTURE.md
├── build/                # build outputs (gitignored)
├── Makefile
└── README.md
```

Notes:

ebpf/vmlinux.h is machine-generated and should be gitignored.

build/ should be gitignored.

## Development Workflow (recommended)
1) Generate vmlinux.h (once per machine)
```bash
sudo bpftool btf dump file /sys/kernel/btf/vmlinux format c > ebpf/vmlinux.h
```
3) Build
```bash
make
```
5) Run (requires root)
```bash
sudo ./build/syscall-guard --pid <PID>
```

(Exact flags will be documented as the CLI stabilizes.)

## How SyscallGuard Works (high level)

Two components:

1. eBPF program (kernel space)
* hooks syscall entry/exit tracepoints
* captures syscall id + selected arguments
* pushes events to user space (ring buffer / perf buffer)

2. User-space controller
* loads eBPF program via libbpf
* reads events
* applies detection:
  * rule-based (policy violations)
  * baseline-based (deviation from learned behavior)
* prints + logs alerts (JSON)

## Demos / Test Programs

Example programs in examples/ are used to produce predictable syscall patterns.

Build examples (future placeholder):
```bash
make examples
```
Run examples:
```bash
./build/examples/normal_io
./build/examples/net_connect
./build/examples/spawn_proc
```

## Baseline Training Mode

Goal:
* learn a program’s “normal” syscall profile
* compare runtime behavior to baseline

Workflow (planned):
1. Train baseline:
```bash
sudo ./build/syscall-guard train --cmd "./build/examples/normal_io" --out baseline.json
```

2. Monitor with baseline:
```bash
sudo ./build/syscall-guard monitor --cmd "./build/examples/normal_io" --baseline baseline.json
```

## Logging Format (planned)

All events/alerts should support JSONL output for analysis:
* logs/events.jsonl
* logs/alerts.jsonl

Example alert fields:
* timestamp
* pid / comm
*syscall_name
* reason (policy / baseline deviation)
* severity score

## Performance Testing (planned)

We will measure overhead under different settings:
* trace all syscalls vs select syscalls
* rule-only vs rule+baseline
* argument capture on/off (high cost)

Measurement tools:
* time
* perf stat
* CPU/memory sampling

Example (placeholder):
```bash
/usr/bin/time -v sudo ./build/syscall-guard monitor --cmd "./build/examples/normal_io"
perf stat -d sudo ./build/syscall-guard monitor --cmd "./build/examples/normal_io"
```

## Useful Debug Commands

Show loaded BPF programs:
```bash
sudo bpftool prog show
```
Show maps:
```bash
sudo bpftool map show
```
Kernel trace logs:
```bash
sudo dmesg -w
```
(If using bpf_printk / debug traces, this helps.)

## Common Pitfalls
vmlinux.h missing
You must generate it:
```bash
sudo bpftool btf dump file /sys/kernel/btf/vmlinux format c > ebpf/vmlinux.h
```

kernel-devel mismatch
Install headers for the running kernel:
```bash
sudo dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

permissions
Loading eBPF usually requires root:
* run with sudo

