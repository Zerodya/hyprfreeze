# hyprfreeze
Hyprfreeze is a utility to suspend a game process (and other programs) in Hyprland.

Useful to:
- Pause single-player games that can't be paused (like Dark Souls)
- Pause during cutscenes to read the subtitles or when something urgent comes up
- Save system resources (excluding RAM) if you need them for another computer task, or if the game's pause menu uses too many

## Installation
### Arch Linux
Hyprfreeze is available in [AUR](https://aur.archlinux.org/packages/hyprfreeze-git).

### Dependencies
- hyprland (to get the pid of the active window with hyprctl)
- jq (to parse json)
- hyprprop (to get the pid of a window by selecting it with your mouse)

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
bind = $mainMod, PAUSE, exec, hyprfreeze -a
```
### Available flags
```
-h, --help      show help message
-a, --active    pause/resume active window
-p, --pid       pause/resume by process id
-n, --name      pause/resume by process name/command
-r, --prop      pause/resume by clicking on window
--info          show information about the process
--dry-run       doesn't actually pause/resume a process, useful with --info
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

## Known issues
- Pausing Wine/Proton games will cause mouse input to not work inside XWayland windows [#1](https://github.com/Zerodya/hyprfreeze/issues/2)
  - Workaround 1: Run the game in `gamescope`
  - Workaround 2: Pause the game from a terminal `hyprfreeze -n eldenring.exe` 
- Pausing Linux native games (e.g. Minecraft) may cause sound to stop in other apps [#2](https://github.com/Zerodya/hyprfreeze/issues/2)
