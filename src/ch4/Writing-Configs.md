
# Comprehensive Guide to BSPWM Configuration

## Executive Summary

This documentation provides an in-depth exploration of **bspwm** (Binary Space Partitioning Window Manager) configuration, derived entirely from authoritative web sources. BSPWM is a sophisticated tiling window manager that represents windows as leaves of a full binary tree, controlled entirely through the **bspc** command-line client. This guide comprehensively covers every flag, command-line option, configuration setting, and advanced technique required to create a highly sophisticated and customized BSPWM environment. The documentation is organized to progress from foundational concepts through complex configuration scenarios, with detailed explanations of each component extracted directly from official documentation and community resources.



## Understanding the Architecture and Fundamental Concepts

### The Binary Space Partitioning Model

BSPWM fundamentally operates differently from other tiling window managers through its binary tree architecture. Unlike traditional grid-based layouts, BSPWM represents windows as the leaves of a full binary tree, where each split divides the available space into exactly two nodes. This architectural choice provides exceptional flexibility for window arrangements and enables sophisticated manipulation of window layouts through tree operations. The window manager only responds to X11 events and messages received on a dedicated socket, which is handled exclusively through the **bspc** client.

### The Client-Server Model


The relationship between BSPWM and its control interface is built on a socket-based client-server model. BSPWM itself runs as a daemon and listens on a socket for messages from **bspc**, the command-line client that sends configuration commands and window management instructions. This separation provides exceptional modularity and allows complete configuration through shell scripts, making BSPWM exceptionally scriptable compared to window managers that require specialized configuration languages. The socket path, by default, follows the pattern `/tmp/bspwm<hostname>_<display>_<screen>-socket`, but can be customized through the `BSPWM_SOCKET` environment variable.



### Configuration File Structure

BSPWM configuration resides in a single shell script located at `$XDG_CONFIG_HOME/bspwm/bspwmrc`, which typically resolves to `~/.config/bspwm/bspwmrc`. This configuration file contains shell commands that invoke **bspc** to set up the window manager environment. The beauty of this approach lies in its simplicity: no special configuration language is required, and users can leverage any shell scripting capabilities to create dynamic, conditional configurations. When BSPWM starts, it executes this shell script, which should contain commands to start necessary daemons (like keyboard managers), set up monitors and desktops, configure display properties, and launch supporting applications.



## BSPWM Command Structure and Domains

### Overview of the BSPC Command

The **bspc** command functions as the exclusive interface to BSPWM, accepting a specific structure that organizes functionality into domains. The general syntax follows: `bspc DOMAIN [SELECTOR] COMMANDS`. This structure allows granular control over window manager behavior, from global settings to desktop-specific configurations to individual window manipulations. Understanding this hierarchical structure is essential for creating sophisticated configurations.

### Available Domains and Their Purposes

BSPWM organizes all functionality into six primary domains:

1. **node**: Controls individual window (node) operations
2. **desktop**: Manages desktop/workspace-level settings
3. **monitor**: Handles monitor-specific configurations
4. **query**: Retrieves metadata and state information
5. **rule**: Defines window matching and application rules
6. **wm**: Controls global window manager states

Additionally, specialized domains handle **config**, **subscribe**, and **quit** operations. The **subscribe** domain enables event-driven scripting, a powerful feature for creating dynamic window management behaviors.



## Node Domain: Individual Window Control

### Node-Specific Commands

The **node** domain manages individual windows (called nodes in BSPWM terminology). When no selector is provided, the command defaults to the focused node. All node operations follow the syntax: `bspc node [NODE_SEL] COMMAND`.

#### Focus Operations: `-f` and `--focus`

The **-f** or **--focus** flag changes focus to a specified node. When used alone, it focuses the selected node; when given a NODE_SEL argument, it focuses that specific node. For example:

- `bspc node -f west` - Focus the window to the west
- `bspc node -f biggest` - Focus the largest window

The focus command respects node selectors and modifiers, enabling complex selection patterns.

#### Activation: `-a` and `--activate`

The **-a** or **--activate** flag differs from focus by setting a node as active without necessarily giving it keyboard focus. This distinction is important in multi-monitor setups where you may want to highlight a window's visual state independently of input focus. The activate command accepts an optional NODE_SEL parameter.

#### Desktop Transfer: `-d` and `--to-desktop`

The **-d** or **--to-desktop** flag sends a node to a specified desktop. The syntax is: `bspc node -d DESKTOP_SEL`. The optional **--follow** flag can be appended to maintain focus on the moved node after transfer. For example:

- `bspc node -d '^2' --follow` - Move focused node to desktop 2 and maintain focus


#### Monitor Transfer: `-m` and `--to-monitor`

The **-m** or **--to-monitor** flag transfers a node to a specified monitor. Syntax: `bspc node -m MONITOR_SEL`. The **--follow** flag similarly maintains focus after the transfer. This is particularly useful in multi-monitor setups for dynamically redistributing windows.

#### Node-to-Node Transfer: `-n` and `--to-node`

The **-n** or **--to-node** flag moves a node to become a sibling of another specified node. This powerful operation enables precise positioning within the binary tree structure. The **--follow** flag again allows focus to follow the transferred node. Example:

- `bspc node -n newest.!automatic.local` - Move focused node to the newest preselected area


#### Swapping Nodes: `-s` and `--swap`

The **-s** or **--swap** flag exchanges positions of two nodes in the tree, effectively swapping their visual positions and tiling spaces. The syntax is: `bspc node -s NODE_SEL`. The **--follow** flag moves focus to the swapped node's new position. This differs from moving: the nodes exchange places rather than one replacing the other.

#### Preselection Direction: `-p` and `--presel-dir`

The **-p** or **--presel-dir** flag enables manual insertion mode by specifying where the next spawned window should appear relative to the current node. Valid directions are: `north`, `south`, `east`, `west`. Additionally, `~DIR` syntax cancels a preselection if it matches the specified direction. For example:

- `bspc node -p east` - Next window spawns to the east of current node
- `bspc node -p ~east` - Cancel east preselection if currently active


#### Preselection Ratio: `-o` and `--presel-ratio`

The **-o** or **--presel-ratio** flag sets the proportion of space allocated to the preselected area. The RATIO parameter must be between 0 and 1. For instance, `bspc node -o 0.4` allocates 40% of space to the preselected area and 60% to the existing content. This works in conjunction with preselection direction.

#### Node Movement: `-v` and `--move`

The **-v** or **--move** flag shifts a node's position by pixel offsets. Syntax: `bspc node -v dx dy`. Both **dx** and **dy** are pixel values representing horizontal and vertical displacement. This command is particularly useful for floating windows, allowing pixel-perfect positioning adjustments.

#### Node Resizing: `-z` and `--resize`

The **-z** or **--resize** flag adjusts node dimensions by specifying an edge and pixel adjustments. Valid edges include: `top`, `left`, `bottom`, `right`, `top_left`, `top_right`, `bottom_right`, `bottom_left`. Syntax: `bspc node -z EDGE dx dy`. The **dx** and **dy** values represent horizontal and vertical pixel adjustments. For example:

- `bspc node -z right 20 0` - Expand right edge by 20 pixels


#### Node Type/Split Cycling: `-y` and `--type`

The **-y** or **--type** flag changes or cycles the split type of a node's parent. Valid arguments are `horizontal`, `vertical`, or `next` to cycle between types. This operation rotates the tree's splitting orientation. Example:

- `bspc node -y next` - Toggle between horizontal and vertical splits


#### Split Ratio Adjustment: `-r` and `--ratio`

The **-r** or **--ratio** flag modifies the split ratio of a node's parent, controlling space distribution between siblings. Syntax: `bspc node -r RATIO` where RATIO is a decimal from 0-1, or `(+|-)(PIXELS|FRACTION)` for relative adjustments. For instance:

- `bspc node -r 0.6` - Set split ratio to 60/40
- `bspc node -r +0.05` - Increase split ratio by 5%


#### Tree Rotation: `-R` and `--rotate`

The **-R** or **--rotate** flag rotates the tree rooted at the selected node. Valid angles are: `90`, `270`, `180`. This operation is useful for dynamically rearranging window hierarchies. Example:

- `bspc node -R 90` - Rotate tree 90 degrees clockwise


#### Tree Flipping: `-F` and `--flip`

The **-F** or **--flip** flag flips the tree rooted at the selected node. Valid directions are: `horizontal`, `vertical`. This mirrors the tree structure along the specified axis. Example:

- `bspc node -F horizontal` - Flip tree horizontally


#### Tree Equalization: `-E` and `--equalize`

The **-E** or **--equalize** flag resets all split ratios within a node's subtree to their default values (typically 0.5). This is useful for restoring a balanced layout after manual ratio adjustments.

#### Tree Balancing: `-B` and `--balance`

The **-B** or **--balance** flag adjusts split ratios within a subtree so all leaves occupy equal area. Unlike equalization, which resets all ratios identically, balancing respects the existing tree structure while equalizing final window areas.

#### Tree Circulation: `-C` and `--circulate`

The **-C** or **--circulate** flag moves windows within a tree in a circular pattern. Valid directions are: `forward`, `backward`. This operation is particularly useful for dynamic window rotation without explicit swapping.

#### Node State: `-t` and `--state`

The **-t** or **--state** flag changes a node's state (tiling mode). Valid states are: `tiled`, `pseudo_tiled`, `floating`, `fullscreen`. The `~` prefix toggles to the previous state if the current state matches, or restores the previous state if no state is specified. Syntax: `bspc node -t STATE`. Examples:

- `bspc node -t floating` - Make window floating
- `bspc node -t ~floating` - Toggle floating state


#### Node Flags: `-g` and `--flag`

The **-g** or **--flag** flag manages binary flags that modify node behavior. Valid flags are: `hidden`, `sticky`, `private`, `locked`, `marked`, `urgent`. The flag syntax accepts optional `=on|off` to explicitly set state. Each flag serves a specific purpose:

- **hidden**: Node is hidden and doesn't occupy tiling space
- **sticky**: Node stays on the focused desktop of its monitor
- **private**: Node resists movement and resizing during automatic insertion
- **locked**: Node ignores the close (`-c`) message
- **marked**: Arbitrary flag used for custom operations; unmarked automatically when sent to a preselected node
- **urgent**: Indicates window urgency (typically set externally); used for window selection

Examples:

- `bspc node -g sticky=on` - Make window sticky
- `bspc node -g marked` - Toggle marked flag


#### Node Layering: `-l` and `--layer`

The **-l** or **--layer** flag changes a node's stacking layer. Valid layers are: `below`, `normal`, `above`. BSPWM maintains three stacking layers where below < normal < above, and within each layer: tiled/pseudo_tiled < floating < fullscreen. Examples:

- `bspc node -l above` - Place window above all others
- `bspc node -l below` - Place window below all others


#### Receptacle Insertion: `-i` and `--insert-receptacle`

The **-i** or `--insert-receptacle` flag creates a receptacle (empty leaf node) at the selected node's position. Receptacles are particularly useful for creating predefined layouts that can be filled with windows later.

#### Node Closure: `-c` and `--close`

The **-c** or **--close** flag closes a node by sending it the close message. This respects the `locked` flag; if set, the message is ignored.

#### Node Termination: `-k` and **--kill**

The **-k** or **--kill** flag forcibly terminates a node's window regardless of flags or settings. This is a hard termination that bypasses all safety mechanisms.

### Node Selector Syntax

Node selectors determine which nodes are affected by commands. They follow the pattern: `[REFERENCE#]DESCRIPTOR[.MODIFIER]*`.

#### Node Descriptors

Descriptors specify the initial node selection:

- **DIR** (north|west|south|east): Relative direction
- **CYCLE_DIR** (next|prev): Cyclic direction
- **any**: Any node
- **first_ancestor**: First non-leaf ancestor
- **last**: Most recently focused
- **newest**: Most recently created
- **older**: Older than focused in history
- **newer**: Newer than focused in history
- **focused**: Currently focused node
- **pointed**: Node under mouse pointer
- **biggest**: Largest node by area
- **smallest**: Smallest node by area
- **<node_id>**: Direct node ID reference


#### Node Modifiers

Modifiers further filter the selection:

- **[!]focused**: Currently/not currently focused
- **[!]active**: Active/not active on desktop
- **[!]automatic**: In automatic/manual insertion mode
- **[!]local**: On/not on current desktop
- **[!]leaf**: Is/isn't a leaf node
- **[!]window**: Has/doesn't have a window
- **[!]STATE**: Matches/doesn't match state (tiled|pseudo_tiled|floating|fullscreen)
- **[!]FLAG**: Has/doesn't have flag (hidden|sticky|private|locked|marked|urgent)
- **[!]LAYER**: On/not on layer (below|normal|above)
- **[!]SPLIT_TYPE**: Split type (horizontal|vertical)
- **[!]same_class**: Same/different class as focused
- **[!]descendant_of**: Is/isn't descendant of reference
- **[!]ancestor_of**: Is/isn't ancestor of reference


#### Path Jumps

Path jumps navigate the tree structure:

- **first|1**: First child
- **second|2**: Second child
- **brother**: Sibling node
- **parent**: Parent node
- **DIR**: Directional jump



## Desktop Domain: Workspace Management

### Desktop-Specific Commands

The **desktop** domain manages workspace-level settings and operations. Syntax: `bspc desktop [DESKTOP_SEL] COMMAND`. If no DESKTOP_SEL is provided, the focused desktop is targeted.

#### Desktop Focus: `-f` and `--focus`

The **-f** or **--focus** flag switches focus to a specified desktop. Example:

- `bspc desktop -f '^2'` - Focus the 2nd desktop
- `bspc desktop -f next` - Focus next desktop


#### Desktop Activation: `-a` and `--activate`

The **-a** or **--activate** flag sets a desktop as active (similar to node activation). This is distinct from focus and is useful in multi-monitor scenarios.

#### Desktop Transfer to Monitor: `-m` and `--to-monitor`

The **-m** or `--to-monitor` flag transfers a desktop to a different monitor. Syntax: `bspc desktop -m MONITOR_SEL`. The optional **--follow** flag moves focus to the transferred desktop.

#### Desktop Swapping: `-s` and `--swap`

The **-s** or **--swap** flag exchanges two desktops. Syntax: `bspc desktop -s DESKTOP_SEL`. The **--follow** flag maintains focus continuity.

#### Layout Selection: `-l` and `--layout`

The **-l** or `--layout` flag sets or cycles the desktop's layout. Valid layouts are: `tiled`, `monocle`, or `next` to cycle. BSPWM provides two built-in layouts:

- **tiled**: Standard binary space partitioning with visible splits
- **monocle**: Fullscreen layout where only the most recently focused tiled/pseudo_tiled window is visible

Examples:

- `bspc desktop -l monocle` - Switch to monocle layout
- `bspc desktop -l next` - Cycle to next layout


#### Desktop Renaming: `-n` and `--rename`

The **-n** or `--rename` flag changes a desktop's name. Syntax: `bspc desktop -n <new_name>`. Desktop names are used in keybindings and configurations.

#### Desktop Cycling: `-b` and `--bubble`

The **-b** or `--bubble` flag moves a desktop within the monitor's desktop list. Direction values are `next` or `prev`. This reorders desktops without changing their content.

#### Desktop Removal: `-r` and `--remove**

The **-r** or `--remove` flag deletes a desktop. Windows on the removed desktop are transferred to the next available desktop.

### Desktop Selector Syntax

Desktop selectors determine which desktops are affected.

#### Desktop Descriptors

- **CYCLE_DIR** (next|prev): Cyclic direction
- **any**: Any desktop
- **last**: Most recently focused
- **newest**: Most recently created
- **older**: Older in history
- **newer**: Newer in history
- **focused**: Currently focused
- **^< n >**: The nth desktop
- **MONITOR_SEL:focused**: Focused desktop on specified monitor
- **<desktop_id>**: Direct ID reference
- **<desktop_name>**: By name


#### Desktop Modifiers

- **[!]focused**: Currently/not focused
- **[!]active**: Active/not active on desktop
- **[!]occupied**: Has/doesn't have windows
- **[!]urgent**: Contains/doesn't contain urgent windows
- **[!]local**: On/not on current monitor
- **[!]LAYOUT**: Matches/doesn't match layout
- **[!]user_LAYOUT**: Custom layout matching



## Monitor Domain: Multi-Monitor Configuration

### Monitor-Specific Commands

The **monitor** domain controls monitor-specific settings and operations. Syntax: `bspc monitor [MONITOR_SEL] COMMAND`.

#### Monitor Focus: `-f` and `--focus`

The **-f** or `--focus` flag switches focus to a specified monitor. Example:

- `bspc monitor -f DP-1` - Focus monitor named DP-1
- `bspc monitor -f next` - Focus next monitor


#### Monitor Swapping: `-s` and `--swap`

The **-s** or `--swap` flag exchanges two monitors. This is useful for reversing display order.

#### Add Desktops: `-a` and `--add-desktops`

The **-a** or `--add-desktops` flag creates new desktops on a monitor. Syntax: `bspc monitor -a <name>...`. Multiple desktop names can be provided. Example:

- `bspc monitor -a I II III IV V` - Create five desktops named I through V


#### Reorder Desktops: `-o` and `--reorder-desktops`

The **-o** or `--reorder-desktops` flag changes the order of desktops on a monitor. Syntax: `bspc monitor -o <name>...`. The order must match existing desktop names.

#### Reset Desktops: `-d` and `--reset-desktops`

The **-d** or `--reset-desktops` flag reconfigures desktops, adding, removing, or renaming as needed. Syntax: `bspc monitor -d <name>...`. This command automatically handles all necessary adjustments.

#### Monitor Rectangle: `-g` and `--rectangle`

The **-g** or `--rectangle` flag manually sets monitor geometry. Syntax: `bspc monitor -g WxH+X+Y`. This is useful for multi-display configurations or unusual setups.

#### Monitor Renaming: `-n` and `--rename`

The **-n** or `--rename` flag changes a monitor's name. Syntax: `bspc monitor -n <new_name>`.

#### Monitor Removal: `-r` and `--remove`

The **-r** or `--remove` flag removes a monitor from management. Desktops on the removed monitor are transferred to remaining monitors.

### Monitor Selector Syntax

Monitor selectors determine which monitors are affected.

#### Monitor Descriptors

- **DIR** (north|west|south|east): Spatial direction
- **CYCLE_DIR** (next|prev): Cyclic direction
- **any**: Any monitor
- **last**: Most recently focused
- **newest**: Most recently added
- **older**: Older in history
- **newer**: Newer in history
- **focused**: Currently focused
- **pointed**: Monitor under pointer
- **primary**: Primary monitor
- **^< n >**: The nth monitor
- **<monitor_id>**: By monitor ID
- **<monitor_name>**: By name (e.g., HDMI-1)


#### Monitor Modifiers

- **[!]focused**: Currently/not focused
- **[!]occupied**: Has/doesn't have occupied desktops



## Query Domain: State Information Retrieval

### Query Commands

The **query** domain retrieves metadata and state information. Syntax: `bspc query COMMANDS [OPTIONS]`.

#### Node Query: `-N` and `--nodes`

The **-N** or `--nodes` flag returns node IDs. Syntax: `bspc query -N [NODE_SEL]`. Without selectors, returns all node IDs. With selectors, returns matching nodes. Example:

- `bspc query -N -n focused` - Get ID of focused node
- `bspc query -N -n .window` - Get IDs of all window nodes


#### Desktop Query: `-D` and `--desktops`

The **-D** or `--desktops` flag returns desktop IDs. Syntax: `bspc query -D [DESKTOP_SEL]`. Returns all desktop IDs when no selector is given.

#### Monitor Query: `-M` and `--monitors`

The **-M** or `--monitors` flag returns monitor IDs. Syntax: `bspc query -M [MONITOR_SEL]`. Returns all monitor IDs without selector.

#### Tree Query: `-T` and `--tree`

The **-T** or `--tree` flag returns the complete tree structure in JSON or text format. This comprehensive output shows all relationships and states. Example:

- `bspc query -T` - Output complete tree state
- `bspc query -T -m DP-1` - Tree for specific monitor


### Query Options

Query results can be filtered and formatted.

#### Monitor Specification: `-m` and `--monitor`

The **-m** or `--monitor` option filters results to a specific monitor. Syntax: `bspc query -m MONITOR_SEL`.

#### Desktop Specification: `-d` and `--desktop`

The **-d** or `--desktop` option filters to a specific desktop. Syntax: `bspc query -d DESKTOP_SEL`.

#### Node Specification: `-n` and `--node`

The **-n** or `--node` option filters to a specific node. Syntax: `bspc query -n NODE_SEL`.

#### Names Output: `--names`

The `--names` flag outputs entity names instead of IDs. This is useful for human-readable output.



## Rule Domain: Window Matching and Defaults

### Rule Management Commands

The **rule** domain defines patterns and default configurations for new windows.

#### Adding Rules: `-a` and `--add`

The **-a** or `--add` flag creates a new rule. Syntax: `bspc rule -a [CLASS[:INSTANCE[:NAME]]] [options]`. The class, instance, and name can be wildcards using `*`. Fields can be escaped with backslash.

Rule properties include:

- **monitor=MONITOR_SEL**: Target monitor
- **desktop=DESKTOP_SEL**: Target desktop
- **node=NODE_SEL**: Target node (for insertion)
- **state=STATE**: Initial state (tiled|pseudo_tiled|floating|fullscreen)
- **layer=LAYER**: Initial layer (below|normal|above)
- **honor_size_hints=(true|false|tiled|floating)**: Whether to respect ICCCM size hints
- **split_dir=DIR**: Preselection direction
- **split_ratio=RATIO**: Preselection ratio
- **hidden=(on|off)**: Initially hidden state
- **sticky=(on|off)**: Sticky flag state
- **private=(on|off)**: Private flag state
- **locked=(on|off)**: Locked flag state
- **marked=(on|off)**: Marked flag state
- **center=(on|off)**: Center floating windows
- **follow=(on|off)**: Focus follows window
- **manage=(on|off)**: Whether window is managed
- **focus=(on|off)**: Initial focus state
- **border=(on|off)**: Show window borders
- **rectangle=WxH+X+Y**: Initial geometry for floating windows

The **-o** or `--one-shot` flag makes the rule apply only once.

Examples:

- `bspc rule -a Firefox desktop=^2 follow=on`
- `bspc rule -a Gimp state=floating follow=off`


#### Removing Rules: `-r` and `--remove`

The **-r** or `--remove` flag deletes rules. Syntax: `bspc rule -r PATTERN`. Patterns can be `head`, `tail`, `^< n >` for index, or a class/instance/name pattern.

#### Listing Rules: `-l` and `--list`

The **-l** or `--list` flag displays all currently active rules.

### External Rules Script

For complex matching logic beyond the built-in rule syntax, BSPWM supports external rule scripts. The **external_rules_command** setting specifies a script path. This script receives window information and outputs rule properties.

Script invocation: `SCRIPT wid class instance name`

Example external rules script:

```bash
#!/bin/sh
case "$2" in
    Firefox)
        echo "desktop=^2 follow=on"
        ;;
    Gimp)
        echo "state=floating"
        ;;
esac
```




## Config Domain: Global Configuration

### Configuration Settings

The **config** domain manages global, monitor-specific, desktop-specific, and node-specific settings.

#### General Syntax

`bspc config [-m MONITOR_SEL|-d DESKTOP_SEL|-n NODE_SEL] <setting> [<value>]`

### Global Settings

#### Color Settings

Colors are specified in hexadecimal format: `#RRGGBB`

- **normal_border_color**: Border color for unfocused windows
- **active_border_color**: Border color for active (desktop-focused) windows
- **focused_border_color**: Border color for focused windows
- **presel_feedback_color**: Color of preselection indicator


#### Layout and Splitting

- **split_ratio**: Default ratio for binary splits (0 < ratio < 1)
- **automatic_scheme**: Algorithm for automatic window placement. Valid values:
    - **longest_side**: Split along longest edge
    - **alternate**: Alternate between horizontal and vertical
    - **spiral**: Create spiral patterns
- **initial_polarity**: Which child receives the new window in automatic mode. Valid values:
    - **first_child**: New window becomes first child
    - **second_child**: New window becomes second child
- **directional_focus_tightness**: Strictness of directional focus algorithm. Valid values:
    - **high**: Stricter matching
    - **low**: Looser matching


#### Insertion and Removal

- **removal_adjustment**: Whether to readjust layout after window removal
- **presel_feedback**: Whether to show preselection feedback


#### Monocle Layout

- **borderless_monocle**: Remove borders in monocle layout
- **gapless_monocle**: Remove gaps in monocle layout
- **top_monocle_padding**, **right_monocle_padding**, **bottom_monocle_padding**, **left_monocle_padding**: Padding for monocle layout
- **single_monocle**: Switch to monocle if only one window remains


#### Status and Behavior

- **borderless_singleton**: Remove borders when only one window exists
- **status_prefix**: Prefix for status messages
- **pointer_motion_interval**: Minimum milliseconds between pointer motion updates
- **pointer_modifier**: Keyboard modifier for pointer actions. Valid values:
    - **shift**, **control**, **lock**
    - **mod1** (Alt), **mod2**, **mod3**
    - **mod4** (Super), **mod5**
- **pointer_action1**, **pointer_action2**, **pointer_action3**: Mouse button actions. Valid values:
    - **move**: Move floating windows
    - **resize_side**: Resize from edge
    - **resize_corner**: Resize from corner
    - **focus**: Focus on click
    - **none**: No action
- **click_to_focus**: Mouse button for focus-on-click. Valid values: **button1**, **button2**, **button3**, **any**, **none**
- **swallow_first_click**: Consume first click when focusing
- **focus_follows_pointer**: Focus window under pointer
- **pointer_follows_focus**: Move pointer to focused window
- **pointer_follows_monitor**: Move pointer to focused monitor


#### EWMH Compatibility

- **mapping_events_count**: How many mapping events to process
- **ignore_ewmh_focus**: Ignore EWMH focus requests
- **ignore_ewmh_fullscreen**: Ignore fullscreen requests. Valid values: **none**, **all**, or comma-separated **enter**, **exit**
- **ignore_ewmh_struts**: Ignore taskbar/panel space reservations
- **center_pseudo_tiled**: Center pseudo_tiled windows


#### Monitor Management

- **remove_disabled_monitors**: Remove monitors that are disabled
- **remove_unplugged_monitors**: Remove unplugged monitors
- **merge_overlapping_monitors**: Merge monitors with overlapping geometry


### Monitor and Desktop Settings

#### Padding

Applied at monitor and desktop levels:

- **top_padding**, **right_padding**, **bottom_padding**, **left_padding**: Space reserved around the desktop edges. Commonly set to bar heights


### Desktop Settings

#### Window Gap

- **window_gap**: Pixel spacing between windows. Can be set negative to create gapless layouts


### Node Settings

#### Borders and Hints

- **border_width**: Border thickness in pixels
- **honor_size_hints**: Respect ICCCM window size hints. Valid values:
    - **true**: Apply to all windows
    - **false**: Don't apply
    - **tiled**: Apply only to tiled windows
    - **floating**: Apply only to floating windows



## Subscribe Domain: Event-Driven Scripting

### Event Subscription

The **subscribe** domain enables reactive scripting based on window manager events. Syntax: `bspc subscribe [OPTIONS] (all|report|monitor|desktop|node|...)*`.

### Subscription Options

#### FIFO Output: `-f` and `--fifo`

The **-f** or `--fifo` flag outputs events to a named FIFO instead of stdout. This enables long-lived subscriptions.

#### Event Count: `-c` and `--count`

The **-c** or `--count` flag exits after receiving COUNT events.

### Available Events

#### Monitor Events

- **monitor_add**: New monitor connected
- **monitor_rename**: Monitor renamed
- **monitor_remove**: Monitor disconnected
- **monitor_swap**: Monitors swapped positions
- **monitor_focus**: Focus changed to monitor
- **monitor_geometry**: Monitor geometry changed


#### Desktop Events

- **desktop_add**: Desktop created
- **desktop_rename**: Desktop renamed
- **desktop_remove**: Desktop deleted
- **desktop_swap**: Desktops swapped
- **desktop_transfer**: Desktop transferred to different monitor
- **desktop_focus**: Desktop focus changed
- **desktop_activate**: Desktop activated
- **desktop_layout**: Layout changed


#### Node Events

- **node_add**: Window added
- **node_remove**: Window removed
- **node_swap**: Windows swapped
- **node_transfer**: Window transferred
- **node_focus**: Window focus changed
- **node_activate**: Window activated
- **node_presel**: Preselection changed
- **node_stack**: Stack order changed
- **node_geometry**: Window geometry changed
- **node_state**: Window state changed
- **node_flag**: Window flag changed
- **node_layer**: Stacking layer changed


#### Report Event

The **report** event outputs the complete current state in a specific format.

### Event Processing Example

Using events to perform actions:

```bash
bspc subscribe node_add | while read -a msg; do
    desk_id=${msg[^1_2]}
    wid=${msg[^1_4]}
    # Make new windows fullscreen
    bspc node "$wid" -t fullscreen
done
```




## Wm Domain: Window Manager State

### Global Window Manager Operations

The **wm** domain controls global window manager state and operations.

#### Dump State: `-d` and `--dump-state`

The **-d** or `--dump-state` flag outputs the complete window manager state. This is useful for backing up configurations or analysis.

#### Load State: `-l` and `--load-state`

The **-l** or `--load-state** flag restores a previously dumped state. Syntax: `bspc wm -l <file_path>`.

#### Add Monitor: `-a` and `--add-monitor`

The **-a** or `--add-monitor` flag manually adds a monitor to management. Syntax: `bspc wm -a <name> WxH+X+Y`. Useful for dynamic monitor addition.

#### Reorder Monitors: `-O` and `--reorder-monitors`

The **-O** or `--reorder-monitors` flag changes the global monitor order. Syntax: `bspc wm -O <name>...`.

#### Adopt Orphans: `-o` and `--adopt-orphans`

The **-o** or `--adopt-orphans` flag brings unmanaged windows under BSPWM control.

#### Record History: `-h` and `--record-history`

The **-h** or `--record-history** flag enables/disables command history logging. Syntax: `bspc wm -h on|off`.

#### Get Status: `-g` and `--get-status`

The **-g** or `--get-status` flag outputs the current status.

#### Restart: `-r` and `--restart`

The **-r** or `--restart** flag restarts BSPWM while preserving windows. This is useful during configuration updates.



## Window States and Flags

### Window States

Each window maintains exactly one state at any time:

#### Tiled

**Tiled** state means the window fills its assigned tiling space without overlapping others. Windows are arranged according to the binary tree structure. This is the default state for new windows.

#### Pseudo-Tiled

**Pseudo-tiled** windows respect ICCCM size hints while being centered within their tiling space. They can be resized but maintain their centered position.

#### Floating

**Floating** windows can be positioned and resized freely anywhere on the desktop. They don't participate in automatic tiling but remain part of the node tree.

#### Fullscreen

**Fullscreen** windows occupy their monitor's entire rectangle with no borders. They're placed above other content. When a fullscreen window is present and a new floating window is created, BSPWM must change the fullscreen window to tiled to display the floating window on top, unless the floating window is placed on the above layer.

### Window Flags

Flags are independent states that can be combined:

#### Hidden

**Hidden** windows don't occupy tiling space and aren't visible. They're useful for implementing scratchpads and minimization-like functionality.

#### Sticky

**Sticky** windows follow the focused desktop on their monitor. When switching desktops, sticky windows appear on the new desktop.

#### Private

**Private** windows resist movement and resizing during automatic insertion. When inserting new windows into automatic mode, private nodes maintain their position and size rather than being split.

#### Locked

**Locked** windows ignore the close message sent by `bspc node -c`. They require forceful termination with `bspc node -k`.

#### Marked

**Marked** is an arbitrary flag useful for custom operations. It's particularly valuable in conjunction with preselection to implement deferred window movement. Marked nodes automatically become unmarked when sent to a preselected node.

#### Urgent

**Urgent** indicates a window requiring attention. It's typically set externally by applications (e.g., when incoming messages arrive). BSPWM uses this flag for window selection.



## Stacking Layers

BSPWM implements three independent stacking layers:

1. **below**: Lowest layer
2. **normal**: Middle layer (default)
3. **above**: Top layer

Within each layer, the window order follows: tiled \& pseudo_tiled < floating < fullscreen. This means a floating window on the "below" layer appears above tiled windows on that layer but below all windows on the "normal" layer.



## Advanced Configuration Patterns and Techniques

### Monitor Setup and Multi-Monitor Configuration

Setting up monitors requires careful use of the monitor domain commands:

```bash
#!/bin/bash
# Example multi-monitor setup

# External monitor setup
bspc monitor eDP1 -d I II III IV V
bspc monitor HDMI1 -d VI VII VIII IX X

# Alternative: Conditional setup
if xrandr | grep -q "HDMI1 connected"; then
    xrandr --output HDMI1 --right-of eDP1 --auto
    bspc monitor eDP1 -d I II III IV
    bspc monitor HDMI1 -d V VI VII VIII
fi
```


### Floating Desktop Configuration

Creating desktops where all windows float by default:

```bash
bspc rule -a "*" -o desktop=floating_desktop state=floating
bspc desktop floating_desktop -l tiled  # or any layout
```


### Scratchpad/Dropdown Terminal Implementation

Using hidden sticky windows to create dropdown terminal functionality:

```bash
# Create dropdown terminal rule
bspc rule -a dropdown -o sticky=on state=floating hidden=on rectangle=800x600+560+240

# Launch dropdown terminal
alacritty --class dropdown -e zsh &

# Toggle script
bspc node any.hidden.sticky -g hidden -f
```


### Receptacle-Based Manual Layouts

Using receptacles to build predefined layouts:

```bash
# Create three-pane layout with receptacles
bspc node -i
bspc node -p west -o 0.5
bspc node -i
bspc node -p south
bspc node -i

# Fill receptacles with windows - they automatically slot into place
```


### External Rules for Complex Logic

When built-in rules aren't sufficient, external rule scripts provide unlimited flexibility:

```bash
#!/bin/bash
# /home/user/.config/bspwm/external_rules

wid=$1
class=$2
instance=$3

case "$class" in
    Firefox)
        if [ "$instance" = "firefox" ]; then
            echo "desktop=^1 follow=on"
        else
            echo "desktop=^2"  # Private browsing to different desktop
        fi
        ;;
    Blender)
        echo "state=floating rectangle=1920x1080+0+0"
        ;;
    *)
        # Default behavior
        ;;
esac
```


### Event-Driven Automatic Layouts

Using subscribe to dynamically adapt behavior:

```bash
# Automatically switch to monocle in fullscreen windows
bspc subscribe node_state | while read -a msg; do
    if [ "${msg[^1_8]}" = "fullscreen" ] && [ "${msg[^1_9]}" = "on" ]; then
        desk_id=$(echo "${msg[^1_1]}" | cut -d':' -f2)
        bspc desktop "$desk_id" -l monocle
    fi
done &
```


### Node ID Tracking and Manipulation

Storing and using node IDs for complex operations:

```bash
# Store node ID and move it later
id=$(bspc query -N -n)
# ... perform other operations ...
bspc node "$id" -d '^2'  # Move stored node to desktop 2
```


### Custom Layouts with Master Stack

While BSPWM provides only tiled and monocle, external scripts enable layouts like master-stack:

```bash
# Balance 3-pane layout
bspc node @/ -B
bspc node @/1 -r 0.66  # Make left side 66% of space
```


### Pointer Action Configuration

Setting up mouse controls for floating window manipulation:

```bash
# Configure pointer modifier and actions
bspc config pointer_modifier mod1          # Use Alt key
bspc config pointer_action1 move           # Alt+Button1 moves
bspc config pointer_action2 resize_side    # Alt+Button2 resizes edge
bspc config pointer_action3 resize_corner  # Alt+Button3 resizes corner
```


### Query-Based Window Selection

Complex window selection using query syntax:

```bash
# Find biggest window on current desktop and close it
biggest=$(bspc query -N -n biggest.local.!fullscreen.window)
bspc node "$biggest" -c

# Focus all windows of same class
class=$(bspc query -N -n | xargs -I {} xprop -id {} WM_CLASS | cut -d'"' -f2 | head -1)
bspc node ".same_class" -f
```


### Tree Manipulation Example

Using rotate, flip, and balance for dynamic layouts:

```bash
# Rotate current desktop tree 90 degrees
bspc node @/ -R 90

# Flip tree horizontally
bspc node @/ -F horizontal

# Balance all split ratios for equal window sizes
bspc node @/ -B

# Equal spacing (reset all ratios)
bspc node @/ -E
```


### Comprehensive Configuration Template

A sophisticated bspwmrc demonstrating multiple techniques:

```bash
#!/bin/bash
# ~/.config/bspwm/bspwmrc

# Start sxhkd for keybindings
sxhkd &

# Monitor setup
if xrandr | grep -q "HDMI1 connected"; then
    xrandr --output HDMI1 --right-of eDP1 --auto
    bspc monitor eDP1 -d 1 2 3 4 5
    bspc monitor HDMI1 -d 6 7 8 9 10
else
    bspc monitor -d 1 2 3 4 5 6 7 8 9 10
fi

# Global settings
bspc config window_gap 12
bspc config border_width 2
bspc config top_padding 0
bspc config bottom_padding 0
bspc config left_padding 0
bspc config right_padding 0

# Colors
bspc config normal_border_color "#3c3836"
bspc config focused_border_color "#b8bb26"
bspc config active_border_color "#a89984"

# Layout and splitting
bspc config split_ratio 0.5
bspc config automatic_scheme longest_side
bspc config initial_polarity second_child

# Pointer
bspc config pointer_modifier mod1
bspc config pointer_action1 move
bspc config pointer_action2 resize_side
bspc config pointer_action3 resize_corner

# EWMH
bspc config ignore_ewmh_focus true
bspc config ignore_ewmh_struts true

# Focus behavior
bspc config focus_follows_pointer false

# Rules
bspc rule -r '*'  # Clear existing rules

bspc rule -a Firefox desktop='^1'
bspc rule -a Thunderbird desktop='^2'
bspc rule -a Slack desktop='^9'
bspc rule -a feh state=floating
bspc rule -a Gimp state=floating follow=on
bspc rule -a St state=floating rectangle=800x600+560+240

# External rules script
bspc config external_rules_command ~/.config/bspwm/external_rules

# Start status bar
polybar main &

# Launch background apps
nm-applet &
```




## So Far

* BSPWM represents a paradigm shift in window manager configuration philosophy. Rather than implementing every feature directly within the window manager, BSPWM provides a powerful, scriptable interface through **bspc** that enables users to build sophisticated configurations entirely from shell scripts. This approach offers unprecedented flexibility and transparency—users can see exactly what commands are being executed and modify them without learning specialized syntax.

* The comprehensive command set across six domains (node, desktop, monitor, query, rule, wm) combined with powerful selector syntax enables precise control over every aspect of window management. Advanced features like event subscription, external rule scripts, and tree manipulation operations provide the foundation for highly customized window management workflows.

* Mastering BSPWM configuration requires understanding the hierarchical structure of selectors, the binary tree model of window arrangement, and how to combine simple commands into complex behaviors. The sophisticated user can leverage all documented flags and options to create configurations that adapt dynamically to their workflow, automating complex window management scenarios through scripting and event handling.

* This documentation provides the authoritative reference for every flag, option, and configuration technique available in BSPWM, extracted from official sources and community expertise, enabling users to build the most sophisticated window management configurations possible.




