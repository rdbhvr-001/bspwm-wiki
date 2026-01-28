## What is bspwm-wiki?

It is a comprehensive and complete documentation for the Binary Space Partitioning Window Manager (BSPWM).

Bspwm is a tiling window manager which uses binary space partition system. (C students must have known what it means). Unlike other dynamic window managers like dwm and awesomewm, bspwm is what you configure and use. It is possible to do anything with correct integration of configs : bspwm + sxhkd + eww or polybar, you will get a perfectly aligned environment that obeys your thoughts. I named this documentation as bspwm-wiki because it is short and aligns with documentation policy of my interest.

> [!TIP]
> If you do not know what exactly is binary space partitioning, don't worry, the documentation will teach you.

## What exactly is the purpose of The bspwm-wiki?

There has not been a very good documentation about the bspwm window manager in the web. Many beginners in linux stumble upon understanding the workflow and the exact process behind bspwm and struggle by copying someone else's configuration blindly hoping that it works on their system too. Real power users do not copy, they build upon an already existing config or build something entirely from the scratch.

Linux punishes weak fundamentals, not because it is cruel towards you, but because it wants to make you stronger. Understanding about The Binary Space Partitioning Window Manager and Binary Space Partitioning before writing your first bspwm configuration will make you write optimised and good configs better than your copycat versions.

Most of the configurations are using bspwm in an underrated way. For example, people rerely use the `query selectors with flags` to batch-manipulate windows. Here's an on-liner that toggles hidden states across all windows...


```bash
if [ -n "$(bspc query -N -n .hidden)" ]; then flag=off; else flag=on; fi
for i in $(bspc query -N); do 
    bspc node $i --flag hidden=$flag
done

```

The above code matters because it unhides or hides all windows dynamically, no tiling rules or scratchpads, reproducible, scriptable UI hacks...

Another powerful feature is the `selector magic : @`, which looks like : `bspc node -f @parent`, which lets you targe relative nodes - parent, first child, brother, etc...

This documentation aims in providing short scripts, cool tips for speeding up workflows, links to several popular bspwm configurations for quick references, a detailed explanation for the working logic behind bspwm, complete manipulation of the windows, advanced configurations and complex logic structures like : **Open alacritty in the second workspace on every sunday nights in horizontally tiled mode along with ncmpcpp in a seperate alacritty window with so and so split ratio**, when you have the habit of listening to music on sunday nights while working!

Everything you want to know about the bspwm window manager, you will get it here...

## Help People Discover bspwm-wiki

1) Help others find about such an extensive documentation for bspwm by [contributing](./Contributing.md) to this project. You can open up pull requests, do corrections and error analysis for the code blocks in this documentation and expand it by adding specific case examples and linking other bspwm related projects.

2) If you find this documentation really useful to you, you can star this [repository](https://github.com/rdbhvr-001/bspwm-wiki.git) which will increase the ease with which this documentation is found and make it easier for people to reach out to this.

> [!NOTE]
> If you have any issues related to codes provided in this documentation, you can open up an issue in the github repository.

## Github repositories

1) [Official BSPWM Binary : baskerville/bspwm](https://github.com/baskerville/bspwm.git)
2) [Official Documentation Repository : rdbhvr-001/bspwm-wiki](https://github.com/rdbhvr-001/bspwm-wiki.git)

## Pages

1) [Documentation : This Page](https://rdbhvr-001.github.io/bspwm-wiki)
