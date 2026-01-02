
# Comprehensive Guide to bspwm's Core Interface: bspc Command and Its Internals

## Overview and Architecture

**bspc** (bspwm socket client) is the foundational command-line interface that enables all user interaction with the **bspwm** window manager. Rather than implementing complex, built-in keyboard handling or graphical menus, bspwm delegates this responsibility entirely to external tools, with bspc acting as the primary bridge between user scripts (commonly managed by sxhkd—the simple X hotkey daemon) and the bspwm daemon itself[^1_1][^1_2][^1_3][^1_4][^1_5].

The architectural elegance of this design cannot be overstated. By separating concerns, bspwm maintains a lean, focused responsibility: managing the window tree structure and responding to X11 events. Meanwhile, bspc becomes the universal command-line API through which virtually all administrative and operational commands flow[^1_1][^1_6][^1_2]. This philosophy—described as "do one thing and do it well"—has enabled bspwm to become highly scriptable and customizable[^1_1].

The communication model between these components follows a clear client-server pattern: **sxhkd** listens for keyboard and pointer events, translates them into **bspc** invocations, and **bspc** then connects to a Unix domain socket maintained by the **bspwm** daemon, transmitting the command and receiving responses[^1_2][^1_7][^1_3]. This architecture is typically represented as:

```
        PROCESS SOCKET
sxhkd --------> bspc <------> bspwm
```

This design ensures that bspc requires the bspwm daemon to be running; without it, commands will fail as there's no daemon to process the messages[^1_1].

## Understanding bspc's Command Structure

### Domain-Based Command Hierarchy

bspc organizes all of its functionality around several primary **domains**, which represent different aspects of the window manager's state that can be queried or modified. The general syntax follows this pattern[^1_8][^1_9]:

```
bspc DOMAIN [SELECTOR] COMMAND [OPTIONS] [ARGUMENTS]
```

The primary domains are:

- **node**: Operations targeting individual windows (which are represented as leaf nodes in the binary tree structure)
- **desktop**: Operations targeting virtual desktops or workspaces
- **monitor**: Operations targeting physical displays or monitors
- **query**: Retrieving information about the current state without modifying it
- **rule**: Defining automatic rules for new windows
- **config**: Reading or setting configuration values
- **wm**: Whole-window-manager operations like restarting or dumping state
- **subscribe**: Listening to events emitted by bspwm

Each domain has its own set of subcommands and options[^1_1][^1_8][^1_9].

### Selectors: The Selection Language

One of the most powerful aspects of bspc is its sophisticated selector language, which allows precise specification of targets for commands. A selector consists of an optional reference, a descriptor, and any number of modifiers[^1_8][^1_9][^1_10]:

```
[REFERENCE#]DESCRIPTOR(.MODIFIER)*
```

This hierarchical structure enables complex queries and operations. The exclamation mark (`!`) can be prepended to any modifier to reverse its meaning[^1_8][^1_9][^1_10].

#### Node Selectors

Node selectors target individual windows or tree nodes. The complete grammar for node selection includes[^1_8][^1_9][^1_10]:

```
NODE_SEL := [NODE_SEL#](DIR|CYCLE_DIR|PATH|any|first_ancestor|last|newest|
            older|newer|focused|pointed|biggest|smallest|
            <node_id>)[.[!]focused][.[!]active][.[!]automatic][.[!]local]
            [.[!]leaf][.[!]window][.[!]STATE][.[!]FLAG][.[!]LAYER]
            [.[!]SPLIT_TYPE][.[!]same_class][.[!]descendant_of]
            [.[!]ancestor_of]
```

The descriptors include directional references (`DIR` meaning north, west, south, or east), cycle directions (next, prev), path-based selection, and absolute identification by node ID[^1_8][^1_9][^1_10].

For example:

```bash
# Focus the window to the east
bspc node east --focus

# Focus the previously focused window
bspc node last --focus

# Focus the biggest window
bspc node biggest --focus

# Focus a window by its ID
bspc node 0x01E00009 --focus

# Focus the focused window that is tiled (not floating)
bspc node focused.tiled --focus

# Focus nodes that are descendants of a specific node
bspc node @parent/descendant_of --focus
```

Modifiers allow filtering based on window properties:

- `[!]focused`: Only consider the currently focused node
- `[!]active`: Only consider active nodes across all desktops
- `[!]automatic`: Only consider nodes in automatic insertion mode
- `[!]local`: Only consider nodes within the reference monitor
- `[!]leaf`: Only consider leaf nodes (actual windows)
- `[!]window`: Only consider nodes that hold windows
- `[!]STATE`: Filter by window state (tiled, pseudo_tiled, floating, fullscreen)
- `[!]FLAG`: Filter by node flags (hidden, sticky, private, locked, marked, urgent)
- `[!]LAYER`: Filter by stacking layer (below, normal, above)
- `[!]SPLIT_TYPE`: Filter by parent split type (horizontal, vertical)
- `[!]same_class`: Only consider windows of the same class as the reference
- `[!]descendant_of`: Only consider descendants of a specific node
- `[!]ancestor_of`: Only consider ancestors of a specific node


#### Desktop Selectors

Desktop selectors work similarly to node selectors but operate on the desktop (workspace) level[^1_8][^1_9][^1_10]:

```
DESKTOP_SEL := [DESKTOP_SEL#](CYCLE_DIR|any|last|newest|older|newer|
               [MONITOR_SEL:](focused|^<n>)|
               <desktop_id>|<desktop_name>)[.[!]focused][.[!]active]
               [.[!]occupied][.[!]urgent][.[!]local]
```

Desktop descriptors include:

- `CYCLE_DIR`: Select next or previous desktop
- `any`: Select the first matching desktop
- `last`: Select the previously focused desktop
- `newest`: Select the newest desktop in the history
- `older`/`newer`: Select desktops relative to the reference in history
- `focused`: Select the currently focused desktop
- `^<n>`: Select the nth desktop (e.g., `^2` for the second desktop)
- `<desktop_id>`: Select by unique ID
- `<desktop_name>`: Select by name

Desktop modifiers include:

- `[!]focused`: Only consider the focused desktop
- `[!]active`: Only consider desktops that are active on their monitor
- `[!]occupied`: Only consider desktops with windows
- `[!]urgent`: Only consider desktops with urgent windows
- `[!]local`: Only consider desktops on the reference monitor


#### Monitor Selectors

Monitor selectors target physical displays[^1_8][^1_9][^1_10]:

```
MONITOR_SEL := [MONITOR_SEL#](DIR|CYCLE_DIR|any|last|newest|older|newer|
               focused|pointed|primary|^<n>|
               <monitor_id>|<monitor_name>)[.[!]focused][.[!]occupied]
```

Monitor descriptors include:

- `DIR`: Select monitors in directional order
- `CYCLE_DIR`: Cycle to next or previous monitor
- `any`: Select any monitor
- `last`: Select the previously focused monitor
- `newest`/`older`/`newer`: Historical monitor selection
- `focused`: Select the currently focused monitor
- `pointed`: Select the monitor containing the pointer
- `primary`: Select the primary monitor
- `^<n>`: Select the nth monitor
- `<monitor_id>`: Select by unique ID
- `<monitor_name>`: Select by name (e.g., `DP-1`, `HDMI-1`)

Monitor modifiers include:

- `[!]focused`: Only consider the focused monitor
- `[!]occupied`: Only consider monitors with occupied desktops


## The Node Domain: Window Management

The **node** domain provides the most direct window manipulation capabilities. Nodes represent either windows (leaf nodes) or internal branching points in the binary tree structure[^1_1][^1_3][^1_11][^1_8].

### Node Commands Overview

The complete set of node commands includes[^1_8][^1_9][^1_10]:

```bash
# Focus operations
bspc node [NODE_SEL] --focus [NODE_SEL]
bspc node [NODE_SEL] --activate [NODE_SEL]

# Movement operations
bspc node [NODE_SEL] --to-desktop DESKTOP_SEL [--follow]
bspc node [NODE_SEL] --to-monitor MONITOR_SEL [--follow]
bspc node [NODE_SEL] --to-node NODE_SEL [--follow]
bspc node [NODE_SEL] --swap NODE_SEL [--follow]

# Preselection (manual insertion mode)
bspc node [NODE_SEL] --presel-dir [~] DIR|cancel
bspc node [NODE_SEL] --presel-ratio RATIO

# Geometric operations
bspc node [NODE_SEL] --move dx dy
bspc node [NODE_SEL] --resize DIRECTION dx dy
bspc node [NODE_SEL] --ratio RATIO|(+|-)(PIXELS|FRACTION)
bspc node [NODE_SEL] --rotate 90|270|180
bspc node [NODE_SEL] --flip horizontal|vertical
bspc node [NODE_SEL] --equalize
bspc node [NODE_SEL] --balance

# Tree operations
bspc node [NODE_SEL] --circulate forward|backward
bspc node [NODE_SEL] --insert-receptacle

# State and property operations
bspc node [NODE_SEL] --state [~](tiled|pseudo_tiled|floating|fullscreen)
bspc node [NODE_SEL] --flag hidden|sticky|private|locked|marked[=on|off]
bspc node [NODE_SEL] --layer below|normal|above

# Lifecycle operations
bspc node [NODE_SEL] --close
bspc node [NODE_SEL] --kill
```


### Practical Node Operations

#### Focus Management

Focus management is perhaps the most frequently used node operation. When a node is focused, it becomes the active window, receives keyboard input, and is typically highlighted with a colored border[^1_1][^1_11]:

```bash
# Focus the focused window (appears redundant but useful in scripts)
bspc node focused --focus

# Focus a specific window by ID
bspc node 0x01E00009 --focus

# Focus the east window relative to currently focused
bspc node east --focus

# Focus the next window in cycle order
bspc node next --focus

# Focus the biggest window on the current desktop
bspc node biggest --focus

# Focus the smallest window on the current desktop
bspc node smallest --focus

# Focus a window that is in floating state
bspc node focused.floating --focus

# Focus a window that is NOT in floating state
bspc node focused.!floating --focus

# Focus all tiled windows on the focused desktop
# (useful in loops for batch operations)
bspc query -N -n focused -n .tiled
```


#### Window Movement and Sending

Moving windows between desktops and monitors is fundamental to bspwm's functionality[^1_1][^1_11]:

```bash
# Send the focused window to desktop 2
bspc node focused --to-desktop ^2

# Send the focused window to desktop 2 and follow it
bspc node focused --to-desktop ^2 --follow

# Send the focused window to the next desktop without following
bspc node focused --to-desktop next

# Send the focused window to the previous desktop
bspc node focused --to-desktop prev

# Send the focused window to the last desktop
bspc node focused --to-desktop last

# Send the focused window to the next monitor
bspc node focused --to-monitor next

# Send the focused window to the primary monitor and follow
bspc node focused --to-monitor primary --follow

# Send all windows on the current desktop to desktop 3
bspc query -N -d focused -n .window | while read node_id; do
    bspc node "$node_id" --to-desktop ^3
done
```

The `--follow` flag is particularly important. Without it, the focus remains where it was. With it, the focus "follows" the node being moved, shifting to wherever the node is sent[^1_1][^1_8].

#### Swapping Windows

Swapping exchanges the positions of two nodes in the tree, making it useful for rearranging windows[^1_1]:

```bash
# Swap the focused window with the east window
bspc node focused --swap east

# Swap the focused window with the next window
bspc node focused --swap next

# Swap with a specific window by ID
bspc node focused --swap 0x01E00009

# Swap focused window with its parent (all siblings swap implicitly)
bspc node focused --swap @parent

# Swap the entire desktop tree with the tree of desktop 2
bspc query -N -d focused -n @parent | head -1 | \
    xargs -I {} bspc node {} --swap @parent
```


#### Preselection: Manual Window Insertion

Preselection allows users to manually specify where the next window will be placed, overriding the automatic insertion scheme[^1_1][^1_11][^1_8][^1_9][^1_10]:

```bash
# Preselect the north direction for the next window
bspc node focused --presel-dir north

# Preselect west with default ratio (0.5)
bspc node focused --presel-dir west

# Preselect south with a specific ratio (30% to the south window)
bspc node focused --presel-dir south
bspc node focused --presel-ratio 0.3

# Preselect east with custom ratio (70% to the east window)
bspc node focused --presel-dir east
bspc node focused --presel-ratio 0.7

# Cancel preselection
bspc node focused --presel-dir cancel

# Toggle preselection (cancel if already set, otherwise set north)
bspc node focused --presel-dir ~north

# Preselect with dynamic ratio adjustment
# If already north, cancel; otherwise set north with ratio change
bspc node focused --presel-dir ~north
bspc node focused --presel-ratio 0.4
```

The preselection feedback visual (border and fill) helps users see where the window will be placed. This can be configured with[^1_8][^1_9][^1_10]:

```bash
# Set the preselection feedback color (RGB hex)
bspc config presel_feedback_color '#ff0000'

# Enable or disable the feedback visual
bspc config presel_feedback true
```


#### Window State Transitions

Windows can exist in four primary states, each representing different layout and interaction modes[^1_1][^1_11][^1_8]:

```bash
# Set window to tiled state (participates in tree layout)
bspc node focused --state tiled

# Set window to pseudo_tiled (tiled position, manually sizable)
bspc node focused --state pseudo_tiled

# Set window to floating (free positioning and sizing)
bspc node focused --state floating

# Set window to fullscreen (occupies entire monitor)
bspc node focused --state fullscreen

# Toggle between current and previous state
bspc node focused --state ~fullscreen

# Set multiple nodes to floating
bspc query -N -n .leaf | while read node_id; do
    bspc node "$node_id" --state floating
done

# Switch back to tiled from fullscreen
bspc node focused --state tiled
```

State transitions are crucial for workflow management. For instance, switching a window to floating state immediately removes it from the tree layout[^1_1].

#### Node Flags: Additional Properties

Beyond state, nodes have several boolean flags that modify behavior[^1_1][^1_11][^1_8]:

```bash
# Hide a window (still in tree but not displayed)
bspc node focused --flag hidden on

# Unhide a hidden window
bspc node focused --flag hidden off

# Make a window sticky (visible on all desktops)
bspc node focused --flag sticky on

# Make a window private (can't be tiled with unrelated windows)
bspc node focused --flag private on

# Lock a node (prevents removal or structural changes)
bspc node focused --flag locked on

# Mark a node (mark for batch operations)
bspc node focused --flag marked on

# Set a node as urgent (needs attention)
bspc node focused --flag urgent on

# Toggle a flag
bspc node focused --flag marked !
```


#### Geometric Operations

Beyond the tree structure, individual nodes support geometric manipulations[^1_1][^1_8]:

```bash
# Resize by moving the right edge 20 pixels right
bspc node focused --resize right 20 0

# Resize by moving the bottom edge 30 pixels down
bspc node focused --resize bottom 0 30

# Resize by moving both right and bottom
bspc node focused --resize bottom 20 30

# Move the window by 10 pixels right and 5 pixels down
bspc node focused --move 10 5

# Adjust the split ratio (controls window size relative to sibling)
bspc node focused --ratio 0.6  # This window gets 60% of space

# Adjust ratio by delta (add 10 pixels or +0.1 fraction)
bspc node focused --ratio +10
bspc node focused --ratio +0.1

# Rotate the split orientation of the parent node
bspc node focused --rotate 90

# Flip the orientation (vertical <-> horizontal)
bspc node focused --flip horizontal

# Equalize the sizes of all children under parent
bspc node focused --equalize

# Balance the tree (adjust ratios for visual balance)
bspc node focused --balance
```


#### Tree Circulation and Recycling

Advanced tree operations include circulation, which rotates windows around the tree[^1_1][^1_11]:

```bash
# Circulate windows forward
bspc node focused --circulate forward

# Circulate windows backward
bspc node focused --circulate backward

# Circulate all windows on the desktop
bspc node @root --circulate forward
```


#### Window Lifecycle

Finally, nodes can be closed or killed[^1_1][^1_8]:

```bash
# Close the focused window gracefully (sends close event)
bspc node focused --close

# Kill the focused window forcefully (terminates process)
bspc node focused --kill

# Close all windows on the current desktop
bspc query -N -d focused -n .window | while read node_id; do
    bspc node "$node_id" --close
done
```


## The Desktop Domain: Virtual Workspace Management

The **desktop** domain manages virtual workspaces, allowing users to organize windows across multiple logical spaces[^1_1][^1_3][^1_11][^1_8].

### Desktop Commands

Complete desktop command syntax[^1_8][^1_9][^1_10]:

```bash
# Focus operations
bspc desktop [DESKTOP_SEL] --focus [DESKTOP_SEL]
bspc desktop [DESKTOP_SEL] --activate [DESKTOP_SEL]

# Movement and organization
bspc desktop [DESKTOP_SEL] --to-monitor MONITOR_SEL [--follow]
bspc desktop [DESKTOP_SEL] --swap DESKTOP_SEL [--follow]

# Layout control
bspc desktop [DESKTOP_SEL] --layout monocle|tiled|cycle_dir

# Administrative operations
bspc desktop [DESKTOP_SEL] --rename <new_name>
bspc desktop [DESKTOP_SEL] --bubble next|prev
bspc desktop [DESKTOP_SEL] --remove
```


### Practical Desktop Operations

#### Desktop Navigation

Moving between desktops is essential for multi-workspace workflows[^1_1][^1_11]:

```bash
# Focus the next desktop on the current monitor
bspc desktop next --focus

# Focus the previous desktop
bspc desktop prev --focus

# Focus desktop by name
bspc desktop 'web' --focus

# Focus the second desktop on the current monitor
bspc desktop ^2 --focus

# Focus a specific desktop by ID
bspc desktop 0x0C00001 --focus

# Focus the last focused desktop
bspc desktop last --focus

# Cycle through unfocused desktops
bspc desktop prev.!focused --focus
```


#### Desktop Layout Control

Each desktop can use one of two primary layouts[^1_1][^1_11]:

```bash
# Switch to tiled layout (auto-partition)
bspc desktop focused --layout tiled

# Switch to monocle layout (one window at a time, like fullscreen)
bspc desktop focused --layout monocle

# Cycle between layouts
bspc desktop focused --layout next

# Get current desktop layout
bspc query -D -d focused --names
```

The layout system is configurable with automatic schemes[^1_1][^1_11]:

```bash
# Set automatic insertion scheme to longest_side
bspc config automatic_scheme longest_side

# Set to alternate (alternates split direction from parent)
bspc config automatic_scheme alternate

# Set to spiral (rotates windows in a spiral pattern)
bspc config automatic_scheme spiral

# Set initial polarity (where first child vs second child appears)
bspc config initial_polarity first_child
bspc config initial_polarity second_child
```


#### Desktop Movement Between Monitors

For multi-monitor setups, desktops can be moved between displays[^1_1][^1_11]:

```bash
# Move the current desktop to the next monitor
bspc desktop focused --to-monitor next

# Move the current desktop to a specific monitor by name
bspc desktop focused --to-monitor DP-1

# Move and follow (focus moves to the desktop on new monitor)
bspc desktop focused --to-monitor next --follow

# Swap two desktops (swaps all their contents)
bspc desktop 1 --swap 2

# Swap with follow
bspc desktop focused --swap next --follow
```


#### Desktop Renaming and Organization

Desktops can be renamed and reordered[^1_1][^1_8]:

```bash
# Rename the focused desktop
bspc desktop focused --rename 'workspace-1'

# Rename a specific desktop
bspc desktop web --rename 'browsing'

# Bubble the desktop (reorder within monitor)
bspc desktop focused --bubble next
bspc desktop focused --bubble prev

# Remove an empty desktop
bspc desktop empty --remove
```


## The Monitor Domain: Multi-Display Management

The **monitor** domain handles operations on physical displays, critical for multi-monitor workflows[^1_1][^1_3][^1_11][^1_8].

### Monitor Commands

Complete monitor command syntax[^1_8][^1_9][^1_10]:

```bash
# Focus operations
bspc monitor [MONITOR_SEL] --focus [MONITOR_SEL]

# Swapping and organization
bspc monitor [MONITOR_SEL] --swap MONITOR_SEL

# Desktop management
bspc monitor [MONITOR_SEL] --add-desktops <name>...
bspc monitor [MONITOR_SEL] --reset-desktops <name>...
bspc monitor [MONITOR_SEL] --reorder-desktops <name>...

# Geometric operations
bspc monitor [MONITOR_SEL] --rectangle WxH+X+Y

# Administrative
bspc monitor [MONITOR_SEL] --rename <new_name>
bspc monitor [MONITOR_SEL] --remove
```


### Practical Monitor Operations

#### Multi-Monitor Navigation

For users with multiple displays[^1_1][^1_11]:

```bash
# Focus the next monitor
bspc monitor next --focus

# Focus the previous monitor
bspc monitor prev --focus

# Focus the primary monitor
bspc monitor primary --focus

# Focus the monitor under the pointer
bspc monitor pointed --focus

# Focus a specific monitor by name (e.g., HDMI-1, DP-1)
bspc monitor DP-1 --focus

# Focus a monitor in a specific direction
bspc monitor north --focus
bspc monitor west --focus
```


#### Desktop Assignment to Monitors

A common configuration task is assigning specific desktops to each monitor[^1_1][^1_11][^1_8]:

```bash
# Add five desktops to the primary monitor
bspc monitor primary --add-desktops 1 2 3 4 5

# Add desktops to another monitor
bspc monitor DP-1 --add-desktops 6 7 8 9 10

# Reset desktops (replaces all existing ones)
bspc monitor primary --reset-desktops I II III IV V

# Reorder desktops on a monitor
bspc monitor focused --reorder-desktops 3 1 2
```


#### Complete Multi-Monitor Configuration

A typical multi-monitor bspwmrc might look like[^1_6][^1_5]:

```bash
#!/bin/bash

# Kill any existing bars
killall -q polybar

# Wait for process termination
while pgrep -u $UID -x polybar >/dev/null; do sleep 1; done

# Get connected monitors using xrandr
# Configure monitors with xrandr first
xrandr --output DP-1 --primary --mode 2560x1440 --rate 60
xrandr --output HDMI-1 --mode 1920x1080 --rate 60 --right-of DP-1

# Configure bspwm for multiple monitors
bspc monitor DP-1 --reset-desktops I II III IV V
bspc monitor HDMI-1 --reset-desktops VI VII VIII IX X

# Global settings
bspc config border_width        2
bspc config window_gap         10
bspc config split_ratio        0.52
bspc config automatic_scheme   alternate
bspc config initial_polarity   second_child
bspc config remove_disabled_monitors true
bspc config remove_unplugged_monitors true

# Launch polybar for each monitor
polybar -c ~/.config/polybar/config primary &
polybar -c ~/.config/polybar/config secondary &

# Start window manager
exec bspwm
```


#### Swapping Monitors

In complex multi-monitor setups, entire monitor configurations can be swapped[^1_1][^1_11]:

```bash
# Swap two monitors
bspc monitor primary --swap DP-1

# This swaps both the desktops and their layouts
```


## The Query Domain: State Inspection

The **query** domain allows users to retrieve information about bspwm's state without modifying it. This is essential for scripting and dynamic configuration[^1_1][^1_3][^1_11][^1_8].

### Query Commands

Complete query command syntax[^1_8][^1_9][^1_10]:

```bash
# Query commands
bspc query --nodes [NODE_SEL]      # List matching node IDs
bspc query --desktops [DESKTOP_SEL] # List matching desktop IDs or names
bspc query --monitors [MONITOR_SEL] # List matching monitor IDs or names
bspc query --tree [SELECTOR]        # Print JSON tree structure

# Query options
bspc query [OPTIONS]
  --monitor [MONITOR_SEL]
  --desktop [DESKTOP_SEL]
  --node [NODE_SEL]
  --names                  # Print names instead of IDs
  --history               # Print node/desktop history
  --stack                 # Print window stacking order
```


### Practical Query Operations

#### Finding Nodes

One of the most critical query operations is finding specific nodes/windows[^1_1][^1_3][^1_11][^1_8][^1_12]:

```bash
# List all node IDs on the focused desktop
bspc query -N -d focused

# List all window nodes (exclude parent nodes)
bspc query -N -n .window

# List all hidden nodes
bspc query -N -n .hidden

# List all floating windows on the focused desktop
bspc query -N -d focused -n .floating

# List all tiled windows on the focused desktop
bspc query -N -d focused -n .tiled

# List the biggest window on the focused desktop
bspc query -N -d focused -n biggest

# List all windows with the same class as the focused window
bspc query -N -n focused.same_class

# Find all urgent windows
bspc query -N -n .urgent

# Find the parent of the focused node
bspc query -N -n focused -n @parent

# List with names
bspc query -N --names -d focused

# Print node IDs and their names
bspc query -N -d focused --names
```

The output from query commands is typically node IDs in hexadecimal format (e.g., `0x01E00009`), which can then be used in subsequent bspc commands[^1_3][^1_13].

#### Desktop and Monitor Queries

Similar queries work for desktops and monitors[^1_1][^1_11][^1_8]:

```bash
# List all desktops
bspc query -D

# List desktops on a specific monitor
bspc query -D -m DP-1

# List desktop names instead of IDs
bspc query -D --names

# List occupied desktops
bspc query -D -d .occupied

# List all monitors
bspc query -M

# List primary monitor
bspc query -M -m primary

# Print monitor names
bspc query -M --names

# List monitors with occupied desktops
bspc query -M -m .occupied
```


#### Tree Queries

For advanced diagnostics and state inspection, tree queries return JSON representations[^1_1][^1_11][^1_8]:

```bash
# Print the tree rooted at the focused desktop
bspc query -T -d focused

# Print the entire window tree on the primary monitor
bspc query -T -m primary

# Print the tree starting from the focused node
bspc query -T -n focused
```

The JSON structure provides complete tree information, useful for debugging and external tools[^1_1][^1_11][^1_8][^1_12].

#### Practical Query Use Cases

Queries enable sophisticated scripting patterns[^1_1][^1_11][^1_14][^1_15]:

```bash
# Move the focused window to a desktop and follow it
id=$(bspc query -N -n focused); \
  bspc node -d ^2; \
  bspc node -f ${id}

# Move all floating windows to a specific desktop
bspc query -N -n .floating | while read node; do
    bspc node "$node" -d ^3
done

# Kill all hidden windows
bspc query -N -n .hidden | while read node; do
    bspc node "$node" -k
done

# Close all windows on a desktop except the focused one
focused_id=$(bspc query -N -n focused)
bspc query -N -d focused -n .window | grep -v "$focused_id" | while read node; do
    bspc node "$node" -c
done

# Get the largest window and focus it
largest=$(bspc query -N -d focused -n biggest)
bspc node "$largest" -f

# List all windows and their states
bspc query -N -d focused --names | while read node; do
    echo "Node: $node"
done
```


## The Rule Domain: Window Auto-Configuration

The **rule** domain defines automatic behaviors for new windows based on their class, instance, or name[^1_1][^1_11][^1_16][^1_8].

### Rule Commands

Complete rule command syntax[^1_8][^1_9]:

```bash
# Add a rule
bspc rule --add (<class_name>|*)[:(<instance_name>|*)[:(<name>|*)]]
           [-o|--one-shot]
           [monitor=MONITOR_SEL|desktop=DESKTOP_SEL|node=NODE_SEL]
           [state=STATE]
           [layer=LAYER]
           [split_dir=DIR]
           [split_ratio=RATIO]
           [(hidden|sticky|private|locked|marked|center|follow|manage|focus|border)=(on|off)]
           [rectangle=WxH+X+Y]

# Remove rules
bspc rule --remove ^<n>|head|tail|(<class_name>|*)[:(<instance_name>|*)[:(<name>|*)]]

# List rules
bspc rule --list
```


### Window Classification

To create effective rules, one must first identify window class and instance names. The standard tool for this is `xprop`[^1_6][^1_16]:

```bash
# Get window properties
xprop

# Then click on the window you want to identify
# Look for the WM_CLASS line, which shows format: "instance", "class"
# Example output: WM_CLASS(STRING) = "firefox", "Firefox"
```


### Common Rule Patterns

Typical bspwmrc rule definitions follow patterns like[^1_6][^1_5][^1_16]:

```bash
#!/bin/bash

# Remove all existing rules first
bspc rule -r '*'

# Define rules for various applications
bspc rule -a Gimp desktop=^8 state=floating
bspc rule -a Chromium desktop=^2
bspc rule -a mplayer2 state=floating
bspc rule -a Kupfer.py focus=on
bspc rule -a Screenkey manage=off
bspc rule -a Pavucontrol state=floating
bspc rule -a Pcmanfm state=floating center=on

# Firefox rules - instance:class format
bspc rule -a firefox desktop=^1 focus=on

# Terminal floating windows
bspc rule -a st-floating state=floating

# Specific floating size
bspc rule -a st rectangle=800x600+10+10 state=floating

# One-shot rules (apply only once)
bspc rule -a 'VirtualBox.*' -o state=floating
```


### Rule Properties

Rules can control numerous window properties[^1_1][^1_11][^1_8]:

```bash
# Monitor and desktop placement
bspc rule -a myapp monitor=DP-1 desktop=^3

# Node placement (insert into specific parent)
bspc rule -a myapp node=@parent

# Window state
bspc rule -a myapp state=floating
bspc rule -a myapp state=fullscreen
bspc rule -a myapp state=pseudo_tiled

# Stacking layer
bspc rule -a myapp layer=above
bspc rule -a myapp layer=below

# Split parameters for new nodes
bspc rule -a myapp split_dir=north split_ratio=0.5

# Node flags
bspc rule -a myapp hidden=on
bspc rule -a myapp sticky=on
bspc rule -a myapp private=on
bspc rule -a myapp locked=on
bspc rule -a myapp marked=on

# Behavior modifiers
bspc rule -a myapp follow=on           # Follow to new desktop
bspc rule -a myapp center=on           # Center window
bspc rule -a myapp manage=off          # Don't manage (no decorations)
bspc rule -a myapp focus=on            # Focus immediately
bspc rule -a myapp border=on           # Draw border

# Floating rectangle specification
bspc rule -a myapp rectangle=640x480+100+100
```


### Advanced Rule Techniques

#### One-Shot Rules

One-shot rules apply only once and are then automatically removed[^1_6][^1_16]:

```bash
# Rule applies only to the first instance
bspc rule -a 'firefox' -o desktop=^1

# Useful for startup applications
bspc rule -a 'rofi' -o state=floating sticky=on
```


#### Pattern Matching Rules

Rules can use wildcards and patterns[^1_1][^1_11]:

```bash
# Match all Firefox-derived applications
bspc rule -a 'firefox*' desktop=^1

# Match any instance of a class
bspc rule -a 'st' state=floating

# Match specific instance:class combination
bspc rule -a 'keepassxc:KeePassXC' state=floating center=on
```


#### Complex Rule Chains

Multiple rules can be layered to create sophisticated behaviors[^1_1][^1_11][^1_8]:

```bash
# Specific application instances first (most specific)
bspc rule -a 'mpv:mpv' state=floating rectangle=960x540+10+10

# General class rules (less specific)
bspc rule -a 'mpv' desktop=^4

# Fallback behaviors
bspc rule -a 'Floating*' state=floating
```


## The Config Domain: System Configuration

The **config** domain allows reading and setting configuration values that control bspwm's behavior[^1_1][^1_11][^1_8][^1_9].

### Global Settings

Global settings affect the entire window manager[^1_8]:

```bash
# Colors (RGB hex format)
bspc config normal_border_color         '#222222'
bspc config active_border_color         '#555555'
bspc config focused_border_color        '#ff0000'
bspc config presel_feedback_color       '#ff0000'

# Dimensions
bspc config border_width                2
bspc config window_gap                  10
bspc config split_ratio                 0.52

# Insertion scheme
bspc config automatic_scheme            alternate
bspc config initial_polarity            second_child

# Layout behavior
bspc config borderless_monocle          true
bspc config gapless_monocle             true
bspc config single_monocle              false
bspc config top_monocle_padding         0
bspc config right_monocle_padding       0
bspc config bottom_monocle_padding      0
bspc config left_monocle_padding        0

# Pointer interaction
bspc config pointer_modifier            mod1
bspc config pointer_motion_interval     25
bspc config pointer_action1             move
bspc config pointer_action2             resize_side
bspc config pointer_action3             resize_corner
bspc config click_to_focus              button1
bspc config focus_follows_pointer       false
bspc config pointer_follows_focus       false
bspc config pointer_follows_monitor     false

# Focus and behavior
bspc config directional_focus_tightness low
bspc config removal_adjustment          unbalanced
bspc config center_pseudo_tiled         true
bspc config honor_size_hints            true

# External rules command
bspc config external_rules_command      "/path/to/rules/script"

# Status reporting
bspc config status_prefix               ''
bspc config mapping_events_count        -1

# EWMH compliance
bspc config ignore_ewmh_focus           false
bspc config ignore_ewmh_fullscreen      'enter'
bspc config ignore_ewmh_struts          false

# Monitor management
bspc config remove_disabled_monitors    true
bspc config remove_unplugged_monitors   true
bspc config merge_overlapping_monitors  false
```


### Per-Monitor and Per-Desktop Settings

Some settings can be applied to specific monitors or desktops[^1_8]:

```bash
# Per-monitor padding
bspc config -m DP-1 top_padding 50
bspc config -m DP-1 bottom_padding 0
bspc config -m DP-1 left_padding 0
bspc config -m DP-1 right_padding 0

# Per-desktop window gap
bspc config -d '^1' window_gap 20
bspc config -d '^2' window_gap 0

# Per-node border width
bspc config -n focused border_width 3
bspc config -n unfocused border_width 1
```


### Reading Configuration

To read current settings[^1_1][^1_11]:

```bash
# Read a specific setting
bspc config border_width

# Read all settings (returns name value pairs)
bspc config

# Check a boolean setting
bspc config focus_follows_pointer
```


### Common Configuration Examples

A typical complete configuration[^1_6][^1_5]:

```bash
#!/bin/bash

# bspwmrc - bspwm configuration script

# Monitor setup (done before bspc commands)
xrandr --output DP-1 --mode 2560x1440 --rate 60 --primary
xrandr --output HDMI-1 --mode 1920x1080 --rate 60 --right-of DP-1

# Setup desktops
bspc monitor DP-1 -d 1 2 3 4 5
bspc monitor HDMI-1 -d 6 7 8 9 10

# Border and window gap
bspc config border_width 2
bspc config window_gap 8

# Colors
bspc config focused_border_color "#ff6600"
bspc config active_border_color "#0066ff"
bspc config normal_border_color "#333333"
bspc config presel_feedback_color "#ff6600"

# Insertion and layout
bspc config split_ratio 0.52
bspc config automatic_scheme alternate
bspc config initial_polarity first_child

# Monocle layout settings
bspc config borderless_monocle true
bspc config gapless_monocle true
bspc config single_monocle false

# Focus behavior
bspc config focus_follows_pointer false
bspc config pointer_follows_focus false

# Rules
bspc rule -r "*"
bspc rule -a Gimp state=floating
bspc rule -a firefox desktop=^1
bspc rule -a Chromium desktop=^2
bspc rule -a mpv state=floating
```


## The Subscribe Domain: Event Streaming

The **subscribe** domain allows external programs to receive real-time notifications about changes in bspwm's state[^1_1][^1_11][^1_8].

### Subscribe Commands

Complete subscribe command syntax[^1_8]:

```bash
# Subscribe to events
bspc subscribe [OPTIONS] (all|report|monitor|desktop|node|...)*

# Options
--fifo                  # Output to FIFO file
--count COUNT           # Exit after COUNT events
```


### Event Types

The available events include[^1_8][^1_9][^1_10]:

```
monitor events:
  monitor_add <monitor_id> <monitor_name> <monitor_geometry>
  monitor_rename <monitor_id> <old_name> <new_name>
  monitor_remove <monitor_id>
  monitor_swap <src_monitor_id> <dst_monitor_id>
  monitor_focus <monitor_id>
  monitor_geometry <monitor_id> <monitor_geometry>

desktop events:
  desktop_add <monitor_id> <desktop_id> <desktop_name>
  desktop_rename <monitor_id> <desktop_id> <old_name> <new_name>
  desktop_remove <monitor_id> <desktop_id>
  desktop_swap <src_monitor_id> <src_desktop_id> <dst_monitor_id> <dst_desktop_id>
  desktop_transfer <src_monitor_id> <src_desktop_id> <dst_monitor_id>
  desktop_focus <monitor_id> <desktop_id>
  desktop_activate <monitor_id> <desktop_id>
  desktop_layout <monitor_id> <desktop_id> tiled|monocle

node events:
  node_add <monitor_id> <desktop_id> <ip_id> <node_id>
  node_remove <monitor_id> <desktop_id> <node_id>
  node_swap <src_monitor_id> <src_desktop_id> <src_node_id> 
            <dst_monitor_id> <dst_desktop_id> <dst_node_id>
  node_transfer <src_monitor_id> <src_desktop_id> <src_node_id>
                <dst_monitor_id> <dst_desktop_id> <dst_node_id>
  node_focus <monitor_id> <desktop_id> <node_id>
  node_activate <monitor_id> <desktop_id> <node_id>
  node_presel <monitor_id> <desktop_id> <node_id> (dir DIR|ratio RATIO|cancel)
  node_stack <node_id_1> below|above <node_id_2>
  node_geometry <monitor_id> <desktop_id> <node_id> <node_geometry>
  node_state <monitor_id> <desktop_id> <node_id> tiled|pseudo_tiled|floating|fullscreen on|off
  node_flag <monitor_id> <desktop_id> <node_id> hidden|sticky|private|locked|marked|urgent on|off
  node_layer <monitor_id> <desktop_id> <node_id> below|normal|above

pointer events:
  pointer_action <monitor_id> <desktop_id> <node_id> move|resize_corner|resize_side begin|end

report events:
  report <status_string>
```


### Practical Subscription Usage

Event subscription is typically used for status bars, external tools, and automation[^1_1][^1_11][^1_8]:

```bash
# Subscribe to all events
bspc subscribe all

# Subscribe to only desktop changes
bspc subscribe desktop

# Subscribe to node-related events only
bspc subscribe node

# Subscribe and exit after 10 events
bspc subscribe --count 10 all

# Parse events in a script
bspc subscribe monitor desktop | while read -r event; do
    case "$event" in
        monitor_focus*)
            echo "Monitor focus changed: $event"
            ;;
        desktop_focus*)
            echo "Desktop focus changed: $event"
            ;;
    esac
done

# Use with FIFO for status bar
bspc subscribe --fifo /tmp/bspwm_fifo report &
while true; do
    read -r status < /tmp/bspwm_fifo
    echo "$status"  # Update polybar or other status system
done
```


### Status Report Format

The report event provides a compact status string with specific format[^1_8][^1_9]:

```
Each report event consists of items separated by colons, where each item has the form `<type><value>`:
```

```
M<monitor_name>     - Monitor
m<monitor_name>     - Focused monitor
O<desktop_name>     - Occupied desktop (has windows)
o<desktop_name>     - Focused desktop
F<desktop_name>     - Active desktop
f<desktop_name>     - Focused and active desktop
U<desktop_name>     - Urgent desktop
u<desktop_name>     - Focused and urgent desktop
L(T|M)              - Layout (Tiled or Monocle)
T(T|P|F|=|@)        - Window state (Tiled, Pseudo-tiled, Floating, Equal, or Parent)
G(S?P?L?M?)         - Geometric flags
```

Example status strings:

```
m1:o1:Lclient:client.focused:O2:o3:o4:T1:L2:Tiled
m1:O1:o2:T1:L3:Monocle
```


## The Wm Domain: Window Manager Operations

The **wm** domain provides system-level operations on the window manager itself[^1_1][^1_11][^1_8].

### Wm Commands

Complete wm command syntax[^1_8][^1_9]:

```bash
# State operations
bspc wm --dump-state
bspc wm --load-state <file_path>

# Monitor operations
bspc wm --add-monitor <name> WxH+X+Y
bspc wm --reorder-monitors <name>...
bspc wm --adopt-orphans

# History and status
bspc wm --record-history on|off
bspc wm --get-status

# Lifecycle
bspc wm --restart
```


### Practical Wm Operations

#### State Dump and Restore

The most complex wm operations involve saving and restoring complete window manager state[^1_1][^1_11][^1_8]:

```bash
# Dump the entire window manager state to a file
bspc wm --dump-state > ~/bspwm_state.json

# Later, restore the state
bspc wm --load-state ~/bspwm_state.json

# This is useful for:
# - Saving layouts before reconfiguring monitors
# - Recreating specific desktop layouts
# - Session persistence
```


#### Monitor Reordering

The reorder-monitors command allows changing monitor order[^1_1][^1_11]:

```bash
# Reorder monitors (affects desktop cycling order)
bspc wm --reorder-monitors DP-1 HDMI-1

# Useful in multi-monitor scripts
xrandr --output DP-1 --mode 2560x1440 --primary
xrandr --output HDMI-1 --mode 1920x1080 --right-of DP-1
bspc wm --reorder-monitors DP-1 HDMI-1
```


#### Adopting Orphaned Windows

When monitors are physically removed or disabled, windows may become orphaned[^1_1][^1_11]:

```bash
# Reassign orphaned windows to remaining monitors
bspc wm --adopt-orphans

# Useful in laptop docking/undocking scenarios
```


#### History Recording

For debugging and advanced use cases, window manager history can be recorded[^1_1][^1_11]:

```bash
# Enable history recording
bspc config record_history true

# Disable it
bspc config record_history false
```


#### Getting Manager Status

The get-status command provides current status information[^1_1][^1_11]:

```bash
# Get the current manager status
bspc wm --get-status

# Returns the same format as report events
```


#### Restarting the Window Manager

One of the most important wm operations is restarting bspwm while preserving state[^1_1][^1_11]:

```bash
# Restart bspwm (preserves window layout)
bspc wm --restart

# Full restart with state loss
killall bspwm; bspwm &
```


## Advanced bspc Patterns and Use Cases

### Complex Scripting Examples

#### Dynamic Workspace Creation

A common pattern is creating workspaces on demand[^1_1][^1_11]:

```bash
#!/bin/bash

# Create a new desktop on the focused monitor if not already present
create_desktop_if_missing() {
    local desktop_name=$1
    
    # Check if desktop already exists
    if bspc query -D -d "$desktop_name" >/dev/null 2>&1; then
        # Desktop exists, just focus it
        bspc desktop "$desktop_name" --focus
    else
        # Create new desktop
        bspc monitor focused --add-desktops "$desktop_name"
        bspc desktop "$desktop_name" --focus
    fi
}

# Usage
create_desktop_if_missing 'work'
```


#### Intelligent Window Movement

Moving windows with context awareness[^1_1][^1_11][^1_14]:

```bash
#!/bin/bash

# Move window to a specific app's desktop
move_to_app_desktop() {
    local app_class=$1
    local app_desktop
    
    # Find desktop where app exists
    app_desktop=$(bspc query -D -d ".occupied" | while read desktop; do
        if bspc query -N -d "$desktop" -n ".window" | \
           xdotool search --class "$app_class" 2>/dev/null >/dev/null; then
            echo "$desktop"
            break
        fi
    done)
    
    if [ -n "$app_desktop" ]; then
        # Move focused window to app's desktop
        bspc node focused --to-desktop "$app_desktop" --follow
    fi
}

# Usage
move_to_app_desktop 'Firefox'
```


#### Window Layout Templates

Pre-configured layouts that can be applied to desktops[^1_1][^1_11][^1_8]:

```bash
#!/bin/bash

# Apply a three-column layout to the focused desktop
layout_three_column() {
    # Add three windows vertically first
    local desktop=$(bspc query -D -d focused)
    
    # First window gets 50% width (splits vertically)
    bspc node -d "$desktop" -n first -p west -o 0.5
    
    # Second window gets 25% width
    bspc node -d "$desktop" -n second -p west -o 0.5
}

# Apply a main+side layout
layout_main_side() {
    bspc node focused --presel-dir east --presel-ratio 0.7
}
```


#### Focus Ring Management

Cycling through windows in focus order[^1_1][^1_11]:

```bash
#!/bin/bash

# Cycle focus through windows
cycle_focus() {
    local direction=${1:-next}
    
    # Get current node
    local current=$(bspc query -N -n focused)
    
    # Get all window nodes on desktop
    local nodes=$(bspc query -N -n ".window" | grep -v "$current")
    
    # Focus next or previous
    local next_node=$(echo "$nodes" | head -1)
    if [ -n "$next_node" ]; then
        bspc node "$next_node" --focus
    fi
}

# Usage
cycle_focus next
cycle_focus prev
```


#### Workspace Stashing

Temporarily hiding windows for focus[^1_1][^1_11]:

```bash
#!/bin/bash

# Hide all windows except focused on current desktop
stash_others() {
    local focused=$(bspc query -N -n focused)
    
    bspc query -N -d focused -n ".window" | while read node; do
        if [ "$node" != "$focused" ]; then
            bspc node "$node" --flag hidden on
        fi
    done
}

# Restore stashed windows
unstash() {
    bspc query -N -n ".hidden" -d focused | while read node; do
        bspc node "$node" --flag hidden off
    done
}
```


#### Interactive Window Selection

Using external tools with bspc for enhanced interactivity[^1_1][^1_11]:

```bash
#!/bin/bash

# Use dmenu to select and focus window
select_window_dmenu() {
    # Get list of windows
    local windows=$(bspc query -N -n ".window" --names)
    
    # Use dmenu for selection
    local selected=$(echo "$windows" | dmenu -l 10)
    
    if [ -n "$selected" ]; then
        # Extract node ID and focus it
        # Note: This is a simplified example
        local node_id=$(bspc query -N -n ".window" --names | \
                       grep "$selected" | head -1)
        bspc node "$node_id" --focus
    fi
}
```


### Integration with Other Tools

#### Polybar Integration

Status bars need constant bspwm state updates[^1_1][^1_11][^1_8]:

```bash
#!/bin/bash

# Module for polybar
get_bspwm_status() {
    bspc subscribe report | while read -r report; do
        # Parse report format
        # Extract relevant parts for display
        
        # Get focused desktop
        focused_desktop=$(echo "$report" | grep -o 'o[0-9]*' | sed 's/o//')
        
        # Get occupied desktops
        occupied=$(echo "$report" | grep -o 'O[0-9]*' | sed 's/O//')
        
        # Format for display
        echo "[$focused_desktop] $occupied"
    done
}
```


#### Dunst Notifications

Sending notifications on bspwm events[^1_1][^1_11]:

```bash
#!/bin/bash

# Watch for desktop changes and notify
bspc subscribe desktop | while read -r event; do
    case "$event" in
        desktop_focus*)
            desktop=$(echo "$event" | awk '{print $NF}')
            dunstify -u low -t 1000 "Focus: Desktop $desktop"
            ;;
    esac
done
```


#### External Rules Commands

Complex rule logic can be delegated to external scripts[^1_1][^1_11][^1_8]:

```bash
#!/bin/bash
# external-rules.sh

# Get window properties
id=$1
class=$2
instance=$3
name=$4

# Complex rule logic
if [[ "$class" == "Firefox" ]]; then
    desktop=1
elif [[ "$class" == "Blender" ]]; then
    desktop=3
    echo "state=fullscreen"
else
    desktop=0
fi

echo "desktop=^$desktop"
```

Then configure bspwm to use it:

```bash
bspc config external_rules_command "/path/to/external-rules.sh"
```


### Performance Optimization

For efficient scripting with large numbers of windows[^1_1][^1_11]:

```bash
# Batch operations
# Instead of:
for i in {1..100}; do
    bspc node $id --resize right 10 0
done

# Do:
bspc node $id --resize right 1000 0

# Query once and reuse results
nodes=$(bspc query -N -d focused -n .window)
echo "$nodes" | while read node; do
    # Use $node multiple times
    bspc node "$node" --state floating
    bspc node "$node" --move 100 100
done

# Use pipes efficiently
bspc query -N -n .marked | xargs -I {} bspc node {} --flag marked off
```


## Internal Implementation Details

### Socket Communication Protocol

```
bspc communicates with bspwm via a Unix domain socket, typically located at `/tmp/bspwm<hostname>_<display>_<screen>-socket`[^1_1][^1_17][^1_2][^1_7]. The protocol is text-based, with commands formatted as strings sent to the socket[^1_1][^1_17][^1_2].
```

The message format follows this pattern:

```
<COMMAND> [SELECTOR] <OPERATIONS>
```

Where each component is space-separated and parameters with special characters are properly quoted[^1_1].

The socket communication flow works as follows[^1_1][^1_17][^1_2]:

1. bspc opens a connection to the bspwm socket
2. Sends the formatted command string
3. Receives a response (typically empty for successful commands, or an error message)
4. For query commands, receives the requested data
5. Closes the connection

This is a one-shot request-response model, with each invocation of bspc creating a new socket connection[^1_1][^1_17][^1_2].

### Selector Parsing and Resolution

The selector system involves complex parsing logic that resolves descriptors and modifiers into specific tree nodes[^1_8][^1_9][^1_10]. The parsing follows a hierarchical approach:

1. **Reference Parsing**: Extract the reference node/desktop/monitor using `#` as separator
2. **Descriptor Parsing**: Parse the primary descriptor (DIR, name, ID, etc.)
3. **Modifier Parsing**: Apply each modifier in sequence with `.` as separator
4. **Resolution**: Walk the tree structure to find the target

For directional selectors, bspwm walks the tree looking for nodes in the specified direction. For ID-based selectors, it performs a hash lookup. For name-based selectors, it searches the tree by name property[^1_1][^1_8].

### Binary Tree Structure

The core data structure in bspwm is a full binary tree where[^1_1][^1_3][^1_11]:

- Each internal node represents a split (horizontal or vertical)
- Each internal node has exactly two children
- Each leaf node holds a window
- Each node has associated geometry (rectangle) and split parameters

When a window is inserted, the tree is restructured to accommodate it according to the insertion mode (automatic or manual). The insertion process[^1_1][^1_3][^1_11]:

1. Finds the insertion point (usually the focused leaf)
2. Creates a new internal node
3. Attaches the existing window as one child
4. Attaches the new window as the other child
5. Assigns split parameters based on insertion mode

When a window is removed[^1_1][^1_3][^1_11]:

1. The containing leaf node is removed
2. The parent internal node is also removed
3. The sibling subtree is promoted to replace the parent

This tree structure enables efficient window layout management and complex tiling patterns[^1_1][^1_3][^1_11].

## Conclusion

The **bspc** command and its underlying infrastructure represent a masterpiece of Unix philosophy applied to window management. By separating concerns between the window manager daemon, the command-line interface, and external tools, bspwm achieves remarkable flexibility and scriptability[^1_1][^1_6][^1_2][^1_3].

Understanding bspc deeply—from basic window operations through advanced query patterns and complex scripting—unlocks the full potential of bspwm. Whether managing a simple two-monitor setup or automating complex workflows across dozens of windows, bspc provides the precise control necessary to bend window management to your specific needs[^1_1][^1_2].

The socket-based communication model, sophisticated selector language, and event streaming capabilities combine to create an environment where external tools can seamlessly integrate with and extend bspwm's functionality. This design has proven so effective that it has influenced other window managers and continues to attract a dedicated community of power users[^1_1][^1_6][^1_2][^1_3][^1_4].

As you become more proficient with bspc, you'll find yourself creating increasingly sophisticated workflows, custom integrations, and automated tasks that would be impossible or extremely difficult to implement in monolithic window managers. The investment in learning bspc's command structure and internals pays dividends through a remarkably powerful and personalized window management system.

[^1_1]: https://linuxcommandlibrary.com/man/bspc

[^1_2]: https://dev.to/l04db4l4nc3r/bspwm-a-bare-bones-window-manager-44di

[^1_3]: https://madnight.github.io/bspwm/

[^1_4]: https://www.linuxlinks.com/bspwm-tiling-window-manager/

[^1_5]: https://forum.endeavouros.com/t/beginners-quick-start-guide-to-endeavour-os-bspwm-edition/13095

[^1_6]: https://wiki.archlinux.org/title/Bspwm

[^1_7]: https://www.reddit.com/r/unixporn/comments/feseh2/getting_started_with_bspwm_for_beginners/

[^1_8]: https://www.dragonflybsd.org/cgi/web-man?command=bspc\&section=1

[^1_9]: https://manpages.debian.org/testing/bspwm/bspc.1.en.html

[^1_10]: https://manpages.ubuntu.com/manpages/focal/man1/bspwm.1.html

[^1_11]: https://dharmx.is-a.dev/bspwm-basics/

[^1_12]: https://www.youtube.com/watch?v=cQNy6yVvfBU

[^1_13]: https://www.reddit.com/r/bspwm/comments/6or5r0/bspc_query_output_and_bspc_format/

[^1_14]: https://www.reddit.com/r/bspwm/comments/oht7on/i_think_im_stupid/

[^1_15]: https://pmc.ncbi.nlm.nih.gov/articles/PMC2687026/

[^1_16]: https://www.youtube.com/watch?v=WBE1NrKX9ZA

[^1_17]: https://www.reddit.com/r/bspwm/comments/l0ol12/bspwms_linux_ipc_socket_protocol_anyone_familiar/

[^1_18]: https://docs.yoctoproject.org/bsp-guide/bsp.html

[^1_19]: https://www.edibon.com/en/computer-controlled-base-unit-for-bs

[^1_20]: https://wiki.st.com/stm32mpu/wiki/OpenSTLinux_BSP_architecture_overview

[^1_21]: https://www.sciencedirect.com/science/article/pii/S0264127524000200

[^1_22]: https://www.bspc.net/resources/documents/710

[^1_23]: https://bbs.archlinux.org/viewtopic.php?id=258194

[^1_24]: https://www.youtube.com/watch?v=Ihvf8urQLJk

[^1_25]: https://www.windriver.com/solutions/learning/board-support-packages

[^1_26]: https://forum.level1techs.com/t/how-to-install-bspwm/54178

[^1_27]: https://github.com/baskerville/bspwm

[^1_28]: https://erproof.com/bpc/sap-bpc-training/sap-bpc-embedded-architecture/

[^1_29]: https://www.youtube.com/watch?v=smZUSnUCjwg

[^1_30]: https://bgdawes.github.io/bspwm-xfce-dotfiles/

[^1_31]: https://www.youtube.com/watch?v=BmAiMAILg94

[^1_32]: https://www.cheat-sheets.org/project/tldr/command/bspc/os/linux/

[^1_33]: https://hexmos.com/freedevtools/tldr/linux/bspc/

[^1_34]: https://wiki.archcraft.io/docs/window-managers/tiling-wm/bspwm/

[^1_35]: https://gist.github.com/amit08255/43ed6efdc1952d88f9a61e86f375e924

[^1_36]: https://github.com/xintron/configs/blob/master/.config/bspwm/bspwmrc

[^1_37]: https://forum.endeavouros.com/t/launching-application-on-fresh-boot-on-bspwm/56595

[^1_38]: https://habr.com/en/articles/779176/

[^1_39]: https://docs.mojolicious.org/IPC/Cmd

[^1_40]: https://metacpan.org/pod/IPC::System::Simple

[^1_41]: https://github.com/baskerville/bspwm/issues/1227

[^1_42]: https://stackoverflow.com/questions/11279602/writing-messages-of-different-types-through-sockets-in-c

[^1_43]: https://beej.us/guide/bgipc/html/

[^1_44]: https://www.reddit.com/r/bspwm/comments/11o0lsj/bspw_multiple_monitor_select_monitordesktop_and/

[^1_45]: https://www.youtube.com/watch?v=KEiur5aZnIM

[^1_46]: https://www.reddit.com/r/programming/comments/11vtu2r/c_commandline_argument_parser/

[^1_47]: https://stackoverflow.com/questions/73917457/bspwm-does-not-switch-focus-to-my-second-screen-it-works-when-going-there-with

[^1_48]: https://github.com/baskerville/bspwm/blob/master/README.md

[^1_49]: https://stackoverflow.com/questions/40248701/how-does-one-provide-ipcrun-with-an-argument-set-by-user-input-that-contains-e

[^1_50]: https://bbs.archlinux.org/viewtopic.php?id=255294

[^1_51]: https://bbs.archlinux.org/viewtopic.php?id=277844

[^1_52]: https://github.com/openbmc/docs/blob/master/designs/redfish-eventservice.md

[^1_53]: https://my-take-on.tech/2020/07/03/some-tricks-for-sxhkd-and-bspwm/

[^1_54]: https://www.pulsar.com.tr/upload/urunler_robo/124/ozellikler/SEL_PROGRAMMING_MANUAL(ME0224-6B).pdf

[^1_55]: https://www.bspc.net/resources/documents/71

[^1_56]: https://www.bspc.net/resources/documents/879

[^1_57]: https://manpages.debian.org/experimental/bspwm/bspc.1.en.html

[^1_58]: https://www.bspc.net/resources/documents/471

[^1_59]: https://www.youtube.com/watch?v=au-ZEcJKNb0

[^1_60]: https://manpages.debian.org/stretch/bspwm/bspc.1

[^1_61]: https://www.bsp.gov.ph/Media_And_Research/Annual%20Report/annrep2022.pdf

[^1_62]: https://bbs.archlinux.org/viewtopic.php?id=292172

[^1_63]: https://github.com/baskerville/bspwm/blob/master/doc/CHANGELOG.md

