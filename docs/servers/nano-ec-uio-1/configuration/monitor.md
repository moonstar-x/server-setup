# Monitor

This server's main use is to monitor the rest of the servers with `gotop`. I've made a nice script that launches all the `gotop` instances through SSH on all the services and presents them on a nice `tmux` session.

## Script

Create the monitor script file:

```bash
mkdir monitor
nano monitor/gotop.sh
```

The script content is as follows:

```bash
#!/bin/bash

SESSION=monitor

tmux new-session -d -s $SESSION

tmux set-option -t $SESSION pane-border-status top
tmux set-option -t $SESSION pane-border-format "#{pane_title}"
tmux set-option -t $SESSION pane-border-style fg=green
tmux set-option -t $SESSION pane-active-border-style fg=green

tmux split-window -h -t ${SESSION}.0
tmux split-window -h -t ${SESSION}.1
tmux select-layout -t "$SESSION" even-horizontal

TOP_PANES=$(tmux list-panes -t $SESSION -F "#{pane_id}" | tr '\n' ' ')

for PID in $TOP_PANES; do
  tmux split-window -v -t $PID
done

ALL_PANES=$(tmux list-panes -t $SESSION -F "#{pane_id}" | tr '\n' ' ')

TITLES=(
  "misc-ec-uio-1"
  "dev-ec-uio-1"
  "nano-ec-uio-1"
  "misc-us-east-1"
  "deploy-us-east-1"
  "dev-eu-west-1"
)

COMMANDS=(
  "ssh user@server.com -t 'gotop'"
  "ssh user@server.com -t 'gotop'"
  "gotop"
  "ssh user@server.com -t 'gotop'"
  "ssh user@server.com -t 'gotop'"
  "ssh user@server.com -t 'gotop'"
)

for PID in $ALL_PANES; do
  i=${PID#%}
  title=${TITLES[$i]}
  cmd=${COMMANDS[$i]}

  tmux select-pane -T $title -t $PID
  tmux send-keys -t $PID "$cmd" C-m 
done

tmux attach-session -t $SESSION
```

## Auto-starting

The idea is this script runs as soon as the Raspberry Pi turns on. For this, we'll edit the `~/.xinitrc` file and add the following content:

```bash
awesome & sleep 15 && xrdb ~/.Xresources && xterm -fullscreen -e "(cd ~/monitor && ./gotop.sh)"
```

This starts the Awesome WM, waits until it's up, loads up the terminal colors and then starts `xterm` in fullscreen mode with the script.
