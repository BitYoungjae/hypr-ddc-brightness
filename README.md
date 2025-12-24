# hypr-ddc-brightness

DDC/CI brightness control for external monitors on Hyprland.

## Why?

I got a Lofree Flow 2 keyboard with a touchbar for brightness control. It didn't work with my external monitors. This script fixes that.

**The problem:**
- SwayOSD uses brightnessctl, which only works with laptop backlights (`/sys/class/backlight/`)
- External monitors need DDC/CI protocol
- Touchbars send rapid events (15-28ms intervals) that overwhelm DDC/CI

**The solution:**
- Delta-based calculation with real DDC read after debounce
- 200ms debounce with flock + timestamps
- Works correctly even after adjusting brightness via monitor buttons

## Requirements

- [ddcutil](https://www.ddcutil.com/)
- [swayosd](https://github.com/ErikReider/SwayOSD)
- [Hyprland](https://hyprland.org/)
- [jq](https://jqlang.github.io/jq/)

## Install

```bash
cp bright ~/.local/bin/
chmod +x ~/.local/bin/bright
bright --init
```

## Usage

```bash
bright +         # +10%
bright -         # -10%
bright + 5       # +5%
bright --init    # Re-detect monitors
```

## Hyprland Config

```bash
unbind = , XF86MonBrightnessUp
unbind = , XF86MonBrightnessDown
binde = , XF86MonBrightnessUp, exec, ~/.local/bin/bright +
binde = , XF86MonBrightnessDown, exec, ~/.local/bin/bright -
binde = ALT, XF86MonBrightnessUp, exec, ~/.local/bin/bright + 1
binde = ALT, XF86MonBrightnessDown, exec, ~/.local/bin/bright - 1
```

## How It Works

1. Each input accumulates delta (+10, +10, +10...)
2. `flock` ensures atomic delta updates
3. OSD shows immediately (estimated value)
4. After 200ms debounce, read actual brightness via DDC
5. Apply delta to real value and send DDC command

## Related

- [SwayOSD DDC/CI issue #87](https://github.com/ErikReider/SwayOSD/issues/87)
- [ddcutil Performance Options](https://www.ddcutil.com/performance_options/)

## License

MIT
