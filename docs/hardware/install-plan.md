# Getting Linux onto metari safely

The machine has **680 GB of data on a 954 GB drive** and is running Windows.
That makes a real installation the riskiest step in the whole project, and the
one most likely to cost you a week.

**The good news: you can postpone it for months.** Stages 0 and 1 below cost
nothing, carry no risk, and cover everything except the L1 hardware layer.
Start there.

## Read this first - three ways to lose your data

Relevant to **Stage 2b (dual boot) only** - Stages 0 and 1 touch none of this.
They are specific to Dell laptops and all three are avoidable.

**1. BitLocker.** Dell ships Windows with device encryption on many configs.
Changing the boot order, disabling Secure Boot, or resizing the partition can
trigger a recovery-key prompt on next boot. If you do not have the key, the
data is gone - permanently, with no workaround.

> Before anything else: check `manage-bde -status` in an admin terminal.
> If protection is on, **save the recovery key somewhere off this laptop**,
> then `manage-bde -protectors -disable C:` before making firmware changes.

**2. Intel RST / "RAID On" SATA mode.** Dell often ships with the storage
controller in RST mode, which makes the NVMe drive **completely invisible to
the Linux installer**. The fix is switching BIOS to AHCI/NVMe - but flipping
it while Windows is installed makes *Windows* unbootable unless you prepare it
first (boot into safe mode once, from Windows, before switching).

Do this in order: save BitLocker key -> suspend BitLocker -> prepare Windows
for AHCI -> change BIOS -> boot Windows once to confirm -> then proceed.

**3. Only ~274 GB free.** Not enough for a comfortable dual boot with room to
grow. Do not start by shrinking to the minimum.

## Recommended path

Four stages, each reversible, each proving something before you risk more.
**The first two cost nothing and carry no risk at all** - and they cover the
large majority of the work. Do not let the later stages feel urgent.

### Stage 0 - VM only (start here, free)

A virtual machine is a file on disk. It cannot touch Windows, the bootloader,
or the partition table, no matter what we break inside it. Setup in
`docs/decisions/0004-dev-environment.md`.

Everything except L1 is fully developable here:

| Layer | In a VM? |
|---|---|
| L0 base image, CI, signing, the update mechanism | yes, completely |
| L2 Windows apps - Wine needs no GPU for productivity software | yes, completely |
| L3 design system, Plasma theming, shell | yes, completely |
| L4 installer, first-run, app management | yes, completely |
| L1 Dell hardware - battery, suspend, WiFi, audio | **no** |

That is months of work at zero cost and zero risk.

### Stage 1 - Live USB (free, ~20 minutes, no risk)

Any spare 8 GB+ USB stick. A Fedora KDE live session boots into RAM - nothing
is installed and no disk is written, so BitLocker and the partition table are
never involved. Power it off and the machine is untouched.

This answers the biggest open hardware question in twenty minutes:

```sh
lspci -nnk | grep -A3 -i network   # the WiFi chipset - the main unknown
lspci -nnk > lspci.txt             # everything else, for docs/hardware/
```

Then check by hand: WiFi, sound, touchpad, brightness keys, external display.

The kernel and drivers are identical to the ones in our image, so the answers
transfer directly. **Battery life and suspend are the exceptions** - a live
session runs from RAM and cannot measure those honestly.

### Stage 2 - Real install (pick one when L1 starts)

Only needed once battery, suspend and thermals actually matter. Two ways:

**a) External USB-C NVMe - costs money, zero risk**

An NVMe enclosure is a small aluminium case you put an M.2 SSD into - the same
form factor as the drive inside the laptop - connected over USB-C. Install
Linux onto it and boot it with F12 at startup.

- **Zero risk.** The internal drive is never touched. No partitioning, no
  BitLocker exposure, no bootloader changes. Unplug it and Windows is exactly
  as it was.
- Fast enough to genuinely daily-drive - USB 3.2 Gen 2 is ~1000 MB/s.
- Lets you complete the entire `inspiron-baseline.md` checklist, find the WiFi
  chipset, and measure battery before committing to anything.

**What to buy** (roughly 60-100 EUR total):

| Option | Price | Note |
|---|---|---|
| Pre-built external SSD (Samsung T7 Shield, Crucial X9/X10) | 500 GB ~50-60, 1 TB ~80-100 | Simplest - unbox and go |
| Enclosure + own NVMe | case ~20-30 + SSD ~35-45 | Cheaper, and the SSD can go internal later |
| USB flash drive | ~15 | **Avoid** for an installed OS - poor random writes, wears out fast |

Look for: USB 3.2 **Gen 2** (10 Gbit/s - Gen 2x2 is wasted money here), an
aluminium housing, and a **JMicron JMS583** or **Realtek RTL9210B** controller.
Cheap no-name enclosures overheat and drop the connection mid-write, which is
genuinely painful during an OS install. 500 GB is plenty.

Also enable USB boot support in BIOS, and note that suspend and power figures
on external boot are representative but not identical to an internal install.
Good enough for finding real problems.

**This purchase is optional.** It buys risk-free experimentation, not
capability - option (b) reaches the same place for free.

**b) Dual boot - free, but the risks at the top of this file are real**

Free up space in Windows first: get Windows down to roughly 400-450 GB used,
leaving ~400 GB for Linux. Shrink from *within Windows* (Disk Management),
never from the Linux installer, and run a defrag first or the shrink will stop
short.

This is where BitLocker and Intel RST actually matter. Work through those three
warnings properly - none of them is hard, all of them are unforgiving.

Stay in this stage until you have daily-driven your own image for a few weeks
and every item on the baseline checklist is green or knowingly accepted.

### Stage 3 - Commit

Wipe Windows only when you no longer reach for it. There is no schedule for
this and no prize for doing it early.

## Keep a Windows escape hatch either way

Even after Stage 3, keep a Windows install available - a VM, or the external
drive with the roles reversed. Two reasons that are not sentimental:

1. **BIOS and firmware.** Dell firmware updates are well covered by fwupd on
   Linux, but if something ever needs Dell's Windows updater, you need Windows.
2. **L2 fallback.** The seamless-VM plan for Office/Adobe (the apps Wine
   cannot run) needs a licensed Windows image anyway. Your existing licence is
   tied to this machine - do not throw it away.

## Install commands

Once you have a target drive, install stock Aurora DX first (that is the
control group for the baseline), then rebase onto your own image:

```sh
# stock, for baseline measurement
# -> install from the Aurora ISO: https://getaurora.dev

# then, once CI has published your image
sudo bootc switch --enforce-container-sigpolicy ghcr.io/<user>/ivy:latest
systemctl reboot

# to compare the Nvidia variant
sudo bootc switch --enforce-container-sigpolicy ghcr.io/<user>/ivy-nvidia:latest
systemctl reboot
```

Rolling back a bad image is the previous entry in the boot menu. That safety
net is why this architecture was chosen - use it without fear.
