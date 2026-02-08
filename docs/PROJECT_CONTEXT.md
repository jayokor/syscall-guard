# SyscallGuard – Project Context

## 1. Project Overview

SyscallGuard is a semester-long OS security project for a graduate operating
systems course (CIS 657).

The project implements a **system call monitoring and analysis tool** using
modern Linux primitives (eBPF + libbpf with CO-RE). The system observes runtime
syscall behavior of user-space programs and flags abnormal or policy-violating
activity.

The project is intentionally **security-adjacent**, mirroring techniques used
by modern Endpoint Detection & Response (EDR) and runtime security tools, while
remaining scoped and educational.

---

## 2. Motivation & Problem Statement

Many modern attacks and misuses of software systems manifest as **unusual
interactions with the operating system**, especially at the system call
boundary between user space and the kernel.

Although modern EDR tools use syscall-level telemetry, open challenges remain:
- performance overhead
- false positives
- explainability of alerts
- tradeoffs between broad monitoring and selective tracing

This project explores those tradeoffs by implementing a small, explainable
syscall monitoring system from first principles.

---

## 3. Scope & Design Philosophy

### In scope
- Observation and analysis of syscalls
- Rule-based detection (policy violations)
- Baseline-based detection (deviation from learned behavior)
- Performance measurement
- Explainable alerts

### Explicitly out of scope
- Full malware detection
- Kernel modification
- Production-grade enforcement
- Heavy ML models
- Bypassing SELinux or kernel protections

Focus is on **clarity, correctness, and learning**, not completeness.

---

## 4. Technical Stack (Locked In)

### Operating System
- Rocky Linux 9.x (Workstation)
- Kernel: 5.x (RHEL 9 series)
- Architecture: x86_64
- SELinux: Enforcing (default)

### Kernel-space
- eBPF programs
- CO-RE (Compile Once, Run Everywhere)
- Tracepoints: `raw_syscalls/sys_enter` (initially)

### User-space
- C (libbpf loader + event handling)
- Python (analysis, baseline scripts)
- JSON/JSONL logging

### Tooling
- clang / LLVM
- libbpf + bpftool
- gcc / make
- perf (optional)
- Git + GitHub
- VS Code

---

## 5. Repository Structure
```bash
syscall-guard/
├── ebpf/ # eBPF programs (kernel space)
│ ├── syscall_guard.bpf.c
│ └── vmlinux.h # generated per machine (gitignored)
├── user/ # user-space loader and logic
│ ├── main.c
│ ├── policy.c
│ └── logging.c
├── include/ # shared headers
│ ├── common.h
│ ├── policy.h
│ └── logging.h
├── scripts/ # analysis / baseline scripts
│ ├── train_baseline.py
│ └── analyze_logs.py
├── examples/ # small test programs
│ ├── normal_io.c
│ ├── net_connect.c
│ └── spawn_proc.c
├── configs/ # policies and runtime config
│ ├── policy.json
│ └── settings.json
├── docs/
│ ├── PROJECT_CONTEXT.md
│ ├── SETUP.md
│ ├── DEVELOPMENT.md
│ └── ARCHITECTURE.md
├── build/ # build artifacts (gitignored)
├── Makefile
└── README.md
```

---

## 6. Current Status

- Rocky Linux installed and configured
- Development toolchain installed
- GitHub repo initialized with structure and docs
- `SETUP.md` and `DEVELOPMENT.md` written
- Ready to implement first functional prototype

---

## 7. Current Milestone (Active)

**Week 1 – Minimal Syscall Tracer**

Goals:
- Load an eBPF program via libbpf
- Attach to `tracepoint/raw_syscalls/sys_enter`
- Stream syscall events to user space
- Print:
  - PID
  - process name
  - syscall ID

This establishes the foundation for all later features.

---

## 8. Planned Milestones (High Level)

- Week 1: Basic syscall tracing
- Week 2: PID filtering + syscall ID → name mapping
- Week 3: Policy-based detection
- Week 4: Baseline learning mode
- Week 5: Performance evaluation
- Week 6+: Demo polish, reporting, optional extensions

---

## 9. Notes for Collaborators (Human or AI)

- Treat this document as the authoritative project summary
- Detailed setup steps live in `SETUP.md`
- Development workflow lives in `DEVELOPMENT.md`
- Keep scope controlled to avoid over-engineering
