# 0005 - Visual direction: Windows 11's logic, ivy's colour

Status: accepted - 2026-08-21

Rendered specification: `design/direction.html`

## Context

Stated goal: "faster, better UI than these Linux things like Aurora, minimal
modern design with simple icons - basically exactly like Windows 11", later
refined to "the taskbar could be smaller, like on Mac" and "some things could
be more premium".

Stock Aurora is unmodified Plasma with a wallpaper. That is the bar we are
clearing, and it is not a high one - most distributions look unconsidered
because nobody does design work, not because Plasma cannot look good.

## Decision

**Adopt Windows 11's spatial logic. Reject its colour and its panel.**

Windows 11's arrangement is what a switcher already knows how to operate:
centred launcher, rounded surfaces, one accent on neutral ground, single-weight
line icons, layered translucent chrome. That familiarity is the product; it is
why we are not inventing a new interaction model.

What we change:

- **Accent is ivy green (`#4FAE7F`), not Windows blue.** This is the entire
  visual difference, and it is enough. Neutrals carry a slight green bias so
  they read as belonging to the accent rather than sitting beside it.
- **A floating dock, not a full-width taskbar.** Detached from the screen edge
  so the wallpaper carries underneath. On a 14-inch panel a full strip is real
  estate we cannot spare. Running applications are marked by a 4 px dot.
- **Clock and system state get their own tile, bottom right.** macOS solves
  this with a menu bar across the top; that costs another row of pixels. A
  separate tile gives the same information without the second panel.

## Typography

Segoe UI is licensed and cannot ship. **Selawik**, released by Microsoft under
an open licence and metric-compatible with Segoe, gives the same texture
legally. Four sizes: 28/600, 15/600, 13/400, 11/600-uppercase. No more.

## What "premium" means here, concretely

The user asked for a more premium feel. It is worth writing down what that
actually consists of, because the instinct is usually to add effects and the
answer is almost always light and space:

- **Layered shadows** - a hairline contact shadow, a mid ambient, a wide soft
  cast. One large blur always reads as a sticker.
- **A 1 px inner highlight** along the top edge of raised surfaces, simulating
  light from above. This is what makes a surface feel like material.
- **Fine grain on the wallpaper.** Most of the difference between a background
  that looks rendered and one that looks photographed.
- **More negative space.** The single strongest cheap-versus-considered signal.

## Deliberate omissions

**No dock magnification.** macOS's zoom-on-hover is its most imitated and least
useful detail - it moves the target you are aiming at. Icons stay put; hover
changes only the background. Recorded here because we will be tempted later.

**Blur is confined to system chrome** - dock, menus, notifications. Not behind
every window. Mica everywhere looks wonderful and costs several watts on a 15 W
processor; battery life is a stated priority (0002). Spend the effect where it
is looked at constantly, withhold it where it mostly blurs wallpaper nobody is
reading.

**All motion is 150 ms, ease-out.** Fast animation reads as a responsive
machine. Slow animation reads as a slow one, whatever the benchmark says.

## Consequences

- Windows applications must inherit this scheme, or the illusion breaks the
  moment one opens. Generating the Wine theme from the live Plasma colour
  scheme is therefore not a nicety - it is what makes this one system rather
  than two. It is the expensive half of L3 and the half worth paying for.
- An icon set must cover Windows applications too. A Wine app appearing in the
  menu with its own filled logo beside our line glyphs breaks the drawing
  style more visibly than any colour mismatch.
- Shipping a colour scheme does **not** restyle an account that already
  exists - Plasma writes user config on first login and it wins over system
  defaults. Existing users select it once; new users get it by default.

## Revisit if

Measured idle power shows chrome blur costs materially more than estimated, or
the Windows-app theming turns out to be unachievable - in which case the
honest response is to soften our own chrome toward Wine's, not to ship two
visual languages and hope nobody notices.
