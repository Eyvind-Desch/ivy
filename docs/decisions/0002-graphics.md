# 0002 - Ship Intel-only by default, Nvidia as a measurable variant

Status: accepted - 2026-08-20

## Context

The reference machine (Dell Inspiron 14 5440, "metari") is a muxless hybrid
laptop:

- **Intel Core 7 150U** with Xe-class integrated graphics - drives the panel
- **Nvidia GeForce MX570 A, 2 GB** - GA107, Ampere; render-offload only, wired
  to no display output

Stated priorities are Windows productivity apps, software development, battery
life and interface quality. **Gaming is explicitly not a priority.**

The base image we chose in 0001 (`aurora-dx`) contains no Nvidia drivers at
all, so this is a decision we cannot avoid making.

## Decision

The default image is **Intel-only**. The Nvidia stack ships as a separate
variant (`recipe-nvidia.yml` -> `nyx-nvidia`), built by the same CI run.

## Why

Against our actual priorities the MX570 A earns very little:

- **2 GB of VRAM.** Marginal for modern GPU work; it will not rescue a heavy
  3D workload, and nothing in our priority list is GPU-bound.
- **It is a battery liability.** On hybrid laptops the dGPU must reach D3cold
  when idle or it silently costs several watts. Ampere enables dynamic power
  management automatically, but regressions across driver releases are a
  recurring, well-documented problem. Battery life is priority #4 on our list;
  the dGPU is the single largest threat to it.
- **It couples us to the driver.** Nvidia kernel modules must match the
  running kernel, need Secure Boot key enrollment via mokutil, and need VRAM
  preservation services to survive suspend. Every one of those is a way for an
  update to break a machine that would otherwise be fine.

The integrated Xe graphics handle Plasma compositing, video decode and
multi-monitor output without any of that.

## Why keep the variant at all

Because this is a judgement call about a tradeoff we have not yet measured,
and the architecture makes measuring it nearly free. Both images build from
one push. Switching between them is one command and a reboot:

```sh
sudo bootc switch ghcr.io/<user>/nyx-nvidia:latest && systemctl reboot
```

Run the battery baseline on each, compare real numbers, and this decision
either stands on evidence or gets reversed on evidence. Deciding by argument
when you can decide by measurement is a waste.

## Consequences

- Default installs have no CUDA and no dGPU offload. If that turns out to
  matter for game-engine work, the variant is one rebase away.
- Two images to keep building. Cost is CI minutes, not maintenance - they
  share `recipes/common.yml`, so features are written once.
- The two images have different names, so a machine's variant choice survives
  updates. It only changes when someone deliberately rebases.

## Revisit if

Measured idle drain with the dGPU present turns out to be negligible (making
the default pointlessly restrictive), or GPU-accelerated work becomes a real
part of the daily workload.
