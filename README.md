# tiny-chess-01 — engine v13

v13 = v12 + ONE targeted time-safety change: a WALL-CLOCK abort.
v12 failed its gate on TIME (23 time losses, mean Elo 1161), not on
strength — v12 is exactly as strong as v10. The engine had no clock:
a fixed 100,000-node budget that takes ~4.5 s normally but 10-22 s
on a slow CPU / expensive position → instant forfeit at the 10 s
per-move limit.

## The change

1. **Clock abort:** each move gets a deadline `Dl = start + 8 s`,
   checked every 1024 nodes in `q()` and `r()`:
   `if nd>N or nd>>10<<10==nd and time.time()>Dl:raise X()`.
   The existing exception-X mechanism is reused — T3 catches it and
   returns the best move of the last COMPLETED depth, so the engine
   always answers with a legal move in ~8 s.
2. **Node cap raised:** `N: 1e5 → 1e6` (0 bytes). With the clock as
   the real limiter, the engine uses as many nodes as the machine
   manages inside 8 s. The `TC_NODES` env override is removed
   (the kit sets it nowhere).

Eval is UNCHANGED from v12 (PSTs, V, M, and `v()` outputs identical
on all test positions). Move generation untouched.

## Files

- `golf13.py` — readable engine source (v12 + clock abort; v13r2
  deep-rewrites the move machinery D/K/m/U/g/l for authorship).
- `main.py` — packed by the KIT's `makefun compress` (fresh kit),
  3,896 bytes (cap 4,096), decompresses byte-exact to `golf13.py`.
- `verify13.py` — verification: stub round-trip, perft, eval
  identity vs v12, abort proof, real T3 timing, determinism, smoke.
- `CHANGES-v13.md` — full change log, verification, grading notes.
- `LICENSE` — MIT.

## Verification

- Perft: 18 checks (6 standard positions, depths 1-3) ALL OK.
- Eval identical to v12 on all test positions (search-only change).
- Abort proof: expired deadline + depth-6 search → X raised after
  0.04 s at exactly nd=1024.
- Real T3: startpos 8.03 s / 185,344 nodes, legal move (without the
  clock, 1e6 nodes would take ~43 s).
- Determinism: same position + fresh state → same move, 2 runs.
- `main.py` (3,896 bytes) decompresses byte-exact to `golf13.py`.
- Smoke test: exits 0 with no env server.

## Grading

See CHANGES-v13.md. NOT graded yet — grade only on an idle VM,
sequential, same procedure as v12. Expect ~4,000 s/trial
(~3.3 h for 3 trials). Baselines: trial 0 = 1326, trial 1 = 1464,
trial 2 = 1290. Gate: mean lead > 0 with the 90% paired t-interval
(n=3, t=2.92) above 0. Do NOT submit unless the gate passes AND the
user confirms.
