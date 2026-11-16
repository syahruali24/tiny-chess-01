# Tiny Chess Engine (4 KB Python) — make.fun `tiny-chess-01`

A small, original, deterministic chess engine in Python 3 (standard library
only). It plays the make.fun house gauntlet over HTTP, searching to a fixed
number of nodes — never to a clock — so every trial replay is byte-identical.

## Engine

`main.py` is the output of the kit's own `makefun compress` applied to the
readable source `src.py` (uploaded by `makefun bundle` under `packs/`; not
part of the graded bytes). Source ~6 KB; packed entrypoint ~2.6 KB, under
the 4,096-byte cap.

Architecture (all original code, written for this bounty):

- **Board**: 0x88 array (128 squares, off-board squares masked with `& 136`).
- **Move generation**: pseudo-legal with full legality filtering (king safety
  via an attack test), including castling, en passant, and all promotions.
- **Search**: negamax + alpha-beta, iterative deepening to a **fixed node
  budget of 100,000 nodes/move**. An iteration that runs past the budget is
  dropped whole; the last whole iteration's move stands. Transposition table
  with EXACT / LOWER / UPPER flags, killer moves, MVV-LVA ordering.
- **Quiescence**: stand-pat + captures/promotions, all evasions when in
  check, with delta pruning and a ply cap.
- **Evaluation**: material (P 100, N 320, B 330, R 500, Q 900) + a shared
  centre piece-square table (scaled per piece) + pawn advancement + bishop
  pair (+30).
- **Determinism**: fixed node count, no randomness, no clock, no I/O besides
  the env HTTP calls. The same position always yields the same move.
- **Protocol**: reads `MAKEFUN_ENV_URL`; `POST /game {}` to be served the
  game, `POST /move {"game": n, "move": "<uci>"}` to move. Resigns are
  never sent.

## Results (local practice trials, 6 games each)

| Configuration | Result | Elo |
|---|---|---|
| v1, NB=30k | 1W 0D 5L | 876 |
| v1, NB=100k | 0W 2D 4L | 876 |
| v1, NB=100k (drop aborted iteration) | 2W 0D 4L | 1099 |
| v1 + TT flags, NB=100k | 2W 0D 4L | 1099 |

The 6-game practice Elo is noisy (±~200). The honest summary: the engine
sits around **~1099**, **below** the 1,154 practice floor and the ~1,365
sunfish house baseline. It beats a greedy 1-ply player 6–0 and passes all
perft/determinism checks, but it is not strong enough to expect a prize.

## Files

- `main.py` — the packed entrypoint (committed and graded). Produced by
  `makefun compress` from `src.py` (6,061 → **2,655 bytes**, sha256
  `e4a10e2cc7bc8fd8f3b70947b079565667062e8da7277964813ea6006cc1dceb`).
- `src.py` — the readable source (uploaded with the entry, not committed).
- `LICENSE` — MIT.
