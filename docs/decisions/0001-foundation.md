# 0001 - Foundation: image-based OS on Fedora Atomic

Status: accepted - 2026-08-20

## Context

Goal: our own Linux-based desktop OS for Dell Inspiron/Vostro laptops, able to
run Windows software at native speed, modern-looking, fast, customizable, and
practical enough to use as a daily driver.

Constraint that shapes everything: **small team, needs visible progress fast.**

## Decision

Build a **distribution**, not a kernel. Define the whole OS as a declarative
recipe in git, built by CI into a signed OCI container image, installed via
`bootc`/rpm-ostree.

Base: `ghcr.io/ublue-os/aurora-dx` (Fedora Atomic + KDE Plasma 6 + dev tooling).

## Why

- **The OS is one readable file.** `recipes/recipe.yml` is the entire system.
  A new contributor understands it in five minutes.
- **Updates are atomic and reversible.** A bad update is rolled back by picking
  the previous entry in the boot menu. We cannot brick our own users.
- **Reproducible.** Same recipe, same image, byte for byte. No snowflake
  machines, no "works on mine".
- **We inherit maintenance we did not sign up for.** Kernel, Mesa, Wayland,
  Plasma, codecs and hardware enablement all come from upstream, already tested.
  Our work is the delta that makes it *ours*.
- **Free CI.** GitHub Actions builds and signs the image; ISOs on demand.

## What we explicitly are NOT doing

- **Not writing a kernel or drivers.** 10+ years, and Windows software would
  never run on it.
- **Not writing a Windows compatibility layer.** Wine already translates Win32
  calls to Linux ones with the CPU running natively - roughly Windows speed.
  Reimplementing it is decades of work for a worse result. We integrate it.
- **Not writing a Wayland compositor.** KDE Plasma 6 is customizable enough to
  carry our identity. Revisit only if the design genuinely cannot be expressed
  in it.

## Known limitations we accept

Wine cannot run software that needs Windows kernel drivers. In practice:
kernel-level anti-cheat, Adobe Creative Cloud, and the full MS Office desktop
suite. Mitigation is a seamless Windows VM (L2, later), where those apps appear
as ordinary windows - not a fix, a fallback. This is a permanent property of
the platform and must be stated honestly to users.

## Consequences

- The base OS is read-only. Software installs as Flatpak, container, Homebrew,
  or gets baked into the image. Occasionally friction; in exchange, the system
  cannot rot.
- A cosign key pair signs the image. `cosign.key` is **secret** (stored as the
  `SIGNING_SECRET` GitHub secret, never committed); `cosign.pub` is committed
  so installations can verify updates. Losing the private key means every
  install has to re-trust a new one.
- We are downstream of Fedora's release cadence. Pin `image-version` to a
  number before a Fedora major release to control when we move.

## Revisit if

Fedora Atomic's read-only base blocks a core feature we cannot work around, or
Aurora upstream is abandoned. Escape hatch: the recipe can be rebased onto
plain `fedora-bootc` or `kinoite` with modest effort - that is why we depend on
Aurora only through one line of YAML.
