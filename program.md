# helix-nanolm

Optimize a small GPT trained from scratch on a large text corpus. The goal is to minimize **validation bits per byte (`val_bpb`)** within a fixed **5-minute wall-clock training budget**.

## Setup

1. Run `uv run prepare.py` once — downloads data shards and trains a BPE tokenizer into `~/.cache/autoresearch/`.
2. Read `prepare.py` to understand the fixed evaluation harness and constants.
3. Read `train.py` to understand the current model architecture, optimizer, and hyperparameters.

## Constraints

- Modify `train.py` freely — architecture, optimizer, hyperparameters, batch size, model size are all in scope.
- Do NOT modify `prepare.py`, `helix.yaml`, `program.md`, or `pyproject.toml`.
- Do not install new packages or add dependencies beyond `pyproject.toml`.
- The `evaluate_bpb` function in `prepare.py` is the ground truth metric — do not change it.

## Metrics

**Primary: `val_bpb` (minimize)** — Validation bits per byte. Lower is better.

Since the training time budget is fixed at 5 minutes, improving throughput (MFU) automatically translates to more gradient steps and potentially lower BPB.

**VRAM** is a soft constraint. Some increase is acceptable for meaningful `val_bpb` gains, but avoid dramatic blowup.

## Simplicity criterion

Simpler is better when all else is equal. A 0.001 `val_bpb` improvement that adds 20 lines of hacky code is not worth it. A 0.001 improvement from deleting code is a win. Weigh complexity cost against improvement magnitude.

## Output format

```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:            8
```

Extract results: `grep "^val_bpb:\|^peak_vram_mb:" run.log`

If the grep output is empty, the run crashed. Check `tail -n 50 run.log` for the stack trace.

## Experiment loop

LOOP FOREVER:

1. Choose an optimization idea. Do not repeat what has already been tried.
2. Modify `train.py`.
3. `git commit` with a short description.
4. Run: `uv run train.py > run.log 2>&1 & echo $! > run.pid; wait $!; rm -f run.pid`
5. Extract results: `grep "^val_bpb:\|^peak_vram_mb:" run.log`
6. If results are empty: crashed. Check `tail -n 50 run.log`. Fix if trivial, log as crash and move on.
7. Append a row to `results.tsv` (tab-separated, columns: commit val_bpb status description).
   - `status` must be exactly `keep`, `discard`, or `crash` — no other values.
8. If `val_bpb` improved (lower): **keep** (commit stays). Status = `keep`.
9. Otherwise: **discard** (`git reset --hard HEAD~1`). Status = `discard`.

**NEVER STOP.** Run until interrupted.
