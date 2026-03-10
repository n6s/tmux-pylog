tmux-log: pipe a tmux pane into a log file so Codex can read it for you.

Quick start:
  ./tmux-log --fresh                              # start/attach session "ops", log pane 0.0 to ~/.cache/tmux-log/window0.log (truncate first)
  ./tmux-log --raw                                # same, but skip ANSI/backspace cleanup
  ./tmux-log -s ops -t 1.0 -l ~/tmux-logs/ops.log # custom session/pane/log path
  ./tmux-log --current --fresh --auto-log         # inside tmux: log the active pane to ~/.cache/tmux-log/<session>/w<pane>.log (truncate first)
  ./tmux-pylog                                    # Python logger for session "ops" with a default 50 MiB per-file cap
  ./tmux-redact                                   # redact a secret from the current pane's log, preserving piping
Then ask Codex to read the log file you pointed tmux at (default: ~/.cache/tmux-log/window0.log).

What it does:
- Ensures the tmux session exists (default `ops`; override with TMUX_LOG_SESSION or -s).
- Pipes the target pane (default `0.0`; override with TMUX_LOG_TARGET or -t) to the log file (default `~/.cache/tmux-log/window0.log`; override with TMUX_LOG_PATH or -l).
- Cleans ANSI/backspaces and carriage returns via ansi2txt when available, otherwise uses sed-based cleanup; use `--raw` to skip cleaning.
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
  tmux pipe-pane -o -t ops:0.0 'ansi2txt | sed -r ":a; s/.\x08//; ta" | tee -a ~/.cache/tmux-log/window0.log'

Dependencies: tmux. Optional: ansi2txt and col for cleaner logs.

tmux-pylog notes:
- Always writes both a raw log and a cleaned log under ~/.cache/tmux-log/<session>/.
- By default, each file is capped at `50M` and pruned back to the newest `40M`.
- Use `--max-bytes SIZE` (or `TMUX_PYLOG_MAX_BYTES`) to override the per-file cap, or `--max-bytes 0` to disable pruning.
- Use `--keep-bytes SIZE` (or `TMUX_PYLOG_KEEP_BYTES`) to control how much recent data remains after pruning.
- Sizes accept raw bytes or `K`, `M`, `G`, `T` suffixes, for example `250M`.
