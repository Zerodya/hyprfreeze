# hyprfreeze
Simple bash script to pause a game process in Hyprland, allowing you to resume it later.
Useful to:
- Pause games during unskippable cutscenes and similar scenarios
- Save system resources (CPU and GPU are free, the process is saved in RAM)

(Also works for any other program; pauses the current active window)
### Usage
Make the script executable and add a bind in your Hyprland config:
```
bind = $mainMod, PAUSE, exec, ~/scripts/hyprfreeze
```
### Known issues
- Mouse input doesn't work inside Xorg/XWayland windows while a Wine/Proton game is paused
  - Workaround: Run the game in gamescope
