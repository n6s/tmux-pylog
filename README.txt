tmux-log: pipe a tmux pane into a log file so Codex can read it for you.

Quick start:
  ./tmux-log --fresh                              # start/attach session "ops", log pane 0.0 to ~/.cache/tmux-log/window0.log (truncate first)
  ./tmux-log --raw                                # same, but skip ANSI/backspace cleanup
  ./tmux-log -s ops -t 1.0 -l ~/tmux-logs/ops.log # custom session/pane/log path
  ./tmux-log --current --fresh --auto-log         # inside tmux: log the active pane to ~/.cache/tmux-log/<session>/w<pane>.log (truncate first)
Then ask Codex to read the log file you pointed tmux at (default: ~/.cache/tmux-log/window0.log).
Window 1 ("codex") is auto-created with commented sample prompts you can edit/copy into Codex.
If ~/venv/ansible9/bin/activate exists (or TMUX_LOG_VENV points elsewhere), that venv is sourced in the codex window.

What it does:
- Ensures the tmux session exists (default `ops`; override with TMUX_LOG_SESSION or -s).
- Pipes the target pane (default `0.0`; override with TMUX_LOG_TARGET or -t) to the log file (default `~/.cache/tmux-log/window0.log`; override with TMUX_LOG_PATH or -l).
- Cleans ANSI/backspaces via ansi2txt | col -b when available; use `--raw` to skip cleaning.
- Creates a helper window at index 1 named "codex" with commented sample prompts and a sample pipe-pane command to log another pane.
- Attaches if you are not already inside tmux; otherwise prints a ready message.

Options:
  -s, --session NAME   tmux session name
  -t, --target WIN.P   target pane to log
  -l, --log PATH       log file path
  -c, --current        use the current tmux pane as target
      --auto-log       choose a per-pane log file under ~/.cache/tmux-log/<session>/
      --raw            do not clean ANSI/backspaces; write raw output
      --fresh          truncate the log file before piping
      --stop           stop piping for the target pane
  -h, --help           show help

Suggested tmux key binding (put in ~/.tmux.conf):
  bind-key L run-shell -b '~/workarea/tmux-log/tmux-log --current --auto-log --fresh'

Manual alternative (without tmux-log):
  tmux new -s ops
  tmux pipe-pane -o -t ops:0.0 'ansi2txt | col -b | tee -a ~/.cache/tmux-log/window0.log'  # or drop filters if not installed

Dependencies: tmux. Optional: ansi2txt and col for cleaner logs.
