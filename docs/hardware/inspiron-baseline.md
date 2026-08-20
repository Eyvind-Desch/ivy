# Hardware baseline - Dell Inspiron / Vostro

This is the most important document in the repo right now. **Fill it in before
writing a single line of tuning code.** Guessing what is broken wastes weeks;
measuring it takes an afternoon.

## Method

1. Install **stock Aurora DX** (`ghcr.io/ublue-os/aurora-dx:stable`) on the
   reference machine - unmodified, no tweaks.
2. Use it normally for one day.
3. Record every result below. Anything marked BROKEN becomes an L1 task.

Stock Aurora is the control group. Our OS only has to fix the delta.

## Machine

| Field | Value |
|---|---|
| Exact model | **Dell Inspiron 14 5440** |
| Hostname | metari |
| CPU | Intel Core 7 150U - Raptor Lake-U refresh, 10 cores (2P + 8E), 12 threads, 15 W class |
| iGPU | Intel Xe-class integrated - drives the panel |
| dGPU | **Nvidia GeForce MX570 A, 2 GB** - GA107 (Ampere), render-offload only |
| RAM | 16 GB |
| Storage | 954 GB NVMe, **680 GB already in use by Windows** |
| No touch input | confirmed from Windows device info |
| Service tag | _fill in_ |
| BIOS version | _fill in_ |
| WiFi chipset | **_fill in - see below, this is the main unknown_** |

Confirm on the machine with `sudo dmidecode -t system -t bios` and
`lspci -nnk`. Paste the raw output into `docs/hardware/lspci.txt` - the
`[vendor:device]` IDs are what you search for when hunting a driver quirk.

## Known before we start

Research done up front, so you can prioritise. Verify each on the real machine.

**Good news**
- The Inspiron 14 5440 line is Ubuntu-certified by Dell (on the i5-1335U
  config). That is a strong signal the core platform works.
- Raptor Lake-U and Xe graphics are mature on Linux. Expect no drama from CPU,
  iGPU, video decode or display output.
- The MX570 A is Ampere, so the *open* Nvidia kernel modules support it. Older
  MX cards (Pascal and earlier) do not - we are on the right side of that line.

**Expect trouble**
- **Fingerprint reader: assume it will not work.** Dell's readers are Goodix
  and ship a Windows-only driver. Test it, but plan the UX without it.
- **WiFi is the real unknown.** This model shipped with different chipsets
  across revisions and there are Linux reports of non-working WiFi on Dell
  5440 machines. Intel = fine, MediaTek = usually fine on current kernels,
  Realtek/Broadcom = potentially painful. **Check this on day one** - have a
  USB tether or USB WiFi dongle ready so a bad chipset does not strand you
  mid-install.
- **dGPU idle drain.** See `docs/decisions/0002-graphics.md`. The default image
  sidesteps this by omitting the Nvidia stack entirely.

**Set expectations**
- The 150U is a 15 W ultramobile chip. The OS can absolutely feel fast - fast
  boot, fast wake, responsive UI - but heavy compilation will not be quick.
  This is a reason to keep image builds in CI rather than local.

## Checklist

Mark each: OK / BROKEN / PARTIAL, plus a note.

### Power and suspend - highest priority
- [ ] Suspend on lid close, and wakes correctly
- [ ] **Battery drain overnight in sleep** (measure %: charge to 100, close lid,
      check after 8h). Modern Dells use s2idle, not S3 - >10%/night is a real
      bug and the #1 reason people abandon Linux laptops.
- [ ] Idle battery life vs. Windows (measure both, note the gap)
- [ ] Fans behave sanely - not always-on, not silent while hot
- [ ] Thermal/performance profile switching works (`powerprofilesctl list`)
- [ ] Charging threshold configurable (Dell supports this via SMBIOS)
- [ ] Idle package power at desktop, screen on, nothing running
      (`sudo powertop` -> should be low single-digit watts; if it is not,
      that is the battery bug, find it here)

### Graphics
- [ ] Panel at native resolution and refresh rate
- [ ] External display over HDMI and USB-C
- [ ] Hardware video decode working (`intel_gpu_top` shows video engine load
      while playing a 4K video - if not, the CPU is decoding and eating battery)
- [ ] Nvidia variant only: dGPU reaches D3cold when idle
      (`cat /sys/bus/pci/devices/*/power_state` -> `D3cold`)
- [ ] Nvidia variant only: resume from suspend without a black screen

### Input and display
- [ ] Touchpad: two-finger scroll, gestures, palm rejection
- [ ] Keyboard backlight + its Fn key
- [ ] Brightness Fn keys
- [ ] Volume / mute / airplane-mode Fn keys
- [ ] External display over HDMI and USB-C
- [ ] Correct scaling on the internal panel

### Audio - a classic Dell weak point
- [ ] Speakers (many recent Dells need SOF firmware + a model-specific quirk)
- [ ] Internal microphone
- [ ] Headphone jack, and switching when plugged in
- [ ] Speaker volume actually loud enough (amp initialisation often fails)

### Everything else
- [ ] WiFi (note chipset - Intel is fine, Realtek/Killer often is not)
- [ ] Bluetooth, incl. reconnect after resume
- [ ] Webcam
- [ ] Fingerprint reader - `fprintd-enroll`. Goodix sensors are frequently
      **unsupported with no fix available**. Find out now, not in month 3.
- [ ] SD card reader
- [ ] Firmware updates: `fwupdmgr get-devices` / `get-updates`
- [ ] Secure Boot (custom images need key enrollment - see BlueBuild docs)

## Delta summary

Once filled in, list only the BROKEN/PARTIAL items here, ordered by how much
they hurt daily use. That ordered list is the L1 backlog - nothing more.

| # | Issue | Impact | Suspected cause | Status |
|---|---|---|---|---|
| 1 | | | | |
