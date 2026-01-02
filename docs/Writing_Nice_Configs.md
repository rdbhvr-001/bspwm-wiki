
# Comprehensive Guide to BSPWM Configuration

## Executive Summary

This documentation provides an in-depth exploration of **bspwm** (Binary Space Partitioning Window Manager) configuration, derived entirely from authoritative web sources. BSPWM is a sophisticated tiling window manager that represents windows as leaves of a full binary tree, controlled entirely through the **bspc** command-line client[^1_1][^1_2]. This guide comprehensively covers every flag, command-line option, configuration setting, and advanced technique required to create a highly sophisticated and customized BSPWM environment. The documentation is organized to progress from foundational concepts through complex configuration scenarios, with detailed explanations of each component extracted directly from official documentation and community resources[^1_1][^1_3][^1_2].

***

## Understanding the Architecture and Fundamental Concepts

### The Binary Space Partitioning Model

BSPWM fundamentally operates differently from other tiling window managers through its binary tree architecture[^1_1][^1_4]. Unlike traditional grid-based layouts, BSPWM represents windows as the leaves of a full binary tree, where each split divides the available space into exactly two nodes[^1_1][^1_4]. This architectural choice provides exceptional flexibility for window arrangements and enables sophisticated manipulation of window layouts through tree operations[^1_1]. The window manager only responds to X11 events and messages received on a dedicated socket, which is handled exclusively through the **bspc** client[^1_1][^1_4].

### The Client-Server Model

```
The relationship between BSPWM and its control interface is built on a socket-based client-server model[^1_1][^1_4]. BSPWM itself runs as a daemon and listens on a socket for messages from **bspc**, the command-line client that sends configuration commands and window management instructions[^1_1][^1_5]. This separation provides exceptional modularity and allows complete configuration through shell scripts, making BSPWM exceptionally scriptable compared to window managers that require specialized configuration languages[^1_5][^1_6]. The socket path, by default, follows the pattern `/tmp/bspwm<hostname>_<display>_<screen>-socket`, but can be customized through the `BSPWM_SOCKET` environment variable[^1_2][^1_7].
```


### Configuration File Structure

BSPWM configuration resides in a single shell script located at `$XDG_CONFIG_HOME/bspwm/bspwmrc`, which typically resolves to `~/.config/bspwm/bspwmrc`[^1_1][^1_8][^1_3]. This configuration file contains shell commands that invoke **bspc** to set up the window manager environment[^1_1][^1_8]. The beauty of this approach lies in its simplicity: no special configuration language is required, and users can leverage any shell scripting capabilities to create dynamic, conditional configurations[^1_5][^1_6]. When BSPWM starts, it executes this shell script, which should contain commands to start necessary daemons (like keyboard managers), set up monitors and desktops, configure display properties, and launch supporting applications[^1_1][^1_8].

***

## BSPWM Command Structure and Domains

### Overview of the BSPC Command

The **bspc** command functions as the exclusive interface to BSPWM, accepting a specific structure that organizes functionality into domains[^1_2]. The general syntax follows: `bspc DOMAIN [SELECTOR] COMMANDS`[^1_2]. This structure allows granular control over window manager behavior, from global settings to desktop-specific configurations to individual window manipulations[^1_2]. Understanding this hierarchical structure is essential for creating sophisticated configurations.

### Available Domains and Their Purposes

BSPWM organizes all functionality into six primary domains[^1_2]:

1. **node**: Controls individual window (node) operations[^1_2]
2. **desktop**: Manages desktop/workspace-level settings[^1_2]
3. **monitor**: Handles monitor-specific configurations[^1_2]
4. **query**: Retrieves metadata and state information[^1_2]
5. **rule**: Defines window matching and application rules[^1_2]
6. **wm**: Controls global window manager state[^1_2]

Additionally, specialized domains handle **config**, **subscribe**, and **quit** operations[^1_2]. The **subscribe** domain enables event-driven scripting, a powerful feature for creating dynamic window management behaviors[^1_2][^1_9].

***

## Node Domain: Individual Window Control

### Node-Specific Commands

The **node** domain manages individual windows (called nodes in BSPWM terminology)[^1_2]. When no selector is provided, the command defaults to the focused node[^1_2]. All node operations follow the syntax: `bspc node [NODE_SEL] COMMAND`[^1_2].

#### Focus Operations: `-f` and `--focus`

The **-f** or **--focus** flag changes focus to a specified node[^1_2]. When used alone, it focuses the selected node; when given a NODE_SEL argument, it focuses that specific node[^1_2]. For example:

- `bspc node -f west` - Focus the window to the west
- `bspc node -f biggest` - Focus the largest window[^1_10]

The focus command respects node selectors and modifiers, enabling complex selection patterns[^1_2].

#### Activation: `-a` and `--activate`

The **-a** or **--activate** flag differs from focus by setting a node as active without necessarily giving it keyboard focus[^1_2]. This distinction is important in multi-monitor setups where you may want to highlight a window's visual state independently of input focus[^1_2]. The activate command accepts an optional NODE_SEL parameter[^1_2].

#### Desktop Transfer: `-d` and `--to-desktop`

The **-d** or **--to-desktop** flag sends a node to a specified desktop[^1_2]. The syntax is: `bspc node -d DESKTOP_SEL`[^1_2]. The optional **--follow** flag can be appended to maintain focus on the moved node after transfer[^1_2]. For example:

- `bspc node -d '^2' --follow` - Move focused node to desktop 2 and maintain focus[^1_2][^1_11]


#### Monitor Transfer: `-m` and `--to-monitor`

The **-m** or **--to-monitor** flag transfers a node to a specified monitor[^1_2]. Syntax: `bspc node -m MONITOR_SEL`[^1_2]. The **--follow** flag similarly maintains focus after the transfer[^1_2]. This is particularly useful in multi-monitor setups for dynamically redistributing windows[^1_2].

#### Node-to-Node Transfer: `-n` and `--to-node`

The **-n** or **--to-node** flag moves a node to become a sibling of another specified node[^1_2]. This powerful operation enables precise positioning within the binary tree structure[^1_2]. The **--follow** flag again allows focus to follow the transferred node[^1_2]. Example:

- `bspc node -n newest.!automatic.local` - Move focused node to the newest preselected area[^1_12][^1_13]


#### Swapping Nodes: `-s` and `--swap`

The **-s** or **--swap** flag exchanges positions of two nodes in the tree, effectively swapping their visual positions and tiling spaces[^1_2][^1_10]. The syntax is: `bspc node -s NODE_SEL`[^1_2]. The **--follow** flag moves focus to the swapped node's new position[^1_2]. This differs from moving: the nodes exchange places rather than one replacing the other[^1_2][^1_10].

#### Preselection Direction: `-p` and `--presel-dir`

The **-p** or **--presel-dir** flag enables manual insertion mode by specifying where the next spawned window should appear relative to the current node[^1_2][^1_14]. Valid directions are: `north`, `south`, `east`, `west`[^1_2]. Additionally, `~DIR` syntax cancels a preselection if it matches the specified direction[^1_2]. For example:

- `bspc node -p east` - Next window spawns to the east of current node
- `bspc node -p ~east` - Cancel east preselection if currently active[^1_2]


#### Preselection Ratio: `-o` and `--presel-ratio`

The **-o** or **--presel-ratio** flag sets the proportion of space allocated to the preselected area[^1_2]. The RATIO parameter must be between 0 and 1[^1_2]. For instance, `bspc node -o 0.4` allocates 40% of space to the preselected area and 60% to the existing content[^1_2]. This works in conjunction with preselection direction[^1_2][^1_15].

#### Node Movement: `-v` and `--move`

The **-v** or **--move** flag shifts a node's position by pixel offsets[^1_2]. Syntax: `bspc node -v dx dy`[^1_2]. Both **dx** and **dy** are pixel values representing horizontal and vertical displacement[^1_2]. This command is particularly useful for floating windows, allowing pixel-perfect positioning adjustments[^1_2].

#### Node Resizing: `-z` and `--resize`

The **-z** or **--resize** flag adjusts node dimensions by specifying an edge and pixel adjustments[^1_2]. Valid edges include: `top`, `left`, `bottom`, `right`, `top_left`, `top_right`, `bottom_right`, `bottom_left`[^1_2]. Syntax: `bspc node -z EDGE dx dy`[^1_2]. The **dx** and **dy** values represent horizontal and vertical pixel adjustments[^1_2]. For example:

- `bspc node -z right 20 0` - Expand right edge by 20 pixels[^1_2]


#### Node Type/Split Cycling: `-y` and `--type`

The **-y** or **--type** flag changes or cycles the split type of a node's parent[^1_2]. Valid arguments are `horizontal`, `vertical`, or `next` to cycle between types[^1_2]. This operation rotates the tree's splitting orientation[^1_2]. Example:

- `bspc node -y next` - Toggle between horizontal and vertical splits[^1_2]


#### Split Ratio Adjustment: `-r` and `--ratio`

The **-r** or **--ratio** flag modifies the split ratio of a node's parent, controlling space distribution between siblings[^1_2]. Syntax: `bspc node -r RATIO` where RATIO is a decimal from 0-1, or `(+|-)(PIXELS|FRACTION)` for relative adjustments[^1_2]. For instance:

- `bspc node -r 0.6` - Set split ratio to 60/40
- `bspc node -r +0.05` - Increase split ratio by 5%[^1_2]


#### Tree Rotation: `-R` and `--rotate`

The **-R** or **--rotate** flag rotates the tree rooted at the selected node[^1_2]. Valid angles are: `90`, `270`, `180`[^1_2]. This operation is useful for dynamically rearranging window hierarchies[^1_2]. Example:

- `bspc node -R 90` - Rotate tree 90 degrees clockwise[^1_2]


#### Tree Flipping: `-F` and `--flip`

The **-F** or **--flip** flag flips the tree rooted at the selected node[^1_2]. Valid directions are: `horizontal`, `vertical`[^1_2]. This mirrors the tree structure along the specified axis[^1_2]. Example:

- `bspc node -F horizontal` - Flip tree horizontally[^1_2]


#### Tree Equalization: `-E` and `--equalize`

The **-E** or **--equalize** flag resets all split ratios within a node's subtree to their default values (typically 0.5)[^1_2]. This is useful for restoring a balanced layout after manual ratio adjustments[^1_2].

#### Tree Balancing: `-B` and `--balance`

The **-B** or **--balance** flag adjusts split ratios within a subtree so all leaves occupy equal area[^1_2]. Unlike equalization, which resets all ratios identically, balancing respects the existing tree structure while equalizing final window areas[^1_2].

#### Tree Circulation: `-C` and `--circulate`

The **-C** or **--circulate** flag moves windows within a tree in a circular pattern[^1_2]. Valid directions are: `forward`, `backward`[^1_2]. This operation is particularly useful for dynamic window rotation without explicit swapping[^1_2][^1_16].

#### Node State: `-t` and `--state`

The **-t** or **--state** flag changes a node's state (tiling mode)[^1_2]. Valid states are: `tiled`, `pseudo_tiled`, `floating`, `fullscreen`[^1_2]. The `~` prefix toggles to the previous state if the current state matches, or restores the previous state if no state is specified[^1_2]. Syntax: `bspc node -t STATE`[^1_2]. Examples:

- `bspc node -t floating` - Make window floating
- `bspc node -t ~floating` - Toggle floating state[^1_2]


#### Node Flags: `-g` and `--flag`

The **-g** or **--flag** flag manages binary flags that modify node behavior[^1_2]. Valid flags are: `hidden`, `sticky`, `private`, `locked`, `marked`, `urgent`[^1_2]. The flag syntax accepts optional `=on|off` to explicitly set state[^1_2]. Each flag serves a specific purpose:

- **hidden**: Node is hidden and doesn't occupy tiling space[^1_1][^1_2][^1_12]
- **sticky**: Node stays on the focused desktop of its monitor[^1_1][^1_2][^1_12]
- **private**: Node resists movement and resizing during automatic insertion[^1_1][^1_2][^1_12]
- **locked**: Node ignores the close (`-c`) message[^1_1][^1_2][^1_12]
- **marked**: Arbitrary flag used for custom operations; unmarked automatically when sent to a preselected node[^1_1][^1_2][^1_12]
- **urgent**: Indicates window urgency (typically set externally); used for window selection[^1_2]

Examples:

- `bspc node -g sticky=on` - Make window sticky
- `bspc node -g marked` - Toggle marked flag[^1_12]


#### Node Layering: `-l` and `--layer`

The **-l** or **--layer** flag changes a node's stacking layer[^1_2]. Valid layers are: `below`, `normal`, `above`[^1_2]. BSPWM maintains three stacking layers where below < normal < above, and within each layer: tiled/pseudo_tiled < floating < fullscreen[^1_2]. Examples:

- `bspc node -l above` - Place window above all others
- `bspc node -l below` - Place window below all others[^1_2]


#### Receptacle Insertion: `-i` and `--insert-receptacle`

The **-i** or `--insert-receptacle` flag creates a receptacle (empty leaf node) at the selected node's position[^1_2][^1_17]. Receptacles are particularly useful for creating predefined layouts that can be filled with windows later[^1_2][^1_17].

#### Node Closure: `-c` and `--close`

The **-c** or **--close** flag closes a node by sending it the close message[^1_2]. This respects the `locked` flag; if set, the message is ignored[^1_2].

#### Node Termination: `-k` and **--kill**

The **-k** or **--kill** flag forcibly terminates a node's window regardless of flags or settings[^1_2]. This is a hard termination that bypasses all safety mechanisms[^1_2].

### Node Selector Syntax

Node selectors determine which nodes are affected by commands[^1_2]. They follow the pattern: `[REFERENCE#]DESCRIPTOR[.MODIFIER]*`[^1_2].

#### Node Descriptors

Descriptors specify the initial node selection[^1_2]:

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
- **<node_id>**: Direct node ID reference[^1_2]


#### Node Modifiers

Modifiers further filter the selection[^1_2]:

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
- **[!]ancestor_of**: Is/isn't ancestor of reference[^1_2]


#### Path Jumps

Path jumps navigate the tree structure[^1_2]:

- **first|1**: First child
- **second|2**: Second child
- **brother**: Sibling node
- **parent**: Parent node
- **DIR**: Directional jump[^1_2]

***

## Desktop Domain: Workspace Management

### Desktop-Specific Commands

The **desktop** domain manages workspace-level settings and operations[^1_2]. Syntax: `bspc desktop [DESKTOP_SEL] COMMAND`[^1_2]. If no DESKTOP_SEL is provided, the focused desktop is targeted[^1_2].

#### Desktop Focus: `-f` and `--focus`

The **-f** or **--focus** flag switches focus to a specified desktop[^1_2]. Example:

- `bspc desktop -f '^2'` - Focus the 2nd desktop
- `bspc desktop -f next` - Focus next desktop[^1_2]


#### Desktop Activation: `-a` and `--activate`

The **-a** or **--activate** flag sets a desktop as active (similar to node activation)[^1_2]. This is distinct from focus and is useful in multi-monitor scenarios[^1_2].

#### Desktop Transfer to Monitor: `-m` and `--to-monitor`

The **-m** or `--to-monitor` flag transfers a desktop to a different monitor[^1_2]. Syntax: `bspc desktop -m MONITOR_SEL`[^1_2]. The optional **--follow** flag moves focus to the transferred desktop[^1_2].

#### Desktop Swapping: `-s` and `--swap`

The **-s** or **--swap** flag exchanges two desktops[^1_2]. Syntax: `bspc desktop -s DESKTOP_SEL`[^1_2]. The **--follow** flag maintains focus continuity[^1_2].

#### Layout Selection: `-l` and `--layout`

The **-l** or `--layout` flag sets or cycles the desktop's layout[^1_2]. Valid layouts are: `tiled`, `monocle`, or `next` to cycle[^1_2]. BSPWM provides two built-in layouts:

- **tiled**: Standard binary space partitioning with visible splits[^1_1][^1_2][^1_18]
- **monocle**: Fullscreen layout where only the most recently focused tiled/pseudo_tiled window is visible[^1_1][^1_2][^1_18]

Examples:

- `bspc desktop -l monocle` - Switch to monocle layout
- `bspc desktop -l next` - Cycle to next layout[^1_2]


#### Desktop Renaming: `-n` and `--rename`

The **-n** or `--rename` flag changes a desktop's name[^1_2]. Syntax: `bspc desktop -n <new_name>`[^1_2]. Desktop names are used in keybindings and configurations[^1_2].

#### Desktop Cycling: `-b` and `--bubble`

The **-b** or `--bubble` flag moves a desktop within the monitor's desktop list[^1_2]. Direction values are `next` or `prev`[^1_2]. This reorders desktops without changing their content[^1_2].

#### Desktop Removal: `-r` and `--remove**

The **-r** or `--remove` flag deletes a desktop[^1_2]. Windows on the removed desktop are transferred to the next available desktop[^1_2].

### Desktop Selector Syntax

Desktop selectors determine which desktops are affected[^1_2].

#### Desktop Descriptors

- **CYCLE_DIR** (next|prev): Cyclic direction
- **any**: Any desktop
- **last**: Most recently focused
- **newest**: Most recently created
- **older**: Older in history
- **newer**: Newer in history
- **focused**: Currently focused
- **^<n>**: The nth desktop
- **MONITOR_SEL:focused**: Focused desktop on specified monitor
- **<desktop_id>**: Direct ID reference
- **<desktop_name>**: By name[^1_2]


#### Desktop Modifiers

- **[!]focused**: Currently/not focused
- **[!]active**: Active/not active on desktop
- **[!]occupied**: Has/doesn't have windows
- **[!]urgent**: Contains/doesn't contain urgent windows
- **[!]local**: On/not on current monitor
- **[!]LAYOUT**: Matches/doesn't match layout
- **[!]user_LAYOUT**: Custom layout matching[^1_2]

***

## Monitor Domain: Multi-Monitor Configuration

### Monitor-Specific Commands

The **monitor** domain controls monitor-specific settings and operations[^1_2]. Syntax: `bspc monitor [MONITOR_SEL] COMMAND`[^1_2].

#### Monitor Focus: `-f` and `--focus`

The **-f** or `--focus` flag switches focus to a specified monitor[^1_2]. Example:

- `bspc monitor -f DP-1` - Focus monitor named DP-1
- `bspc monitor -f next` - Focus next monitor[^1_2]


#### Monitor Swapping: `-s` and `--swap`

The **-s** or `--swap` flag exchanges two monitors[^1_2]. This is useful for reversing display order[^1_2].

#### Add Desktops: `-a` and `--add-desktops`

The **-a** or `--add-desktops` flag creates new desktops on a monitor[^1_2]. Syntax: `bspc monitor -a <name>...`[^1_2]. Multiple desktop names can be provided[^1_2]. Example:

- `bspc monitor -a I II III IV V` - Create five desktops named I through V[^1_2]


#### Reorder Desktops: `-o` and `--reorder-desktops`

The **-o** or `--reorder-desktops` flag changes the order of desktops on a monitor[^1_2]. Syntax: `bspc monitor -o <name>...`[^1_2]. The order must match existing desktop names[^1_2].

#### Reset Desktops: `-d` and `--reset-desktops`

The **-d** or `--reset-desktops` flag reconfigures desktops, adding, removing, or renaming as needed[^1_2]. Syntax: `bspc monitor -d <name>...`[^1_2]. This command automatically handles all necessary adjustments[^1_2].

#### Monitor Rectangle: `-g` and `--rectangle`

The **-g** or `--rectangle` flag manually sets monitor geometry[^1_2]. Syntax: `bspc monitor -g WxH+X+Y`[^1_2]. This is useful for multi-display configurations or unusual setups[^1_2].

#### Monitor Renaming: `-n` and `--rename`

The **-n** or `--rename` flag changes a monitor's name[^1_2]. Syntax: `bspc monitor -n <new_name>`[^1_2].

#### Monitor Removal: `-r` and `--remove`

The **-r** or `--remove` flag removes a monitor from management[^1_2]. Desktops on the removed monitor are transferred to remaining monitors[^1_2].

### Monitor Selector Syntax

Monitor selectors determine which monitors are affected[^1_2].

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
- **^<n>**: The nth monitor
- **<monitor_id>**: By monitor ID
- **<monitor_name>**: By name (e.g., HDMI-1)[^1_2]


#### Monitor Modifiers

- **[!]focused**: Currently/not focused
- **[!]occupied**: Has/doesn't have occupied desktops[^1_2]

***

## Query Domain: State Information Retrieval

### Query Commands

The **query** domain retrieves metadata and state information[^1_2]. Syntax: `bspc query COMMANDS [OPTIONS]`[^1_2].

#### Node Query: `-N` and `--nodes`

The **-N** or `--nodes` flag returns node IDs[^1_2]. Syntax: `bspc query -N [NODE_SEL]`[^1_2]. Without selectors, returns all node IDs[^1_2]. With selectors, returns matching nodes[^1_2][^1_19]. Example:

- `bspc query -N -n focused` - Get ID of focused node
- `bspc query -N -n .window` - Get IDs of all window nodes[^1_2]


#### Desktop Query: `-D` and `--desktops`

The **-D** or `--desktops` flag returns desktop IDs[^1_2]. Syntax: `bspc query -D [DESKTOP_SEL]`[^1_2]. Returns all desktop IDs when no selector is given[^1_2].

#### Monitor Query: `-M` and `--monitors`

The **-M** or `--monitors` flag returns monitor IDs[^1_2]. Syntax: `bspc query -M [MONITOR_SEL]`[^1_2]. Returns all monitor IDs without selector[^1_2].

#### Tree Query: `-T` and `--tree`

The **-T** or `--tree` flag returns the complete tree structure in JSON or text format[^1_2][^1_20]. This comprehensive output shows all relationships and states[^1_2]. Example:

- `bspc query -T` - Output complete tree state
- `bspc query -T -m DP-1` - Tree for specific monitor[^1_2][^1_20]


### Query Options

Query results can be filtered and formatted[^1_2].

#### Monitor Specification: `-m` and `--monitor`

The **-m** or `--monitor` option filters results to a specific monitor[^1_2]. Syntax: `bspc query -m MONITOR_SEL`[^1_2].

#### Desktop Specification: `-d` and `--desktop`

The **-d** or `--desktop` option filters to a specific desktop[^1_2]. Syntax: `bspc query -d DESKTOP_SEL`[^1_2].

#### Node Specification: `-n` and `--node`

The **-n** or `--node` option filters to a specific node[^1_2]. Syntax: `bspc query -n NODE_SEL`[^1_2].

#### Names Output: `--names`

The `--names` flag outputs entity names instead of IDs[^1_2]. This is useful for human-readable output[^1_2].

***

## Rule Domain: Window Matching and Defaults

### Rule Management Commands

The **rule** domain defines patterns and default configurations for new windows[^1_2][^1_7].

#### Adding Rules: `-a` and `--add`

The **-a** or `--add` flag creates a new rule[^1_2]. Syntax: `bspc rule -a [CLASS[:INSTANCE[:NAME]]] [options]`[^1_2]. The class, instance, and name can be wildcards using `*`[^1_2]. Fields can be escaped with backslash[^1_2].

Rule properties include[^1_2]:

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
- **rectangle=WxH+X+Y**: Initial geometry for floating windows[^1_2][^1_7]

The **-o** or `--one-shot` flag makes the rule apply only once[^1_1][^1_3][^1_2][^1_7].

Examples:

- `bspc rule -a Firefox desktop=^2 follow=on`[^1_3][^1_2][^1_7]
- `bspc rule -a Gimp state=floating follow=off`[^1_3][^1_2][^1_7]


#### Removing Rules: `-r` and `--remove`

The **-r** or `--remove` flag deletes rules[^1_2]. Syntax: `bspc rule -r PATTERN`[^1_2]. Patterns can be `head`, `tail`, `^<n>` for index, or a class/instance/name pattern[^1_2].

#### Listing Rules: `-l` and `--list`

The **-l** or `--list` flag displays all currently active rules[^1_2].

### External Rules Script

For complex matching logic beyond the built-in rule syntax, BSPWM supports external rule scripts[^1_2][^1_21][^1_7]. The **external_rules_command** setting specifies a script path[^1_2]. This script receives window information and outputs rule properties[^1_2][^1_7].

Script invocation: `SCRIPT wid class instance name`[^1_2][^1_7]

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


***

## Config Domain: Global Configuration

### Configuration Settings

The **config** domain manages global, monitor-specific, desktop-specific, and node-specific settings[^1_2][^1_7].

#### General Syntax

`bspc config [-m MONITOR_SEL|-d DESKTOP_SEL|-n NODE_SEL] <setting> [<value>]`[^1_2]

### Global Settings

#### Color Settings

Colors are specified in hexadecimal format: `#RRGGBB`[^1_2]

- **normal_border_color**: Border color for unfocused windows[^1_2]
- **active_border_color**: Border color for active (desktop-focused) windows[^1_2]
- **focused_border_color**: Border color for focused windows[^1_2]
- **presel_feedback_color**: Color of preselection indicator[^1_2]


#### Layout and Splitting

- **split_ratio**: Default ratio for binary splits (0 < ratio < 1)[^1_2]
- **automatic_scheme**: Algorithm for automatic window placement[^1_2]. Valid values:
    - **longest_side**: Split along longest edge[^1_1][^1_2][^1_22]
    - **alternate**: Alternate between horizontal and vertical[^1_1][^1_2][^1_22]
    - **spiral**: Create spiral patterns[^1_1][^1_2][^1_22]
- **initial_polarity**: Which child receives the new window in automatic mode[^1_2]. Valid values:
    - **first_child**: New window becomes first child[^1_1][^1_2][^1_22]
    - **second_child**: New window becomes second child[^1_1][^1_2][^1_22]
- **directional_focus_tightness**: Strictness of directional focus algorithm[^1_2]. Valid values:
    - **high**: Stricter matching
    - **low**: Looser matching[^1_2]


#### Insertion and Removal

- **removal_adjustment**: Whether to readjust layout after window removal[^1_2]
- **presel_feedback**: Whether to show preselection feedback[^1_2]


#### Monocle Layout

- **borderless_monocle**: Remove borders in monocle layout[^1_2]
- **gapless_monocle**: Remove gaps in monocle layout[^1_2]
- **top_monocle_padding**, **right_monocle_padding**, **bottom_monocle_padding**, **left_monocle_padding**: Padding for monocle layout[^1_2]
- **single_monocle**: Switch to monocle if only one window remains[^1_2]


#### Status and Behavior

- **borderless_singleton**: Remove borders when only one window exists[^1_2]
- **status_prefix**: Prefix for status messages[^1_2]
- **pointer_motion_interval**: Minimum milliseconds between pointer motion updates[^1_2]
- **pointer_modifier**: Keyboard modifier for pointer actions[^1_2]. Valid values:
    - **shift**, **control**, **lock**
    - **mod1** (Alt), **mod2**, **mod3**
    - **mod4** (Super), **mod5**[^1_2][^1_23]
- **pointer_action1**, **pointer_action2**, **pointer_action3**: Mouse button actions[^1_2]. Valid values:
    - **move**: Move floating windows
    - **resize_side**: Resize from edge
    - **resize_corner**: Resize from corner
    - **focus**: Focus on click
    - **none**: No action[^1_2][^1_24][^1_23]
- **click_to_focus**: Mouse button for focus-on-click[^1_2]. Valid values: **button1**, **button2**, **button3**, **any**, **none**[^1_2]
- **swallow_first_click**: Consume first click when focusing[^1_2]
- **focus_follows_pointer**: Focus window under pointer[^1_2]
- **pointer_follows_focus**: Move pointer to focused window[^1_2]
- **pointer_follows_monitor**: Move pointer to focused monitor[^1_2]


#### EWMH Compatibility

- **mapping_events_count**: How many mapping events to process[^1_2]
- **ignore_ewmh_focus**: Ignore EWMH focus requests[^1_2]
- **ignore_ewmh_fullscreen**: Ignore fullscreen requests[^1_2]. Valid values: **none**, **all**, or comma-separated **enter**, **exit**[^1_2]
- **ignore_ewmh_struts**: Ignore taskbar/panel space reservations[^1_2]
- **center_pseudo_tiled**: Center pseudo_tiled windows[^1_2]


#### Monitor Management

- **remove_disabled_monitors**: Remove monitors that are disabled[^1_2]
- **remove_unplugged_monitors**: Remove unplugged monitors[^1_2]
- **merge_overlapping_monitors**: Merge monitors with overlapping geometry[^1_2]


### Monitor and Desktop Settings

#### Padding

Applied at monitor and desktop levels[^1_2]:

- **top_padding**, **right_padding**, **bottom_padding**, **left_padding**: Space reserved around the desktop edges[^1_2]. Commonly set to bar heights[^1_25][^1_26]


### Desktop Settings

#### Window Gap

- **window_gap**: Pixel spacing between windows[^1_2][^1_25][^1_26]. Can be set negative to create gapless layouts[^1_26]


### Node Settings

#### Borders and Hints

- **border_width**: Border thickness in pixels[^1_2]
- **honor_size_hints**: Respect ICCCM window size hints[^1_2]. Valid values:
    - **true**: Apply to all windows
    - **false**: Don't apply
    - **tiled**: Apply only to tiled windows
    - **floating**: Apply only to floating windows[^1_2]

***

## Subscribe Domain: Event-Driven Scripting

### Event Subscription

The **subscribe** domain enables reactive scripting based on window manager events[^1_2][^1_9][^1_27]. Syntax: `bspc subscribe [OPTIONS] (all|report|monitor|desktop|node|...)*`[^1_2].

### Subscription Options

#### FIFO Output: `-f` and `--fifo`

The **-f** or `--fifo` flag outputs events to a named FIFO instead of stdout[^1_2]. This enables long-lived subscriptions[^1_2].

#### Event Count: `-c` and `--count`

The **-c** or `--count` flag exits after receiving COUNT events[^1_2].

### Available Events

#### Monitor Events

- **monitor_add**: New monitor connected
- **monitor_rename**: Monitor renamed
- **monitor_remove**: Monitor disconnected
- **monitor_swap**: Monitors swapped positions
- **monitor_focus**: Focus changed to monitor
- **monitor_geometry**: Monitor geometry changed[^1_2][^1_9]


#### Desktop Events

- **desktop_add**: Desktop created
- **desktop_rename**: Desktop renamed
- **desktop_remove**: Desktop deleted
- **desktop_swap**: Desktops swapped
- **desktop_transfer**: Desktop transferred to different monitor
- **desktop_focus**: Desktop focus changed
- **desktop_activate**: Desktop activated
- **desktop_layout**: Layout changed[^1_2][^1_9]


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
- **node_layer**: Stacking layer changed[^1_2][^1_9]


#### Report Event

The **report** event outputs the complete current state in a specific format[^1_2].

### Event Processing Example

Using events to perform actions[^1_9]:

```bash
bspc subscribe node_add | while read -a msg; do
    desk_id=${msg[^1_2]}
    wid=${msg[^1_4]}
    # Make new windows fullscreen
    bspc node "$wid" -t fullscreen
done
```


***

## Wm Domain: Window Manager State

### Global Window Manager Operations

The **wm** domain controls global window manager state and operations[^1_2].

#### Dump State: `-d` and `--dump-state`

The **-d** or `--dump-state` flag outputs the complete window manager state[^1_2]. This is useful for backing up configurations or analysis[^1_2].

#### Load State: `-l` and `--load-state`

The **-l** or `--load-state** flag restores a previously dumped state[^1_9]. Syntax: `bspc wm -l <file_path>`[^1_9].

#### Add Monitor: `-a` and `--add-monitor`

The **-a** or `--add-monitor` flag manually adds a monitor to management[^1_2]. Syntax: `bspc wm -a <name> WxH+X+Y`[^1_2]. Useful for dynamic monitor addition[^1_2].

#### Reorder Monitors: `-O` and `--reorder-monitors`

The **-O** or `--reorder-monitors` flag changes the global monitor order[^1_2]. Syntax: `bspc wm -O <name>...`[^1_2].

#### Adopt Orphans: `-o` and `--adopt-orphans`

The **-o** or `--adopt-orphans` flag brings unmanaged windows under BSPWM control[^1_2].

#### Record History: `-h` and `--record-history`

The **-h** or `--record-history** flag enables/disables command history logging[^1_9]. Syntax: `bspc wm -h on|off`[^1_9].

#### Get Status: `-g` and `--get-status`

The **-g** or `--get-status` flag outputs the current status[^1_2].

#### Restart: `-r` and `--restart`

The **-r** or `--restart** flag restarts BSPWM while preserving windows[^1_9]. This is useful during configuration updates[^1_9][^1_75].

***

## Window States and Flags

### Window States

Each window maintains exactly one state at any time[^1_1][^1_2]:

#### Tiled

**Tiled** state means the window fills its assigned tiling space without overlapping others[^1_1][^1_2][^1_28]. Windows are arranged according to the binary tree structure[^1_1][^1_2][^1_28]. This is the default state for new windows[^1_1][^1_2].

#### Pseudo-Tiled

**Pseudo-tiled** windows respect ICCCM size hints while being centered within their tiling space[^1_1][^1_2][^1_28]. They can be resized but maintain their centered position[^1_1][^1_2][^1_28].

#### Floating

**Floating** windows can be positioned and resized freely anywhere on the desktop[^1_1][^1_2][^1_28]. They don't participate in automatic tiling but remain part of the node tree[^1_1][^1_2][^1_28].

#### Fullscreen

**Fullscreen** windows occupy their monitor's entire rectangle with no borders[^1_1][^1_2][^1_28]. They're placed above other content[^1_1][^1_2][^1_28]. When a fullscreen window is present and a new floating window is created, BSPWM must change the fullscreen window to tiled to display the floating window on top, unless the floating window is placed on the above layer[^1_29].

### Window Flags

Flags are independent states that can be combined[^1_1][^1_2]:

#### Hidden

**Hidden** windows don't occupy tiling space and aren't visible[^1_1][^1_2][^1_12]. They're useful for implementing scratchpads and minimization-like functionality[^1_1][^1_2][^1_12].

#### Sticky

**Sticky** windows follow the focused desktop on their monitor[^1_1][^1_2][^1_12]. When switching desktops, sticky windows appear on the new desktop[^1_1][^1_2][^1_12].

#### Private

**Private** windows resist movement and resizing during automatic insertion[^1_1][^1_2][^1_12]. When inserting new windows into automatic mode, private nodes maintain their position and size rather than being split[^1_1][^1_2][^1_12][^1_30].

#### Locked

**Locked** windows ignore the close message sent by `bspc node -c`[^1_1][^1_2][^1_12]. They require forceful termination with `bspc node -k`[^1_1][^1_2][^1_12].

#### Marked

**Marked** is an arbitrary flag useful for custom operations[^1_1][^1_2][^1_12]. It's particularly valuable in conjunction with preselection to implement deferred window movement[^1_1][^1_2][^1_12]. Marked nodes automatically become unmarked when sent to a preselected node[^1_1][^1_2][^1_12].

#### Urgent

**Urgent** indicates a window requiring attention[^1_2]. It's typically set externally by applications (e.g., when incoming messages arrive)[^1_2]. BSPWM uses this flag for window selection[^1_2].

***

## Stacking Layers

BSPWM implements three independent stacking layers[^1_2][^1_31][^1_32]:

1. **below**: Lowest layer
2. **normal**: Middle layer (default)
3. **above**: Top layer

Within each layer, the window order follows: tiled \& pseudo_tiled < floating < fullscreen[^1_2][^1_31][^1_32]. This means a floating window on the "below" layer appears above tiled windows on that layer but below all windows on the "normal" layer[^1_2][^1_31][^1_32].

***

## Advanced Configuration Patterns and Techniques

### Monitor Setup and Multi-Monitor Configuration

Setting up monitors requires careful use of the monitor domain commands[^1_3][^1_33]:

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

Creating desktops where all windows float by default[^1_1]:

```bash
bspc rule -a "*" -o desktop=floating_desktop state=floating
bspc desktop floating_desktop -l tiled  # or any layout
```


### Scratchpad/Dropdown Terminal Implementation[^1_3][^1_34]

Using hidden sticky windows to create dropdown terminal functionality:

```bash
# Create dropdown terminal rule
bspc rule -a dropdown -o sticky=on state=floating hidden=on rectangle=800x600+560+240

# Launch dropdown terminal
alacritty --class dropdown -e zsh &

# Toggle script
bspc node any.hidden.sticky -g hidden -f
```


### Receptacle-Based Manual Layouts[^1_17]

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


### External Rules for Complex Logic[^1_7]

When built-in rules aren't sufficient, external rule scripts provide unlimited flexibility[^1_7]:

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


### Event-Driven Automatic Layouts[^1_9]

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


### Node ID Tracking and Manipulation[^1_35]

Storing and using node IDs for complex operations:

```bash
# Store node ID and move it later
id=$(bspc query -N -n)
# ... perform other operations ...
bspc node "$id" -d '^2'  # Move stored node to desktop 2
```


### Custom Layouts with Master Stack[^1_16]

While BSPWM provides only tiled and monocle, external scripts enable layouts like master-stack[^1_16]:

```bash
# Balance 3-pane layout
bspc node @/ -B
bspc node @/1 -r 0.66  # Make left side 66% of space
```


### Pointer Action Configuration

Setting up mouse controls for floating window manipulation[^1_24][^1_23]:

```bash
# Configure pointer modifier and actions
bspc config pointer_modifier mod1          # Use Alt key
bspc config pointer_action1 move           # Alt+Button1 moves
bspc config pointer_action2 resize_side    # Alt+Button2 resizes edge
bspc config pointer_action3 resize_corner  # Alt+Button3 resizes corner
```


### Query-Based Window Selection[^1_19][^1_20]

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


***

## Conclusion

BSPWM represents a paradigm shift in window manager configuration philosophy. Rather than implementing every feature directly within the window manager, BSPWM provides a powerful, scriptable interface through **bspc** that enables users to build sophisticated configurations entirely from shell scripts[^1_1][^1_5][^1_6]. This approach offers unprecedented flexibility and transparency—users can see exactly what commands are being executed and modify them without learning specialized syntax[^1_1][^1_5][^1_6].

The comprehensive command set across six domains (node, desktop, monitor, query, rule, wm) combined with powerful selector syntax enables precise control over every aspect of window management[^1_2]. Advanced features like event subscription, external rule scripts, and tree manipulation operations provide the foundation for highly customized window management workflows[^1_2][^1_9].

Mastering BSPWM configuration requires understanding the hierarchical structure of selectors, the binary tree model of window arrangement, and how to combine simple commands into complex behaviors[^1_1][^1_2]. The sophisticated user can leverage all documented flags and options to create configurations that adapt dynamically to their workflow, automating complex window management scenarios through scripting and event handling[^1_9][^1_6].

This documentation provides the authoritative reference for every flag, option, and configuration technique available in BSPWM, extracted from official sources and community expertise, enabling users to build the most sophisticated window management configurations possible[^1_1][^1_3][^1_2][^1_7][^1_9][^1_6].

***

## References

All information in this documentation is sourced directly from the following authoritative web resources:

[^1_1] Bspwm Basics - dead airspace
[^1_36] BSPWM Cheatsheet - Garuda Linux
[^1_8] BSPWM Getting Started Guide - bspwm-doc
[^1_3] BSPWM - ArchWiki
[^1_4] Bspwm: A Bare-Bones Window Manager - Dev.to
[^1_2] bspwm(1) - Arch Manual Pages
[^1_5] Getting Started with BSPWM for Beginners - Reddit
[^1_33] Bspwm - Archcraft
[^1_12] Node Flags Explanation - Reddit
[^1_29] Fullscreen with Tiling Windows - Reddit
[^1_18] Cycling Layouts in BSPWM - Reddit
[^1_30] ELI5 Flags - Reddit
[^1_28] BSPWM Functioning - EndeavourOS Discovery
[^1_17] Manual Tiling with Receptacles - Brodie Robertson YouTube
[^1_25] Gap Between Window and Polybar - Reddit
[^1_14] Insertion Modes - madnight.github.io
[^1_21] CHANGELOG - madnight GitHub
[^1_15] Chinese Documentation - CSDN
[^1_26] BSPWMRC Example - GitHub Gist
[^1_37] BSPWM Package - SUSE PackageHub
[^1_19] BSPC Query Output Format - Reddit
[^1_35] Focusing Nodes by Class - Reddit
[^1_22] Ubuntu Manual Pages
[^1_20] BSPWM Query Language - YouTube
[^1_10] BSPWM Node Commands - YouTube
[^1_11] Move Window to Different Desktop - Reddit
[^1_7] Configuring BSPWM - Krython
[^1_9] BSPC Subscribe Events - YouTube
[^1_6] Configuration Script for BSPWM - YouTube
[^1_24] Mouse Drag and Resize - Reddit
[^1_16] Master Stack Layout - YouTube
[^1_23] Pointer Move/Resize - GitHub Issue
[^1_31] Stacking Layers - Reddit
[^1_32] Ubuntu Manual Pages (Stacking)


[^1_1]: https://dharmx.is-a.dev/bspwm-basics/

[^1_2]: https://man.archlinux.org/man/bspwm.1

[^1_3]: https://wiki.archlinux.org/title/Bspwm

[^1_4]: https://dev.to/l04db4l4nc3r/bspwm-a-bare-bones-window-manager-44di

[^1_5]: https://www.reddit.com/r/unixporn/comments/feseh2/getting_started_with_bspwm_for_beginners/

[^1_6]: https://www.youtube.com/watch?v=au-ZEcJKNb0

[^1_7]: https://krython.com/post/configuring-bspwm-window-manager/

[^1_8]: https://github.com/IntrepidPig/bspwm-doc/blob/master/getting-started.md

[^1_9]: https://www.youtube.com/watch?v=dER9xkB50no

[^1_10]: https://www.youtube.com/watch?v=smZUSnUCjwg

[^1_11]: https://www.reddit.com/r/bspwm/comments/s1zkc6/move_window_to_different_desktop_and_follow/

[^1_12]: https://www.reddit.com/r/bspwm/comments/161o8zq/new_to_bspwm_what_are_node_flags/

[^1_13]: https://www.reddit.com/r/bspwm/comments/muys88/you_can_move_a_node_without_swapping_positions/

[^1_14]: https://madnight.github.io/bspwm/

[^1_15]: https://blog.csdn.net/weixin_32999557/article/details/154185130

[^1_16]: https://www.youtube.com/watch?v=4TN-tMYVpBw

[^1_17]: https://www.youtube.com/watch?v=9YUreBjtAGo

[^1_18]: https://www.reddit.com/r/bspwm/comments/q6crzg/is_it_possible_to_cycle_between_layouts_in_bspwm/

[^1_19]: https://www.reddit.com/r/bspwm/comments/6or5r0/bspc_query_output_and_bspc_format/

[^1_20]: https://www.youtube.com/watch?v=cQNy6yVvfBU

[^1_21]: https://madnight.github.io/bspwm/CHANGELOG/

[^1_22]: https://manpages.ubuntu.com/manpages/focal/man1/bspwm.1.html

[^1_23]: https://github.com/baskerville/bspwm/issues/678

[^1_24]: https://www.reddit.com/r/bspwm/comments/d83ntm/how_do_i_move_and_resize_a_floating_window_with/

[^1_25]: https://www.reddit.com/r/bspwm/comments/juv038/gap_between_window_and_polybar_after_the_polybar/

[^1_26]: https://gist.github.com/fokusov/adc6a0afee013982037c5ef1f93dcc92

[^1_27]: https://www.reddit.com/r/bspwm/comments/fry5bb/ideas_for_bspc_subscribe/

[^1_28]: https://discovery.endeavouros.com/window-tiling-managers/bspwm-2/2021/03/

[^1_29]: https://www.reddit.com/r/bspwm/comments/g9ynnu/fulscreen_with_tiling_windows/

[^1_30]: https://www.reddit.com/r/bspwm/comments/cqtux4/eli5_flags/

[^1_31]: https://www.reddit.com/r/bspwm/comments/d4lnhx/is_it_possible_to_change_stacking_layers_order/

[^1_32]: https://manpages.ubuntu.com/manpages/noble/man1/bspwm.1.html

[^1_33]: https://wiki.archcraft.io/docs/window-managers/tiling-wm/bspwm/

[^1_34]: https://ersi.vivaldi.net/2019/05/11/bspwm-by-arcolinux-still-looking-for-the-perfect-desktop/

[^1_35]: https://www.reddit.com/r/bspwm/comments/oht7on/i_think_im_stupid/

[^1_36]: https://wiki.garudalinux.org/en/bspwm-cheatsheet

[^1_37]: https://packagehub.suse.com/packages/bspwm/0_9_5-bp150_2_4/

[^1_38]: https://www.reddit.com/r/archlinux/comments/ruz9ww/arch_tiling_window_manager/

[^1_39]: https://forum.endeavouros.com/t/beginners-quick-start-guide-to-endeavour-os-bspwm-edition/13095

[^1_40]: https://github.com/jaagr/polybar/issues/982

[^1_41]: https://bgdawes.github.io/bspwm-xfce-dotfiles/

[^1_42]: https://pkgs.alpinelinux.org/package/v3.18/main/armv7/musl

[^1_43]: https://www.youtube.com/watch?v=WBE1NrKX9ZA

[^1_44]: https://www.linux.org.ru/forum/general/17833769

[^1_45]: https://www.youtube.com/watch?v=XTcf8g54RuU

[^1_46]: https://thegeeks.ir/discover?q=%23shell_prompt

[^1_47]: http://dev.james-ball.co.uk/blog/2020/05/01/bspwm/

[^1_48]: https://www.instructables.com/Bspwm-Installation-and-Configuration/

[^1_49]: https://man.archlinux.org/man/bspwm.1.en

[^1_50]: https://www.youtube.com/watch?v=Ihvf8urQLJk

[^1_51]: https://www.reddit.com/r/bspwm/comments/1pjjb74/bspwm_window_inspector_a_simple_shell_script_for/

[^1_52]: https://www.reddit.com/r/bspwm/comments/p2ee07/how_to_switch_focus_from_floating_window_to_a/

[^1_53]: https://www.dragonflybsd.org/cgi/web-man?command=bspc\&section=1

[^1_54]: https://github.com/baskerville/bspwm/issues/977

[^1_55]: https://www.reddit.com/r/linuxmasterrace/comments/meas3r/desktop_window_manager/

[^1_56]: https://github.com/baskerville/bspwm/issues/224

[^1_57]: https://github.com/dharmx/vile

[^1_58]: https://www.reddit.com/r/bspwm/comments/18k0ah8/bspwm_freeze_when_resizemove_floating_windows/

[^1_59]: https://www.reddit.com/r/PowerShell/comments/1ebswfn/azure_nsg_modification_updateadd_rules/

[^1_60]: https://docs.oracle.com/en/middleware/bpm/12.2.1.3/rules-reference/rules-language-reference-oracle-business-process-management.pdf

[^1_61]: https://www.reddit.com/r/paloaltonetworks/comments/17u67i1/change_rules_from_service_any_to/

[^1_62]: https://www.reddit.com/r/kde/comments/eqcrxm/how_to_disable_kde_from_applying_the_wallpaper/

[^1_63]: https://community.exoscale.com/reference/cli/exo/compute/security-group/rule/add/

[^1_64]: https://github.com/baskerville/bspwm/issues/925

[^1_65]: https://github.com/baskerville/bspwm/blob/master/README.md

[^1_66]: https://docs.oracle.com/cd/E23521_01/doc.60/e23528/sbpms_creating_rules.htm

[^1_67]: https://manpages.debian.org/experimental/bspwm/bspc.1.en.html

[^1_68]: https://www.ibm.com/docs/he/power9?topic=commands-rules-command

[^1_69]: https://bbs.archlinux.org/viewtopic.php?id=257000

[^1_70]: https://www.reddit.com/r/bspwm/comments/88xv9x/how_to_focus_node_by_wmclass/

[^1_71]: https://stackoverflow.com/questions/49053369/how-to-implement-tree-command-in-bash-shell-script

[^1_72]: https://www.reddit.com/r/bspwm/comments/hytkod/alternate_configurations_not_working_in_bspwm/

[^1_73]: https://lib.rs/rendering/data-formats

[^1_74]: https://www.reddit.com/r/bspwm/comments/123f88p/changing_layout_of_node/

[^1_75]: https://www.reddit.com/r/bspwm/comments/l5ya33/question_about_node_ids_and_querying_windows/

[^1_76]: https://www.reddit.com/r/bspwm/comments/6jj6le/move_window_to_workspace_and_then_switch_to_that/

[^1_77]: https://www.linuxlinks.com/bspwm-tiling-window-manager/

[^1_78]: https://stackoverflow.com/questions/10606685/display-directory-tree-output-like-tree-command

[^1_79]: https://git.riyyi.com/riyyi/dotfiles/commit/7a08a720d0e2cad41408a233e84ad5179ce1911e?style=split\&whitespace=

[^1_80]: https://github.com/sonic-net/SONiC/blob/master/doc/warm-reboot/swss_warm_restart.md

[^1_81]: https://gist.github.com/wimstefan/64e17d257d07b666079628a7e1859823

[^1_82]: https://www.reddit.com/r/bspwm/comments/jvsjsl/scripts_internal_rules_and_external_rules/

[^1_83]: https://github.com/baskerville/bspwm/issues/680

[^1_84]: https://bbs.archlinux.org/viewtopic.php?id=305231

[^1_85]: https://stackoverflow.com/questions/73917457/bspwm-does-not-switch-focus-to-my-second-screen-it-works-when-going-there-with

[^1_86]: https://github.com/baskerville/bspwm/issues/628

[^1_87]: https://github.com/baskerville/bspwm

[^1_88]: https://forum.endeavouros.com/t/bspwm-rules/36451

[^1_89]: https://www.cbit.ac.in/wp-content/uploads/2022/12/Final-2021-Part-2-Proofs_merged.pdf

[^1_90]: https://stackoverflow.com/questions/40257320/how-to-define-window-stacking-order

[^1_91]: https://www.reddit.com/r/bspwm/comments/kgdsbs/noob_question_how_to_drag_floating_windows_around/

[^1_92]: https://www.gitam.edu/sites/default/files/docs/others/faculty_publications.pdf

[^1_93]: https://forums.freebsd.org/threads/bspwm-problem.54795/

[^1_94]: https://github.com/baskerville/bspwm/issues/754

[^1_95]: https://gist.github.com/panther03/c823f5864d1245da713fe1e48946c051

[^1_96]: https://github.com/baskerville/bspwm/issues/298

[^1_97]: https://www.reddit.com/r/bspwm/comments/arv1aw/bspc_to_use_super_key_for_resizing/

[^1_98]: https://github.com/baskerville/bspwm/blob/master/doc/CHANGELOG.md

