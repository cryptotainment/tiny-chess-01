# Tiny Chess — a 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/cryptotainment/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/cryptotainment/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in one Python file (`main.py`, 3,908 bytes), written for the [make.fun tiny-chess-01 bounty](https://make.fun/bounty/tiny-chess-01). It plays the house gauntlet through the env at `MAKEFUN_ENV_URL` (`POST /game`, `/move`, `/resign`).

## Design (v5)

- 0x88 board, alpha-beta search with iterative deepening to a **fixed budget of 100,000 nodes a move** (never a clock — replay-deterministic).
- Quiescence on captures and promotions, plus check evasions when in check (no stand-pat while in check; mate in quiescence scores `-MATE + ply`), with a quiescence ply cap of 14.
- Transposition table (Zobrist keys, deterministic seed); killer moves; MVV-LVA capture ordering; material + piece-square evaluation (standard simplified-evaluation tables, written out here).
- Move generation verified by perft against python-chess on 6 positions (startpos to depth 4: 197,281; Kiwipete to depth 3: 97,862; and more), **0 failures**; Zobrist key stable across make/unmake.
- Env driver: `POST /game`, `/move`, `/resign` to `MAKEFUN_ENV_URL` (urllib, stdlib only); falls back to the env's first legal move if its own best move is ever refused (never happened in tests).
- Speed: ~30–60k nodes/s; a 100k-node move averages ~1.7–2.1 s, slowest seen 3.54 s (move limit 10 s). No timeouts in any trial.

## Practice results

Mean Elo ~1,305 over 3 practice trials against the gauntlet anchors (855 / 1,073 / 1,199 / 1,392 / 1,497 / 1,596), vs the house baseline (sunfish-NNUE at 50k nodes) mean ~1,365. Competitive with the baseline — it equals the baseline's score on two of three trials — and it clears the 1,154 practice floor on 2 of 3 instances. It does not demonstrate beating the baseline's mean, so it is entered as a competitive engine, not a proven stronger one.

## The file

`main.py` is the `makefun compress` packing (LZMA stub) of the readable source, which lives in the work directory as `dev/packed_source.py` — it is NOT part of the submitted snapshot (it is over the per-file cap). The readable source is sent beside the transcripts at submit time, so the transcript gate counts its lines. Rebuild after any change to the source with:

    python dev/pack.py [--engine dev/engine_v5.py] [--nodes N]
    cp dev/packed_source.py repo/source.py
    (cd repo && node cli/makefun.mjs compress source.py --out main.py)
    rm repo/source.py                # must not be committed

MIT licensed, see LICENSE.
