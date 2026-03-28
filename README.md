# helix-nanolm

Train a small GPT from scratch on `karpathy/climbmix-400b-shuffle` and let an autonomous agent optimize it. Inspired by [karpathy/autoresearch](https://github.com/karpathy/autoresearch).

The agent has a fixed **5-minute training budget** per experiment. It modifies `train.py` — architecture, optimizer, hyperparameters — and tries to minimize **validation bits per byte (`val_bpb`)**.

## Quickstart

```bash
pip install 'helices[claude]'

git clone https://github.com/VectorInstitute/helix-nanolm.git
cd helix-nanolm

uv run prepare.py   # one-time: download data shards + train tokenizer (~10 shards, ~5 min)
helix run
```

**CUDA users:** install PyTorch with CUDA support before `uv sync`:
```bash
pip install torch --index-url https://download.pytorch.org/whl/cu128
```

## Metric

**Primary: `val_bpb` (minimize)** — Vocabulary-size-independent validation bits per byte, evaluated on a pinned held-out shard.

## Scope

The agent may only modify `train.py`. All other files are read-only.

## Hardware

Runs on CUDA (tested on H100), Apple Silicon (MPS), or CPU. CUDA is strongly recommended for actual research — Mac/CPU runs are slow but useful for development.

---

Built with [helix](https://github.com/VectorInstitute/helix).
