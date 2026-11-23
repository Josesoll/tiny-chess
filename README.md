# tiny-chess

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

An original chess engine that fits in 4,096 bytes of Python. `main.py` is
the whole engine: a minified build carried as an lzma payload with a tiny
unpacker, playing the make.fun gauntlet through the env API
(`POST /game`, `POST /move`). No model calls, standard library only, and
fully deterministic: fixed node budgets per opponent, never a clock.

Version 3 was rebuilt after version 2 stalled at 1,364 / 1,279. Two of v2's
features were liabilities: a null-move bug deleted the piece on a1 from
the board mid-search, so the engine hallucinated mates and sent illegal
moves; and late-move pruning threw away defensive quiet moves, so the
engine believed it was winning material and played losing sacrifices.
The null move now restores the position by hand, and late-move pruning
was removed outright.

## How it plays

- **Search:** iterative-deepening PVS negamax, a transposition table with
  exact/lower/upper bounds and mate-distance adjustment, null-move
  pruning, late-move reductions, check extensions, and a quiescence
  search over captures, promotions and check evasions (delta pruning).
  Repetition inside the search scores a draw. No aspiration windows and
  no late-move pruning. Budgets run 40,000 against depth-2 up to 120,000
  against nodes-2M; the slowest move stays around four seconds on one
  CPU, well inside the 10-second limit.
- **Move ordering:** transposition-table move first, then captures by
  MVV-LVA, then killer, then history.
- **Evaluation:** material (100/320/330/500/900) plus PeSTO-style
  middlegame/endgame piece-square tables, interpolated by game phase
  (24 in the opening, 0 with bare kings), a bishop-pair bonus and a small
  tempo bonus. Tables are generated from formulas inside the source.
- **Board:** a 64-square array with precomputed attack rays, make/unmake
  with a full state stack, and incremental scores and Zobrist hash.

Measured against the kit's real env and scorer: **1,654 Elo on practice
trial-0** (4 wins, 2 draws, no losses) and **2,112 on practice trial-1**
(6 wins, no losses — the trial cap), against v1's 1,279 on both trials
and v2's 1,364 / 1,279. Move generation was verified against perft
counts on six test positions.

## Run it

```sh
MAKEFUN_ENV_URL=<env url> python3 main.py
```

---
<sub>Entered on [make.fun](https://make.fun/?ref=w9gyap1e&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
