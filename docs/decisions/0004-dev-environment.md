# 0004 - Development environment: WSL2 builds, VM iterates, hardware decides

Status: accepted - 2026-08-20

## Context

Development happens on the target machine itself (Windows 11 Home, Inspiron 14
5440) - which is also the machine we eventually want to install onto. Until we
are ready to touch its disk, we need somewhere to run and look at the OS.

Three candidates: WSL2, a desktop VM, or real hardware via external drive.

## Decision

Use all three, for three different jobs. They are not alternatives.

| Tool | Job | Cannot do |
|---|---|---|
| **WSL2 + podman** | build images locally, `cosign`, git exec bits | run our OS |
| **VMware Workstation Pro** | daily iteration: boot, UI/design, L2 Windows apps | battery, suspend, hardware, real speed |
| **External USB-C NVMe** | the truth: power, WiFi, audio, thermals, feel | nothing - this is the real thing |

## Why WSL2 cannot run this OS

Not a configuration problem - a structural one. **bootc explicitly does not
support WSL2.** WSL runs Microsoft's own custom kernel, which is incompatible
with bootc-based images, and `bootc-image-builder` produces no WSL output
format for exactly that reason.

Even if it booted, WSL is missing precisely the parts we are building:

- no real boot process, so no ostree deployments and no rollback - the entire
  safety model from 0001 is absent
- Microsoft's kernel, so no Dell kernel modules, no `platform_profile`, no
  power management, no suspend - our whole L1 layer is untestable
- no display server session; WSLg forwards individual app windows, so a full
  Plasma shell would run nested and janky - useless for judging L3 design work
- Wine inside WSL2 would mean Windows -> Linux VM -> Wine -> Windows app, on a
  host that is already Windows. Meaningless as a test.

WSL2 is still worth setting up - as the **build environment**. Podman for local
image builds before pushing, `cosign` for the signing key, and a Linux git that
handles executable bits correctly (see the README warning about `--chmod=+x`).

## Why VMware Workstation Pro for the VM

Broadcom made Workstation Pro **free for all use** in November 2024 - personal,
educational and commercial, with no license key. A Broadcom account is needed
for the download; that is the only friction.

It matters that from 17.5.2 onward it layers on the Windows Hypervisor
Platform, so it **coexists with WSL2** rather than fighting it for the
hypervisor. On Windows 11 Home that is decisive: Hyper-V Manager is not
available on Home, and we want WSL2 running at the same time anyway.

VirtualBox is the fallback if the Broadcom account is unacceptable - free and
simpler, but weaker 3D acceleration, which is the one thing that matters when
evaluating a desktop shell.

Suggested VM shape: UEFI firmware, Secure Boot off, 4 vCPU, 6 GB RAM, 80 GB
disk, 3D acceleration enabled.

> Disk check: the laptop has roughly 274 GB free. An 80 GB VM fits, but it is a
> meaningful bite. Putting the VM on an external drive is reasonable.

## The workflow that makes this fast

The obvious approach - rebuild an ISO for every change - is far too slow. Do
this instead:

1. Install stock Aurora DX into the VM **once**, from the upstream ISO.
2. Rebase it onto our image once CI has published it.
3. For every subsequent change: push, wait for CI, then in the VM:

```sh
sudo bootc upgrade && systemctl reboot
```

Roughly thirty seconds per iteration, and no ISO builds at all. Snapshot the VM
right after step 2 so a broken image is one rollback away.

This is the same mechanism real users get for updates, so we are testing the
update path continuously rather than only the install path.

## What the VM will lie to you about

Do not draw conclusions from a VM on any of these:

- **Battery life and idle power** - no meaningful measurement exists in a VM
- **Suspend and resume** - the hardest L1 problem, and completely untestable
- **Anything Dell-specific** - SMBIOS, thermals, fan curves, Fn keys
- **Graphics behaviour** - the MX570 question from 0002 needs real hardware
- **Speed** - a 15 W CPU running a VM will feel slower than the real install

Which is to say: the VM validates that the OS *works*. Only the external drive
validates that it is *good*. Both stages are necessary; do not let the VM's
convenience delay ordering the enclosure.

## Consequences

- Two environments to keep in sync. Acceptable: both consume the same published
  image, so there is no separate "dev build" to drift.
- L2 work (`ivy-winapp`) can be developed almost entirely in the VM - Wine does
  not need a GPU for productivity software. Good news for progress this week.
- The L2 v4 Windows-VM fallback needs nested virtualisation and will not be
  developed inside the VM. That is a real-hardware task, later.
