# files/

Overlays and scripts baked into the image at build time.

## `system/`

Copied onto the image root exactly as-is. The path here is the path on the
installed system:

```
files/system/etc/modprobe.d/dell-audio.conf   ->   /etc/modprobe.d/dell-audio.conf
files/system/usr/share/wallpapers/ivy/        ->   /usr/share/wallpapers/ivy/
```

Use this for anything declarative: kernel module options, udev rules, systemd
units, default configs, wallpapers, branding.

## `scripts/`

Shell scripts run during the build, referenced by filename from a
`type: script` module in the recipe. Use these only when a file drop cannot do
the job - a script is imperative, harder to reason about, and runs on every
build.

Rule of thumb: **prefer `system/` over `scripts/`.** A config file states what
the system should be; a script states how it got there.

## Nothing here yet

Correct. Files land here as L1 hardware fixes are found - see
`docs/hardware/inspiron-baseline.md`.
