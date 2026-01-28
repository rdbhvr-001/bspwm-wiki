## Introduction to `bspc` command

Bspwm is controlled and configured via the `bspc` command.  In practice one writes a shell script (typically $XDG_CONFIG_HOME/bspwm/bspwmrc) that calls bspc config to set various options.  The general syntax is:

```bash
bspc config [ -m MONITOR | -d DESKTOP | -n NODE ] < setting > [< value >]
```

This gets or sets the value of `< setting >`. Without selectors, the setting applies globally; with -m, -d, or -n it targets a specific monitor, desktop or node.  Each option affects bspwm’s behavior: for example, bspc config border_width 3 sets the width of window borders to 3 pixels.  The settings below comprise every configurable parameter (as of the latest stable bspwm) and include the option’s role, valid values, default behavior, and examples.

## Window Borders and Colors

Bspwm uses window borders to indicate focus and state.  The global border color settings are:

**`normal_border_color:`** color for an unfocused window’s border.

**`active_border_color:`** color for a focused window on an unfocused monitor.

**`focused_border_color`:** color for a focused window on the active monitor.
Each of these accepts any color string #RRGGBB or named X color.  For example, to make the focused window’s border red:

```bash
bspc config focused_border_color "#FF0000"
```

> [!TIP]
> You can query the current color with bspc config focused_border_color

A special feedback color, `presel_feedback_color`, governs the border drawn during manual split-preselection (preselection feedback).  For instance, setting

```bash
bspc config presel_feedback_color "#00FF00"
```

will draw a green highlight where a split will occur.

The `border_width` option (a node setting) sets the thickness in pixels of the window border. Its default is 1 pixel; to make borders thicker, e.g.:

```bash
bspc config border_width 2
```

These parameters control the visual frame around windows.  In the **monocle layout** (one-window fullscreen mode), two special flags exist: **borderless_monocle** (boolean) removes borders when in monocle mode, and **gapless_monocle** removes any window gaps in monocle layout.  Finally, **single_monocle** (boolean) forces the layout to switch to monocle when only one window is present. For example:

```bash
bspc config borderless_monocle true
bspc config gapless_monocle true
bspc config single_monocle true
```

These would eliminate borders and gaps in monocle layout and make that layout automatic for lone windows.

## Splitting and Layout

The `split_ratio` setting defines how a window is partitioned when a new window is inserted in automatic mode. It is a fraction between 0 and 1. For example,

```bash
bspc config split_ratio 0.60
```

makes new splits 60/40 by default (i.e. the first child gets 60% of the space).  The default is 0.5 (even split) unless changed in your config.

**Window spacing is controlled by `window_gap`** (desktop setting), which is the number of pixels of space between tiled windows.  To add a 10px gap everywhere, use:

```bash
bspc config window_gap 10
```

**Monocle-layout** padding can also be set. The options top_monocle_padding, right_monocle_padding, bottom_monocle_padding, and left_monocle_padding (all in pixels) add blank space at the screen edges when in monocle mode. For example, to center a monocle window with a 20px top margin:

```bash
bspc config top_monocle_padding 20
```

> [!IMPORTANT]
> In summary, `split_ratio` and `window_gap` affect tiling splits and inter-window gaps, while the monocle flags/padding fine-tune fullscreen behavior.

### Automatic Tiling and Preselection

Bspwm can tile windows automatically or under manual preselection. The automatic_scheme option chooses the insertion algorithm: it accepts longest_side, alternate, or spiral.  This determines how the binary partitioning tree expands. For example:

```bash
bspc config automatic_scheme spiral
```

switches to the **“spiral”** insertion pattern.

> [!NOTE]
> By default bspwm uses `alternate` automatic scheme which alternates between vertical and horizontal splits.

The `initial_polarity` option determines on which side a new window is attached in an automatic split when a node has only one child. It can be first_child or second_child. For instance:

```bash
bspc config initial_polarity first_child
```

makes new windows attach on the first (left or top) side of the split by default. (By default this is usually second_child.)

**Other related options:** `directional_focus_tightness` (high or low) tweaks how strictly bspwm decides whether a window is “in the DIR side” of another for focus commands. `removal_adjustment` (boolean) controls whether bspwm adjusts (re-splits) the sibling when a node is removed from the tree; turning it off can leave odd splits if windows are closed.

**Preselection (manual tiling)** can be toggled with pointer commands or key bindings. The presel_feedback setting (boolean, defaults to true) enables the visible overlay that shows where a manual split will occur. If you disable it (bspc config presel_feedback false), bspwm will still respect manual split commands but won’t draw the highlighted region.

In short, these settings define how bspwm splits windows: the scheme, where new windows attach, and how strictly the tiling is adjusted or visualized.

### Pointer and Focus Behavior

Bspwm lets you use the mouse (with a modifier key) to move/resize windows and to control focus. The pointer_modifier setting specifies which keyboard modifier (e.g. mod4 for the Super/Windows key) enables pointer actions. For example:

```bash
bspc config pointer_modifier mod4
```

Means holding Super while clicking/draggings acts on windows.

By default, `pointer_modifier+Button1` moves a window, `Button2` resizes by dragging a side, and `Button3` resizes by dragging a corner. These defaults correspond to `pointer_action1=move`, `pointer_action2=resize_side`, `pointer_action3=resize_corner`. You can reassign them; e.g.:

```bash
bspc config pointer_action1 resize_side
bspc config pointer_action2 move
```

swaps the actions for button1 and button2. Setting pointer_action< n > to none disables that action.

Focus via mouse clicking is controlled by click_to_focus. It takes button1, button2, button3, any, or none. The default is button1 (left click focuses). For example, to focus windows with a middle-click:

```bash
bspc config click_to_focus button2
```

The `swallow_first_click` flag (boolean) prevents the click event that focuses a window from being passed to the application. If true, clicking to focus will not (for example) click buttons in the newly focused window; this is useful if you want a click to only change focus.

**Pointer and focus warping:** `focus_follows_pointer` (boolean) makes focus follow the mouse pointer; if enabled, simply moving the mouse over a window will focus it. Conversely, `pointer_follows_focus` (boolean) warps the pointer to the center of the newly focused window, and `pointer_follows_monitor` does the same for the newly focused monitor. These are all off by default, but can be turned on with e.g.:

```bash
bspc config focus_follows_pointer true
bspc config pointer_follows_focus true
```

**In summary, the pointer settings let you pick a modifier key and mouse button actions to move/resize windows, and to configure click-to-focus and pointer warping behaviors.**

### EWMH Hints and Miscellaneous

Bspwm can ignore or honor certain EWMH hints (state requests from other applications) via these settings.

`ignore_ewmh_focus` (boolean) ignores focus requests from EWMH-compliant clients. If true, external programs cannot change window focus.

`ignore_ewmh_fullscreen` can be none, all, or a comma-separated list enter,exit. It blocks clients that try to put windows into fullscreen (setting the _NET_WM_STATE_FULLSCREEN hint). For example, bspc config ignore_ewmh_fullscreen all will prevent all EWMH fullscreen changes.

`ignore_ewmh_struts` (boolean) ignores the EWMH strut hints that panels and docks use to reserve screen space. If you have an always-on-top panel, setting this to true makes bspwm ignore its struts (useful if your panel is external and you don’t want bspwm to avoid it).

`center_pseudo_tiled` (boolean, defaults to true) determines whether pseudo-tiled windows (tiled windows that respect size hints) are centered in their area. For example, floating dialogs can be treated as pseudo-tiled; if true, bspwm will center them in their split. You can turn it off with bspc config center_pseudo_tiled false.

`honor_size_hints` (boolean) tells bspwm to apply ICCCM size hints from applications. Enabling this makes bspwm honor the minimum/maximum size hints that some X programs request.

`mapping_events_count` (integer) sets how many mapping notify events bspwm should process. By default it only handles one (focusing windows, etc.), but a negative value means “handle all.” This is a rare, low-level option usually left at 0 or 1.

> [!IMPORTANT]
> These options govern how bspwm reacts to external hints or special windows. Use them to integrate bspwm with other desktop components (EWMH panels, fullscreen requests) or to tweak size-hint behavior.

### Monitor and Desktop Settings

Padding at screen edges is controlled by the `top_padding`, `right_padding`, `bottom_padding`, and `left_padding` settings (monitor/desktop settings). These are pixel values added as empty space on each side of the screen or desktop. This is commonly used to leave room for panels or docks. For example, to leave a 24px gap at the top (e.g. for a status bar):

```bash
bspc config top_padding 24
bspc config bottom_padding 0
bspc config left_padding 0
bspc config right_padding 0
```

With this, bspwm will tile windows below that 24px area.

Bspwm can also manage multiple monitors dynamically. The boolean flags remove_disabled_monitors, remove_unplugged_monitors, and merge_overlapping_monitors (all default false) adjust bspwm’s behavior on monitor changes. **For instance, if you set bspc config remove_unplugged_monitors true, any monitor that is unplugged (e.g. a laptop lid closing) will be treated as disconnected and its desktops moved elsewhere. These are advanced options for multi-head setups**.

### Global Misc Settings

**status_prefix:** a string prefixed to each bspc event line (used in status scripts). For example, setting

```bash
bspc config status_prefix "[bspwm] "
```
causes all window/desktop events sent to status bars to start with “ ”.

**external_rules_command:** path to an external script that provides window rule assignments. If set, bspwm will call this command each time a new window appears, passing it the window’s ID, class, and instance, and expecting key=value outputs for rule keys (like split_ratio=0.7, desktop=2, etc.). For example:

```bash
bspc config external_rules_command "$HOME/.config/bspwm/rules.sh"
```

where rules.sh might output something like center=on follow=on. This allows dynamic, programmatic control of window placement.

> [!NOTE]
> These global options are less frequently changed, but status_prefix can help integrate bspwm with custom panels, and external_rules_command enables advanced window rule logic.

### Usage Examples

Below is a sample snippet of a bspwmrc configuring several options (annotated for clarity):

```bash

# Set focused/unfocused border colors
bspc config focused_border_color "#3585ce"    # blue border for focused windows
bspc config normal_border_color "#444444"     # gray for others

# Set default split ratio and gaps
bspc config split_ratio 0.55                  # 55/45 split by default
bspc config window_gap 10                     # 10px gap between windows

# Monocle layout tweaks
bspc config borderless_monocle true           # no borders in monocle mode
bspc config top_monocle_padding 20            # 20px padding on top in monocle

# Pointer/mouse actions
bspc config pointer_modifier mod4             # use Super key for pointer actions
bspc config pointer_action3 none              # disable corner resize
bspc config click_to_focus any                # any click focuses window
bspc config swallow_first_click true         # do not forward the click event

# Automatic tiling scheme
bspc config automatic_scheme alternate        # alternate insert (default)
bspc config initial_polarity second_child     # new windows on second child by default

# Layout hints
bspc config focus_follows_pointer true        # focus on mouse-over
bspc config center_pseudo_tiled false        # disable centering pseudo-tiled

# Padding for panels
bspc config top_padding 30                   # leave 30px at top for panel
```

Each bspc config command above changes an aspect of bspwm’s layout or behavior, as documented. For a complete reference of all options and their meanings, see the bspwm manual (man page).

