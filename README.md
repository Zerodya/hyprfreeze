# hyprfreeze
Hyprfreeze is a utility to suspend a game process (and other programs) in Hyprland.

Useful to:
- Pause single-player games that can't normally be paused (Elden Ring, Baldur's Gate 3, ...)
- Pause cutscenes to read the subtitles or if you suddenly need to leave your desk
- Save system resources (excluding RAM) if you need them for another computer task, or if the game's pause menu uses too many

Note: Running games in [gamescope](https://github.com/ValveSoftware/gamescope) is highly recommended. (See [#1](https://github.com/Zerodya/hyprfreeze/issues/1))

## Installation
### Arch Linux
Hyprfreeze is available in [AUR](https://aur.archlinux.org/packages/hyprfreeze-git).

### Dependencies
- `hyprland` to get the pid of the active window with hyprctl
- `jq` to parse json
- `psmisc` contains 'pstree' which is required to list child processes
  ### Optional
- [`hyprprop`](https://github.com/vilari-mickopf/hyprprop) to get the pid of a window by selecting it with your mouse
  ### Highly Recommended
- [`gamescope`](https://github.com/ValveSoftware/gamescope) fixes mouse input not working in other XWayland windows after pausing a Wine game. It's also the superior way to game in Wayland anyway.

### Manual
Clone this repo and symlink the `hyprfreeze` script to a directory in your `PATH`:
```bash
$ git clone https://github.com/Zerodya/hyprfreeze.git Hyprfreeze
$ ln -s $(pwd)/Hyprfreeze/hyprfreeze $HOME/.local/bin
$ chmod +x Hyprfreeze/hyprfreeze
```

## Usage
Add a bind in your Hyprland config to pause the current active window:
```
# ~/.config/hypr/hyprland.conf

...

# Toggle freeze on active window
bind = , PAUSE, exec, hyprfreeze -a
```
### Available flags
```
-h, --help            show help message

-a, --active          toggle suspend by active window
-p, --pid             toggle suspend by process id
-n, --name            toggle suspend by process name/command
-r, --prop            toggle suspend by clicking on window

-s, --silent          don't send notification
-t, --notif-timeout   notification timeout in milliseconds (default 5000)
--info                show information about the process
--dry-run             doesn't actually suspend/resume a process, useful with --info
--debug               enable debug mode
```
### Examples:
```
# Pause game by process name
hyprfreeze -n eldenring.exe
```
```
# Get info about a process by clicking on its window, without suspending it
hyprfreeze -r --info --dry-run
```
## Disclaimer
There is always the risk, although slim, that an application may crash.

This is intrinsically related to modifying running processes and is not something that Hyprfreeze can prevent.

Please make sure to **save your data** before using hyprfreeze.
