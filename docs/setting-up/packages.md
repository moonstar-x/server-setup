# Packages

We'll install a handful of packages once we're done with the OS installation.

## Screen

We'll also require *screen* which is a wonderful tool capable of improving multitasking on a single shell window with the ability to *background* processes. It also lets you recover shell windows when there's a connection loss.

### Installation

Installing *screen* is as simple as running the following command:

```bash
sudo apt-get install screen
```

### Configuration

Create a `~/.screenrc` file and add the following:

```text
termcapinfo xterm* ti@:te@
```

This enables the mouse wheel inside *screen*.

### Usage

At first it may seem a little complicated and maybe even intimidating to use this tool but once you get used to it you'll realize how useful it really is.

First start up *screen* by typing:

```bash
screen -S <socket name>
```

Here's a table with the keys and actions you can use with *screen*:

| Keys         | Action                    |
|--------------|---------------------------|
| `Ctrl-a` `c` | Creates a new window.     |
| `Ctrl-a` `n` | Switches between windows. |
| `Ctrl-a` `d` | Detaches from screen.     |
| `Ctrl-a` `n` | Switches between screens. |

To reattach to a *screen*:

```bash
screen -r <screen pid>
```

or,

```bash
screen -rd <screen socket name>
```

## The Rest

Install the rest of the packages with the following:

```bash
sudo apt-get install net-tools neofetch nload
```
