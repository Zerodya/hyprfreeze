# hyprfreeze
Simple bash script to pause a game process in Hyprland, allowing you to resume it later.
Useful to:
- Pause games during unskippable cutscenes and similar scenarios
- Save system resources (CPU and GPU are free, the process is saved in RAM)

(Also works for any other program; pauses the current active window)
### Usage
Make the script executable, then in your Hyprland config add a bind to the script e.g. :
```
bind = $mainMod, PAUSE, exec, ~/scripts/hyprfreeze
```
