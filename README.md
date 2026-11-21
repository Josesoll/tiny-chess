# tiny-chess

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Josesoll/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

An original chess engine that fits in 4,094 bytes of Python. `main.py` is the
whole engine: it carries a minified build of the engine as an lzma/base85
payload with a tiny unpacker, and plays the make.fun house gauntlet through
the env API (`POST /game`, `POST /move`). No model calls, standard library
only, and fully deterministic: the search runs to a fixed node budget, never
to a clock, so a replayed trial makes the same moves byte for byte.

## How it plays

- **Search:** iterative-deepening negamax alpha-beta with principal-variation
  search, a transposition table with exact/lower/upper bounds (Zobrist
  hashing, incremental), check extensions, and a quiescence search over
  captures, promotions and check evasions. The budget is 40,000 nodes per
  move in total across all depths; only completed depths count, so the worst
  move stays a few seconds on one CPU, well inside the 10-second limit.
  (The budget was tuned against the kit's practice trials: 1,279 Elo at
  40,000 nodes on two of them, against a 1,154 qualification floor.)
- **Move ordering:** transposition-table move first, then captures by
  MVV-LVA, then promotions.
- **Evaluation:** material (100/320/330/500/900) plus piece-square tables for
  pawn, knight, bishop, rook and king; the queen table is the average of the
  rook and bishop tables. Tables are stored delta-encoded inside the source.
- **Board:** a 64-square array with precomputed knight/king/slider attack
  rays, make/unmake with a full state stack, and incremental score and hash.

Move generation was verified against published perft node counts on five
standard test positions (start position, Kiwipete, and three others,
including castling-, en-passant- and promotion-heavy suites), and the packed
`main.py` was exercised end to end against the kit's real env and scorer on
the practice trials.

## Run it

```sh
MAKEFUN_ENV_URL=<env url> python3 main.py
```

---
<sub>Entered on [make.fun](https://make.fun/?ref=w9gyap1e&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
