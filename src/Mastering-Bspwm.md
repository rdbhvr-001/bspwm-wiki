### How to master the bspwm?

To master **bspwm** (Binary Space Partitioning Window Manager), you must move beyond simple configuration and understand it as a programmable environment. Because `bspwm` is merely a C program controlled by a socket client (`bspc`), mastery is equivalent to mastering the `bspc` command and the logic of binary trees.

Here is the exhaustive curriculum for total bspwm mastery, structured from architectural concepts to advanced kernel-level scripting.

### The Theoretical Foundation

> [!IMPORTANT]
> You cannot predict window behavior without understanding the data structure.

1) **The Full Binary Tree**: Understand how every desktop is a root node that splits recursively.
2) **Internal Nodes vs. Leaves**:
    * *Leaves* hold windows (`client`).
    * *Internal Nodes* hold structure (Split type: Horizontal/Vertical + Split Ratio).
3) **Traversal Paths**: Understanding how `next`/`prev` cycles through the tree (pre-order, in-order, post-order traversal logic).
4) **Tree Rotation**: How "circulate" commands (`bspc node -C`) physically rotate the tree structure.


### The Control Interface

Mastering every subcommand of the binary space partitioning controller.

1) **Node Selection (The Query Language)**:
    * **Directional**: `north`, `south`, `east`, `west`.
    * **History**: `last`, `older`, `newer`.
    * **Family**: `parent`, `brother`, `ancestor`, `descendant`.
    * **Path Notations**: `@/` (root), `@/1` (left child), `@/2` (right child).
    * **Modifiers**: `.floating`, `.tiled`, `.pseudo_tiled`, `.locked`, `.sticky`, `.private`, `.hidden`.
2) **Node Manipulation**:
    * **Movement**: Moving nodes within a desktop, across desktops, or across monitors (`bspc node -m`, `-d`, `-n`).
    * **Resizing**: Differential resizing (`-z top -20 0`) vs. Ratio resizing.
    * **Swapping**: Exchanging the position of two nodes (`bspc node -s`).
    * **Flags**: Toggling `hidden` (minimized), `sticky` (global), `private` (ignored by automatic splitting), `locked` (immune to closing).
3) **Preselection (`-p`)**:
    * Manually defining the *split direction* and *ratio* for the *next* window.
    * Visualizing preselection feedback colors.
    * Canceling preselection (`bspc node -p cancel`).
4) **Receptacles (`-i`)**:
    * Inserting empty nodes (leaves without windows) to reserve layout space.
    * Dumping windows into receptacles.


### The Input Daemon

`bspwm` handles zero keyboard input. You must master `sxhkd` to drive it.

* **Chord Chains**: Binding sequences (`super + a ; b`).
* **Brace Expansion**: Creating compact matrices of commands (`super + {h,j,k,l}` mapped to `{west,south,north,east}`).
* **Command Replay**: Using `@` to run commands on key release vs. press.
* **Sync vs. Async**: Understanding when a command blocks `sxhkd` and when it doesn't.
* **Mode Simulation**: Creating "modal" editing (like Vim) using chord chains or dynamic config reloading.


### Logic \& Automation

1) **Standard Rules (`bspc rule`)**:
    * Static assignment: `bspc rule -a Firefox desktop='^2' follow=on`.
    * One-shot rules (`-o`): Rules that apply only to the very next window spawned.
2) **External Rules Command**:
    * Writing shell scripts that intercept `Window ID`, `Class`, and `Instance` before the window is mapped.
    * Implementing complex logic (e.g., "If GIMP opens and I am on Monitor 1, float it; if Monitor 2, tile it").
3) **Event Subscription (`bspc subscribe`)**:
    * Listening to the event stream: `node_add`, `node_remove`, `desktop_focus`, `monitor_add`.
    * Building daemon scripts that react to changes (e.g., "When I switch to Desktop 5, automatically change the wallpaper").
4) **State Dumping (`bspc query -T`)**:
    * Reading the JSON state of the entire window manager.
    * Writing scripts to save/load layouts (parsing JSON to reconstruct trees using receptacles).


### Layout Management

1) **Automatic Schemes**:
    * `spiral`: Windows split the largest node, spiraling inward.
    * `longest_side`: Windows always split the longest edge (standard tiling).
    * `alternate`: Windows alternate H/V splits.
2) **Manual Layouts**: Building custom grids on the fly using preselection.
3) **Padding \& Gaps**:
    * `window_gap`: Space between nodes.
    * `top_padding`, `left_padding`, etc.: Reserving space for bars/panels.
    * Configuring per-monitor or per-desktop padding (e.g., Mono-monitor setup needs different padding than Dual).


### The Ecosystem

1) **Polybar / Lemonbar**:
    * Parsing the `bspwm` internal report (`bspc subscribe report`) to display workspace tags (Occupied, Free, Urgent, Focused).
2) **X11 Tools**:
    * `xprop`: Finding `WM_CLASS` and `WM_NAME` for rules.
    * `xwininfo`: Debugging geometry.
    * `xtitle`: Getting dynamic window titles for scripting.
    * `xdo` / `xdotool`: Sending fake input or managing windows that `bspc` cannot touch.
3) **Compositors (Picom)**:
    * Handling transparency, shadows, and blur.
    * Managing opacity rules for focused vs. unfocused nodes.


### Advanced Hacks \& Workflows

1) **Scratchpads**:
    * Using the `hidden` flag and unique IDs to create "drop-down" terminals that toggle visibility.
    * Managing a "Scratchpad Desktop" (e.g., Desktop 10) vs. Hidden Nodes.
2) **Swallowing**:
    * implementing terminal swallowing (launching an image viewer "eats" the terminal window until the viewer closes) via generic scripts or tools like `devour`.
3) **Dynamic Gaps**:
    * Scripting gaps to disappear when only one window is present (`smart_gaps`).
4) **Urgency Hints**:
    * Handling `WM_HINTS` (flashing red borders) when a background window needs attention.


### Troubleshooting \& Debugging

1) **The Socket**: Understanding `/tmp/bspwm_...socket`.
2) **Logs**: Checking `~/.xsession-errors`.
3) **Visual Debugging**: Using `bspc query -T` piped to `jq` to visualize why a window is stuck in a specific split.

