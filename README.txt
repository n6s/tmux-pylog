tmux new -s ops                                  # Start a tmux "ops" session
Ctrl-b c                                         # new windows for AI and sessions
tmux pipe-pane -o -t ops:0.0 'tee -a ~/tmux-logs/context.log'  # start logging for first window
tmux pipe-pane -o -t ops:1.0 'tee -a ~/tmux-logs/window-b.log'  # start logging for second window

aiq wraps OpenAI to answer questions using log context.

Usage:
  aiq [options] question...
  aiq setup-tmux [--fresh]           # create/prepare tmux session for logging + AI panes

Options:
  -f, --file PATH       Add a context file (repeatable). Use for logs or KB/notes. Defaults to ~/.cache/aiq/context.log if none given.
  -l, --lines N         Tail N lines per context file (default: $LINES or 200).
  -s, --summary PATH    Rolling session summary file (default: ~/.cache/aiq/summary.txt). Use --reset-summary to clear first.
  -m, --model NAME      Model to use (default: $AIQ_MODEL or gpt-4o-mini).
  -h, --help            Show help.

Examples:
  aiq "Summarize anomalies or differences between the two hosts."
  LINES=400 aiq "Look further back in the history for patterns."
  aiq -f /var/log/syslog -f /var/log/auth.log "Anything suspicious in the last few minutes?"
  aiq -f ~/tmux-logs/window-a.log "Check for errors"                 # override default context file
  aiq --reset-summary "Start fresh"                                  # clear summary first
  aiq setup-tmux                                                      # start/attach ops session with logging
  aiq setup-tmux --fresh                                              # same, but truncates default context + summary

Requires OPENAI_API_KEY in the environment. Missing context files are noted in the prompt instead of failing. Rolling summaries are kept at ~/.cache/aiq/summary.txt by default to simulate session memory.

Context vs summary:
- Context files: raw input the model reads each run (logs, KB snippets, notes). Default: ~/.cache/aiq/context.log (tail N lines). Provide multiple with -f.
- Summary file: compact rolling memory maintained by aiq across runs (not tailed like logs). Default: ~/.cache/aiq/summary.txt; reset with --reset-summary.

tmux helper:
- `aiq setup-tmux` creates/uses session `ops`, pipes pane 0.0 of window 0 to ~/.cache/aiq/context.log (via ansi2txt when available to strip TUI escape codes), ensures an AI window at index 1, and attaches if you are not already inside tmux.
- `aiq setup-tmux --fresh` additionally truncates the default context log and summary file for a clean slate.
