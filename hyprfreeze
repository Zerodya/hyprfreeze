#!/usr/bin/env bash

function printHelp() {
	cat <<EOF
Usage: hyprfreeze (-a | -p <pid> | -n <name> | -r) [options]

Utility to suspend a game process (and other programs) in Hyprland.

Options:
  -h, --help            show help message

  -a, --active          toggle suspend by active window
  -p, --pid             toggle suspend by process id
  -n, --name            toggle suspend by process name/command
  -r, --prop            toggle suspend by clicking on window (hyprprop/swayprop must be installed)

  -s, --silent          don't send notification
  -t, --notif-timeout   notification timeout in milliseconds (default 5000)
  --info                show information about the process
  --dry-run             doesn't actually suspend/resume a process, useful with --info
  --debug               enable debug mode
EOF
}

function debugPrint() {
  if [ "$DEBUG" -eq 1 ]; then
    echo "[DEBUG] $1"
  fi
}

function toggleFreeze() {
  # Skip this function if --dry-run flag was provided
  if [[ $DRYRUN == "1" ]]; then return 0; fi

  # Get pids of process tree
  PIDS=$(pstree -p "$PID" | grep -oP '\(\K[^\)]+')

  debugPrint "PIDs: $PIDS"

  # Prevent suspending itself
  local pid_of_script=$$
  if echo "$PIDS" | grep -q "$pid_of_script"; then
    echo "You are trying to suspend the hyprfreeze process."
    exit 1
  fi

  # Suspend or resume processes
  if [[ "$(ps -o state= "$PID")" == T ]]; then
    debugPrint "Resuming processes..."
    kill -CONT $PIDS 2>/dev/null && echo "Resumed $(ps -p "$PID" -o comm= 2>/dev/null) (PID $PID)" || exit 1
  else
    debugPrint "Suspending processes..."
    kill -STOP $PIDS 2>/dev/null && echo "Suspended $(ps -p "$PID" -o comm= 2>/dev/null) (PID $PID)" || exit 1
  fi
}

function getPidByActive() {
  debugPrint "Getting PID by active window..."
  if [[ "hyprland" == "$DESKTOP" ]]; then
    PID=$(hyprctl activewindow -j | jq '.pid')
  elif [[ "sway" == "$DESKTOP" ]]; then
    PID=$(swaymsg -t get_tree | jq '.. | select(.type?) | select(.focused==true) | .pid')
  else
    echo "Detecting the active window is currently not supported on $DESKTOP."
    exit 1
  fi
  debugPrint "PID by active window: $PID"

  # Die if PID is not numeric (e.g. "null" if there is no active window)
  if ! [[ $PID == ?(-)+([[:digit:]]) ]]; then
    echo "Cannot freeze window with invalid PID $PID."
    exit 1
  fi
}

function getPidByPid() {
  debugPrint "Getting PID by PID: $1"
  # Check if process pid exists
  if ! ps -p "$1" &>/dev/null; then
    echo "Process ID $1 not found"
    exit 1
  fi

  PID=$1
}

function getPidByName() {
  debugPrint "Getting PID by name: $1"
  # Check if process name exists
  if ! pidof -x "$1" >/dev/null; then
    echo "Process name $1 not found"
    exit 1
  fi

  # Get last process if there are multiple
  PID=$(pidof "$1" | awk '{print $NF}')
  debugPrint "PID by name: $PID"
}

function getPidByProp() {
  debugPrint "Getting PID by prop..."
  if [[ "hyprland" == "$DESKTOP" ]]; then
    if ! [ $(command -v hyprprop) ]; then
      echo "You need to install 'hyprprop' to use this feature. (https://github.com/vilari-mickopf/hyprprop)"
      exit 1
    fi

    PID=$(hyprprop | jq '.pid')
  elif [[ "sway" == "$DESKTOP" ]]; then
    if ! [ $(command -v swayprop) ]; then
      echo "You need to install 'swayprop' to use this feature. (https://git.alternerd.tv/alterNERDtive/swayprop)"
      exit 1
    fi

    PID=$(swayprop | jq '.pid')
  else
    echo "Selecting the target window by mouse is currently not supported on $DESKTOP."
    exit 1
  fi
  debugPrint "PID by prop: $PID"
}

function detectEnvironment() {
  if [ $(command -v loginctl) ]; then
    local Desktop
    eval "$(loginctl show-session $(loginctl | grep $(whoami) | awk '{print $1}') -p Desktop)"
    DESKTOP=$Desktop
  fi

  if [ -z "$DESKTOP" ]; then
    DESKTOP=$XDG_SESSION_DESKTOP
  fi

  if [ -z "$DESKTOP" ]; then
    debugPrint "Could not determine desktop environment via \`loginctl\`, and XDG_SESSION_DESKTOP is not set. Assuming Hyprland."
    DESKTOP="hyprland"
  fi

  # Enforce lowercase for consistent matching
  DESKTOP="${DESKTOP,,}"

  debugPrint "Desktop: $DESKTOP"
}

function printInfo() {
  debugPrint "Printing process info..."
  echo -e "$(tput bold)Process tree:$(tput sgr0)"
  ps -p "$PID" 2>/dev/null && pstree -p "$PID"

  echo -e "\n$(tput bold)Process threads:$(tput sgr0)"
  ps -eLo pid,tid,comm | grep "$PID" 2>/dev/null

  echo -e "\n$(tput bold)Process ID$(tput sgr0) = $PID \
    \n$(tput bold)Process name$(tput sgr0) = $(ps -p "$PID" -o comm= 2>/dev/null) \
    \n$(tput bold)Process state$(tput sgr0) = $(ps -o state= -p "$PID" 2>/dev/null)\n"
}

function sendNotification() {
  debugPrint "Sending notification..."
  local title
  title=$( [[ "$(ps -p "$PID" -o state=)" == T ]] &&
    echo "Suspended $(ps -p "$PID" -o comm= 2>/dev/null)" ||
    echo "Resumed $(ps -p "$PID" -o comm= 2>/dev/null)")

  local message="PID $PID"

  notify-send "${title}" "${message}" -t "$NOTIF_TIMEOUT" -a Hyprfreeze
}

function args() {
  # Track required flags
  local required_flag_count=0

  # Parse options
  local options="hap:n:rst:"
  local long_options="help,active,pid:,name:,prop,silent,notif-timeout:,info,dry-run,debug"
  local parsed_args
  parsed_args=$(getopt -o "$options" --long "$long_options" -n "$(basename "$0")" -- "$@")

  eval set -- "$parsed_args"
  while true; do
    case $1 in
      -h | --help)
        printHelp
        exit 0
        ;;
      -a | --active)
        ((required_flag_count++))
        FLAG_ACTIVE=true
        ;;
      -p | --pid)
        ((required_flag_count++))
        shift
        FLAG_PID="$1"
        ;;
      -n | --name)
        ((required_flag_count++))
        shift
        NAME_FLAG="$1"
        ;;
      -r | --prop)
        ((required_flag_count++))
        FLAG_PROP=true
        ;;
      -s | --silent)
        SILENT=1
        ;;
      -t | --notif-timeout)
        shift
        NOTIF_TIMEOUT="$1"
        ;;
      --info)
        INFO=1
        ;;
      --dry-run)
        DRYRUN=1
        ;;
      --debug)
        DEBUG=1
        ;;
      --)
        shift # Skip -- argument
        break
        ;;
      *)
        exit 1
        ;;
    esac
    shift
  done

  # Check if more than one required flag is provided, or if none was provided
  if [ $required_flag_count -ne 1 ]; then
    printHelp
    exit 1
  fi
}

function main() {
  debugPrint "Starting main function..."
  # Get pid by a required flag
  if [ "$FLAG_ACTIVE" = true ]; then
    getPidByActive
  elif [ -n "$FLAG_PID" ]; then
    getPidByPid "$FLAG_PID"
  elif [ -n "$NAME_FLAG" ]; then
    getPidByName "$NAME_FLAG"
  elif [ "$FLAG_PROP" = true ]; then
    getPidByProp
  fi

  # Suspend or resume process
  toggleFreeze

  # Run these functions after PID is obtained
  if [ $INFO -eq 1 ]; then printInfo; fi
  if [ $SILENT -ne 1 ]; then sendNotification; fi

  debugPrint "End of main function."
}

SILENT=0
NOTIF_TIMEOUT=5000
INFO=0
DRYRUN=0
DEBUG=0

args "$@"

detectEnvironment

main
