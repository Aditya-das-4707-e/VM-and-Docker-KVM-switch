# VM and Docker KVM Switch

A small, focused guide and helper for switching kernel-based virtualization (KVM) on/off on Linux to make VirtualBox and Docker Desktop (or other KVM-using tools such as QEMU or Android Studio emulator) work reliably on the same machine.

Many Linux systems can only have either KVM enabled (for Docker Desktop, QEMU, Android emulator, etc.) or KVM modules unloaded (needed for older VirtualBox versions). This repository documents the quick commands and provides a minimal workflow to switch between the two modes.

## Table of Contents
- About
- When to use each mode
- Commands
  - MODE 1 — VirtualBox mode (disable KVM)
  - MODE 2 — Docker Desktop mode (enable KVM)
- Quick toggle script (optional)
- Verification
- Safety & troubleshooting
- Contributing
- License
- Author

## About
This repo contains a simple text file describing how to enable or disable KVM modules so you can run VirtualBox or Docker Desktop and other KVM-dependent virtualization tools without conflicts.

The instructions are deliberately minimal — they use modprobe to insert or remove kernel modules and show how to check the result.

## When to use each mode
- MODE 1 — VirtualBox mode (disable KVM): Use this when you want to run VirtualBox and it’s having issues because KVM modules are present.
- MODE 2 — Docker Desktop mode (enable KVM): Use this when you want Docker Desktop, the Android Studio emulator, QEMU, or any other tool that relies on KVM.

## Commands

### MODE 1 — VirtualBox mode (disable KVM)
Use this when you want to run VirtualBox.

Run as root or use sudo:
```bash
sudo modprobe -r kvm_intel kvm
```

Check that KVM modules are unloaded:
```bash
lsmod | grep kvm
```
Expected: no output (should be empty). Now VirtualBox should work properly.

### MODE 2 — Docker Desktop mode (enable KVM)
Use this when you want Docker Desktop, Android Studio emulator, QEMU, etc.

Run as root or use sudo:
```bash
sudo modprobe kvm
sudo modprobe kvm_intel
```

Check that KVM modules are loaded:
```bash
lsmod | grep kvm
```
Expected: output showing `kvm` and `kvm_intel` (or the appropriate vendor module, e.g., `kvm_amd` on AMD systems).

Note: If your CPU uses AMD virtualization, replace `kvm_intel` with `kvm_amd` where appropriate.

## Quick toggle script (optional)
You can create a small script to switch modes quickly. Example (save as `kvm-switch.sh`, make executable with `chmod +x kvm-switch.sh`):

```bash
#!/usr/bin/env bash
# Simple KVM on/off toggler
set -e

usage() {
  echo "Usage: $0 [virtualbox|docker|status]"
  exit 1
}

case "$1" in
  virtualbox)
    echo "Switching to VirtualBox mode (disabling KVM)..."
    sudo modprobe -r kvm_intel kvm || true
    lsmod | grep kvm || true
    ;;
  docker)
    echo "Switching to Docker mode (enabling KVM)..."
    sudo modprobe kvm
    # use kvm_intel or kvm_amd depending on CPU vendor
    sudo modprobe kvm_intel || sudo modprobe kvm_amd || true
    lsmod | grep kvm || true
    ;;
  status)
    echo "KVM modules status:"
    lsmod | grep kvm || echo "No kvm modules loaded"
    ;;
  *)
    usage
    ;;
esac
```

Be careful running scripts that use sudo; inspect before running.

## Verification
- After disabling: `lsmod | grep kvm` should return nothing.
- After enabling: `lsmod | grep kvm` should show `kvm` and `kvm_intel` (or `kvm_amd`).

You can also check VirtualBox and Docker behavior after switching:
- Start VirtualBox VM when in VirtualBox mode.
- Start Docker Desktop or run a QEMU VM when in Docker mode.

## Safety & troubleshooting
- Use the correct CPU vendor module:
  - Intel: kvm_intel
  - AMD: kvm_amd
- If a module fails to remove because it's in use, find and stop the process using it (e.g., a running VM) before removing.
- Kernel updates or driver changes may change module names or behavior.
- Persistent configuration changes (blacklisting modules) are possible but are more intrusive — prefer the manual toggling approach unless you know what you’re doing.
- If VirtualBox still complains after unloading KVM modules, reboot and try again (some systems may require a reboot to fully clear module state).
- When enabling KVM, ensure your CPU virtualization support (VT-x / AMD-V) is enabled in BIOS/UEFI.

## Contributing
This repository is intentionally small and focused. If you have improvements (more complete scripts, distro-specific notes, service checks, or safety improvements), open an issue or submit a PR.

Please include:
- The distribution and version you tested on (e.g., Ubuntu 24.04.3 LTS x86_64).
- CPU vendor (Intel/AMD).
- Exact commands or script changes and why they are needed.

## Author
Aditya-das-4707-e

---

This README collects the short instructions from the repository and expands them with context, example script, safety notes, and troubleshooting steps to make the repository easier to use.
