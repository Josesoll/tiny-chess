# tiny-chess

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

An original chess engine that fits in 4,096 bytes of Python. `main.py` is the
whole engine: it carries a minified build of the engine as an lzma payload
with a tiny unpacker, and plays the make.fun house gauntlet through the env
API (`POST /game`, `POST /move`). No model calls, standard library only, and
fully deterministic: the search runs to a fixed node budget per opponent,
never to a clock, so a replayed trial makes the same moves byte for byte.

## How it plays

- **Search:** iterative-deepening negamax alpha-beta with principal-variation
  search, aspiration windows, late-move reductions, a transposition table
  with exact/lower/upper bounds and mate-distance adjustment (Zobrist
  hashing, incremental), check extensions, and a quiescence search over
  captures, promotions and check evasions. Repetition inside the search
  scores a draw. The node budget depends on the opponent (30,000 against
  depth-2 up to 160,000 against nodes-2M), so the strongest foes get the
  deepest search; the slowest move stays a few seconds on one CPU, well
  inside the 10-second limit.
- **Move ordering:** transposition-table move first, then captures by
  MVV-LVA, then the killer move, then the history heuristic.
- **Evaluation:** material (100/320/330/500/900) plus PeSTO-style
  middlegame/endgame piece-square tables, interpolated by game phase
  (phase 24 in the opening, 0 with bare kings), a bishop-pair bonus and a
  small tempo bonus. Tables are generated from formulas inside the source.
- **Board:** a 64-square array with precomputed knight/king/slider attack
  rays, make/unmake with a full state stack, and incremental middlegame and
  endgame scores and hash.

Measured against the kit's real env and scorer: **1,364 Elo on practice
trial-0** (3 wins, 1 draw, 2 losses) and **1,279 on trial-1** (3 wins,
3 losses), against the v1 engine's 1,279 on both trials. Move generation
was verified against published perft counts on seven test positions, and
in self-play at 20,000 nodes a move this engine scored 4.5/6 against v1.

## Run it

```sh
MAKEFUN_ENV_URL=<env url> python3 main.py
```

---
<sub>Entered on [make.fun](https://make.fun/?ref=w9gyap1e&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
