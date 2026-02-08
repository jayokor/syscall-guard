# SyscallGuard Setup (Rocky Linux 9)

This document describes how to set up a fresh Rocky Linux 9 system to build and run SyscallGuard.

## Supported Platforms
- Tested on: Rocky Linux 9.x (Workstation)
- Should work on: AlmaLinux 9, RHEL 9, Fedora (with minor package name differences), Ubuntu (different package manager)

---

## 1. Update system

```bash
sudo dnf update -y
sudo reboot
```

After reboot:
```bash
uname -r
```

## 2. Install core build tools:
```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y \
git make cmake pkgconf \
elfutils-libelf-devel zlib-devel \
jq
```

Verify:
```bash
gcc --version
make --version
git --version
```

## 3. Install kernel headers + kernel-devel (required)
```bash
sudo dnf install -y \
kernel-devel-$(uname -r) \
kernel-headers-$(uname -r)
```

Verify:
```bash
ls /usr/src/kernels/$(uname -r)
```

## 4. Install LLVM/Clang (required for eBPF)
```bash
sudo dnf install -y \
clang llvm llvm-devel lld
```

Verify:
```bash
clang --version
llvm-config --version
```

5. Install libbpf + bpftool (industry standard)

```bash
sudo dnf install -y \
libbpf libbpf-devel \
bpftool
```

Verify:

```bash
pkg-config --modversion libbpf
sudo bpftool prog show
```

6. Optional tools (recommended)

```bash
perf (useful for overhead measurement)
sudo dnf install -y perf
perf --version
strace (useful for cross-checking)
sudo dnf install -y strace
strace -V
Python (log parsing + baseline scripts)
sudo dnf install -y python3 python3-pip
python3 --version
pip3 --version
```

8. GitHub SSH (recommended)

Generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

Add the public key to GitHub, then verify:

```bash
ssh -T git@github.com
```

8. Clone the repo

```bash
git clone git@github.com:<your-username>/syscall-guard.git
cd syscall-guard
```

10. Generate vmlinux.h (required for CO-RE)

SyscallGuard uses BTF + CO-RE. Generate ebpf/vmlinux.h:

```bash
sudo bpftool btf dump file /sys/kernel/btf/vmlinux format c > ebpf/vmlinux.h
ls -lh ebpf/vmlinux.h
10. Quick eBPF feature check
bpftool feature probe | less
```

Look for `CONFIG_BPF=y` and `CONFIG_BPF_SYSCALL=y`.

11. Build & run (placeholder)

(Commands here will be updated once the Makefile is finalized.)

Example (future):

```
make
sudo ./build/syscall-guard --pid <PID>
Troubleshooting
"kernel-devel-$(uname -r) not found"
```

Your running kernel might be newer than installed headers.

Fix:

```bash
sudo dnf update -y
sudo reboot
sudo dnf install -y kernel-devel-$(uname -r) kernel-headers-$(uname -r)
USB / permission issues
```

eBPF loading typically requires root:

Use sudo for the runner / loader.

SELinux

Rocky uses SELinux by default. We generally keep it enabled.
Check status:

getenforce
