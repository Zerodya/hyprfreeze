# ❄️ hyprfreeze 

Hyprfreeze is a utility to suspend a game process (and other programs) in Hyprland.

https://github.com/Zerodya/hyprfreeze/assets/73220426/541318e2-441a-485a-91c5-f58d4f65926a

**Useful to:**
- Pause single-player games that can't normally be paused (Elden Ring, Baldur's Gate 3, ...)
- Pause cutscenes to read the subtitles or if you suddenly need to leave your desk
- Save system resources (excluding RAM) if you need them for another computer task, or if the game's pause menu uses too many

> **Note:** This repo is in **maintenance-only** mode as it is feature-complete for my scope. I use this tool in Hyprland everyday and will update it to fix potential bugs or when `hyprctl` changes its syntax.

## Dependencies
- `hyprland` to get the PID of the active window
- `jq` to parse json
- `psmisc` contains 'pstree' which is required to list child processes
### Optional
- [`hyprprop`](https://github.com/vilari-mickopf/hyprprop) to get the pid of a window by selecting it with your mouse

## Installation
### Arch Linux
Hyprfreeze is available in the [AUR](https://aur.archlinux.org/packages/hyprfreeze-git). (Maintained by [Aethar](https://github.com/Aethar01))

### Manual
Clone this repo and symlink the `hyprfreeze` script to a directory in your `PATH`:
```bash
git clone https://github.com/Zerodya/hyprfreeze.git Hyprfreeze
ln -s $(pwd)/Hyprfreeze/hyprfreeze $HOME/.local/bin
```
Then install completions for your shell:

**Bash:**
```bash
sudo ln -s $(pwd)/completions/bash/hyprfreeze /usr/share/bash-completion/completions/
```
**Zsh:**
```bash
sudo ln -s $(pwd)/completions/zsh/_hyprfreeze /usr/share/zsh/site-functions/
```
**Fish:**
```bash
sudo ln -s $(pwd)/completions/fish/hyprfreeze.fish /usr/share/fish/vendor_completions.d/
```
## Usage
Add a bind in your Hyprland config to pause the current active window:
```bash
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
-r, --prop            toggle suspend by clicking on window (hyprprop must be installed)

-s, --silent          don't send notification
-t, --notif-timeout   notification timeout in milliseconds (default 5000)
--dry-run             doesn't actually suspend/resume a process
--debug               enable debug mode
--no-xorg-workaround  skip the XWayland mouse capture workaround (other XWayland windows may become non-interactable)

```
### Example:
```bash
# Pause game by process name
hyprfreeze -n eldenring.exe
```

## Caveats
- System audio may stop when pausing Linux native games (no Proton/Wine) like Minecraft. This is a Pipewire [issue](https://gitlab.freedesktop.org/pipewire/pipewire/-/issues/3509). 
To fix it, run the game with the env variable: `ALSOFT_DRIVERS=pulse`

- Pausing XWayland games (no Gamescope or Wayland proton) may stop the mouse from working in other XWayland apps like Discord. A workaround has been implemented that quickly switches workspaces to release the mouse capture before pausing the window.

## Disclaimer
There is always the risk, although slim, that an application may crash.

This is intrinsically related to modifying running processes and is not something that Hyprfreeze can prevent.

Please make sure to **save your data** before using hyprfreeze.
