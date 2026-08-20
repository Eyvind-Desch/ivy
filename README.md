# ivy

> Working name. Pick the real one before the first push - it becomes the image
> URL and renaming later means rebasing every installation by hand.

A modern Linux desktop for Dell laptops: fast, genuinely customizable, designed
rather than themed, and able to run Windows software at native speed.

**The entire operating system is defined in [`recipes/`](recipes/).**
Everything else in this repo supports those files.

Reference machine: **Dell Inspiron 14 5440** ("metari") - Core 7 150U, 16 GB,
Intel Xe + Nvidia MX570 A. Specs and known issues in
[`docs/hardware/inspiron-baseline.md`](docs/hardware/inspiron-baseline.md).

## How it works

You do not install this OS by copying files onto a disk. CI builds it into a
signed container image; machines boot that image directly and update by pulling
a newer one. A bad update is undone by picking the previous entry in the boot
menu.

```
recipes/  ->  GitHub Actions  ->  signed OCI image  ->  your laptop
                                    |
                                    +->  ISO (only to install on a new machine)
```

## Two variants

Both build from every push and share [`recipes/common.yml`](recipes/common.yml),
so features are written once.

| Image | Graphics | Use |
|---|---|---|
| `ivy` | Intel only, dGPU absent | **default** - best battery, no Secure Boot or driver coupling |
| `ivy-nvidia` | + Nvidia MX570 open drivers | for measuring what the dGPU actually costs |

Switching between them is one command and a reboot. That is deliberate: the
dGPU tradeoff should be settled by measured battery numbers, not by argument.
Reasoning in [`docs/decisions/0002-graphics.md`](docs/decisions/0002-graphics.md).

## Layers

| Layer | What | Where | State |
|---|---|---|---|
| L0 | Base OS - kernel, Plasma 6, Wayland, dev tooling | inherited from Aurora DX | done |
| L1 | Dell hardware: power, suspend, audio, Fn keys | `files/`, `recipes/` | **next** |
| L2 | Windows apps: isolated prefixes, `.exe` installer UX, VM fallback | `files/` | **v0 written, untested** |
| L3 | Identity: design system, Plasma theme, branding | `design/` | planned |
| L4 | Experience: installer, first-run, update UI | | planned |

L0 already boots. Every layer after it is refinement - the project is never in
a state where you have nothing working.

## Roadmap

**Week 0 - do not skip.** Read
[`docs/hardware/install-plan.md`](docs/hardware/install-plan.md) before touching
the laptop. It has 680 GB of data on it and Dell firmware has three specific
ways to eat that data. Save your BitLocker key first.

**Week 1 - first boot.** Install stock Aurora DX (on an external USB-C NVMe -
zero risk to Windows) and fill in
[`docs/hardware/inspiron-baseline.md`](docs/hardware/inspiron-baseline.md).
That measured list is the real spec. In parallel: push this repo, let CI build,
rebase the machine onto your own image. You now have your own OS running.

**Weeks 2-4 - L1.** Work the broken list, hardest-hitting item first. Battery
drain in sleep and audio are the usual Dell offenders. This is what makes it
*yours for this hardware*.

**Weeks 4-8 - L2.** The feature nobody else gets right: double-click a `.exe`,
get a real installer window, get a real app in the menu with its real icon that
uninstalls cleanly. No terminal, no prefix management, no "which Wine version".
Design in [`docs/decisions/0003-windows-apps.md`](docs/decisions/0003-windows-apps.md);
v0 is `files/system/usr/bin/ivy-winapp` - written, never run, debug it first.

**Month 3+ - L3.** Design tokens first - type scale, spacing, color, motion -
then implement as a Plasma theme. Design decisions before CSS.

**Month 4 - daily-drive it for 30 days.** Every annoyance becomes an issue.
This is the step that separates a real OS from a nice screenshot.

## Setup (once)

1. **Create the GitHub repo** and push this directory to it.
2. **Generate the signing key** (needs Linux/WSL2 with `cosign` installed):
   ```sh
   cosign generate-key-pair          # leave the password empty
   ```
   Commit `cosign.pub`. Put the contents of `cosign.key` into a repository
   secret named `SIGNING_SECRET`, then delete the local file - it is
   `.gitignore`d, but do not rely on that.
3. **Push.** Watch the `build-os` workflow. First run takes ~15 minutes.
4. **Install on the Dell** - follow
   [`docs/hardware/install-plan.md`](docs/hardware/install-plan.md), then:
   ```sh
   sudo bootc switch --enforce-container-sigpolicy ghcr.io/<your-user>/ivy:latest
   systemctl reboot
   ```
5. **ISO** (only when installing on a fresh machine): run the `build-iso`
   workflow manually and download the artifact.

## Working on Windows

Builds run in GitHub Actions, so no local Linux is required to ship. For fast
iteration you want **WSL2 with podman** to build the image locally before
pushing, and a drive you can boot test images from.

Full reasoning in
[`docs/decisions/0004-dev-environment.md`](docs/decisions/0004-dev-environment.md).
Short version: **WSL2 builds, a VM iterates, real hardware decides.** WSL2
cannot run this OS at all - bootc is structurally incompatible with it.

Once installed in the VM, every subsequent test is `sudo bootc upgrade &&
systemctl reboot` - about thirty seconds, no ISO rebuilds.

**Executable bits.** Windows has no Unix permission bits, so scripts committed
from here land in the image as non-executable and silently do nothing. Git can
still record the bit - set it explicitly for every script you add under
`files/system/usr/bin/`:

```sh
git update-index --chmod=+x files/system/usr/bin/ivy-winapp
```

This will bite you at least once. When something "installed fine but does
nothing", check this first.

## Repo layout

```
recipes/
  common.yml         everything shared by both variants
  recipe.yml         default image (Intel graphics)
  recipe-nvidia.yml  variant with the MX570 stack
files/               overlays copied onto the image, plus build scripts
design/              design tokens, mockups, branding
docs/
  decisions/         why we chose what we chose - read 0001 first
  hardware/          install plan and measured behaviour of the machine
```

## Unverified

The BlueBuild module syntax and action versions in this scaffold were written
from current upstream docs but have not been run. Expect to correct exact YAML
keys on the first CI run - the structure is right, individual field names may
need a look at <https://blue-build.org/reference/recipe/>.
