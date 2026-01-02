# Getting started with bspwm window manager

bspwm is a minimal tiling window manager: you mainly configure the WM via a shell script (`bspwmrc`) and handle keybindings via `sxhkd` (`sxhkdrc`). [^1_1]
A good “getting started” path is: install `bspwm` + `sxhkd`, copy the example configs into `~/.config/…`, make `bspwmrc` executable, then start `sxhkd` and `bspwm` from your X session. [^1_1]

## Install and start

- Install `bspwm` (the WM) and `sxhkd` (hotkey daemon). [^1_1]
- Start `bspwm` via Xinit (or a display manager session), since bspwm is an X11 window manager. [^1_1]


## Create initial config

- Copy the example configs from `/usr/share/doc/bspwm/examples/` into:
    - `~/.config/bspwm/bspwmrc`
    - `~/.config/sxhkd/sxhkdrc` [^1_1]
- Make sure `bspwmrc` is executable, because it’s a shell script that configures bspwm by running `bspc` commands. [^1_1]
- ArchWiki’s example uses these commands (adjust if your distro differs):
    - `install -Dm755 /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/bspwmrc` [^1_1]
    - `install -Dm644 /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/sxhkdrc` [^1_1]


## Learn the core concepts

- `bspwmrc` is where you set window manager behavior (desktops, gaps, rules, etc.) by sending messages with `bspc`. [^1_1]
- `sxhkdrc` is where you define keybindings that typically call `bspc` (focus/move windows, switch desktops, launch apps). [^1_1]
- bspwm doesn’t handle keyboard input directly, so a hotkey daemon like `sxhkd` is required for shortcuts. [^1_1]


## First troubleshooting checklist

If you boot into a blank screen and keys don’t work, check these first:

- `sxhkd` is installed and actually started (and started in the background, since it blocks). [^1_1]
- `~/.config/bspwm/bspwmrc` is executable. [^1_1]
- Your `sxhkdrc` terminal command matches a terminal you have installed (the example config may reference `urxvt`). [^1_1]


***

# The BSPWM Compendium: Architecture, Configuration, and Mastery

## Table of Contents

1. **Core Architecture**: The Binary Tree Philosophy
2. **Installation \& Environment**: Dependencies and X11 Integration
3. **The Nervous System**: `sxhkd` (Simple X Hotkey Daemon)
4. **The Brain**: `bspwmrc` Configuration
5. **Control Plane**: Mastering `bspc` (Binary Space Partitioning Control)
6. **Window Management**: Selections, Preselection, and Receptacles
7. **Advanced Logic**: External Rules \& Scripting
8. **The Ecosystem**: Polybar, Picom, and Rofi Integration

***

## 1. Core Architecture: The Binary Tree Philosophy

Unlike dynamic tilers (like DWM or Xmonad) that use predefined layout lists (master/stack, grid, spiral), bspwm is a **manual** tiler based on a data structure known as a **full binary tree**.

### 1.1 The Data Structure

Every monitor contains at least one desktop. Every desktop is the root of a binary tree.

* **Nodes**: The fundamental unit. A node is either a *leaf* (holding a window) or an *internal node* (holding two children).
* **Leaves**: The actual application windows (e.g., Firefox, Terminal).
* **Internal Nodes**: Containers that define the split direction (Horizontal/Vertical) and the split ratio (e.g., 0.5 for equal halves) between their two children.

![BSPWM Binary Tree Structure](https://ppl-ai-code-interpreter-files.s3.amazonaws.com/web/direct-files/d39d3cac9a5be5c54100567c8bfeb59b/8e10f7d6-a29f-4cff-a6b6-271b3d8d3b07/161c80ce.png)

BSPWM Binary Tree Structure

### 1.2 The Implication of "Manual" Tiling

In standard tilers, you "switch layouts." In bspwm, you manipulate the tree directly.

* **Automatic Mode**: Bspwm decides where the next window goes based on the `automatic_scheme` (usually "spiral" or "longest_side").
* **Manual Mode**: You explicitly "preselect" a region (North, South, East, West) on an existing node. The next window splits *that specific node* in the chosen direction.

This architecture makes bspwm strictly **deterministic**. The layout is exactly what you build, nothing more, nothing less.

***

## 2. Installation \& Environment

Bspwm adheres strictly to the Unix philosophy: "Do one thing and do it well." It only manages windows. It does **not** handle:

* Keyboard input (handled by `sxhkd`)
* Compositing (handled by `picom`)
* Bars/Panels (handled by `polybar` or `lemonbar`)
* Wallpaper (handled by `feh` or `nitrogen`)


### 2.1 Dependencies (Arch Linux / General)

You must install both the window manager and the hotkey daemon.

```bash
# Arch Linux
sudo pacman -S bspwm sxhkd

# Debian/Ubuntu
sudo apt install bspwm sxhkd
```


### 2.2 X11 Session Entry

Because bspwm is just a binary, it must be launched via `.xinitrc` or a Display Manager (LightDM/GDM).

**File**: `~/.xinitrc`

```bash
# Load resources
xrdb -merge ~/.Xresources &

# Start the hotkey daemon in the background (CRITICAL)
sxhkd &

# Start the compositor (optional but recommended)
picom &

# Set wallpaper
feh --bg-fill ~/Pictures/wall.jpg &

# Set cursor shape (fixes "X" cursor bug)
xsetroot -cursor_name left_ptr &

# Launch bspwm (must be the last line, no '&')
exec bspwm
```


***

## 3. The Nervous System: `sxhkd`

`sxhkd` is an X daemon that reacts to input events and executes commands. It is independent of bspwm but essential for it.

**Config Location**: `~/.config/sxhkd/sxhkdrc`

### 3.1 Syntax \& Chords

`sxhkd` uses a unique syntax that allows for "chord chains" (pressing keys in sequence) and "brace expansion" (compacting similar commands).

#### Brace Expansion (The Power Feature)

Instead of writing 4 separate lines for moving windows, you write one:

```bash
# Move window node (super + shift + {h,j,k,l})
super + shift + {h,j,k,l}
    bspc node -v {-20 0, 0 20, 0 -20, 20 0}
```

* When you press `Super + Shift + h`, it executes `bspc node -v -20 0`.
* When you press `Super + Shift + j`, it executes `bspc node -v 0 20`.


#### Chord Chains

You can create "modes" without actual modes.

```bash
# Press Super + o, release, then press 'r' to reload config
super + o ; r
    bspc wm -r
```


### 3.2 Essential Keybindings Table

| Key Chord | Command | Description |
| :-- | :-- | :-- |
| `super + Return` | `$TERMINAL` | Launch terminal |
| `super + w` | `bspc node -c` | Close focused window |
| `super + {h,j,k,l}` | `bspc node -f {west,south,north,east}` | Focus window directionally |
| `super + {1-9}` | `bspc desktop -f ^{1-9}` | Switch to desktop 1-9 |
| `super + shift + {1-9}` | `bspc node -d ^{1-9}` | Send window to desktop 1-9 |


***

## 4. The Brain: `bspwmrc`

**File**: `~/.config/bspwm/bspwmrc`
**Requirement**: Must be executable (`chmod +x bspwmrc`).

This file is simply a shell script. It runs once at startup. If you change it, you must reload bspwm (`bspc wm -r`) to re-run it.

### 4.1 Monitor Configuration

You must explicitly map desktops to monitors. If you plug in a second monitor, bspwm won't use it until you tell it to.

```bash
#!/bin/sh

# Clean setup for one or two monitors
if [[ $(xrandr -q | grep "HDMI-0 connected") ]]; then
    # Dual monitor setup
    bspc monitor DP-0 -d I II III IV V
    bspc monitor HDMI-0 -d VI VII VIII IX X
else
    # Single monitor setup
    bspc monitor DP-0 -d I II III IV V VI VII VIII IX X
fi
```


### 4.2 Global Aesthetics

```bash
# Border width (pixels)
bspc config border_width         2

# Window gap (space between windows)
bspc config window_gap          12

# Split ratio (0.5 = 50/50 split)
bspc config split_ratio          0.50

# Border colors
bspc config normal_border_color "#4c566a"
bspc config focused_border_color "#88c0d0"

# Focus follows mouse (true/false)
bspc config focus_follows_pointer true
```


***

## 5. Control Plane: Mastering `bspc`

`bspc` (Binary Space Partitioning Control) is the command-line client. You use it in `bspwmrc`, `sxhkdrc`, and your own scripts. Understanding `bspc` is understanding bspwm.

### 5.1 Selectors: The Query Language

Almost every `bspc` command requires you to select a target.

* **Nodes**: `-n` (Target a window)
* **Desktops**: `-d` (Target a workspace)
* **Monitors**: `-m` (Target a screen)


#### Advanced Node Selectors

You can target nodes based on relationships, flags, or history.

* `focused`: The currently active window.
* `east`, `west`, `north`, `south`: Directional neighbors.
* `last`: The previously focused node.
* `biggest`: The largest node on the desktop.
* `@parent`: The internal node holding the current window and its sibling.

**Example**: Swap the current window with the biggest window on the desktop:

```bash
bspc node -s biggest.local
```


### 5.2 Node Modifications

* **Flags**: Nodes can be `hidden`, `sticky` (follows you across desktops), `private` (ignored by some queries), or `locked` (cannot be closed).

```bash
bspc node -g sticky  # Toggle sticky
bspc node -t floating # Float the window
```

* **Preselection**: The killer feature. Define where the *next* window opens.

```bash
bspc node -p south   # Preselect the bottom half
bspc node -p cancel  # Cancel preselection
```


***

## 6. Window Management Dynamics

### 6.1 Manual vs. Automatic

By default, bspwm splits windows automatically (spiraling inward).
To take control, you use **Preselection**.

1. **Input**: User presses `Super + Ctrl + L` (East).
2. **Visual Feedback**: Bspwm highlights the right half of the current node in a distinct color (configurable via `presel_feedback_color`).
3. **Action**: User opens a terminal.
4. **Result**: The terminal spawns exactly in that highlighted area.

### 6.2 Receptacles

A **Receptacle** is a "placeholder" node—a leaf that contains no window. It reserves space in the tree layout.

* **Use case**: You want to build a complex layout (e.g., 3 small windows on the left, one big on the right) *before* you open the applications.
* **Command**: `bspc node -i` (Insert receptacle).


### 6.3 Swallowing (Terminal Suppression)

"Swallowing" is when you launch a GUI app (like MPV or Sxiv) from a terminal, and the terminal disappears, being "swallowed" by the new app. When the app closes, the terminal returns.

* Bspwm does not do this natively.
* **Solution**: Use an external listener script or a wrapper tool like `devour` in your sxhkd config.

***

## 7. Advanced Logic: External Rules

This is bspwm's most powerful configuration feature. Instead of a simple "If app is Firefox -> move to desktop 2" list, you can point bspwm to a script.

**Config**: `bspc config external_rules_command ~/.config/bspwm/external_rules`

### 7.1 The External Rules Script

Bspwm passes the Window ID, Class, Instance, and Title to this script as arguments. The script prints commands to `stdout` that bspwm executes for that specific window.

**Example: `~/.config/bspwm/external_rules`**

```bash
#!/bin/sh

wid=$1
class=$2
instance=$3
title=$(xtitle "$wid")

# If it's a Picture-in-Picture window (Firefox/Chrome)
if [ "$title" = "Picture-in-Picture" ]; then
    echo "state=floating"
    echo "sticky=on"
    echo "border=off"
    exit 0
fi

# If it's GIMP, move to desktop 8 and follow it
if [ "$class" = "Gimp" ]; then
    echo "desktop=^8"
    echo "follow=on"
    exit 0
fi

# Complex logic: If 2 nodes already exist on desktop 1, send next one to desktop 2
count=$(bspc query -N -d ^1 | wc -l)
if [ "$count" -ge 2 ]; then
    echo "desktop=^2"
fi
```

This allows for logic impossible in other window managers (e.g., "Float this window only if it's Tuesday and I'm on Monitor 2").

***

## 8. The Ecosystem

A "naked" bspwm is just black screens. You need the ecosystem to make it a workstation.

### 8.1 Polybar (Status Bar)

Polybar has a built-in `bspwm` module. It can display the focused desktop, occupied desktops, and urgent alerts.
**Module Config**:

```ini
[module/bspwm]
type = internal/bspwm
label-focused = %name%
label-focused-background = #3b4252
label-focused-underline= #88c0d0
label-occupied = %name%
label-urgent = %name%!
label-empty =
```


### 8.2 Rofi (Application Launcher)

Rofi works perfectly as a menu. Bind it in `sxhkd`:

```bash
super + space
    rofi -show drun -show-icons
```


### 8.3 Scratchpads (Hidden Windows)

Bspwm creates scratchpads easily using node flags.

1. **Hide**: `bspc node -g hidden -f`
2. **Show**: `bspc node <selector> -g hidden=off -f`

**Scripted Toggle (Sticky Scratchpad)**:

```bash
#!/bin/sh
id=$(xdotool search --class "scratchpad_term" | tail -1)

if [ -z "$id" ]; then
    # Launch if not running
    alacritty --class scratchpad_term &
else
    # Toggle visibility
    if bspc query -N -n "$id.hidden"; then
        bspc node "$id" -g hidden=off -d focused -f
    else
        bspc node "$id" -g hidden=on
    fi
fi
```


***

## 9. Troubleshooting \& Debugging

### 9.1 The Blank Screen of Death

If you log in and see nothing:

1. **Check Executable**: Did you `chmod +x ~/.config/bspwm/bspwmrc`?
2. **Check sxhkd**: Is `sxhkd` running? Switch to TTY (`Ctrl+Alt+F2`), login, and run `pidof sxhkd`.
3. **Check Terminal**: Does your `sxhkdrc` launch a terminal you actually have installed? (Default is often `urxvt`, you might have `alacritty` or `kitty`).

### 9.2 Tree Visualization

To understand why your windows are splitting weirdly, dump the tree state:

```bash
bspc query -T -d | jq .
```

This outputs the JSON representation of the current desktop's binary tree.

### 9.3 Orphaned Nodes

Sometimes a "ghost" node remains. Cleanup:

```bash
# Remove the node explicitly
bspc node <node_id> -k
```


[^2_1]: https://madnight.github.io/bspwm/

[^2_2]: https://www.youtube.com/watch?v=94dA0uB1qp4

[^2_3]: https://www.reddit.com/r/bspwm/comments/6or5r0/bspc_query_output_and_bspc_format/

[^2_4]: https://www.cs.ucdavis.edu/~amenta/w11/29toth.pdf

[^2_5]: https://www.youtube.com/watch?v=ikstju_vJP8

[^2_6]: https://github.com/IntrepidPig/bspwm-doc/blob/master/documentation.md

[^2_7]: https://www.geeksforgeeks.org/dsa/binary-space-partitioning/

[^2_8]: https://dharmx.is-a.dev/bspwm-basics/

[^2_9]: https://www.dragonflybsd.org/cgi/web-man?command=bspc\&section=1

[^2_10]: https://cgvr.cs.uni-bremen.de/teaching/geom_literatur/A%20Tutorial%20on%20Binary%20Space%20Partitioning%20Trees%20-%20Bruce%20F.%20Naylor.pdf

[^2_11]: https://vagnerinacio.com/blog/whybspwm/

[^2_12]: https://www.youtube.com/watch?v=Ihvf8urQLJk

[^2_13]: https://www.reddit.com/r/bspwm/comments/f7vi62/what_is_binary_space_partitioning_doing_for_bspwm/

[^2_14]: https://wiki.archlinux.org/title/Bspwm

[^2_15]: https://www.youtube.com/watch?v=smZUSnUCjwg

[^2_16]: https://en.wikipedia.org/wiki/Binary_space_partitioning

[^2_17]: https://dev.to/l04db4l4nc3r/bspwm-a-bare-bones-window-manager-44di

[^2_18]: https://manpages.debian.org/testing/bspwm/bspc.1.en.html

[^2_19]: https://github.com/baskerville/bspwm

[^2_20]: https://gist.github.com/wimstefan/64e17d257d07b666079628a7e1859823


---
# The BSPWM Compendium: Internals, Commands, and Advanced Architecture

## 1. Architectural Internals

### 1.1 The Binary Space Partitioning (BSP) Model

At its core, **bspwm** differs fundamentally from dynamic tilers (like i3, dwm, or xmonad) because it maps windows directly to the leaves of a **full binary tree**. There are no "lists" or "stacks" of windows; there is only the tree.

#### The `node_t` Structure

Understanding bspwm requires understanding the C structure that represents every node. A `node` is the atomic unit of the window manager.

* **Internal Nodes**: These are purely structural. They do not hold windows. Their sole purpose is to define a region of space and how it is split.
    * *Properties*: `split_type` (Horizontal or Vertical), `split_ratio` (0.0 to 1.0, default 0.5), `first_child` (pointer), `second_child` (pointer).
* **Leaf Nodes**: These are the containers for actual X11 windows.
    * *Properties*: `client` (Window ID), `vacuity` (Occupied or Empty/Receptacle).

![BSPWM Binary Tree Structure](https://ppl-ai-code-interpreter-files.s3.amazonaws.com/web/direct-files/d39d3cac9a5be5c54100567c8bfeb59b/8e10f7d6-a29f-4cff-a6b6-271b3d8d3b07/161c80ce.png)

BSPWM Binary Tree Structure

When you "move" a window in bspwm, you are actually performing tree rotation or swapping pointers in this linked structure.

### 1.2 The IPC Architecture (Inter-Process Communication)

`bspwm` itself is a daemon that does not read a configuration file at runtime. It opens a Unix domain socket and waits for byte-stream commands.

* **The Daemon (`bspwm`)**: Sits in an event loop (using `select` or `epoll`), listening for two things simultaneously:

1. **X11 Events**: `MapRequest`, `DestroyNotify`, `EnterNotify` (from the X server).
2. **Socket Messages**: Commands from `bspc`.
* **The Client (`bspc`)**: A lightweight binary that connects to the socket, sends a null-terminated string (e.g., `node\0-f\0west\0`), waits for a response, and prints it to stdout.

**Implication**: Your `bspwmrc` is just a shell script that spams `bspc` commands to the socket. If `bspc` fails, `bspwm` remains completely unconfigured (black screen).

***

## 2. The `bspc` Command Language

```
`bspc` is not just a controller; it is a query language. Almost every command follows the pattern: `bspc <DOMAIN> <ACTION> <SELECTOR>`.
```


### 2.1 Selectors and Descriptors

A selector uniquely identifies a node, desktop, or monitor.
**Syntax**: `[REFERENCE#]DESCRIPTOR(.MODIFIER)*`

* **Reference**: The starting point. Defaults to `focused`. You can set it arbitrarily: `0x00200005#...` starts the search from window `0x00200005`.
* **Descriptors**:
    * **Directional**: `north`, `south`, `east`, `west`.
    * **Cyclic**: `next`, `prev` (follows the tree traversal order).
    * **Genealogical**: `parent`, `brother`, `first_child`, `second_child`, `ancestor`, `descendant`.
    * **History**: `last` (last focused), `older`, `newer`.
* **Modifiers**: Filters that refine the selection.
    * `.local` (same desktop), `.leaf` (ignore internal nodes), `.floating`, `.tiled`.
    * `!`: Negation. `.!hidden` means "not hidden".

**Example**: "Focus the first non-hidden floating window on the current desktop."

```bash
bspc node -f .floating.!hidden.local.first
```


### 2.2 Path Selectors (`@`)

You can address nodes by their path from the root of the tree.

* `@/`: The root of the current desktop.
* `@/1`: The first child (left/top) of the root.
* `@/2`: The second child (right/bottom) of the root.
* `@/2/1`: The first child of the second child of the root.

This is useful for scripts that need to traverse the tree deterministically.

***

## 3. Command Reference

### 3.1 Node Operations (`bspc node`)

* **Insertion (`-i`)**: Inserts a "receptacle" (empty leaf) at the given split.
    * `bspc node -i` creates a blank space where you can later dump a window.
* **Preselection (`-p`)**: Marks a node to be split in a specific way for the *next* window.
    * `bspc node -p south -o 0.75`: "The next window will open in the bottom 25% of this node."
* **Flags (`-g`)**:
    * `hidden`: Removes the window from the tree layout but keeps it in memory (minimized).
    * `sticky`: Keeps the window visible across all desktop switches on that monitor.
    * `private`: Bspwm will try not to split this node.
    * `marked`: Tags a node for batch operations.


### 3.2 Desktop \& Monitor Operations

* **Reordering**: `bspc monitor -o I II III IV` reorders desktops.
* **Swapping**: `bspc desktop -s prev` swaps the current desktop with the previous one.
* **Bubbling**: `bspc desktop -b next` "bubbles" the desktop (moves it one slot over in the list).


### 3.3 The Query System (`bspc query`)

This is the most powerful tool for scripting.

* `bspc query -T`: Dumps the **entire world state** (monitors -> desktops -> nodes) as a JSON object.
* `bspc query -N`: Lists node IDs.
    * **Use Case**: Count windows on current desktop:

```bash
bspc query -N -d focused -n .leaf.!hidden | wc -l
```


***

## 4. Advanced Logic \& Scripting

### 4.1 External Rules

Bspwm allows you to offload window placement logic to an external script. This is defined by `external_rules_command`.

**How it works**:

1. A new window appears.
2. Bspwm pauses.
3. It calls your script with arguments: `WindowID`, `Class`, `Instance`.
4. Your script prints key-value pairs to stdout (e.g., `state=floating`, `desktop=^3`).
5. Bspwm applies these settings *before* mapping the window.

**Example Script**:

```bash
#!/bin/sh
wid=$1
class=$2
instance=$3

# Float all "Save As" dialogs
if [ "$class" = "Gtkwave" ] && [ "$instance" = "file_chooser" ]; then
    echo "state=floating"
    echo "center=on"
fi
```


### 4.2 State Restoration (Layouts)

Because the state is purely data (JSON), you can save and restore layouts.

* **Save**: `bspc query -T -d > layout.json`.
* **Restore**: A script is needed to parse this JSON. It would iterate through the tree, using `bspc node -i` to create receptacles in the exact structure of the saved tree, and then launch applications to fill those receptacles.


### 4.3 The Subscriber (`bspc subscribe`)

For status bars (like Polybar) or event-driven scripts.
`bspc subscribe report` streams a continuous string describing the state:
`W:m:DP-0:O:I:f:II:o:III:m:HDMI-0:o:IV:o:V`

* `W`: Start of report.
* `m:DP-0`: Monitor DP-0.
* `O:I`: Desktop I is Occupied.
* `f:II`: Desktop II is Free (empty).
* `F:III`: Desktop III is Focused.

This allows bars to update instantly without polling.

***

## 5. Event Flow: A Step-by-Step

When you press `Super + Enter` (Terminal):

1. **X Server** receives KeyPress.
2. **sxhkd** (listening for keys) matches `super + Return`.
3. **sxhkd** executes `$TERMINAL` (e.g., Alacritty).
4. **Alacritty** starts and asks X11 for a window (`CreateWindow`).
5. **X Server** sends `MapRequest` to bspwm.
6. **bspwm** intercepts:
    * Runs `external_rules_command`.
    * Consults the `bspwmrc` rules (`bspc rule -a ...`).
    * Finds the "insertion point" (focused node or preselection).
    * Splits the tree node.
    * Resizes existing windows.
7. **bspwm** grants the MapRequest.
8. **X Server** draws the window.

This entire chain happens in milliseconds, driven by the binary tree logic that ensures no two windows ever overlap (unless floating).

[^3_1]: https://dharmx.is-a.dev/bspwm-basics/

[^3_2]: https://www.dragonflybsd.org/cgi/web-man?command=bspc\&section=1

[^3_3]: https://github.com/IntrepidPig/bspwm-doc/blob/master/documentation.md

[^3_4]: https://github.com/baskerville/bspwm/issues/896

[^3_5]: https://wiki.archlinux.org/title/Bspwm

[^3_6]: https://developers.google.com/nest/device-access/subscribe-to-events

[^3_7]: https://www.reddit.com/r/bspwm/comments/6or5r0/bspc_query_output_and_bspc_format/

[^3_8]: https://bbs.archlinux.org/viewtopic.php?id=306187

[^3_9]: https://www.youtube.com/watch?v=smZUSnUCjwg

[^3_10]: https://www.bmc.com/blogs/pub-sub-publish-subscribe/

[^3_11]: https://madnight.github.io/bspwm/CHANGELOG/

[^3_12]: https://www.reddit.com/r/linux/comments/4i7tja/what_has_been_the_most_transformative_or/

[^3_13]: https://dev.to/l04db4l4nc3r/bspwm-a-bare-bones-window-manager-44di

[^3_14]: https://www.reddit.com/r/bspwm/comments/fry5bb/ideas_for_bspc_subscribe/

[^3_15]: https://packagehub.suse.com/packages/bspwm/0_9_5-bp151_2_14/

[^3_16]: https://www.jvt.me/archives/

[^3_17]: https://github.com/baskerville/bspwm

[^3_18]: https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/platform_events_subscribe_lc.htm

[^3_19]: https://www.reddit.com/r/bspwm/comments/d71twt/how_do_you_select_nodes/

[^3_20]: https://pages.sachachua.com/.emacs.d/Sacha.html


---

To master **bspwm** (Binary Space Partitioning Window Manager), you must move beyond simple configuration and understand it as a programmable environment. Because `bspwm` is merely a C program controlled by a socket client (`bspc`), mastery is equivalent to mastering the `bspc` command and the logic of binary trees.

Here is the exhaustive curriculum for total bspwm mastery, structured from architectural concepts to advanced kernel-level scripting.

### 1. The Theoretical Foundation (The "Binary" in BSPWM)

You cannot predict window behavior without understanding the data structure.

* **The Full Binary Tree**: Understand how every desktop is a root node that splits recursively.
* **Internal Nodes vs. Leaves**:
    * *Leaves* hold windows (`client`).
    * *Internal Nodes* hold structure (Split type: Horizontal/Vertical + Split Ratio).
* **Traversal Paths**: Understanding how `next`/`prev` cycles through the tree (pre-order, in-order, post-order traversal logic).
* **Tree Rotation**: How "circulate" commands (`bspc node -C`) physically rotate the tree structure.


### 2. The Control Interface (`bspc`)

Mastering every subcommand of the binary space partitioning controller.

* **Node Selection (The Query Language)**:
    * **Directional**: `north`, `south`, `east`, `west`.
    * **History**: `last`, `older`, `newer`.
    * **Family**: `parent`, `brother`, `ancestor`, `descendant`.
    * **Path Notations**: `@/` (root), `@/1` (left child), `@/2` (right child).
    * **Modifiers**: `.floating`, `.tiled`, `.pseudo_tiled`, `.locked`, `.sticky`, `.private`, `.hidden`.
* **Node Manipulation**:
    * **Movement**: Moving nodes within a desktop, across desktops, or across monitors (`bspc node -m`, `-d`, `-n`).
    * **Resizing**: Differential resizing (`-z top -20 0`) vs. Ratio resizing.
    * **Swapping**: Exchanging the position of two nodes (`bspc node -s`).
    * **Flags**: Toggling `hidden` (minimized), `sticky` (global), `private` (ignored by automatic splitting), `locked` (immune to closing).
* **Preselection (`-p`)**:
    * Manually defining the *split direction* and *ratio* for the *next* window.
    * Visualizing preselection feedback colors.
    * Canceling preselection (`bspc node -p cancel`).
* **Receptacles (`-i`)**:
    * Inserting empty nodes (leaves without windows) to reserve layout space.
    * Dumping windows into receptacles.


### 3. The Input Daemon (`sxhkd`)

`bspwm` handles zero keyboard input. You must master `sxhkd` to drive it.

* **Chord Chains**: Binding sequences (`super + a ; b`).
* **Brace Expansion**: Creating compact matrices of commands (`super + {h,j,k,l}` mapped to `{west,south,north,east}`).
* **Command Replay**: Using `@` to run commands on key release vs. press.
* **Sync vs. Async**: Understanding when a command blocks `sxhkd` and when it doesn't.
* **Mode Simulation**: Creating "modal" editing (like Vim) using chord chains or dynamic config reloading.


### 4. Logic \& Automation (The "Programmable" Layer)

* **Standard Rules (`bspc rule`)**:
    * Static assignment: `bspc rule -a Firefox desktop='^2' follow=on`.
    * One-shot rules (`-o`): Rules that apply only to the very next window spawned.
* **External Rules Command**:
    * Writing shell scripts that intercept `Window ID`, `Class`, and `Instance` before the window is mapped.
    * Implementing complex logic (e.g., "If GIMP opens and I am on Monitor 1, float it; if Monitor 2, tile it").
* **Event Subscription (`bspc subscribe`)**:
    * Listening to the event stream: `node_add`, `node_remove`, `desktop_focus`, `monitor_add`.
    * Building daemon scripts that react to changes (e.g., "When I switch to Desktop 5, automatically change the wallpaper").
* **State Dumping (`bspc query -T`)**:
    * Reading the JSON state of the entire window manager.
    * Writing scripts to save/load layouts (parsing JSON to reconstruct trees using receptacles).


### 5. Layout Management

* **Automatic Schemes**:
    * `spiral`: Windows split the largest node, spiraling inward.
    * `longest_side`: Windows always split the longest edge (standard tiling).
    * `alternate`: Windows alternate H/V splits.
* **Manual Layouts**: Building custom grids on the fly using preselection.
* **Padding \& Gaps**:
    * `window_gap`: Space between nodes.
    * `top_padding`, `left_padding`, etc.: Reserving space for bars/panels.
    * Configuring per-monitor or per-desktop padding (e.g., Mono-monitor setup needs different padding than Dual).


### 6. The Ecosystem (Integration)

* **Polybar / Lemonbar**:
    * Parsing the `bspwm` internal report (`bspc subscribe report`) to display workspace tags (Occupied, Free, Urgent, Focused).
* **X11 Tools**:
    * `xprop`: Finding `WM_CLASS` and `WM_NAME` for rules.
    * `xwininfo`: Debugging geometry.
    * `xtitle`: Getting dynamic window titles for scripting.
    * `xdo` / `xdotool`: Sending fake input or managing windows that `bspc` cannot touch.
* **Compositors (Picom)**:
    * Handling transparency, shadows, and blur.
    * Managing opacity rules for focused vs. unfocused nodes.


### 7. Advanced "Hacks" \& Workflows

* **Scratchpads**:
    * Using the `hidden` flag and unique IDs to create "drop-down" terminals that toggle visibility.
    * Managing a "Scratchpad Desktop" (e.g., Desktop 10) vs. Hidden Nodes.
* **Swallowing**:
    * implementing terminal swallowing (launching an image viewer "eats" the terminal window until the viewer closes) via generic scripts or tools like `devour`.
* **Dynamic Gaps**:
    * Scripting gaps to disappear when only one window is present (`smart_gaps`).
* **Urgency Hints**:
    * Handling `WM_HINTS` (flashing red borders) when a background window needs attention.


### 8. Troubleshooting \& Debugging

* **The Socket**: Understanding `/tmp/bspwm_...socket`.
* **Logs**: Checking `~/.xsession-errors`.
* **Visual Debugging**: Using `bspc query -T` piped to `jq` to visualize why a window is stuck in a specific split.

