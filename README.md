# tiny-chess-01 — a 4 KB chess engine

`main.py` (4,081 bytes) is the whole entry: a chess engine of my own design,
packed with lzma (payload + unpacker, as the rules allow) to fit the size
cap. Python standard library only.

Design:

- 0x88 mailbox board; moves packed into ints (from, to, promotion).
- Negamax search with alpha-beta and principal-variation search, iterative
  deepening under a fixed node budget (115k nodes a move before ply 100 of
  a game, 60k after) — never a clock, so every move is a pure function of
  the position and the game history. Null-move pruning; move ordering by
  transposition-table move, captures by MVV-LVA, promotions, then a killer
  move per ply.
- One unified search: at depth 0 it becomes quiescence (stand-pat, captures
  and promotions, all evasions when in check), with repetitions and the
  hundred-ply rule scored as draws at every node. The transposition table
  stores bound flags and the best move.
- Incremental evaluation: material plus piece-square tables (my own
  values), updated on make/unmake. The king's table is tapered: the
  endgame king table is blended in by a phase computed from the material
  on the board, so the king centralises as pieces come off. Zobrist
  hashing drives the table and repetition detection.
- Games start from the standard start position (the env plays 8 book
  plies first); the driver replays the game's moves each turn, which also
  builds the repetition history, then searches and answers.

Measured locally against the kit's practice gauntlet (kit env, this
machine, 6 games per instance, colours alternating):

- instance 0: 4.5/6 — Elo 1545 (beat depth-2/3/4 and nodes-100k, drew
  nodes-250k, lost to nodes-500k)
- instance 1: 5.0/6 — Elo 1654 (lost only to nodes-250k)
- instance 2: 3.5/6 — Elo 1364 (beat the depth rungs, drew nodes-100k and
  nodes-250k, lost to nodes-500k)

Mean practice Elo 1521 across the three instances (qualification floor
1154). A repeated run of instance 1 reproduced every game move for move.
