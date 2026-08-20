# 0003 - Windows applications: one prefix per app, zero jargon

Status: accepted - 2026-08-20

## Context

The headline feature: run Windows software at native speed. The engine for that
already exists and we are not rebuilding it (0001). Wine translates Win32 calls
to Linux ones with the CPU executing natively.

So the problem is **not** compatibility. It is that the existing experience is
hostile:

> Download `setup.exe`. Install Bottles. Create a bottle. Choose a runner
> version. Guess which winetricks verbs it needs. Run the installer. Find the
> binary inside the prefix. Hand-write a launcher. Hunt down an icon.

On Windows that whole sequence is: double-click, Next, Next, Finish, and the
app is in the menu with its icon.

**Closing that gap is the product.** It is a design and integration problem,
which is exactly the kind we can win at as a small team - unlike writing a
compatibility layer, which we would lose at.

## Decision

### 1. One app, one prefix. Always.

Each installed Windows app gets its own isolated prefix under
`~/.local/share/nyx/apps/<id>/`. Never a shared prefix.

Shared prefixes are how every Wine setup eventually dies: one installer drops a
broken runtime and unrelated apps start failing weeks later, with no way to
attribute the damage. Isolation costs disk space, which we have, and buys
attributability and clean uninstall, which we do not otherwise get.

Uninstall is therefore `rm -rf` of one directory plus its launchers. No
leftovers, no registry rot, no "reinstall Windows to fix it".

### 2. The vocabulary is banned from the UI

The words **prefix, bottle, runner, winetricks, WINEPREFIX, DXVK** never appear
in the default interface. They are implementation detail. A user installing a
tax program does not need to learn Wine's architecture, any more than a Windows
user needs to know what a DLL search path is.

They stay available in an advanced view and in the CLI, because the escape
hatch matters when something breaks. They are just not the default experience.

### 3. Launchers are generated, not hand-made

After an installer finishes, we detect what it created and produce real
`.desktop` entries with the app's own icon extracted from its executable. The
app appears in the Plasma menu next to native apps, is searchable, and pins to
the task manager like anything else.

This is the moment the OS feels different from every other distro. It is worth
disproportionate polish.

### 4. Honesty before effort

A compatibility database ships with the image: app -> known status, required
dependencies, notes. If an app is known-broken we say so **before** the user
spends fifteen minutes discovering it. If it needs a specific runtime we
install it silently rather than failing with a Win32 error dialog.

Seeded from WineHQ AppDB plus our own testing. It updates with the OS image,
which means a compatibility fix ships like a system update - one of the real
advantages of image-based delivery.

### 5. The VM is the fallback, and we say so plainly

Wine cannot run software that needs Windows kernel drivers. In practice:
kernel-level anti-cheat, Adobe Creative Cloud, and the full Office desktop
suite.

For those, a Windows VM with seamless RDP windowing (the WinApps approach) -
the app appears as an ordinary window, not a desktop-in-a-window. Fine for
productivity software, useless for 3D.

This needs a licensed Windows image, which is the practical reason
`docs/hardware/install-plan.md` says keep the existing licence. With 16 GB of
RAM, budget 4-6 GB for the VM.

**We state this limitation in the product, up front.** Overpromising here is
how projects like this lose trust permanently.

## Build order

Each stage is independently useful. Do not attempt them at once.

| Stage | Deliverable | Proves |
|---|---|---|
| **v0** | CLI: `.exe` in, isolated prefix, app installed | the pipeline works at all |
| **v1** | Automatic `.desktop` + real icon extraction | the "wow" moment - stop here and evaluate |
| **v2** | Compatibility database with pre-applied fixes | it works on apps we did not hand-test |
| **v3** | Kirigami GUI: progress, app list, uninstall | it is usable by someone who is not you |
| **v4** | Seamless Windows VM fallback | the 10% Wine cannot reach |

v0 is in `files/system/usr/bin/nyx-winapp` - **written but never executed.**
Treat it as a specification you debug, not working code.

## Implementation notes

**Do not write a Wine manager.** The orchestrator is thin: it shells out to
`wine`/`umu`, and its real work is prefix lifecycle, launcher generation and
the compatibility database. Bottles stays installed as the manual escape hatch
for anything the automation cannot handle.

**Let `winemenubuilder` do the hard part.** Wine already watches for Start Menu
shortcuts during installation and emits `.desktop` files with extracted icons.
Rather than parsing `.lnk` files ourselves, we snapshot the applications
directory before install, diff after, and adopt whatever appeared. Far less
code and it tracks Wine's own improvements. `icoutils` is the fallback for
installers that create no shortcut.

**GUI toolkit: Kirigami/QML.** We chose Plasma 6 (0001); matching Qt keeps the
app native-looking for free and shares the design system with L3.

## Consequences

- Disk usage is higher than a shared prefix. Acceptable, and the isolation is
  worth more than the gigabytes.
- We own a compatibility database - a real ongoing maintenance commitment, and
  the thing that will most differentiate the OS after a year.
- A per-app prefix means per-app first-run cost (a few seconds of `wineboot`).
  Hide it behind the installer progress UI in v3.

## Revisit if

Per-app prefixes prove unworkable on disk (they will not, at ~400 MB each), or
upstream produces something that already does this well - in which case we
should adopt it rather than compete with it.
