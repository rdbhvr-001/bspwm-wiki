## BSPC Query Commands and Flags

The bspc query interface provides read-only commands to inspect BSPWM’s internal state.  The core query subcommands are:

`-N, --nodes` – list matching node IDs (windows or internal tree nodes).

`-D, --desktops` – list matching desktop IDs (or names with --names).

`-M, --monitors` – list matching monitor IDs (or names with --names).

`-T, --tree` – print a JSON representation of the matching object (monitor, desktop or node).


Each of the above accepts optional selectors (given in brackets) to filter which items match. For example, 

```bash
bspc query -N  # (with no selector) 
```

prints all node IDs on the current desktop (including parent and receptacle nodes and leaf windows).

To list only window (leaf) nodes, one can add a node selector:
```bash
bspc query -N -n .window
```
lists just the actual window nodes.

In general a selector like `.window`, `.leaf`, `.tiled`, etc., may be appended (after a `-n` or `-d` flag) to match only nodes with that property.  (Any valid node, desktop or monitor selector can be used according to the BSPWM reference.)

For example, if two windows are open on the focused desktop, running:

```bash
bspc query -N

# Outputs
# 0x01A00001
# 0x01B00001
```

lists their **node IDs** (hex values may vary).
Using a selector, 

```bash
bspc query -N -n .window
```

would return the same IDs in this case, whereas without `-n .window` BSPWM might include an **extra internal parent** node ID as well.

**Desktop listing (-D, --desktops):** The `-D` subcommand lists the IDs of matching desktops.  By default it outputs hex IDs, but adding `--names` prints desktop names.  For example, on a monitor with desktops named “1”, “2”, “3”, 

```bash
bspc query -D --names 
```

might output:

```bash
1
2
3
```

while the query -

```bash
bspc query -D 
```

(IDs only) might output:

```c
0x00A00001
0x00A00002
0x00A00003
```

**The -D command can also be scoped:** e.g. `bspc query -D -m HDMI-1` lists only desktops on monitor “HDMI-1”.  Similarly, using -D -d focused lists only the (ID or name of the) currently focused desktop.

**Monitor listing (-M, --monitors):** The `-M` subcommand lists **monitor IDs**.  With `--names`, it prints monitor names (the X11 output names).  For example, 

```bash 
bspc query -M --names 
```

might output:

```bash
HDMI-1
DP-1
```

on a two-monitor setup.  Without `--names`, it might show corresponding hexadecimal IDs.  Monitors can also be filtered: `bspc query -M -n focused` lists just the focused monitor (which yields a single ID/name).

**Tree output (-T, --tree):** The `-T` flag instructs bspc query to output a **JSON-formatted tree** of the matching object.  By itself (with no selectors) `bspc query -T` prints a JSON object for the focused desktop: it includes the *layout, nodes,* and *metadata* for all windows on that desktop.  You can combine it with other flags to select the target: for example, 

```bash
bspc query -T -d focused # or simply bspc query -T 
```

You can also query subtrees: e.g. 

```bash 
bspc query -T -n focused
``` 

emits the JSON tree rooted at the currently focused window (containing just that leaf).  The output is verbose JSON; in scripts you might pipe it through jq or similar to extract fields.

**Scope options (-m, -d, -n):** The flags `-m [MON_SEL]`, `-d [DESK_SEL]`, and `-n [NODE_SEL]` constrain a query to a specific monitor, desktop, or node.  For example, `bspc query -N -d 2` lists nodes on desktop index 2.  (Selectors can be numeric indices, names, or special syntax like `^1` for the first desktop.)  Likewise, `bspc query -D -m ^2` lists desktops on the second monitor (by order).  These options accept the same selector syntax as used in other bspc commands.  Omitting the descriptor is allowed: e.g. 

```bash
bspc query -M -d focused 
```

first selects the focused desktop then lists its monitor.

**Name output (--names):** The `--names` flag causes -M or -D to output human-readable names instead of hex IDs.  It has no effect with -N (node IDs always appear as hex) or -T (JSON always uses IDs internally).  For instance, bspc query -D --names on desktops named “web” and “code” would print those names.

**Deprecated query flags:** In older BSPWM versions (≈0.8.x), there was a -W, `--windows` subcommand that listed window (leaf) node IDs.  For example, forum discussions show:

> [!NOTE]
> `bspc query -W -d $DESKTOP | wc -l` will return the number of windows on $DESKTOP.
This `-W` option was removed in **BSPWM 0.9** in favor of using `-N` with a `.window` selector.  (In modern BSPWM, you would do `bspc query -N -n .window` to get the same list of window IDs.)



**Examples:**  Here are some illustrative commands. To list all node IDs on the current desktop:

`bspc query -N`

To list only window (leaf) nodes:

```bash
bspc query -N -n .window
```

To list all desktop names on monitor HDMI-1:

```bash
bspc query -D -m HDMI-1 --names
```

To show the JSON tree of the focused desktop:

```bash
bspc query -T -d focused
```

In summary, all bspc query subcommands across BSPWM releases are `-N/--nodes`, `-D/--desktops`, `-M/--monitors`, and `-T/--tree`, along with the filtering flags `-m/--monitor`, `-d/--desktop`, `-n/--node`, and the formatting flag `--names`.

> [!WARNING]
> The `-W/--windows` command existed in older 0.8-series releases but is no longer present and is currently deprecated. We recommend you to avoid the usage of `--windows` flag in the configuration.

## The query selectors

**Selectors** choose a **node (window)**, **desktop (workspace)** or **monitor** for `bspc` commands. The general shape is an optional reference, a descriptor, and zero or more modifiers, written like:

```c
[REFERENCE#]DESCRIPTOR(.MODIFIER)*
```

The **reference** is another selector (used when computing a relative target). **Modifiers** narrow or invert the descriptor. All of the concrete descriptors and modifiers below are taken from the official bspc manual — use them exactly as shown. 

> [!TIP]
> In order to flip the descriptor, you have to prefix it with and exclamatory mark `!`. Yeah, the one which looks like the `inverted i` is used to invert the meaning , i.e negate the meaning of the discriptor.


### Node selectors (windows / tree leaves)

A node selector (NODE_SEL) picks a node (a leaf or internal node in BSPWM’s binary tree). Its descriptor can be a direction (`north|west|south|east`), a cyclic direction (`next|prev`), an explicit path, `any`, `first_ancestor`, `last`, `newest`, `older`, `newer`, `focused`, `pointed`, `biggest`, `smallest`, or a `raw node id`. Examples and detailed meanings follow; each paragraph explains one descriptor and includes an example showing how to use it with `bspc query -N`.

`focused` selects the currently focused node. To print the node id for the focused window:

```bash
bspc query -N focused
```

`pointed` selects the leaf under the pointer (mouse). Useful when combining pointer actions with scripts:

```bash
bspc query -N pointed
```

**Directional descriptors** (`north`, `south`, `east`, `west`) select a window in that spatial direction relative to the reference node (default reference is the focused node). For example, to get the node id to the west of the focused window:

```bash
bspc query -N west
```

**Cyclic descriptors** `next` / `prev` walk the tree in a depth-first in-order traversal; 

```bash
bspc query -N next 
```

returns the next node in that ordering relative to the reference.

**`any`** returns the first node matching any appended modifiers; it’s a convenience for “give me a node that matches these constraints”. For example, to find any floating window on the current desktop:

```bash
bspc query -N any.floating
```

**`first_ancestor`** jumps up the tree from the reference node and returns the first ancestor that matches appended modifiers (useful when you want the container rather than the leaf). Example:

```bash
bspc query -N first_ancestor.window
```

**`last returns`** the previously focused node relative to the reference node (i.e., focus history), and newest, older, newer are history-based descriptors referring to nodes in the focus history. Example asking for the newest node in the focused-node history:

```bash
bspc query -N newest
```

**`biggest and smallest`** select the leaf with the largest or smallest onscreen area (size metrics come from the window tree). Example:

```bash
bspc query -N biggest.window
```

**A raw `< node_id >` also works:** if you already have a node id (e.g., 0x80000d), you can pass it as the descriptor to target exactly that node:

```bash
bspc query -N 0x80000d
```

**Paths let you jump around the tree explicitly.** The `PATH` form is written with a leading `@` and optional `desktop prefix`, then jumps (`first|1`, `second|2`, `brother`, `parent`, or a `DIR`).

For instance, `@/first` starts from the root of the default (or referenced) desktop and jumps to its `first child`; to query that node id:

```bash
bspc query -N @/first
```

> [!IMPORTANT]
> **Modifiers refine matching.** For *nodes* you can append `.focused`, `.active`, `.automatic`, `.local`, `.leaf`, `.window`, a *state* like `.floating` or `.tiled`, *flags* like `.hidden` or `.urgent`, *layer* `.above`, `.normal`, `.below`, *split type* `.horizontal` / `.vertical`, or *relations* like `.same_class`, `.descendant_of`, .`ancestor_of`. Prefix any modifier with `!` to invert it. 

Example: return the node id of the focused window **only if it is floating:**

```bash
bspc query -N focused.floating
```

Use `!` to invert: find a focused node **that is not floating:**

```bash
bspc query -N focused.!floating
```

> [!NOTE]
> Everything in this node section follows the canonical NODE_SEL grammar in the man page.


### Desktop selectors (workspaces)

Desktop selectors (DESKTOP_SEL) pick a desktop. Descriptors include `next|prev` (cyclic), `any`, `last`, `newest`, `older`, `newer`, `focused`, `numeric ^< n >` (nth desktop), *desktop id*, or *desktop name*; you may also prefix with a monitor selector MONITOR_SEL: to pick the nth desktop on a particular monitor.

Example to print the name or id of the focused desktop:

```bash
bspc query -D focused --names
```

To pick the second desktop on the currently focused monitor use the `^<n>` notation:

> [!NOTE]
> You have to quote the `^<n>` as shown in the code below. Otherwise the shell (bash, zsh or any POSIX shell) wil expand '^' and you may run into errors or unexpected behaviours.

```bash
bspc query -D '^2' --names
```

To pick the nth desktop on a specific monitor by monitor selector (e.g., primary):

```bash
bspc query -D primary:^3 --names
```

**Desktop modifiers include** `.focused` (only consider focused desktops), `.active` (desktops that are focused on their monitor), `.occupied` (contain windows), `.urgent`, and `.local` (limit to the reference monitor).

For example, **to list IDs of all occupied desktops on the current monitor**:

```bash
bspc query -D any.occupied
```

If you leave out a descriptor where allowed (the descriptor can be omitted for `-D/-M/-N` in some cases), the default is the focused target for that domain. The desktop selector details and modifier names are from the bspc manual. 


### Monitor selectors

**Monitor selectors (MONITOR_SEL) choose monitors.** Descriptors are `north|west|south|east` (spacial DIR relative to the reference monitor), `next|prev` (cycle), `any`, `last`, `newest`, `older`, `newer`, `focused`, `pointed`, `primary`, `^<n>` (nth monitor), *monitor id*, or *monitor name*.

For example, **to print the names of all monitors**:

```bash
bspc query -M --names
```

To target the primary monitor:

```bash
bspc query -M primary --names
```

Monitor modifiers are `.focused` and `.occupied` (where the focused desktop on that monitor is occupied). For example, to find the monitor under the pointer and get its id:

```bash
bspc query -M pointed
```

### Practical bspc query examples using selectors

Show node ids for all windows on the focused desktop (returns a list of node ids):

```bash 
bspc query -N
```
Show node ids for windows that are floating on the focused desktop:

```bash
bspc query -N any.floating
```

Show the id of the window under the mouse pointer:

```bash
bspc query -N pointed
```

Show the id (or name with --names) of the currently focused desktop:

```bash
bspc query -D focused --names
```

Show names of all monitors in visual order:

```bash
bspc query -M --names
```

Return the previously focused desktop (useful when scripting focus toggles):

```bash
bspc query -D last --names
```

Select the first ancestor of the focused node that is a container (useful to operate on the subtree enclosing a window):

```bash
bspc query -N first_ancestor
```

Combine selectors and modifiers to target a node by class relation: for example, choose any node that has the same class as the focused node but is not floating:

```bash
bspc query -N any.same_class.!floating
```

## Notes, pitfalls and best practices

> [!NOTE]
> * Selectors are evaluated relative to a reference (default is the focused item).
>
> * Use explicit references when you need deterministic behavior across monitors/desktops, for example prefix a path with a desktop selector `@[DESKTOP_SEL:]`....
>
> * Remember to quote shell-sensitive tokens like `^2` or tokens containing `!`.
>
> * The `!` inversion applies to individual modifiers (e.g., `.!occupied`), **not to whole descriptors.**
>
> * The man page or the bspwm-wiki(this) is the authoritative source for the exact names, allowed modifiers and the path jump syntax — consult it when you need exhaustive precision. 


