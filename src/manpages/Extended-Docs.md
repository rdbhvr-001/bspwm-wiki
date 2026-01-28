
# BSPWM(1)                                 Bspwm Manual                                 BSPWM(1)

NAME
bspwm - Binary space partitioning window manager

SYNOPSIS
bspwm [-h|-v|-c CONFIG_PATH]

       bspc --print-socket-path
    
       bspc DOMAIN [SELECTOR] COMMANDS
    
       bspc COMMAND [OPTIONS] [ARGUMENTS]
    DESCRIPTION
bspwm is a tiling window manager that represents windows as the leaves of a full binary
tree.

       It is controlled and configured via bspc.
    OPTIONS
-h
Print the synopsis and exit.

       -v
           Print the version and exit.
    
       -c CONFIG_PATH
           Use the given configuration file.
    
       --print-socket-path
           Print the bspwm socket path and exit.
    COMMON DEFINITIONS
DIR         := north | west | south | east
CYCLE_DIR   := next | prev

SELECTORS
Selectors are used to select a target node, desktop, or monitor. A selector can either
describe the target relatively or name it globally.

       Selectors consist of an optional reference, a descriptor and any number of
       non-conflicting modifiers as follows:
    
           [REFERENCE#]DESCRIPTOR(.MODIFIER)*
    
       The relative targets are computed in relation to the given reference (the default
       reference value is focused).
    
       An exclamation mark can be prepended to any modifier in order to reverse its meaning.
    
       The following characters cannot be used in monitor or desktop names: #, :, ..
    
       The special selector %<name> can be used to select a monitor or a desktop with an
       invalid name.
    Node
Select a node.

           NODE_SEL := [NODE_SEL#](DIR|CYCLE_DIR|PATH|any|first_ancestor|last|newest|
                                   older|newer|focused|pointed|biggest|smallest|
                                   <node_id>)[.[!]focused][.[!]active][.[!]automatic][.[!]local]
                                             [.[!]leaf][.[!]window][.[!]STATE][.[!]FLAG][.[!]LAYER][.[!]SPLIT_TYPE]
                                             [.[!]same_class][.[!]descendant_of][.[!]ancestor_of]
    
           STATE := tiled|pseudo_tiled|floating|fullscreen
    
           FLAG := hidden|sticky|private|locked|marked|urgent
    
           LAYER := below|normal|above
    
           SPLIT_TYPE := horizontal|vertical
    
           PATH := @[DESKTOP_SEL:][[/]JUMP](/JUMP)*
    
           JUMP := first|1|second|2|brother|parent|DIR
    
       Descriptors
    
           DIR
               Selects the window in the given (spacial) direction relative to the reference
               node.
    
           CYCLE_DIR
               Selects the node in the given (cyclic) direction relative to the reference node
               within a depth-first in-order traversal of the tree.
    
           PATH
               Selects the node at the given path.
    
           any
               Selects the first node that matches the given selectors.
    
           first_ancestor
               Selects the first ancestor of the reference node that matches the given
               selectors.
    
           last
               Selects the previously focused node relative to the reference node.
    
           newest
               Selects the newest node in the history of the focused node.
    
           older
               Selects the node older than the reference node in the history.
    
           newer
               Selects the node newer than the reference node in the history.
    
           focused
               Selects the currently focused node.
    
           pointed
               Selects the leaf under the pointer.
    
           biggest
               Selects the biggest leaf.
    
           smallest
               Selects the smallest leaf.
    
           <node_id>
               Selects the node with the given ID.
    
       Path Jumps
    
           The initial node is the focused node (or the root if the path starts with /) of the
           reference desktop (or the selected desktop if the path has a DESKTOP_SEL prefix).
    
           1|first
               Jumps to the first child.
    
           2|second
               Jumps to the second child.
    
           brother
               Jumps to the brother node.
    
           parent
               Jumps to the parent node.
    
           DIR
               Jumps to the node holding the edge in the given direction.
    
       Modifiers
    
           [!]focused
               Only consider the focused node.
    
           [!]active
               Only consider nodes that are the focused node of their desktop.
    
           [!]automatic
               Only consider nodes in automatic insertion mode. See also --presel-dir under
               Node in the DOMAINS section below.
    
           [!]local
               Only consider nodes in the reference desktop.
    
           [!]leaf
               Only consider leaf nodes.
    
           [!]window
               Only consider nodes that hold a window.
    
           [!](tiled|pseudo_tiled|floating|fullscreen)
               Only consider windows in the given state.
    
           [!]same_class
               Only consider windows that have the same class as the reference window.
    
           [!]descendant_of
               Only consider nodes that are descendants of the reference node.
    
           [!]ancestor_of
               Only consider nodes that are ancestors of the reference node.
    
           [!](hidden|sticky|private|locked|marked|urgent)
               Only consider windows that have the given flag set.
    
           [!](below|normal|above)
               Only consider windows in the given layer.
    
           [!](horizontal|vertical)
               Only consider nodes with the given split type.
    Desktop
Select a desktop.

           DESKTOP_SEL := [DESKTOP_SEL#](CYCLE_DIR|any|last|newest|older|newer|
                                         [MONITOR_SEL:](focused|^<n>)|
                                         <desktop_id>|<desktop_name>)[.[!]focused][.[!]active]
                                                                     [.[!]occupied][.[!]urgent][.[!]local]
                                                                     [.[!]LAYOUT][.[!]user_LAYOUT]
    
           LAYOUT := tiled|monocle
    
       Descriptors
    
           CYCLE_DIR
               Selects the desktop in the given direction relative to the reference desktop.
    
           any
               Selects the first desktop that matches the given selectors.
    
           last
               Selects the previously focused desktop relative to the reference desktop.
    
           newest
               Selects the newest desktop in the history of the focused desktops.
    
           older
               Selects the desktop older than the reference desktop in the history.
    
           newer
               Selects the desktop newer than the reference desktop in the history.
    
           focused
               Selects the currently focused desktop.
    
           ^<n>
               Selects the nth desktop. If MONITOR_SEL is given, selects the nth desktop on
               the selected monitor.
    
           <desktop_id>
               Selects the desktop with the given ID.
    
           <desktop_name>
               Selects the desktop with the given name.
    
       Modifiers
    
           [!]focused
               Only consider the focused desktop.
    
           [!]active
               Only consider desktops that are the focused desktop of their monitor.
    
           [!]occupied
               Only consider occupied desktops.
    
           [!]urgent
               Only consider urgent desktops.
    
           [!]local
               Only consider desktops inside the reference monitor.
    
           [!](tiled|monocle)
               Only consider desktops with the given layout.
    
           [!](user_tiled|user_monocle)
               Only consider desktops which have the given layout as userLayout.
    Monitor
Select a monitor.

           MONITOR_SEL := [MONITOR_SEL#](DIR|CYCLE_DIR|any|last|newest|older|newer|
                                         focused|pointed|primary|^<n>|
                                         <monitor_id>|<monitor_name>)[.[!]focused][.[!]occupied]
    
       Descriptors
    
           DIR
               Selects the monitor in the given (spacial) direction relative to the reference
               monitor.
    
           CYCLE_DIR
               Selects the monitor in the given (cyclic) direction relative to the reference
               monitor.
    
           any
               Selects the first monitor that matches the given selectors.
    
           last
               Selects the previously focused monitor relative to the reference monitor.
    
           newest
               Selects the newest monitor in the history of the focused monitors.
    
           older
               Selects the monitor older than the reference monitor in the history.
    
           newer
               Selects the monitor newer than the reference monitor in the history.
    
           focused
               Selects the currently focused monitor.
    
           pointed
               Selects the monitor under the pointer.
    
           primary
               Selects the primary monitor.
    
           ^<n>
               Selects the nth monitor.
    
           <monitor_id>
               Selects the monitor with the given ID.
    
           <monitor_name>
               Selects the monitor with the given name.
    
       Modifiers
    
           [!]focused
               Only consider the focused monitor.
    
           [!]occupied
               Only consider monitors where the focused desktop is occupied.
    WINDOW STATES
tiled
Its size and position are determined by the window tree.

       pseudo_tiled
           A tiled window that automatically shrinks but doesn’t stretch beyond its floating
           size.
    
       floating
           Can be moved/resized freely. Although it doesn’t use any tiling space, it is still
           part of the window tree.
    
       fullscreen
           Fills its monitor rectangle and has no borders.
    NODE FLAGS
hidden
Is hidden and doesn’t occupy any tiling space.

       sticky
           Stays in the focused desktop of its monitor.
    
       private
           Tries to keep the same tiling position/size.
    
       locked
           Ignores the node --close message.
    
       marked
           Is marked (useful for deferred actions). A marked node becomes unmarked after being
           sent on a preselected node.
    
       urgent
           Has its urgency hint set. This flag is set externally.
    STACKING LAYERS
There’s three stacking layers: BELOW, NORMAL and ABOVE.

       In each layer, the windows are orderered as follows: tiled & pseudo-tiled < floating <
       fullscreen.
    RECEPTACLES
A leaf node that doesn’t hold any window is called a receptacle. When a node is
inserted on a receptacle in automatic mode, it will replace the receptacle. A
receptacle can be inserted on a node, preselected and killed. Receptacles can therefore
be used to build a tree whose leaves are receptacles. Using the appropriate rules, one
can then send windows on the leaves of this tree. This feature is used in
examples/receptacles to store and recreate layouts.

DOMAINS
Node
General Syntax

           node [NODE_SEL] COMMANDS
    
           If NODE_SEL is omitted, focused is assumed.
    
       Commands
    
           -f, --focus [NODE_SEL]
               Focus the selected or given node.
    
           -a, --activate [NODE_SEL]
               Activate the selected or given node.
    
           -d, --to-desktop DESKTOP_SEL [--follow]
               Send the selected node to the given desktop. If --follow is passed, the focused
               node will stay focused.
    
           -m, --to-monitor MONITOR_SEL [--follow]
               Send the selected node to the given monitor. If --follow is passed, the focused
               node will stay focused.
    
           -n, --to-node NODE_SEL [--follow]
               Send the selected node on the given node. If --follow is passed, the focused
               node will stay focused.
    
           -s, --swap NODE_SEL [--follow]
               Swap the selected node with the given node. If --follow is passed, the focused
               node will stay focused.
    
           -p, --presel-dir [~]DIR|cancel
               Preselect the splitting area of the selected node (or cancel the preselection).
               If ~ is prepended to DIR and the current preselection direction matches DIR,
               then the argument is interpreted as cancel. A node with a preselected area is
               said to be in "manual insertion mode".
    
           -o, --presel-ratio RATIO
               Set the splitting ratio of the preselection area.
    
           -v, --move dx dy
               Move the selected window by dx pixels horizontally and dy pixels vertically.
    
           -z, --resize top|left|bottom|right|top_left|top_right|bottom_right|bottom_left dx
           dy
               Resize the selected window by moving the given handle by dx pixels horizontally
               and dy pixels vertically.
    
           -y, --type CYCLE_DIR|horizontal|vertical
               Set or cycle the splitting type of the selected node.
    
           -r, --ratio RATIO|(+|-)(PIXELS|FRACTION)
               Set the splitting ratio of the selected node (0 < RATIO < 1).
    
           -R, --rotate 90|270|180
               Rotate the tree rooted at the selected node.
    
           -F, --flip horizontal|vertical
               Flip the tree rooted at selected node.
    
           -E, --equalize
               Reset the split ratios of the tree rooted at the selected node to their default
               value.
    
           -B, --balance
               Adjust the split ratios of the tree rooted at the selected node so that all
               windows occupy the same area.
    
           -C, --circulate forward|backward
               Circulate the windows of the tree rooted at the selected node.
    
           -t, --state ~|[~]STATE
               Set the state of the selected window. If ~ is present and the current state
               matches STATE, then the argument is interpreted as its last state. If the
               argument is just ~ with STATE omitted, then the state of the selected window is
               set to its last state.
    
           -g, --flag hidden|sticky|private|locked|marked[=on|off]
               Set or toggle the given flag for the selected node.
    
           -l, --layer below|normal|above
               Set the stacking layer of the selected window.
    
           -i, --insert-receptacle
               Insert a receptacle node at the selected node.
    
           -c, --close
               Close the windows rooted at the selected node.
    
           -k, --kill
               Kill the windows rooted at the selected node.
    Desktop
General Syntax

           desktop [DESKTOP_SEL] COMMANDS
    
           If DESKTOP_SEL is omitted, focused is assumed.
    
       COMMANDS
    
           -f, --focus [DESKTOP_SEL]
               Focus the selected or given desktop.
    
           -a, --activate [DESKTOP_SEL]
               Activate the selected or given desktop.
    
           -m, --to-monitor MONITOR_SEL [--follow]
               Send the selected desktop to the given monitor. If --follow is passed, the
               focused desktop will stay focused.
    
           -s, --swap DESKTOP_SEL [--follow]
               Swap the selected desktop with the given desktop. If --follow is passed, the
               focused desktop will stay focused.
    
           -l, --layout CYCLE_DIR|monocle|tiled
               Set or cycle the layout of the selected desktop.
    
           -n, --rename <new_name>
               Rename the selected desktop.
    
           -b, --bubble CYCLE_DIR
               Bubble the selected desktop in the given direction.
    
           -r, --remove
               Remove the selected desktop.
    Monitor
General Syntax

           monitor [MONITOR_SEL] COMMANDS
    
           If MONITOR_SEL is omitted, focused is assumed.
    
       Commands
    
           -f, --focus [MONITOR_SEL]
               Focus the selected or given monitor.
    
           -s, --swap MONITOR_SEL
               Swap the selected monitor with the given monitor.
    
           -a, --add-desktops <name>...
               Create desktops with the given names in the selected monitor.
    
           -o, --reorder-desktops <name>...
               Reorder the desktops of the selected monitor to match the given order.
    
           -d, --reset-desktops <name>...
               Rename, add or remove desktops depending on whether the number of given names
               is equal, superior or inferior to the number of existing desktops.
    
           -g, --rectangle WxH+X+Y
               Set the rectangle of the selected monitor.
    
           -n, --rename <new_name>
               Rename the selected monitor.
    
           -r, --remove
               Remove the selected monitor.
    Query
General Syntax

           query COMMANDS [OPTIONS]
    
       Commands
    
           The optional selectors are references.
    
           -N, --nodes [NODE_SEL]
               List the IDs of the matching nodes.
    
           -D, --desktops [DESKTOP_SEL]
               List the IDs (or names) of the matching desktops.
    
           -M, --monitors [MONITOR_SEL]
               List the IDs (or names) of the matching monitors.
    
           -T, --tree
               Print a JSON representation of the matching item.
    
       Options
    
           -m,--monitor [MONITOR_SEL|MONITOR_MODIFIERS], -d,--desktop
           [DESKTOP_SEL|DESKTOP_MODIFIERS], -n,--node [NODE_SEL|NODE_MODIFIERS]
               Constrain matches to the selected monitors, desktops or nodes.
    
           --names
               Print names instead of IDs. Can only be used with -M and -D.
    Wm
General Syntax

           wm COMMANDS
    
       Commands
    
           -d, --dump-state
               Dump the current world state on standard output.
    
           -l, --load-state <file_path>
               Load a world state from the given file. The path must be absolute.
    
           -a, --add-monitor <name> WxH+X+Y
               Add a monitor for the given name and rectangle.
    
           -O, --reorder-monitors <name>...
               Reorder the list of monitors to match the given order.
    
           -o, --adopt-orphans
               Manage all the unmanaged windows remaining from a previous session.
    
           -h, --record-history on|off
               Enable or disable the recording of node focus history.
    
           -g, --get-status
               Print the current status information.
    
           -r, --restart
               Restart the window manager
    Rule
General Syntax

           rule COMMANDS
    
       Commands
    
           -a, --add (<class_name>|*)[:(<instance_name>|*)[:(<name>|*)]] [-o|--one-shot]
           [monitor=MONITOR_SEL|desktop=DESKTOP_SEL|node=NODE_SEL] [state=STATE] [layer=LAYER]
           [honor_size_hints=(true|false|tiled|floating)] [split_dir=DIR] [split_ratio=RATIO]
           [(hidden|sticky|private|locked|marked|center|follow|manage|focus|border)=(on|off)]
           [rectangle=WxH+X+Y]
               Create a new rule. Colons in the instance_name, class_name, or name fields can
               be escaped with a backslash.
    
           -r, --remove ^<n>|head|tail|(<class_name>|*)[:(<instance_name>|*)[:(<name>|*)]]...
               Remove the given rules.
    
           -l, --list
               List the rules.
    Config
General Syntax

           config [-m MONITOR_SEL|-d DESKTOP_SEL|-n NODE_SEL] <setting> [<value>]
               Get or set the value of <setting>.
    Subscribe
General Syntax

           subscribe [OPTIONS] (all|report|monitor|desktop|node|...)*
               Continuously print events. See the EVENTS section for the description of each
               event.
    
       Options
    
           -f, --fifo
               Print a path to a FIFO from which events can be read and return.
    
           -c, --count COUNT
               Stop the corresponding bspc process after having received COUNT events.
    Quit
General Syntax

           quit [<status>]
               Quit with an optional exit status.
    EXIT CODES
If the server can’t handle a message, bspc will return with a non-zero exit code.

SETTINGS
Colors are in the form \#RRGGBB, booleans are true, on, false or off.

       All the boolean settings are false by default unless stated otherwise.
    Global Settings
normal_border_color
Color of the border of an unfocused window.

       active_border_color
           Color of the border of a focused window of an unfocused monitor.
    
       focused_border_color
           Color of the border of a focused window of a focused monitor.
    
       presel_feedback_color
           Color of the node --presel-{dir,ratio} message feedback area.
    
       split_ratio
           Default split ratio.
    
       status_prefix
           Prefix prepended to each of the status lines.
    
       external_rules_command
           Absolute path to the command used to retrieve rule consequences. The command will
           receive the following arguments: window ID, class name, instance name, and
           intermediate consequences. The output of that command must have the following
           format: key1=value1 key2=value2 ...  (the valid key/value pairs are given in the
           description of the rule command).
    
       automatic_scheme
           The insertion scheme used when the insertion point is in automatic mode. Accept the
           following values: longest_side, alternate, spiral.
    
       initial_polarity
           On which child should a new window be attached when adding a window on a single
           window tree in automatic mode. Accept the following values: first_child,
           second_child.
    
       directional_focus_tightness
           The tightness of the algorithm used to decide whether a window is on the DIR side
           of another window. Accept the following values: high, low.
    
       removal_adjustment
           Adjust the brother when unlinking a node from the tree in accordance with the
           automatic insertion scheme.
    
       presel_feedback
           Draw the preselection feedback area. Defaults to true.
    
       borderless_monocle
           Remove borders of tiled windows for the monocle desktop layout.
    
       gapless_monocle
           Remove gaps of tiled windows for the monocle desktop layout.
    
       top_monocle_padding, right_monocle_padding, bottom_monocle_padding,
       left_monocle_padding
           Padding space added at the sides of the screen for the monocle desktop layout.
    
       single_monocle
           Set the desktop layout to monocle if there’s only one tiled window in the tree.
    
       borderless_singleton
           Remove borders of the only window on the only monitor regardless its layout.
    
       pointer_motion_interval
           The minimum interval, in milliseconds, between two motion notify events.
    
       pointer_modifier
           Keyboard modifier used for moving or resizing windows. Accept the following values:
           shift, control, lock, mod1, mod2, mod3, mod4, mod5.
    
       pointer_action1, pointer_action2, pointer_action3
           Action performed when pressing pointer_modifier + button<n>. Accept the following
           values: move, resize_side, resize_corner, focus, none.
    
       click_to_focus
           Button used for focusing a window (or a monitor). The possible values are: button1,
           button2, button3, any, none. Defaults to button1.
    
       swallow_first_click
           Don’t replay the click that makes a window focused if click_to_focus isn’t none.
    
       focus_follows_pointer
           Focus the window under the pointer.
    
       pointer_follows_focus
           When focusing a window, put the pointer at its center.
    
       pointer_follows_monitor
           When focusing a monitor, put the pointer at its center.
    
       mapping_events_count
           Handle the next mapping_events_count mapping notify events. A negative value
           implies that every event needs to be handled.
    
       ignore_ewmh_focus
           Ignore EWMH focus requests coming from applications.
    
       ignore_ewmh_fullscreen
           Block the fullscreen state transitions that originate from an EWMH request. The
           possible values are: none, all, or a comma separated list of the following values:
           enter, exit.
    
       ignore_ewmh_struts
           Ignore strut hinting from clients requesting to reserve space (i.e. task bars).
    
       center_pseudo_tiled
           Center pseudo tiled windows into their tiling rectangles. Defaults to true.
    
       remove_disabled_monitors
           Consider disabled monitors as disconnected.
    
       remove_unplugged_monitors
           Remove unplugged monitors.
    
       merge_overlapping_monitors
           Merge overlapping monitors (the bigger remains).
    Monitor and Desktop Settings
top_padding, right_padding, bottom_padding, left_padding
Padding space added at the sides of the monitor or desktop.

Desktop Settings
window_gap
Size of the gap that separates windows.

Node Settings
border_width
Window border width.

       honor_size_hints
           If true, apply ICCCM window size hints to all windows. If floating, only apply them
           to floating and pseudo tiled windows. If tiled, only apply them to tiled windows.
           If false, don’t apply them. Defaults to false.
    POINTER BINDINGS
click_to_focus
Focus the window (or the monitor) under the pointer if the value isn’t none.

       pointer_modifier + button1
           Move the window under the pointer.
    
       pointer_modifier + button2
           Resize the window under the pointer by dragging the nearest side.
    
       pointer_modifier + button3
           Resize the window under the pointer by dragging the nearest corner.
    
       The behavior of pointer_modifier + button<n> can be modified through the
       pointer_action<n> setting.
    EVENTS
report
See the next section for the description of the format.

       monitor_add <monitor_id> <monitor_name> <monitor_geometry>
           A monitor is added.
    
       monitor_rename <monitor_id> <old_name> <new_name>
           A monitor is renamed.
    
       monitor_remove <monitor_id>
           A monitor is removed.
    
       monitor_swap <src_monitor_id> <dst_monitor_id>
           A monitor is swapped.
    
       monitor_focus <monitor_id>
           A monitor is focused.
    
       monitor_geometry <monitor_id> <monitor_geometry>
           The geometry of a monitor changed.
    
       desktop_add <monitor_id> <desktop_id> <desktop_name>
           A desktop is added.
    
       desktop_rename <monitor_id> <desktop_id> <old_name> <new_name>
           A desktop is renamed.
    
       desktop_remove <monitor_id> <desktop_id>
           A desktop is removed.
    
       desktop_swap <src_monitor_id> <src_desktop_id> <dst_monitor_id> <dst_desktop_id>
           A desktop is swapped.
    
       desktop_transfer <src_monitor_id> <src_desktop_id> <dst_monitor_id>
           A desktop is transferred.
    
       desktop_focus <monitor_id> <desktop_id>
           A desktop is focused.
    
       desktop_activate <monitor_id> <desktop_id>
           A desktop is activated.
    
       desktop_layout <monitor_id> <desktop_id> tiled|monocle
           The layout of a desktop changed.
    
       node_add <monitor_id> <desktop_id> <ip_id> <node_id>
           A node is added.
    
       node_remove <monitor_id> <desktop_id> <node_id>
           A node is removed.
    
       node_swap <src_monitor_id> <src_desktop_id> <src_node_id> <dst_monitor_id>
       <dst_desktop_id> <dst_node_id>
           A node is swapped.
    
       node_transfer <src_monitor_id> <src_desktop_id> <src_node_id> <dst_monitor_id>
       <dst_desktop_id> <dst_node_id>
           A node is transferred.
    
       node_focus <monitor_id> <desktop_id> <node_id>
           A node is focused.
    
       node_activate <monitor_id> <desktop_id> <node_id>
           A node is activated.
    
       node_presel <monitor_id> <desktop_id> <node_id> (dir DIR|ratio RATIO|cancel)
           A node is preselected.
    
       node_stack <node_id_1> below|above <node_id_2>
           A node is stacked below or above another node.
    
       node_geometry <monitor_id> <desktop_id> <node_id> <node_geometry>
           The geometry of a window changed.
    
       node_state <monitor_id> <desktop_id> <node_id> tiled|pseudo_tiled|floating|fullscreen
       on|off
           The state of a window changed.
    
       node_flag <monitor_id> <desktop_id> <node_id>
       hidden|sticky|private|locked|marked|urgent on|off
           One of the flags of a node changed.
    
       node_layer <monitor_id> <desktop_id> <node_id> below|normal|above
           The layer of a window changed.
    
       pointer_action <monitor_id> <desktop_id> <node_id> move|resize_corner|resize_side
       begin|end
           A pointer action occurred.
    
       Please note that bspwm initializes monitors before it reads messages on its socket,
       therefore the initial monitor events can’t be received.
    REPORT FORMAT
Each report event message is composed of items separated by colons.

       Each item has the form <type><value> where <type> is the first character of the item.
    
       M<monitor_name>
           Focused monitor.
    
       m<monitor_name>
           Unfocused monitor.
    
       O<desktop_name>
           Occupied focused desktop.
    
       o<desktop_name>
           Occupied unfocused desktop.
    
       F<desktop_name>
           Free focused desktop.
    
       f<desktop_name>
           Free unfocused desktop.
    
       U<desktop_name>
           Urgent focused desktop.
    
       u<desktop_name>
           Urgent unfocused desktop.
    
       L(T|M)
           Layout of the focused desktop of a monitor.
    
       T(T|P|F|=|@)
           State of the focused node of a focused desktop.
    
       G(S?P?L?M?)
           Active flags of the focused node of a focused desktop.
    ENVIRONMENT VARIABLES
BSPWM_SOCKET
The path of the socket used for the communication between bspc and bspwm. If it
isn’t defined, then the following path is used:
/tmp/bspwm<host_name>_<display_number>_<screen_number>-socket.

CONTRIBUTORS
•   Steven Allen : steven at stebalien.com

       •   Thomas Adam <thomas at xteddy.org>
    
       •   Ivan Kanakarakis <ivan.kanak at gmail.com>
    AUTHOR
Bastien Dejean : nihilhill at gmail.com 

Bspwm 0.9.12                              10/08/2025                                  BSPWM(1)

Deeply explain all off these in seperate subheadings how to what to and with example  each of the feature in bspwm in 40000 words

Perfect! Now I have comprehensive information. Let me create a detailed, in-depth guide about bspwm. I'll now generate the comprehensive answer.

# A Comprehensive In-Depth Guide to BSPWM: The Binary Space Partitioning Window Manager

This comprehensive guide provides an exhaustive explanation of BSPWM (Binary Space Partitioning Window Manager), covering all major features, domains, commands, and configurations with detailed examples and practical applications. BSPWM is a sophisticated tiling window manager that organizes windows using binary space partitioning algorithms, represented as leaves in a full binary tree structure. Unlike traditional window managers that require mouse interactions for window management, BSPWM operates through a client-server socket architecture where the `bspc` command client communicates with the `bspwm` server daemon, allowing keyboard-driven, highly customizable window management.



## I. Introduction to BSPWM Architecture and Core Concepts

### A. What is Binary Space Partitioning (BSP)?

**Binary Space Partitioning** is a mathematical technique originally developed for three-dimensional computer graphics that recursively divides a rectangular space by splitting it along axis-aligned lines. In the context of BSPWM, this algorithm is applied to monitor screen rectangles, creating a hierarchical binary tree structure where each internal node represents a split (either horizontal or vertical) and each leaf node represents a window or empty receptacle.

The fundamental principle behind BSP in BSPWM is elegant: every node in the tree either has exactly zero children (leaf nodes) or exactly two children (internal nodes). Each internal node manages a rectangular area and decides how to split it between its two child nodes using two parameters: the split type (horizontal or vertical) and the split ratio (a value between 0 and 1, where 0.5 means equal distribution). This mathematical approach ensures that every pixel on the screen is always accounted for and that the layout remains perfectly tiled without gaps or overlaps, except for borders and user-configured gaps.

**Example of BSP in Action:**
When you start with a single window filling the entire monitor (node 1), and open a second window (node 2), BSPWM creates an internal node (node b) that splits the screen according to the automatic insertion scheme. If the split is vertical with a ratio of 0.5, the first window gets 50% of the width on the left and the second window gets 50% on the right. Opening a third window (node 3) causes the tree to reorganize, with one of the existing nodes being displaced to create space for the new window.

### B. Architecture: Client-Server Socket Model

BSPWM operates using a three-component architecture where the window manager itself (the `bspwm` daemon) is separate from its control program (`bspc`) and keyboard binding manager (`sxhkd`). This separation of concerns provides several advantages:

```
1. **bspwm daemon**: Runs in the background listening on a Unix domain socket (`/tmp/bspwm<hostname>_<display>_<screen>-socket`), managing all window operations and desktop state
```

2. **bspc client**: Sends commands to the bspwm socket to manage windows, desktops, monitors, and query the window tree
3. **sxhkd daemon**: A separate process that intercepts X11 keyboard and pointer events and translates them into `bspc` commands

This architecture means BSPWM doesn't handle any keyboard input directly—it only responds to socket messages. This makes BSPWM completely independent of any particular key binding system and allows the same window manager to be used with different hotkey daemons.

### C. Fundamental Concepts: Monitors, Desktops, and Nodes

**Monitors** are physical displays or virtual output devices (detected via RandR or Xinerama protocols) that act as containers holding rectangular regions. Each monitor displays one focused desktop at a time. When you have multiple monitors, each can show a different desktop.

**Desktops** (also called workspaces or virtual desktops) are independent tree structures that contain windows. A monitor shows one desktop at a time, but can switch between them. Each desktop has its own layout (tiled or monocle) and contains its own hierarchy of windows.

**Nodes** are the abstract tree nodes—either internal nodes (which define splits) or leaf nodes (which hold windows or are empty receptacles). Every window in BSPWM is represented as a leaf node in the binary tree. Empty leaf nodes without windows are called **receptacles** and serve as placeholders for future window insertions.

**Example Monitor/Desktop/Node Hierarchy:**

```
Monitor 1 (1920x1080)
├── Desktop 1 (focused) - containing Tree T1
│   ├── Node 1 (window: Firefox)
│   ├── Node 2 (window: Terminal)
│   └── Node 3 (receptacle)
├── Desktop 2 - containing Tree T2
│   └── Node 1 (window: Editor)
└── Desktop 3 - containing Tree T3
    └── (empty)

Monitor 2 (1920x1080)
└── Desktop 4 - containing Tree T4
    └── Node 1 (window: Media Player)
```




## II. Selectors: The Foundation of BSPWM Control

Selectors are the core mechanism through which you target windows, desktops, or monitors for operations. Understanding selectors is absolutely essential because almost every `bspc` command uses them to specify which object to operate on.

### A. Selector Syntax and Structure

The general syntax for a selector is:

```
[REFERENCE#]DESCRIPTOR(.MODIFIER)*
```

This breaks down as:

- **REFERENCE**: An optional reference node/desktop/monitor (defaults to focused)
- **DESCRIPTOR**: The primary selection method (direction, cycling, ID, etc.)
- **MODIFIERS**: Zero or more filtering conditions, each can be negated with `!`

For example: `node east.!floating.window` means "select the node to the east that is not floating and has a window in it".

### B. Node Selectors in Depth

**Node Selectors** target specific windows/nodes in the tree. The complete syntax is:

```
NODE_SEL := [NODE_SEL#](DIR|CYCLE_DIR|PATH|any|first_ancestor|last|newest|older|newer|focused|pointed|biggest|smallest|<node_id>)[.[!]focused][.[!]active][.[!]automatic][.[!]local][.[!]leaf][.[!]window][.[!]STATE][.[!]FLAG][.[!]LAYER][.[!]SPLIT_TYPE][.[!]same_class][.[!]descendant_of][.[!]ancestor_of]
```

**Node Selector Descriptors** (primary selection methods):


| Descriptor | Behavior | Example |
| :-- | :-- | :-- |
| `north\|south\|east\|west` (DIR) | Select window spatially in given direction | `bspc node west -f` |
| `next\|prev` (CYCLE_DIR) | Cycle within tree traversal order | `bspc node next -f` |
| `any` | Select first matching node | `bspc node any.floating -f` |
| `focused` | Currently focused node | `bspc node focused -t floating` |
| `pointed` | Node under mouse pointer | `bspc node pointed -c` |
| `biggest\|smallest` | Largest or smallest leaf | `bspc node biggest -s focused` |
| `last` | Previously focused node | `bspc node last -f` |
| `newest\|older\|newer` | History-based selection | `bspc node newest -n @^1` |
| `<node_id>` | Specific node by ID | `bspc node 0x00800001 -t floating` |
| `@PATH` | Path-based navigation | `bspc node @/first/brother -f` |

**Node Selector Modifiers** (filtering conditions):

```
# States
.tiled              - Only tiled windows
.pseudo_tiled       - Only pseudo-tiled windows
.floating           - Only floating windows
.fullscreen         - Only fullscreen windows

# Flags
.hidden             - Hidden (not displayed)
.sticky             - Stays on focused desktop
.private            - Resists movement/resize
.locked             - Can't be closed
.marked             - Custom marking
.urgent             - Has urgency hint

# Layers
.below              - Below normal layer
.normal             - Normal stacking layer
.above              - Above normal layer

# Tree structure
.leaf               - Leaf nodes only
.window             - Nodes with windows only
.same_class         - Windows of same application class
.descendant_of      - All descendants
.ancestor_of        - All ancestors
```

**Practical Node Selector Examples:**

```bash
# Focus the window to the east
bspc node east -f

# Focus the largest window that's not fullscreen
bspc node biggest.!fullscreen -f

# Select first tiled window on current desktop
bspc node .local.tiled -f

# Move focused window to the second child position
bspc node -n @/second -f

# Select all windows of Firefox class
bspc node .same_class -t floating
```


### C. Desktop Selectors

**Desktop Selectors** target specific workspaces and have the syntax:

```
DESKTOP_SEL := [DESKTOP_SEL#](CYCLE_DIR|any|last|newest|older|newer|[MONITOR_SEL:](focused|^<n>)|<desktop_id>|<desktop_name>)[.[!]focused][.[!]active][.[!]occupied][.[!]urgent][.[!]local][.[!]LAYOUT][.[!]user_LAYOUT]
```

**Desktop Selector Descriptors:**


| Descriptor | Behavior |
| :-- | :-- |
| `next\|prev` | Cycle to next/previous desktop |
| `focused` | Currently active desktop |
| `^<n>` | Nth desktop (^1 = first) |
| `^1\|^2\|^3...` | Numbered desktops |
| `<desktop_name>` | Desktop by name |
| `occupied` | Desktops with windows |
| `urgent` | Desktops with urgent windows |

**Practical Desktop Selector Examples:**

```bash
# Focus the second desktop
bspc desktop ^2 -f

# Send window to desktop named "work" and follow
bspc node -d work --follow

# Focus an occupied desktop
bspc desktop .occupied -f

# List all unoccupied desktops
bspc query -D -d .!occupied
```


### D. Monitor Selectors

**Monitor Selectors** target specific physical displays:

```
MONITOR_SEL := [MONITOR_SEL#](DIR|CYCLE_DIR|any|last|newest|older|newer|focused|pointed|primary|^<n>|<monitor_id>|<monitor_name>)[.[!]focused][.[!]occupied]
```

**Practical Monitor Selector Examples:**

```bash
# Focus the monitor to the right
bspc monitor east -f

# Focus the primary monitor
bspc monitor primary -f

# List all monitors
bspc query -M --names

# Send desktop to second monitor
bspc desktop -m ^2 --follow
```


### E. Path Selectors: Advanced Tree Navigation

Path selectors allow you to navigate the tree structure directly using spatial relationships:

```
PATH := @[DESKTOP_SEL:][[/]JUMP](/JUMP)*
JUMP := first|1|second|2|brother|parent|DIR
```

**Path Navigation Examples:**

```bash
# Focus first child of focused node
bspc node @/first -f

# Focus the second child
bspc node @/second -f

# Focus parent of focused node
bspc node @/parent -f

# Focus brother (sibling) of focused node
bspc node @/brother -f

# Navigate using directions within path
bspc node @/first/east -f

# Cross-desktop path navigation
bspc node @^2:/first -f  # First child of desktop 2
```




## III. Window States: How BSPWM Manages Window Modes

BSPWM provides four distinct window states that determine how windows are positioned and rendered. Understanding these states is crucial for effective window management.

### A. Tiled State

**Tiled** windows are fully managed by the BSPWM tiling algorithm. Their size and position are entirely determined by their position in the binary tree and the tree's split ratios. Windows in this state cannot overlap, and the entire monitor space is divided among them.

**Characteristics of Tiled Windows:**

- Respects and fills the rectangular region allocated by the tree
- Cannot overlap other tiled or pseudo-tiled windows
- Automatically resizes when the tree structure changes
- Follows the configured border width and gaps
- Cannot be freely moved or resized by the user (only through tree operations)

**Example Configuration:**

```bash
# Set focused window to tiled
bspc node focused -t tiled

# Set all windows of Firefox to tiled
bspc node .same_class -t tiled

# Create rule for specific application
bspc rule -a libroffice -t tiled
```


### B. Pseudo-Tiled State

**Pseudo-tiled** (also called sticky floating or "forced") windows are a hybrid between tiled and floating. They are positioned and sized according to the tree structure like tiled windows, but they automatically shrink to respect the application's size hints (if enabled) and don't stretch beyond their requested size.

**Characteristics of Pseudo-Tiled Windows:**

- Positioned like tiled windows (respects tree allocation)
- Can shrink to respect window size hints
- Won't stretch beyond application's preferred size
- Useful for dialog windows, floating applications that you want managed by the tree
- Still participate in the binary tree layout

**Example Use Cases:**

```bash
# Set to pseudo-tiled (good for dialogs)
bspc node focused -t pseudo_tiled

# Rule for GIMP tools palette (prefers fixed size)
bspc rule -a Gimp -t pseudo_tiled

# This window takes its tree space but respects its size hints
```


### C. Floating State

**Floating** windows are completely freed from the tiling tree structure. They can be positioned and resized freely anywhere on the monitor, overlapping other windows. However, floating windows are still part of the tree structure (just not as visible participants) and still have a position in the tree hierarchy.

**Characteristics of Floating Windows:**

- Can be moved and resized freely with keyboard or mouse
- Can overlap other windows
- Still counted in the tree but don't participate in tiling
- Can be moved to different desktops
- Useful for temporary windows, dialogs, notifications

**Example Operations:**

```bash
# Set focused window to floating
bspc node focused -t floating

# Move floating window by pixels
bspc node focused -v -20 0    # Move left 20 pixels
bspc node focused -v 0 20     # Move down 20 pixels

# Resize floating window
bspc node focused -z bottom 0 20  # Expand bottom side

# Focus floating windows only
bspc node .floating -f
```


### D. Fullscreen State

**Fullscreen** windows fill the entire monitor rectangle with no borders or gaps. Fullscreen windows are always on top (in the NORMAL stacking layer by default) and completely hide other windows beneath them. Important: BSPWM uses different stacking layers which affect fullscreen behavior.

**Characteristics of Fullscreen Windows:**

- Fills entire monitor dimensions
- No borders, gaps, or padding applied
- Appears above all other windows
- Can be overridden by ABOVE-layer windows
- Useful for games, presentations, full-screen applications

**Example Operations:**

```bash
# Toggle current window fullscreen
bspc node focused -t fullscreen

# Set specific application to fullscreen
bspc rule -a mpv -t fullscreen

# Make fullscreen window exit
bspc node focused -t tiled  # Return to tiled

# Focus under fullscreen window (requires switching states)
bspc node focused.fullscreen -t floating
```


### E. State Toggling and Dynamic Switching

BSPWM supports elegant state toggling using the tilde `~` character, which switches between the current state and the previous state:

```bash
# Toggle between current and previous state
bspc node focused -t ~

# Toggle to specific state (if currently in that state, revert)
bspc node focused -t ~floating  # If floating, goes back; if not, becomes floating

# Create sxhkd binding for cycling through states
super + {t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating}
    
# Or with toggling
super + shift + {t,s,f}
    bspc node -t {~tiled,~pseudo_tiled,~floating}
```




## IV. Node Flags: Behavioral Modifiers for Windows

Node flags are boolean properties that modify how BSPWM treats a particular window beyond just its state. These flags enable powerful window management patterns and advanced configurations.

### A. Hidden Flag

**Hidden** windows are completely removed from the layout and hidden from view. They don't occupy any tiling space and can be toggled to reappear later.

**Usage Examples:**

```bash
# Hide focused window
bspc node focused -g hidden

# Show all hidden windows on current desktop
bspc node .hidden -g hidden=off

# Toggle hidden state
bspc node focused -g hidden=toggle

# Script to create scratchpad functionality
#!/bin/bash
if bspc node focused -g | grep hidden; then
    bspc node focused -g hidden=off -f
else
    bspc node focused -g hidden=on
fi
```


### B. Sticky Flag

**Sticky** windows automatically follow the focused desktop of their monitor. This means if you switch desktops on a monitor, the sticky window moves with you, remaining visible on every desktop you visit.

**Use Cases:**

- Always-visible status windows
- Quick-reference applications
- Floating notes or widgets
- Monitor-persistent applications

**Configuration:**

```bash
# Make focused window sticky
bspc node focused -g sticky

# Rule for always-visible terminal
bspc rule -a sticky_terminal -g sticky=on -t floating

# Create sticky note/dashboard window
st -n sticky_widget &  # Custom class
bspc rule -a sticky_widget -g sticky=on state=floating

# Sxhkd binding
super + shift + s
    bspc node focused -g sticky
```


### C. Private Flag

**Private** windows attempt to maintain their tiling position and size even when other windows are added or removed from the tree. This prevents automatic resizing when the tree structure changes.

**Behavior:**

- Resists repositioning when tree changes
- Useful for preventing windows from being moved around
- Maintains size hints more strictly

**Example:**

```bash
# Make a media player private so it stays in place
bspc node id:0x00800001 -g private

# Rule for video players
bspc rule -a vlc -g private=on

# Practical use: video window that shouldn't move when opening other windows
```


### D. Locked Flag

**Locked** windows ignore the `node -c` (close) command, preventing accidental closure through keyboard shortcuts. They can only be closed using `node -k` (kill) or through the application's own UI.

**Usage:**

```bash
# Lock focused window against closure
bspc node focused -g locked

# Lock specific application
bspc rule -a firefox -g locked=on

# Toggle lock state
bspc node focused -g locked=toggle

# In sxhkd, you might have:
super + shift + l
    bspc node focused -g locked
    
# And then close uses -c which is ignored for locked nodes
super + shift + w
    bspc node focused -c  # Silently fails for locked windows
```


### E. Marked Flag

**Marked** nodes are flagged with an arbitrary marker that serves as a building block for custom scripting. A marked node automatically becomes unmarked after being sent to a preselected node.

**Key Characteristics:**

- Completely arbitrary—doesn't change behavior by itself
- Used for deferred operations (mark now, operate later)
- Automatically unmarks after preselection
- Enables powerful multi-step workflows

**Practical Examples:**

```bash
# Mark a node for later operation
bspc node focused -g marked

# Move all marked nodes to preselected location
bspc node newest.marked.local -n newest.!automatic.local

# Sxhkd workflow example:
# Step 1: Mark windows
super + m
    bspc node focused -g marked

# Step 2: Navigate and preselect area
super + p; {h,j,k,l}
    bspc node -p {west,south,north,east}

# Step 3: Move marked nodes to preselection
super + y
    bspc node newest.marked.local -n newest.!automatic.local

# Practical workflow: mark multiple windows, define their layout with
# preselections, then move them all at once
```


### F. Urgent Flag

**Urgent** windows have their urgency hint set externally (usually by applications requesting attention—like chat notifications). This flag indicates that a window requires user attention.

**BSPWM Behavior with Urgent:**

- Desktop containing urgent window can be highlighted differently
- Can be used to style borders in unique colors
- Can drive notifications to user

**Configuration:**

```bash
# Style urgent windows differently
bspc config urgent_border_color "#FF0000"

# Focus urgent windows
bspc node .urgent -f

# Clear urgency (focus the window)
bspc node focused -f
```


### G. Managing Multiple Flags Simultaneously

Flags can be combined and managed together:

```bash
# Set multiple flags at once
bspc node focused -g hidden=on -g sticky=on -g locked=on

# Toggle specific flag
bspc node focused -g marked=toggle

# Create comprehensive window configuration
bspc rule -a myapp \
    -g hidden=off \
    -g sticky=on \
    -g private=on \
    state=floating
```




## V. Stacking Layers: Window Z-Order Management

BSPWM implements three stacking layers that determine the Z-order (depth) in which windows are rendered. This provides fine control over which windows appear on top.

### A. Layer Overview

```
ABOVE ────────── (highest Z-order)
  ├── Fullscreen > Floating > Tiled
NORMAL ────────── (default)
  ├── Fullscreen > Floating > Tiled
BELOW ────────── (lowest Z-order)
  ├── Fullscreen > Floating > Tiled
```

Within each layer, windows follow a consistent drawing order: fullscreen windows appear above floating windows, which appear above tiled/pseudo-tiled windows.

### B. Setting and Managing Layers

**The ABOVE Layer:**
Used for permanently visible windows that should never be hidden:

```bash
# Make window appear above everything
bspc node focused -l above

# Typical use: notification windows, always-on-top utilities
bspc rule -a notification -l above

# Floating panel that should stay visible
```

**The NORMAL Layer:**
The default layer for most windows:

```bash
# Return to normal layer
bspc node focused -l normal

# Most windows stay here
```

**The BELOW Layer:**
For background windows:

```bash
# Place window below everything
bspc node focused -l below

# Use case: background wallpaper application, monitor display
bspc rule -a wallpaper_app -l below
```


### C. Practical Layer Usage

```bash
# Create an always-on-top floating window
bspc node floating_app -l above

# Desktop setup: wallpaper below, then tiled windows, then floating on top
bspc node .wallpaper -l below
bspc node .tiled -l normal
bspc node .floating -l above

# Advanced: fullscreen overlay
# When you want a fullscreen window but something above it
bspc rule -a overlay_app -t floating -l above -r

# Solve fullscreen problems: put floating windows on above layer
bspc config external_rules_command ~/.config/bspwm/external_rules

# In external_rules script:
[[ "$(xdotool getactivewindow getwindowname)" == *"Overlay"* ]] && \
    echo "layer=above"
```




## VI. Receptacles: The Empty Node Concept

**Receptacles** are leaf nodes in the tree that don't hold any window. They serve as placeholders for future window placements and enable powerful tree-building patterns.

### A. Understanding Receptacles

When you insert a receptacle at a node's location, it becomes an empty leaf in the tree. When a new window is opened and directed to a receptacle (in automatic mode), the receptacle is replaced by the window node. This allows you to pre-build the entire tree structure before opening windows.

**Receptacle Key Properties:**

- Leaf nodes without windows
- Can be inserted explicitly at any location
- Automatically replaced when window inserted in automatic mode
- Can be preselected and killed manually
- Useful for creating fixed layouts


### B. Creating and Managing Receptacles

```bash
# Insert receptacle at focused node
bspc node focused -i

# Insert receptacle on specific location
bspc node -n @/first -i

# Kill a receptacle
bspc node receptacle_id -k

# Query all receptacles on desktop
bspc query -N -n .leaf.!window

# List receptacle IDs
bspc query -N -n .receptacle
```


### C. Receptacle-Based Layout System

Receptacles enable a sophisticated layout system using dump/load:

```bash
#!/bin/bash
# Build layout with receptacles
bspc desktop ^1 -l monocle

# Create tree structure using receptacles
bspc node -i           # Receptacle 1
bspc node -i           # Receptacle 2
bspc node -i           # Receptacle 3

# Now when you open windows, they fill these receptacles in order
# Window 1 goes to receptacle 1
# Window 2 goes to receptacle 2
# Window 3 goes to receptacle 3

# Dump this state
bspc wm -d > ~/.config/bspwm/layouts/my_layout.json

# Later, restore with:
bspc wm -l ~/.config/bspwm/layouts/my_layout.json
```


### D. Receptacle vs. Preselection Comparison

| Aspect | Receptacle | Preselection |
| :-- | :-- | :-- |
| Visual Feedback | None | Colored preview area |
| Automatic Replacement | Yes, in auto mode | No, requires manual action |
| Multiple Instances | Yes, can have many | Yes, one per node |
| Persistence | Until window inserted | Canceled on split |
| Use Case | Pre-build layouts | Interactive splitting |

```bash
# Using receptacles for fixed layout
bspc node -i; bspc node -i; bspc node -i  # 3 receptacles
# Open windows - they fill the receptacles

# Using preselection for interactive layout
bspc node -p west      # Next window splits to west
# Open window - it appears to the west
```




## VII. Insertion Modes: Automatic vs. Manual Window Placement

BSPWM provides two insertion modes that determine how new windows are positioned in the tree: automatic and manual modes.

### A. Automatic Insertion Mode (Default)

In **automatic** mode, new windows are inserted according to one of three schemes that automatically determine where they should go:

**How Automatic Mode Works:**
When in automatic insertion mode at node N, a new window takes the space of N, and N is displaced into a new branch. The tree automatically reorganizes to accommodate the new window.

**The Three Automatic Schemes:**

**1. Spiral Scheme (Default: "alternate")**
The spiral scheme creates a rotating pattern where windows are arranged in a spiral from outside inward (or inside outward, depending on `initial_polarity`). This is the most common and visually appealing default:

```bash
# Enable spiral (alternate) scheme
bspc config automatic_scheme alternate

# How it works visually:
# With 4 windows:
#   ┌─────────────────┐
#   │       1         │
#   │  ┌────────┐     │
#   │  │   2    │  4  │
#   │  │ ┌──┐   │     │
#   │  │ │3 │   │     │
#   │  │ └──┘   │     │
#   │  └────────┘     │
#   └─────────────────┘

# Sxhkd binding to switch schemes
super + shift + a
    bspc config automatic_scheme alternate && \
    notify-send "Spiral mode"
```

**2. Longest Side Scheme**
The longest_side scheme always splits the longest side of the available rectangle, creating a more balanced layout:

```bash
# Enable longest side scheme
bspc config automatic_scheme longest_side

# How it works: always splits the longest rectangle edge
# Tends to create more balanced, square-ish windows

# Practical comparison:
# Spiral creates:     │ Longest_side creates:
#    │ │ │ │ │         │  │  │
#    ├─┼─┼─┼─┤         ├──┼──┤
#    │ │ │ │ │         │  │  │
#    ├─┴─┴─┴─┤         └──┴──┘

super + shift + l
    bspc config automatic_scheme longest_side && \
    notify-send "Longest side mode"
```

**3. Alternate Scheme**
Alternates between horizontal and vertical splits in a consistent pattern:

```bash
# Note: Recent versions use "alternate" for spiral
# Check documentation for your version
bspc config automatic_scheme spiral  # May be labeled "spiral" in newer versions
```


### B. Manual Insertion Mode (Preselection)

In **manual** mode, you explicitly specify where the next window should be placed using preselection. This allows precise control over window placement:

**How Manual Mode Works:**

1. Select a node
2. Preselect a direction (north, south, east, west)
3. When a new window opens, it automatically appears in the preselected direction
4. The preselection is consumed and automatic mode resumes
```bash
# Enable preselection on focused node
bspc node -p west       # Next window splits to the west (left)
bspc node -p north      # Next window splits to the north (top)
bspc node -p south      # Next window splits to the south (bottom)
bspc node -p east       # Next window splits to the east (right)

# Cancel preselection
bspc node -p cancel

# Adjust preselection ratio (default 0.5)
bspc node -o 0.3        # Next window gets 30%, current gets 70%
bspc node -o 0.7        # Next window gets 70%, current gets 30%

# Example workflow:
# 1. Preselect split direction
bspc node -p west
# 2. Open new window (e.g., Alt+Enter in sxhkd)
# 3. New window appears to the west
# 4. Preselection is consumed, back to automatic mode
```


### C. Configuration Options for Insertion

```bash
# Set automatic scheme
bspc config automatic_scheme {alternate|longest_side|spiral}

# Set initial polarity (where to attach on single-node tree)
bspc config initial_polarity first_child   # Attach as first child
bspc config initial_polarity second_child  # Attach as second child

# Set directional focus tightness (affects directional selection)
bspc config directional_focus_tightness high   # Strict direction matching
bspc config directional_focus_tightness low    # Loose direction matching

# Enable/disable removal adjustment
bspc config removal_adjustment true   # Adjust tree when removing windows
```


### D. Practical Insertion Mode Usage

```bash
# Sxhkd complete example:
# Focus node and cycle insertion schemes
super + shift + {a,l,s}
    bspc config automatic_scheme {alternate,longest_side,spiral} && \
    notify-send "Scheme: $(bspc config automatic_scheme)"

# Preselection workflow
super + ctrl + {h,j,k,l}
    bspc node -p {west,south,north,east}

# Clear preselection
super + ctrl + space
    bspc node -p cancel

# Adjust preselection ratio
super + {minus,equal}
    RATIO=$(bspc config split_ratio); \
    bspc config split_ratio $(bc <<< "scale=2; $RATIO - 0.05") || \
    bspc config split_ratio $(bc <<< "scale=2; $RATIO + 0.05")

# Open window (in preselection, goes to preselected location)
super + Return
    terminal
```




## VIII. Node Operations: Commands for Window Management

The `node` domain contains the most frequently used commands for managing individual windows.

### A. Focus Operations

**Focus** moves keyboard and mouse input to a specific window:

```bash
# Focus specific selector
bspc node SELECTOR -f

# Focus east window
bspc node east -f

# Focus the biggest window
bspc node biggest -f

# Focus floating windows
bspc node .floating -f

# Focus window by ID
bspc node 0x00800001 -f

# Focus and follow (for multi-monitor):
bspc node NODE_SEL -f --follow

# Sxhkd focus example:
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

super + {comma,period}
    bspc node -f {prev,next}.local.!hidden.window

super + bracket{left,right}
    bspc desktop -f {prev,next}.local
```


### B. Activation Operations

**Activate** sets a node as the active node of its desktop (for multi-monitor awareness):

```bash
# Activate focused node
bspc node focused -a

# This differs from focus in multi-monitor setups
# Focus changes which window receives input
# Activate sets which is "most recent" on that desktop
```


### C. Moving Windows Between Desktops

**Send to Desktop** moves a window to a different workspace:

```bash
# Send to specific desktop
bspc node -d DESKTOP_SEL

# Send to desktop 2
bspc node -d ^2

# Send to named desktop
bspc node -d "work"

# Send and follow (keep focus on window)
bspc node -d ^2 --follow

# Sxhkd example:
super + shift + {1-9,0}
    bspc node -d '^{1-9,10}' --follow

# Move to next desktop
super + shift + bracket{left,right}
    bspc node -d {prev,next}.local --follow
```


### D. Moving Windows Between Monitors

**Send to Monitor** moves windows to different physical displays:

```bash
# Send to specific monitor
bspc node -m MONITOR_SEL

# Send to east monitor
bspc node -m east --follow

# Send to primary monitor
bspc node -m primary --follow

# Sxhkd example:
super + shift + m
    bspc node -m next --follow

# Send to specific monitor and follow
super + shift + {Left,Right}
    bspc node -m {west,east} --follow
```


### E. Swapping Nodes

**Swap** exchanges the position of two nodes in the tree:

```bash
# Swap with another node
bspc node -s NODE_SEL

# Swap with brother (sibling)
bspc node -s @brother

# Swap with previous window
bspc node -s prev

# Swap and follow (focus stays on window being swapped)
bspc node -s NODE_SEL --follow

# Practical example:
# Swap with biggest window
super + g
    bspc node -s biggest.window

# Swap with last focused
super + shift + g
    bspc node -s last
```


### F. Resizing and Adjusting Windows

**Move** repositions floating windows:

```bash
# Move floating window by pixels
bspc node -v dx dy

# Move left 20 pixels
bspc node -v -20 0

# Move down 20 pixels
bspc node -v 0 20

# Sxhkd example for floating window movement:
super + shift + {Left,Down,Up,Right}
    bspc node -v {-20 0,0 20,0 -20,20 0}
```

**Resize** changes window dimensions:

```bash
# Resize by moving a specific edge
bspc node -z EDGE dx dy

# EDGE options: top, left, bottom, right, top_left, top_right, bottom_left, bottom_right

# Expand bottom edge by 20 pixels
bspc node -z bottom 0 20

# Contract left edge by 20 pixels
bspc node -z left 20 0

# Sxhkd resizing mode:
super + alt + {h,j,k,l}
    bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

# Or contract:
super + alt + shift + {h,j,k,l}
    bspc node -z {left 20 0,bottom 0 -20,top 0 20,right -20 0}

# Continuous resize mode (like vim motions):
super + r : {h,j,k,l}
    bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}
```


### G. Managing Split Ratios (tiled windows)

**Ratio** adjusts how space is divided between windows:

```bash
# Set specific ratio (0 < ratio < 1)
bspc node -r RATIO

# Set to 0.6 (60/40 split)
bspc node -r 0.6

# Adjust by relative amount
bspc node -r +0.05    # Increase ratio by 5%
bspc node -r -0.05    # Decrease ratio by 5%

# Change by pixel amount
bspc node -r +10px    # Increase by 10 pixels
bspc node -r -10px    # Decrease by 10 pixels

# Sxhkd ratio adjustment:
super + shift + {minus,equal}
    bspc node -r {-0.05,+0.05}

# Balance splits
super + b
    bspc node @focused -B
```


### H. Tree Manipulation Operations

**Rotate** spins the tree structure at a node:

```bash
# Rotate 90 degrees clockwise
bspc node -R 90

# Rotate 180 degrees
bspc node -R 180

# Rotate 270 degrees (or -90)
bspc node -R 270

# Sxhkd binding:
super + {comma,period}
    bspc node @focused -R {90,-90}
```

**Flip** mirrors the tree:

```bash
# Flip horizontally (left-right)
bspc node -F horizontal

# Flip vertically (top-bottom)
bspc node -F vertical

# Sxhkd bindings:
super + {slash,backslash}
    bspc node @focused -F {horizontal,vertical}
```

**Circulate** rotates windows in the tree:

```bash
# Circulate windows forward (clockwise)
bspc node -C forward

# Circulate windows backward (counter-clockwise)
bspc node -C backward

# Practical use: rotate order of windows without changing layout
super + shift + c
    bspc node -C forward

# This moves: window1 -> window2 -> window3 -> window1
```

**Equalize** resets all split ratios to default (0.5):

```bash
# Reset all ratios in tree
bspc node -E

# Useful when tree becomes unbalanced
super + shift + e
    bspc node -E

# Undo manual resizing
```

**Balance** adjusts ratios so all leaves occupy equal area:

```bash
# Balance the tree
bspc node -B

# After resizing, rebalance:
super + shift + b
    bspc node -B

# Different from equalize:
# Equalize: all ratios = 0.5
# Balance: adjusts all ratios so leaf areas are equal
```


### I. Window State and Flag Management

**State** changes window state:

```bash
# Set specific state
bspc node -t STATE

# States: tiled, pseudo_tiled, floating, fullscreen

# Toggle to previous state
bspc node -t ~

# Toggle to specific state (if already in state, revert)
bspc node -t ~floating  # floating <-> previous state

# Sxhkd state cycling:
super + {t,shift+t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating,fullscreen}

# Individual toggles:
super + shift + t
    bspc node -t floating
super + shift + f
    bspc node -t fullscreen
```

**Flag** manages node flags:

```bash
# Set flag on/off
bspc node -g FLAG={on|off}

# Toggle flag
bspc node -g FLAG

# Flags: hidden, sticky, private, locked, marked, urgent

# Examples:
bspc node -g hidden=on       # Hide window
bspc node -g locked=toggle   # Toggle lock
bspc node -g marked          # Mark (toggle)

# Sxhkd bindings:
super + ctrl + {m,x,y,z}
    bspc node -g {marked,locked,sticky,private}

# Sticky window:
super + shift + s
    bspc node -g sticky=toggle
```

**Layer** sets stacking layer:

```bash
# Set layer
bspc node -l LAYER

# Layers: below, normal, above

bspc node -l above    # Always on top
bspc node -l normal   # Standard (default)
bspc node -l below    # Background

# Sxhkd binding:
super + {comma,period}
    bspc node -l {below,above}
```


### J. Closing and Killing Windows

**Close** sends a close signal to the window:

```bash
# Close focused window (gracefully)
bspc node -c

# Close window by selector
bspc node NODE_SEL -c

# Sxhkd:
super + shift + w
    bspc node -c
```

**Kill** forcefully terminates the window:

```bash
# Kill focused window
bspc node -k

# Kill all windows on desktop
bspc node -k

# Use kill when close doesn't work
super + shift + q
    bspc node -k
```


### K. Inserting and Managing Receptacles

**Insert Receptacle** creates empty placeholder nodes:

```bash
# Insert receptacle at focused node location
bspc node -i

# Build layout with receptacles before opening windows
bspc node -i; bspc node -i; bspc node -i

# Then open windows and they fill receptacles

# Script to create grid layout:
#!/bin/bash
# Create 2x2 grid of receptacles
bspc desktop -l tiled
for i in {1..3}; do bspc node -i; done
```




## IX. Desktop Operations: Workspace Management

The `desktop` domain manages virtual desktops/workspaces.

### A. Creating and Managing Desktops

**Add Desktops** to a monitor:

```bash
# Create desktops
bspc monitor -a NAMES

# Add desktops to current monitor
bspc monitor -a I II III IV V

# Multiple desktops at once:
bspc monitor -a I II III IV V VI VII VIII IX X

# Sxhkd configuration (in bspwmrc):
bspc monitor -a 1 2 3 4 5 6 7 8 9 10
```

**Reset Desktops** to match a list:

```bash
# Replace desktop list
bspc monitor -d NAMES

# Removes, adds, or renames desktops to match list
bspc monitor -d one two three
```

**Rename Desktop:**

```bash
# Rename desktop
bspc desktop -n NAME

# Example:
bspc desktop -n "work"
bspc desktop ^2 -n "media"
```

**Remove Desktop:**

```bash
# Remove empty desktop
bspc desktop -r

# Remove desktop 5
bspc desktop ^5 -r
```


### B. Focusing Desktops

**Focus Desktop:**

```bash
# Focus specific desktop
bspc desktop SELECTOR -f

# Focus desktop 2
bspc desktop ^2 -f

# Focus next desktop
bspc desktop next -f

# Focus previous
bspc desktop prev -f

# Focus by name
bspc desktop "work" -f

# Sxhkd bindings:
super + {1-9,0}
    bspc desktop -f '^{1-9,10}'

# Cycle through desktops
super + {Right,Left}
    bspc desktop -f {next,prev}.local
```


### C. Desktop Layout

**Layout** switches between tiling modes:

```bash
# Switch layout
bspc desktop -l LAYOUT

# Layouts: tiled (default), monocle

# Set to monocle (one window fullscreen)
bspc desktop -l monocle

# Cycle layout
bspc desktop -l next

# Sxhkd:
super + m
    bspc desktop -l next

# Or explicit:
super + {t,f}
    bspc desktop -l {tiled,monocle}
```

**Monocle Layout Options:**

```bash
# Borderless monocle (no window borders)
bspc config borderless_monocle true

# Gapless monocle (no gaps)
bspc config gapless_monocle true

# Padding in monocle
bspc config top_monocle_padding 20
bspc config left_monocle_padding 20
bspc config right_monocle_padding 20
bspc config bottom_monocle_padding 20

# Single window uses monocle
bspc config single_monocle true
```


### D. Moving and Swapping Desktops

**Send Node to Desktop:**

```bash
# Send focused window to desktop
bspc node -d DESKTOP_SEL

# Send to desktop 2
bspc node -d ^2

# Send and follow (keep focus on window)
bspc node -d ^2 --follow

# Send to named desktop
bspc node -d "work" --follow

# Sxhkd example:
super + shift + {1-9,0}
    bspc node -d '^{1-9,10}' --follow
```

**Swap Desktops:**

```bash
# Swap position of two desktops
bspc desktop -s DESKTOP_SEL

# Swap focused with desktop 2
bspc desktop -s ^2

# Sxhkd:
super + shift + {Left,Right}
    bspc desktop -s {prev,next}.local
```

**Move Desktop to Monitor:**

```bash
# Move desktop to different monitor
bspc desktop -m MONITOR_SEL

# Move desktop to east monitor
bspc desktop -m east

# Sxhkd:
super + alt + {Left,Right}
    bspc desktop -m {west,east}
```


### E. Bubble and Reorder Desktops

**Bubble** moves desktop in focus order:

```bash
# Bubble left (move before previous)
bspc desktop -b prev

# Bubble right (move after next)
bspc desktop -b next

# Sxhkd:
super + shift + {comma,period}
    bspc desktop -b {prev,next}
```




## X. Monitor Operations: Multi-Display Management

Monitor operations manage physical displays or virtual outputs.

### A. Monitor Configuration

**Add Monitor:**

```bash
# Add virtual monitor
bspc wm -a NAME WIDTHxHEIGHT+X+Y

# Example: virtual 1920x1080 at position 0,0
bspc wm -a virtual_monitor 1920x1080+0+0
```

**Reset Monitor Rectangle:**

```bash
# Set monitor dimensions and position
bspc monitor -g WIDTHxHEIGHT+X+Y

# Set to 1920x1080 at 0,0
bspc monitor -g 1920x1080+0+0

# Get current monitor dimensions
bspc query -M -m focused -T | jq '.rectangle'
```

**Rename Monitor:**

```bash
# Rename monitor
bspc monitor -n NAME

# Rename primary monitor
bspc monitor primary -n "Main"
```

**Remove Monitor:**

```bash
# Remove monitor
bspc monitor -r

# Must be empty or desktops transferred first
```


### B. Focus and Swap

**Focus Monitor:**

```bash
# Focus specific monitor
bspc monitor SELECTOR -f

# Focus east monitor
bspc monitor east -f

# Focus primary
bspc monitor primary -f

# Focus next monitor
bspc monitor next -f

# Sxhkd:
super + shift + {h,j,k,l}
    bspc monitor -f {west,south,north,east}
```

**Swap Monitors:**

```bash
# Swap positions of monitors
bspc monitor -s MONITOR_SEL

# Swap with east monitor
bspc monitor -s east

# Sxhkd:
super + shift + m
    bspc monitor -s next
```


### C. Monitor-Specific Configuration

```bash
# Set padding per monitor
bspc config -m MONITOR top_padding 20

# Set for specific monitor
bspc config -m primary top_padding 20

# Window gap per monitor
bspc config -m eDP1 window_gap 10
```




## XI. Query Operations: Inspecting the Window Tree

The `query` domain allows you to inspect and extract information about the window tree.

### A. Querying Nodes

**Query Nodes:**

```bash
# List node IDs matching selector
bspc query -N -n SELECTOR

# Get focused window ID
bspc query -N -n focused

# Get all tiled windows
bspc query -N -n .tiled

# Get all windows on current desktop
bspc query -N -d focused

# Use in scripts:
FOCUSED_ID=$(bspc query -N -n focused)
echo "Focused window ID: $FOCUSED_ID"

# Get multiple nodes:
for node in $(bspc query -N -n .floating); do
    echo "Floating window: $node"
done
```


### B. Querying Desktops

**Query Desktops:**

```bash
# List desktop names
bspc query -D --names

# List desktop IDs
bspc query -D

# Get occupied desktops
bspc query -D -d .occupied

# Get empty desktops
bspc query -D -d .!occupied

# Get urgent desktops
bspc query -D -d .urgent

# Script example:
EMPTY_DESK=$(bspc query -D -d .!occupied | head -1)
echo "First empty desktop: $EMPTY_DESK"
```


### C. Querying Monitors

**Query Monitors:**

```bash
# List monitor names
bspc query -M --names

# List monitor IDs
bspc query -M

# Get focused monitor
bspc query -M -m focused --names

# Get all monitors
for monitor in $(bspc query -M); do
    echo "Monitor: $monitor"
done
```


### D. Tree Visualization

**Query Tree:**

```bash
# Print entire tree as JSON
bspc query -T

# Query desktop tree
bspc query -T -d focused

# Query node subtree
bspc query -T -n focused

# Pretty print with jq:
bspc query -T -d focused | jq .

# Extract specific info:
bspc query -T | jq '..[].id'  # All node IDs
bspc query -T | jq '..[].client.class'  # All window classes
```




## XII. Window Rules: Automatic Window Configuration

Rules apply automatic configurations to windows based on matching criteria.

### A. Built-in Rules

**Basic Rule Syntax:**

```bash
bspc rule -a CLASS[:INSTANCE[:NAME]] [options]

# Options:
# State: state=STATE (tiled|pseudo_tiled|floating|fullscreen)
# Desktop: desktop=DESKTOP_SEL
# Monitor: monitor=MONITOR_SEL
# Flags: hidden=on|off, sticky=on|off, private=on|off, locked=on|off, marked=on|off
# Layer: layer=LAYER (below|normal|above)
# Splitting: split_dir=DIR, split_ratio=RATIO
# Size hints: honor_size_hints=true|false|tiled|floating
# Layout: rectangle=WxH+X+Y (for floating windows)
# Other: center=on|off, follow=on|off, manage=on|off, focus=on|off, border=on|off

# One-shot: -o|--one-shot (applies only once)
```


### B. Creating Rules

**Common Rule Examples:**

```bash
# Firefox always opens on desktop 2
bspc rule -a Firefox desktop=^2

# GIMP tools floating, sticky
bspc rule -a Gimp -t floating -g sticky=on

# Media player fullscreen
bspc rule -a vlc -t fullscreen

# Dialog windows floating
bspc rule -a '*' -t floating -o  # One-shot for unmatched dialogs

# Locked terminal
bspc rule -a Termite -g locked=on

# Application on specific monitor
bspc rule -a Firefox monitor=HDMI-1

# Combination rule
bspc rule -a libreoffice \
    -t pseudo_tiled \
    desktop=^3 \
    -g sticky=on \
    split_ratio=0.7

# Multiple class selectors
bspc rule -a {Firefox,Chrome} desktop=^2
```


### C. Matching Patterns

**Class Matching:**

```bash
# Exact class match
bspc rule -a Firefox state=floating

# Class and instance
bspc rule -a Firefox:Preferences state=floating

# Class, instance, and name
bspc rule -a "VLC:vlc::*" state=floating

# Wildcard patterns
bspc rule -a "*" state=floating -o  # Fallback rule
bspc rule -a "* notification*" state=floating layer=above

# Negation
bspc rule -a Firefox -r  # Remove Firefox rules
```


### D. List and Manage Rules

**Rule Management:**

```bash
# List all rules
bspc rule -l

# List rules numbered
bspc rule -l | cat -n

# Remove rule by pattern
bspc rule -r Firefox

# Remove rule by number
bspc rule -r ^3

# Remove all rules
bspc rule -r '*'

# Remove specific rules
bspc rule -r Firefox Chromium
```


### E. External Rules Command

For more complex rules, use an external script:

```bash
# Configure external rules
bspc config external_rules_command ~/.config/bspwm/external_rules

# Script receives: WID CLASS INSTANCE [INTERMEDIATE_RULES]
# Output format: key1=value1 key2=value2 ...

# Example external_rules script:
#!/bin/bash

wid=$1
class=$2
instance=$3

# Get window geometry with xdotool
geometry=$(xdotool getwindowgeometry "$wid" | awk '/Geometry/ {print $2}')

# Custom logic
if [[ "$class" == "Firefox" ]]; then
    echo "desktop=^2"
elif [[ "$geometry" == "1920x1080"* ]]; then
    # Large window - make floating
    echo "state=floating"
fi

# Can be very complex:
if xdotool getwindowname "$wid" | grep -q "Settings"; then
    echo "state=floating layer=above center=on"
fi

# Make executable:
chmod +x ~/.config/bspwm/external_rules
```




## XIII. Events and Subscriptions: Reactive Programming with BSPWM

BSPWM generates comprehensive events for every state change, enabling real-time monitoring and scripting.

### A. Event Types

**Available Events:**

```
Monitor Events:
- monitor_add <monitor_id> <name> <geometry>
- monitor_remove <monitor_id>
- monitor_swap <src_monitor_id> <dst_monitor_id>
- monitor_focus <monitor_id>
- monitor_geometry <monitor_id> <geometry>
- monitor_rename <monitor_id> <old_name> <new_name>

Desktop Events:
- desktop_add <monitor_id> <desktop_id> <name>
- desktop_remove <monitor_id> <desktop_id>
- desktop_swap <src_monitor_id> <src_desktop_id> <dst_monitor_id> <dst_desktop_id>
- desktop_transfer <src_monitor_id> <src_desktop_id> <dst_monitor_id>
- desktop_focus <monitor_id> <desktop_id>
- desktop_activate <monitor_id> <desktop_id>
- desktop_layout <monitor_id> <desktop_id> <layout>
- desktop_rename <monitor_id> <desktop_id> <old_name> <new_name>

Node Events:
- node_add <monitor_id> <desktop_id> <ip_id> <node_id>
- node_remove <monitor_id> <desktop_id> <node_id>
- node_swap <src_monitor_id> <src_desktop_id> <src_node_id> <dst_monitor_id> <dst_desktop_id> <dst_node_id>
- node_transfer <src_monitor_id> <src_desktop_id> <src_node_id> <dst_monitor_id> <dst_desktop_id> <dst_node_id>
- node_focus <monitor_id> <desktop_id> <node_id>
- node_activate <monitor_id> <desktop_id> <node_id>
- node_presel <monitor_id> <desktop_id> <node_id> (dir DIR|ratio RATIO|cancel)
- node_stack <node_id_1> (below|above) <node_id_2>
- node_geometry <monitor_id> <desktop_id> <node_id> <geometry>
- node_state <monitor_id> <desktop_id> <node_id> <state> (on|off)
- node_flag <monitor_id> <desktop_id> <node_id> <flag> (on|off)
- node_layer <monitor_id> <desktop_id> <node_id> <layer>
- pointer_action <monitor_id> <desktop_id> <node_id> <action> (begin|end)
```


### B. Subscribing to Events

**Basic Subscribe:**

```bash
# Subscribe to specific event type
bspc subscribe node_add | while read -a msg; do
    # msg array contains: monitor_id desktop_id ip_id node_id
    node_id="${msg[^1_4]}"
    echo "New node added: $node_id"
    # Perform action
done

# Subscribe to multiple event types
bspc subscribe node_add node_remove | while read -a msg; do
    event_type="${msg[^1_0]}"
    case "$event_type" in
        node_add)
            echo "Node added"
            ;;
        node_remove)
            echo "Node removed"
            ;;
    esac
done

# Subscribe to all events
bspc subscribe all | while read line; do
    echo "$line"
done
```


### C. Practical Event Scripts

**Example 1: Auto-fullscreen New Windows:**

```bash
#!/bin/bash
# Make every new window fullscreen automatically

bspc subscribe node_add | while read -a msg; do
    monitor_id="${msg[^1_1]}"
    desktop_id="${msg[^1_2]}"
    node_id="${msg[^1_4]}"
    
    # Auto-fullscreen
    bspc node "$node_id" -t fullscreen
done
```

**Example 2: Auto-floating by Window Class:**

```bash
#!/bin/bash
# Make certain applications floating automatically

FLOATING_APPS=("mpv" "VLC" "Zathura")

bspc subscribe node_add | while read -a msg; do
    node_id="${msg[^1_4]}"
    
    # Get window class (requires extra lookup)
    class=$(xdotool getactivewindow getwindowname 2>/dev/null || echo "")
    
    for app in "${FLOATING_APPS[@]}"; do
        if [[ "$class" == *"$app"* ]]; then
            bspc node "$node_id" -t floating
            break
        fi
    done
done
```

**Example 3: Status Bar Integration:**

```bash
#!/bin/bash
# Update polybar on window events

bspc subscribe node_focus desktop_focus | while read -a msg; do
    # Update status bar
    pkill -SIGUSR1 polybar
    
    # Or update custom status
    echo "$(date): $(bspc query -N -n focused)" > /tmp/bspwm_status
done
```

**Example 4: Sticky Window Management:**

```bash
#!/bin/bash
# Automatically make certain windows sticky

bspc subscribe node_add node_focus | while read -a msg; do
    node_id="${msg[^1_4]}"
    
    # Make all floating windows sticky automatically
    state=$(bspc query -N -n "$node_id" -T | jq -r '.client.state')
    
    if [[ "$state" == "floating" ]]; then
        bspc node "$node_id" -g sticky=on
    fi
done
```


### D. Using FIFO for Event Processing

Alternative method using named pipes:

```bash
# Get FIFO path
FIFO_PATH=$(bspc --print-socket-path)

# Create monitoring script
#!/bin/bash
bspc subscribe -f | nc -l localhost 5000 &

# In another terminal, connect:
nc localhost 5000 | while read event; do
    echo "$event"
done
```




## XIV. Configuration Settings: Customizing BSPWM Behavior

BSPWM provides numerous settings to customize appearance and behavior.

### A. Global Settings

**Color Settings:**

```bash
# Set border colors (in hex #RRGGBB)
bspc config normal_border_color "#484848"           # Unfocused, unfocused monitor
bspc config focused_border_color "#1F8999"          # Focused, focused monitor
bspc config active_border_color "#CCCCCC"           # Focused, unfocused monitor
bspc config presel_feedback_color "#7C7C7C"         # Preselection area

# Practical configuration:
bspc config normal_border_color "#3c3836"
bspc config focused_border_color "#d79921"
bspc config active_border_color "#a89984"
bspc config presel_feedback_color "#458588"

# Test colors in terminal:
bspc config normal_border_color "#FF0000"  # Test red
# Changes apply immediately to new windows
```

**Dimension Settings:**

```bash
# Border width
bspc config border_width 2              # 0-10 pixels

# Window gap (space between windows)
bspc config window_gap 10               # Default: 12

# Padding (space from screen edge)
bspc config top_padding 20
bspc config bottom_padding 0
bspc config left_padding 0
bspc config right_padding 0

# Combined padding:
for side in top right bottom left; do
    bspc config "${side}_padding" 20
done

# Per-monitor padding:
bspc config -m HDMI-1 top_padding 30
```

**Insertion and Splitting:**

```bash
# Default split ratio
bspc config split_ratio 0.5            # 0 < ratio < 1

# Automatic insertion scheme
bspc config automatic_scheme alternate  # or longest_side

# Initial attachment direction
bspc config initial_polarity first_child  # or second_child

# Directional focus strictness
bspc config directional_focus_tightness high  # or low

# Removal behavior
bspc config removal_adjustment true
```

**Layout Options:**

```bash
# Monocle-specific settings
bspc config borderless_monocle true     # Remove borders in monocle
bspc config gapless_monocle true        # Remove gaps in monocle

# Monocle padding
bspc config top_monocle_padding 20
bspc config bottom_monocle_padding 0
bspc config left_monocle_padding 0
bspc config right_monocle_padding 0

# Single window behavior
bspc config single_monocle false        # Switch to monocle with 1 window

# Singleton settings
bspc config borderless_singleton true   # Remove border on single window
```


### B. Focus and Click Behavior

```bash
# Focus follows mouse pointer
bspc config focus_follows_pointer true

# Mouse moves to focused window
bspc config pointer_follows_focus false

# Mouse moves when focus changes to different monitor
bspc config pointer_follows_monitor false

# Click button to focus
bspc config click_to_focus button1     # button1, button2, button3, any, none

# Don't replay click that gave focus
bspc config swallow_first_click true
```


### C. Pointer and Mouse Settings

```bash
# Pointer movement interval (milliseconds)
bspc config pointer_motion_interval 10  # Minimum between motion events

# Modifier key for pointer actions (Alt=mod1, Super=mod4, etc.)
bspc config pointer_modifier mod4

# Pointer actions (button 1, 2, 3)
bspc config pointer_action1 move        # Super+button1: move window
bspc config pointer_action2 resize_side # Super+button2: resize side
bspc config pointer_action3 resize_corner  # Super+button3: resize corner

# Possible values: move, resize_side, resize_corner, focus, none

# Example for alternative bindings:
bspc config pointer_modifier mod1       # Alt instead of Super
bspc config pointer_action1 focus       # Just focus, don't move
bspc config pointer_action2 move
bspc config pointer_action3 resize_corner
```


### D. EWMH and Compatibility Settings

```bash
# Ignore EWMH focus requests from applications
bspc config ignore_ewmh_focus true

# Ignore EWMH fullscreen requests
bspc config ignore_ewmh_fullscreen none    # none, all, or: enter,exit

# Ignore window struts (panel reservations)
bspc config ignore_ewmh_struts false

# Example: allow fullscreen but not fullscreen changes
bspc config ignore_ewmh_fullscreen "exit"  # Ignore exit, allow enter

# Pseudo-tiled centering
bspc config center_pseudo_tiled true
```


### E. Monitor Management

```bash
# Remove disabled monitors
bspc config remove_disabled_monitors false

# Remove unplugged monitors
bspc config remove_unplugged_monitors true

# Merge overlapping monitors
bspc config merge_overlapping_monitors true

# Mapping events to handle
bspc config mapping_events_count -1     # -1 = all events
```


### F. History Recording

```bash
# Record focus history
bspc config record_history true

# Use in scripts:
bspc wm -h on    # Enable history recording
bspc node older -f  # Focus previously focused
bspc wm -h off   # Disable history
```


### G. Size Hints

```bash
# Honor window size hints (ICCCM hints)
bspc config honor_size_hints false

# Options:
# false: ignore all hints
# true: apply to all windows
# tiled: apply only to tiled windows
# floating: apply only to floating windows
# floating|tiled: apply to both

# Example:
bspc config honor_size_hints floating   # Dialogs respect requested size
```


### H. Practical Configuration File Example

```bash
#!/bin/bash
# ~/.config/bspwm/bspwmrc

# Monitor setup
bspc monitor -d I II III IV V VI VII VIII IX X

# Appearance
bspc config border_width 2
bspc config window_gap 10
bspc config split_ratio 0.5

bspc config normal_border_color "#3c3836"
bspc config focused_border_color "#d79921"
bspc config active_border_color "#a89984"

# Layout
bspc config borderless_monocle false
bspc config gapless_monocle false

# Behavior
bspc config automatic_scheme alternate
bspc config split_ratio 0.5
bspc config focus_follows_pointer true

# Rules
bspc rule -a Firefox desktop=^2
bspc rule -a Gimp state=floating sticky=on
bspc rule -a mpv state=floating

# Start additional daemons
sxhkd &
polybar main &
```




## XV. Query and World State Management

Advanced commands for inspecting and managing BSPWM's global state.

### A. Dumping and Loading State

**Dump State:**

```bash
# Get current state as JSON
bspc wm -d > state.json

# This includes:
# - Monitor configuration
# - Desktop hierarchy
# - Window tree structure
# - Focus history
# - All settings

# Inspect structure:
bspc wm -d | jq '.' | less
```

**Load State:**

```bash
# Restore state from file
bspc wm -l /path/to/state.json

# Requires absolute path
bspc wm -l ~/.config/bspwm/layouts/default.json

# Used internally for restart:
bspc wm -r  # Dumps state, restarts, loads state
```


### B. Report Format

**Status Messages:**

```bash
# Get current status
bspc wm -g

# Returns format: M<name>:O<name>:F<name>:...

# Report format codes:
# M<name>    - Focused monitor
# m<name>    - Unfocused monitor
# O<name>    - Occupied focused desktop
# o<name>    - Occupied unfocused desktop
# F<name>    - Free focused desktop
# f<name>    - Free unfocused desktop
# U<name>    - Urgent focused desktop
# u<name>    - Urgent unfocused desktop
# L<T|M>     - Layout (T=tiled, M=monocle)
# T<T|P|F|=|@>  - Node state (T=tiled, P=pseudo-tiled, F=floating, ==fullscreen, @=manual)
# G<flags>   - Flags

# Example script to parse:
#!/bin/bash
bspc wm -g | while IFS=: read -ra items; do
    for item in "${items[@]}"; do
        type="${item:0:1}"
        value="${item:1}"
        
        case "$type" in
            M) echo "Focused monitor: $value" ;;
            O) echo "Occupied desktop: $value" ;;
            L) echo "Layout: $value" ;;
        esac
    done
done
```


### C. World State Operations

**Restart BSPWM:**

```bash
# Restart, preserving all state
bspc wm -r

# Equivalent to:
# 1. Dump state
# 2. Kill bspwm
# 3. Restart bspwm
# 4. Load state

# Sxhkd binding:
super + alt + r
    bspc wm -r
```

**Adopt Orphans:**

```bash
# Manage windows from previous session
bspc wm -o

# If X session crashed, windows still exist
# This command brings them under BSPWM management
```




## XVI. Practical Sxhkd Configuration for BSPWM

A complete, real-world sxhkd configuration demonstrating all major bindings.

### A. Complete Sxhkd Configuration Example

```bash
#!/bin/bash
# ~/.config/sxhkd/sxhkdrc

# Application launching
super + Return
    kitty

super + d
    rofi -show drun

super + slash
    sxhkd-help

# WM control
super + alt + {q,r}
    bspc {quit,wm -r}

# Window state
super + {t,shift+t,s,f}
    bspc node -t {tiled,pseudo_tiled,floating,fullscreen}

# Window flags
super + ctrl + {m,x,y,z}
    bspc node -g {marked,locked,sticky,private}

# Focus operations
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

super + {comma,period}
    bspc node -f {prev,next}.local.!hidden.window

super + shift + {h,j,k,l}
    bspc node -s {west,south,north,east}

super + g
    bspc node -s biggest.window

# Desktop navigation
super + {1-9,0}
    bspc desktop -f '^{1-9,10}'

super + bracket{left,right}
    bspc desktop -f {prev,next}.local

super + shift + {1-9,0}
    bspc node -d '^{1-9,10}' --follow

# Preselection (manual splitting)
super + ctrl + {h,j,k,l}
    bspc node -p {west,south,north,east}

super + ctrl + space
    bspc node -p cancel

super + ctrl + {minus,equal}
    bspc config split_ratio $(bspc config split_ratio){-,+}$(echo 0.05)

# Floating window management
super + shift + {Left,Down,Up,Right}
    bspc node -v {-20 0,0 20,0 -20,20 0}

super + alt + {h,j,k,l}
    bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

# Tree manipulation
super + {comma,period}
    bspc node @focused -R {90,-90}

super + {slash,backslash}
    bspc node @focused -F {horizontal,vertical}

super + shift + c
    bspc node -C forward

super + shift + e
    bspc node -E

super + b
    bspc node @focused -B

# Closing windows
super + shift + w
    bspc node -c

super + shift + q
    bspc node -k

# Monitor operations
super + shift + m
    bspc node -m next --follow

super + shift + {Left,Right}
    bspc monitor -f {west,east}

# Layout control
super + m
    bspc desktop -l next

# Scratchpad/hidden window
super + shift + space
    bash ~/.config/bspwm/toggle_scratchpad.sh
```


### B. Advanced Sxhkd Patterns

**Resize Mode:**

```bash
# Enter resize mode with Super+E, then use hjkl
super + e : {h,j,k,l}
    bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

super + e : shift + {h,j,k,l}
    bspc node -z {right -20 0,top 0 20,bottom 0 -20,left 20 0}

# Exit with Escape key (default)
```

**Multi-key Sequences:**

```bash
# Super+P enters preselect mode
super + p : {h,j,k,l}
    bspc node -p {west,south,north,east}

super + p : shift + {h,j,k,l}
    bspc node -p {east,north,south,west}

super + p : c
    bspc node -p cancel

super + p : space
    bspc node -p cancel
```

**Workspace Management:**

```bash
# Super+W enters workspace mode
super + w : {1-9,0}
    bspc desktop -f '^{1-9,10}'

super + w : shift + {1-9,0}
    bspc node -d '^{1-9,10}' --follow

super + w : {Left,Right}
    bspc desktop -f {prev,next}.local
```




## XVII. Real-World Usage Scenarios

### A. Setting Up a Developer Workflow

```bash
#!/bin/bash
# ~/.config/bspwm/setup_dev.sh

# Create desktop setup
bspc desktop ^1 -n code
bspc desktop ^2 -n terminal
bspc desktop ^3 -n browser
bspc desktop ^4 -n reference

# Rule for always-open applications
bspc rule -a Zathura desktop=^4 state=floating

# Code desktop layout (2 columns)
bspc desktop code -l tiled
bspc config -d code window_gap 15

# Terminal desktop (single window)
bspc desktop terminal -l monocle
bspc config -d terminal borderless_monocle true

# Browser floating on specific desktop
bspc rule -a Firefox desktop=^3 --follow

# Always open terminal on terminal desktop
bspc rule -a kitty desktop=^2
```


### B. Multi-Monitor Setup

```bash
#!/bin/bash
# ~/.config/bspwm/bspwmrc

# Get monitor names
PRIMARY=$(xrandr -q | grep " connected" | head -1 | awk '{print $1}')
SECONDARY=$(xrandr -q | grep " connected" | tail -1 | awk '{print $1}')

# Configure primary monitor
bspc monitor "$PRIMARY" -d I II III IV V

# Configure secondary monitor
if [[ -n "$SECONDARY" ]]; then
    bspc monitor "$SECONDARY" -d VI VII VIII IX X
fi

# Set padding per monitor
bspc config -m "$PRIMARY" top_padding 30
bspc config -m "$SECONDARY" top_padding 30

# Move specific apps to secondary
bspc rule -a "VLC" -m "$SECONDARY"
```


### C. Gaming Setup

```bash
#!/bin/bash
# Create gaming desktop

# Separate desktop for games
bspc desktop ^9 -n gaming

# Games go fullscreen on dedicated desktop
bspc rule -a "Steam" desktop=^9 state=floating follow=on
bspc rule -a "wine" desktop=^9 state=fullscreen
bspc rule -a ".*Game.*" state=fullscreen

# Disable window gaps for games
bspc config -d gaming window_gap 0

# Disable borders for games
bspc config -d gaming border_width 0

# Sxhkd binding to switch to gaming desktop
super + alt + g
    bspc desktop -f ^9
```


### D. Productivity Mode

```bash
#!/bin/bash
# Simplified, distraction-free setup

# Single application per desktop
bspc config focus_follows_pointer false
bspc config split_ratio 0.5

# Balanced window gaps
bspc config window_gap 8
bspc config border_width 1

# Monocle for focus
bspc rule -a "Writer" desktop=^1 state=monocle
bspc rule -a "code-oss" desktop=^2 state=monocle
bspc rule -a "Firefox" desktop=^3 state=monocle

# Sxhkd bind to toggle distraction-free mode
super + alt + f
    bash ~/.config/bspwm/toggle_focus_mode.sh
```


## Conclusion

* BSPWM represents a paradigm shift in window management philosophy, moving from direct graphical manipulation to declarative, keyboard-driven tree manipulation. Its strength lies in the separation of concerns: the window manager handles the tree logic, `bspc` provides the control interface, and `sxhkd` handles keyboard bindings. This architecture enables extraordinary flexibility and customization.

* The binary space partitioning algorithm ensures that screen real estate is always optimally allocated, with no wasted space or overlapping windows in tiled mode. The rich selector syntax allows targeting any window configuration with surgical precision. Advanced features like receptacles, preselection, flags, and event subscriptions enable sophisticated window management workflows that would be difficult or impossible in traditional window managers.

* Whether you're a developer seeking efficient workspace management, a power user who wants complete control over your desktop environment, or someone simply tired of mouse-driven window management, BSPWM provides the tools and flexibility to create a window management system tailored exactly to your needs. The learning curve is steep, but the rewards in productivity and customization justify the investment.
