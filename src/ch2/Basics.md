
# Binary Tree Structure and Partitioning in BSPWM: A Comprehensive Explanation

## Overview and Core Concepts

BSPWM (Binary Space Partitioning Window Manager) is a **tiling window manager that represents windows as the leaves of a full binary tree**. The fundamental innovation of bspwm is its use of binary space partitioning—a recursive subdivision algorithm that divides two-dimensional screen space into increasingly smaller rectangular regions—to manage window layout and positioning. Unlike traditional stacking window managers or even other tiling managers like i3 that use arbitrary tree structures, bspwm constrains itself to strictly binary trees, meaning **each internal node can have exactly zero or two children**. This architectural decision provides both elegance and flexibility, allowing users to manipulate the window hierarchy with sophisticated tree-based operations while maintaining simplicity in the core data structure.

The genius of bspwm lies in how it translates the abstract concept of binary space partitioning into practical window management. When you open windows in bspwm, each new window becomes a leaf node in the binary tree. Every time a second window is added as a sibling to an existing window, the window manager automatically creates an **internal node (also called a parent node)** that serves as a container for both windows. This internal node represents the split between the two windows—it stores information about whether the split is horizontal or vertical and what ratio the rectangle is divided by. The recursive nature of this process means that adding a third window creates a new internal node with its own children, and this pattern continues indefinitely, building increasingly complex tree structures.

Understanding the binary tree structure in bspwm requires grasping several fundamental concepts from data structures and computer science. **Nodes** are the fundamental units of the tree; each node contains data and pointers to its children. In bspwm's case, leaf nodes hold window references, while internal nodes store splitting information. **Leaf nodes** are nodes with no children and represent actual windows currently open on your screen. **Internal nodes** are nodes with at least one child; in bspwm, internal nodes always have exactly two children due to the binary tree constraint. **Parent nodes** have child nodes branching from them, **sibling nodes** share the same parent, and **root nodes** are the topmost nodes from which all other nodes descend.

The hierarchical nature of this structure provides profound practical benefits. Because bspwm maintains a strict binary tree, you can perform complex operations on entire subtrees, treating them as single units. This means you can rotate, flip, balance, or resize multiple windows at once by operating on their common parent node. This is fundamentally different from other tiling managers where window relationships are less formally defined and thus harder to manipulate programmatically.

## The Binary Tree Data Structure Foundation

### Basic Node Structure

At the heart of bspwm's implementation lies a simple but powerful data structure: the binary tree node. In C (the language bspwm is written in), a basic binary tree node looks like this:

```c
typedef struct node_t node_t;

struct node_t {
    int data;
    node_t* left;
    node_t* right;
};
```

However, bspwm's actual node structure is considerably more sophisticated. Based on the source code examined in bspwm documentation, the **actual node structure in bspwm contains**:

- **first_child and second_child pointers**: Instead of "left" and "right," bspwm names these children to reflect the order of window creation (first window to be spawned as first child, second window as second child)
- **Parent pointer**: Links back to the parent node for upward tree traversal
- **Window reference**: For leaf nodes, a pointer to the actual X11 window being managed
- **Rectangle data**: The bounding box (coordinates and dimensions) that the node occupies on screen
- **Split parameters**: For internal nodes, information about whether the split is horizontal or vertical, and the ratio determining how the rectangle is divided between the two children

This enriched node structure enables bspwm to not only maintain the tree hierarchy but also directly manage window geometry without requiring additional layout calculation passes. Every piece of information needed to render the windows is stored within the tree itself.

### Tree Hierarchy in BSPWM's Display System

BSPWM organizes the display hierarchy into multiple levels, each with its own data structure. **At the top level are monitors**, which are represented as a doubly linked list rather than a tree, because there is typically no hierarchical relationship between monitors—they exist as peers. Each monitor structure contains:

- Name and ID
- Physical rectangle (coordinates and dimensions)
- Pointer to desktops contained within it
- Pointers to previous and next monitors (doubly linked list structure)

**Within each monitor are desktops** (also called workspaces), which are also organized as a doubly linked list within each monitor. Each desktop contains:

- Name and ID
- Layout information (tiled or monocle)
- **A pointer to the root of the binary tree** of windows on that desktop
- Window gaps and border width settings
- Pointers to previous and next desktops

**At the bottom of this hierarchy are windows**, which are organized as a binary tree rooted at the desktop level. The tree only exists at the window level; the monitor and desktop layers are linear structures. This three-layer hierarchy (monitors as linked list → desktops as linked list → windows as binary tree) provides the complete organizational framework for bspwm.

## Binary Tree Partitioning Mechanisms

### Fundamental Partitioning Concept

The core principle of binary space partitioning is **recursive subdivision of space into two convex sets using hyperplanes as partitions**. In two-dimensional screen space, these hyperplanes are simply lines—either horizontal or vertical. Each time bspwm inserts a new window, it doesn't just place it anywhere; instead, it recursively subdivides the available rectangle into two smaller rectangles by drawing either a horizontal or vertical line across it.

The **split is characterized by two parameters**:

1. **Split type**: Whether the division is horizontal (line drawn left-to-right) or vertical (line drawn top-to-bottom)
2. **Split ratio**: A floating-point number between 0 and 1 representing where the line is drawn relative to the rectangle's width or height

For example, if a 1920×1080 rectangle is split vertically with a ratio of 0.5, the result is two rectangles: one that is 960 pixels wide (0.5 × 1920) and another that is also 960 pixels wide (0.5 × 1920). If the ratio is 0.35, the first child gets 672 pixels (0.35 × 1920) and the second child gets 1248 pixels.

### Perfect Binary Trees and Full Binary Trees

BSPWM specifically implements what computer scientists call a "**full binary tree**"—a binary tree where every internal node has exactly zero or two children. This is crucial because it means that **you never have a situation where an internal node has only one child**. In theoretical terms, this ensures that the tree structure remains mathematically clean and predictable.

The significance of this constraint cannot be overstated. In other tiling managers, you might have a container with a single window inside, creating unnecessary hierarchy. In bspwm's full binary tree approach, if you have two windows, there's exactly one internal node connecting them. If you add a third window, another internal node is created to organize one of the existing windows with the new window. This property makes tree operations, rotations, and transformations much more straightforward to reason about and implement.

### Window Insertion Points and Automatic Mode Schemes

When a new window is opened in bspwm, it must be inserted somewhere into the tree. By default, the window is inserted at the **focused window** (or focused leaf node). The **insertion mode** determines how the tree structure changes to accommodate the new window. There are two primary insertion modes: **automatic mode** and **manual mode**, with automatic mode being the default.

In **automatic mode**, bspwm uses one of three schemes to determine where and how to split the tree:

#### 1. Alternate Scheme (Default)

The **alternate scheme splits the window based on the split type of the parent node's parent**. This creates a predictable pattern where splits alternate between horizontal and vertical. Here's how it works:

When you open your first window on an empty desktop, it simply fills the desktop rectangle—no splits occur because there's only one window (one leaf node, no internal nodes needed).

When you open a second window, an internal node is created. The **initial polarity** setting determines which child gets which side. By default, the second child receives the split direction specified by the initial polarity. If initial polarity is "second_child" (the default), the new window appears on the right (for vertical split) or bottom (for horizontal split).

When you open a third window, bspwm looks at the split type of the parent node's parent. If the grandparent split is horizontal, the parent splits vertically, and vice versa. This alternating pattern continues recursively, creating a naturally balanced tile layout.

#### 2. Longest Side Scheme

The **longest side scheme examines the dimensions of the focused window's rectangle and makes the split perpendicular to the longest side**. If the window is wider than it is tall, it splits vertically (creating two narrower windows). If taller than wide, it splits horizontally (creating two shorter windows).

This scheme is particularly useful for monitors of unusual aspect ratios or when you want layouts that naturally adapt to the display geometry. For example, on an ultra-wide monitor (like 3440×1440), the longest side scheme would consistently split vertically, creating tall narrow columns rather than short wide rows.

#### 3. Spiral Scheme

The **spiral scheme inserts the new window at the position of the focused window, pushing the existing window deeper into the tree**. This creates a spiral-like tiling pattern. When you add a new window, instead of creating a sibling alongside the focused window, the focused window becomes the child of a new internal node, and the new window becomes the other child.

The name comes from the visual pattern that emerges: windows gradually spiral outward as more are added, similar to a spiral staircase viewed from above. Each new window takes the place of the current focused window, and the previous window is displaced to a specific child position (first_child or second_child) of the new internal node, based on a spiral polarity setting.

### Manual Mode and Preselection

While automatic mode provides sophisticated algorithms for placement, bspwm also offers **manual mode**, which gives users explicit control over where new windows appear. Manual mode is activated through **preselection**—the user specifies where the next window should be split using cardinal directions: north, south, east, or west.

When a user issues a command like `bspc node -p north`, they are preselecting the northern area of the focused window as the insertion point for the next window. This command converts the focused window's insertion mode from automatic to manual. The **preselection creates a visual feedback area** showing where the new window will appear when it's opened.

The **preselection ratio can be adjusted** to control the split distribution. For example, setting the ratio to 0.7 means the preselected area gets 70% of the focused window's rectangle. When the new window opens in the preselected area, that ratio becomes the split ratio of the new internal node.

Preselection is particularly powerful because it allows users to build complex, custom layouts by manually specifying window placements. This transforms bspwm from a purely automatic layout manager into a hybrid that supports both automatic and manual tiling paradigms.

## Internal Tree Operations and Manipulations

### Tree Rotations and Flips

One of the most sophisticated features of bspwm is its support for **tree rotations and flips**—operations that restructure the tree without affecting which windows are visible, only their relative positions and hierarchy.

**A tree rotation changes which nodes are parents and which are children**, effectively restructuring the internal organization of the tree. This is conceptually similar to rotations in self-balancing binary search trees used in algorithms like AVL trees, but with a different purpose. In AVL trees, rotations rebalance the tree for search efficiency. In bspwm, rotations restructure the tiling layout.

When you rotate a subtree, the visual arrangement of windows changes. For example, rotating a subtree 90 degrees clockwise causes windows that were stacked vertically to appear horizontally, or vice versa. The command `bspc node @/2 -R 90` rotates the subtree rooted at the second child of the desktop root by 90 degrees clockwise.

**Tree flips** are simpler operations that mirror the tree structure. A horizontal flip reverses the left-right ordering of children, while a vertical flip reverses the top-bottom ordering. These operations are useful for quickly rearranging window positions without changing their individual sizes or the overall layout structure.

### Equalize and Balance Operations

**The equalize operation resets all split ratios in a subtree to their default values** (typically 0.5, giving each child equal space). This is useful when splits have been manually adjusted to non-uniform ratios and you want to quickly return to equal distribution.

**The balance operation is more sophisticated: it adjusts split ratios so that all leaf nodes (windows) in a subtree occupy approximately equal areas**. This is different from equalization because it calculates what split ratios are needed to achieve equal window sizes, rather than just resetting them. If your tree has uneven depth (some windows deeper in the tree than others), balance will automatically adjust the split ratios to compensate, ensuring all windows get roughly the same screen real estate.

### Window Swapping and Moving

Bspwm provides commands to **swap windows or subtrees in the tree**. Swapping exchanges the positions of two windows or entire subtrees. For example, you can swap the left and right halves of a split, or swap two specific windows regardless of their position in the tree.

**Window moving is different from swapping**: it transplants a window from one position in the tree to another. When a window is moved to a new location, it takes its subtree with it, maintaining any child windows it may have had. This operation requires restructuring the tree to remove the window from its current location and reinsert it elsewhere.

### Resize and Ratio Operations

While rotations and flips restructure the tree itself, **resize and ratio operations adjust the split parameters without changing the tree structure**. You can modify the split ratio of any internal node to give more space to one child or the other.

For example, `bspc node -r 0.3` sets the split ratio of the focused node's parent to 0.3, giving the focused node 30% of its parent's rectangle and the sibling node 70%. You can also make **relative adjustments using increment and decrement operators**: `bspc node -r +0.1` increases the focused node's allocation by 10 percentage points.

## Split Types and Geometric Relationships

### Horizontal and Vertical Splits

**Horizontal splits draw a line left-to-right**, dividing the rectangle into top and bottom sections. The split ratio determines what percentage of the height goes to the first child versus the second child. A horizontal split with ratio 0.6 gives 60% of the height to the first child (top) and 40% to the second child (bottom).

**Vertical splits draw a line top-to-bottom**, dividing the rectangle into left and right sections. A vertical split with ratio 0.4 gives 40% of the width to the first child (left) and 60% to the second child (right).

The recursive nature of these splits is crucial to understanding bspwm's behavior. When you add a fourth window to a tree with three windows, bspwm analyzes the current split orientations and recursively applies the chosen scheme (alternate, longest side, or spiral) to determine both the split type and the location of the new split.

### Directional Focus and Movement

Bspwm implements **directional focus and movement operations** that work by examining the spatial relationships between nodes. When you press a key like "focus west," bspwm doesn't just move through the tree in a fixed order; instead, it calculates which window is in the western direction relative to the currently focused window and moves focus there.

This spatial awareness is computed using the rectangles stored in each node. By comparing the bounding boxes of windows, bspwm can determine which windows are adjacent in which directions, enabling intuitive directional navigation even when the tree structure doesn't directly reflect the spatial layout.

## Leaf Nodes, Receptacles, and Window States

### Leaf Nodes and Their Properties

**A leaf node is any node in the tree with no children**. In bspwm, leaf nodes can be one of two things: either they hold a window reference (a regular window), or they are **receptacles**—empty leaf nodes that don't hold any window.

When you open a window, it becomes a leaf node with a window reference. If the tree already has windows and you add a new one, the focused window (which was a leaf) becomes an internal node with two children: one holding the original window and one being a new leaf (either a receptacle or the new window, depending on whether you've preselected an area).

### Receptacles

**A receptacle is a leaf node that contains no window**. Receptacles are manually inserted into the tree to create placeholder positions where windows can be placed later. They're useful for creating complex predefined layouts without needing any windows to be open initially.

You can insert a receptacle with the command `bspc node -i` (insert receptacle). Once inserted, the receptacle acts like any other leaf node in the tree—it occupies space and participates in the layout. When a new window is opened, if its insertion point is a receptacle, the receptacle is replaced by the new window in-place, maintaining the structure you carefully constructed.

Receptacles are particularly powerful when combined with bspwm's state-saving and state-loading capabilities. You can create a tree structure with receptacles representing where you want windows to go, save this structure, and then use scripting to automatically place windows in the receptacle positions when you restart.

### Window States

In addition to their position in the tree, windows have **states that determine how they interact with the tiling system**:

- **Tiled**: The window's size and position are determined by the tree structure. This is the default state for new windows.
- **Pseudo-tiled**: A tiled window that automatically shrinks to fit its content but doesn't stretch beyond a certain size.
- **Floating**: The window can be moved and resized freely by the user. While not part of the tiling layout, floating windows still exist as nodes in the tree.
- **Fullscreen**: The window fills the entire monitor and has no borders.

The distinction between these states is important because it affects how the tree renders. Tiled windows participate in the recursive subdivision algorithm; floating and fullscreen windows do not.

### Node Flags

Beyond states, nodes can have **multiple flags set simultaneously**:

- **hidden**: The window doesn't occupy tiling space
- **sticky**: The window stays visible on the focused desktop of its monitor
- **private**: The window tries to maintain its tiling position even when other windows are added or removed
- **locked**: The window ignores close commands
- **marked**: Used for deferred actions and batch operations
- **urgent**: Has its urgency hint set (usually by the application)


## Tree Traversal and Query Mechanisms

### Depth-First and Breadth-First Traversal

To work with the binary tree, bspwm must be able to traverse it—visit each node systematically. There are two fundamental approaches to tree traversal that apply to bspwm's tree:

**Depth-first search (DFS) traverses as far down a branch as possible before exploring the next branch**. It's implemented using a stack (either an explicit stack or implicitly via recursion). In bspwm's context, DFS is useful for operations that affect entire subtrees—you explore all descendants of a node before moving to the next branch.

**Breadth-first search (BFS) explores nodes level by level, visiting all nodes at depth N before visiting nodes at depth N+1**. It's implemented using a queue and is useful for operations that need to consider all nodes at the same depth level.

### Traversal Methods: Inorder, Preorder, and Postorder

For binary trees specifically, there are three common **depth-first traversal methods** that differ in when the node itself is processed relative to its children:

**Inorder traversal: Left → Root → Right**. Visit the left subtree, then the node itself, then the right subtree. For a binary search tree, inorder traversal visits nodes in sorted order.

**Preorder traversal: Root → Left → Right**. Visit the node first, then the left subtree, then the right subtree. This is useful when you need to process a node before its descendants (like when printing a tree structure).

**Postorder traversal: Left → Right → Root**. Visit the left subtree, then the right subtree, then the node itself. This is useful when you need to process children before the parent (like when deleting a tree or computing aggregates from the bottom up).

BSPWM's `bspc query` command uses these traversal concepts internally. For example, when you query all windows in a desktop, bspwm performs a complete traversal of that desktop's tree, visiting every leaf node to collect window IDs.

### Selectors and Cyclic Traversal

Bspwm provides a powerful **selector system for targeting nodes**. Selectors combine a descriptor (like `focused`, `pointed`, `biggest`, `smallest`) with optional modifiers (like `.leaf`, `.window`, `.descendant_of`).

For cyclic traversal, bspwm implements a special traversal called **"depth-first in-order traversal"** that's defined by the manual pages but differs from standard inorder traversal. This traversal mode is used by commands like `bspc node -c` to cycle through windows in a deterministic order.

## Practical Rendering and Layout Generation

### How the Tree Becomes a Layout

The binary tree structure alone doesn't directly render windows; instead, the tree provides the instructions for how to partition screen space. **The rendering process is conceptually straightforward**: starting from the root node (the desktop rectangle), recursively subdivide based on the split type and ratio of each internal node until reaching leaf nodes (windows).

For each window, its final rectangle is calculated by tracing the path from the root to that leaf node in the tree, accumulating the subdivisions along the way. For example, if window W is the second child of its parent, and that parent is the first child of its parent, then W's rectangle is calculated as:

1. Start with the desktop rectangle (e.g., 1920×1080)
2. Apply the root's split (e.g., vertical at 0.5, yielding a 960-pixel-wide rectangle for W's branch)
3. Apply the parent's split (e.g., horizontal at 0.6, yielding a 576-pixel-high rectangle for W)

### Layouts Beyond Binary Tiling

While bspwm implements only two built-in layouts (tiled and monocle), its binary tree structure can support complex layouts through external scripting. The **monocle layout displays one window full-screen**, with others hidden but still present in the tree. The **tiled layout is the default binary tree rendering**.

More sophisticated layouts like "tall" (master-stack), "grid," and "spiral" can be created by writing scripts that listen to bspwm events and manipulate window states, positions, and ratios based on custom rules. These scripts might set certain windows to floating state, adjust split ratios programmatically, or create receptacles in specific patterns.

## Advanced Tree Concepts and Optimization

### Tree Balance and Optimization

While bspwm doesn't automatically rebalance its tree for search efficiency (as self-balancing BSTs do), it does provide operations to optimize layouts. **The balance operation ensures that all leaf nodes occupy approximately equal areas**, addressing one common aesthetic concern with deeply-nested trees where some windows get disproportionately small space.

The **tree's depth** (the maximum distance from root to leaf) can affect how users perceive responsiveness when navigating. However, because the tree depth corresponds to visual nesting depth, users typically prefer shallower trees and naturally avoid creating excessively deep nesting.

### Computational Complexity

The binary tree structure provides excellent algorithmic properties. Most operations on bspwm trees are **O(log n) in the number of windows** because the tree is typically reasonably balanced (though not formally balanced). Navigating to a specific window requires at most O(log n) comparisons. Even complex operations like rotating a subtree or applying balance operations are efficient because they operate on subtrees rather than re-rendering the entire layout.

## The Intersection of Theory and Practice

### From Computer Science to Window Management

The genius of bspwm is how it translates established computer science concepts into elegant window management. Binary space partitioning as a rendering algorithm comes from 3D graphics (developed in 1969). Self-balancing trees come from database and algorithm design. Tree rotations and flips come from advanced data structure theory.

By constraining window management to a strict binary tree, bspwm inherits decades of theory and optimization techniques. This means bspwm users, often unknowingly, are using sophisticated algorithmic structures that have been proven optimal for various problems.

### Extensibility Through Tree Manipulation

The binary tree structure also makes bspwm uniquely extensible. Unlike window managers with more complex internal hierarchies, bspwm's simplicity means that **external scripts can manipulate the tree through well-defined bspc commands** without risking corruption or inconsistency. You can write scripts that listen to window events and automatically adjust tree structure, create custom layouts, or build sophisticated multi-monitor coordination systems.

This extensibility is perhaps bspwm's greatest strength: it's not trying to be everything to everyone out of the box. Instead, it provides a clean, mathematically sound foundation (the binary tree) and powerful manipulation tools (bspc), allowing users to build their ideal workflow on top.

## So Far

* The binary tree structure and partitioning system in bspwm represents an elegant fusion of theoretical computer science and practical software engineering. By adopting binary space partitioning as its core organizational principle, bspwm achieves several important properties: **deterministic layout behavior, powerful tree manipulation operations, efficient algorithms, mathematical rigor, and extensibility through scripting**.

* Understanding bspwm requires moving beyond thinking of it as merely a window manager and appreciating it as a direct implementation of computer science data structures and algorithms. The binary tree isn't just an implementation detail—it's the defining characteristic that makes bspwm flexible, powerful, and philosophically different from other tiling managers. Whether you use bspwm with its default automatic schemes or extend it with custom scripts, you're working with a professionally designed data structure that has proven optimal across multiple domains of computer science.

* For users who take time to understand the tree structure, bspwm unlocks capabilities that are difficult or impossible in other window managers: subtree rotations, surgical tree restructuring, mathematically defined layouts, and scriptable tree manipulation. This depth of capability, grounded in solid theory and elegant implementation, is why bspwm continues to appeal to advanced users and why learning its tree structure and partitioning mechanisms is well worth the effort.

