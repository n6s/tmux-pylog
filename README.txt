tmux new -s ops    # Start a tmux “ops” session
Ctrl-b c             # new windows for AI and sessions
tmux pipe-pane -o -t ops:0.0 'tee -a ~/tmux-logs/window-a.log'    # start logging for first window
tmux pipe-pane -o -t ops:1.0 'tee -a ~/tmux-logs/window-b.log'    # start logging for second window
aiq "Summarize anomalies or differences between the two hosts."
LINES=400 aiq "Look further back in the history for patterns."

