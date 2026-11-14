# tiny-chess-01 — engine v12

v12 = v10 + ONE targeted eval change: an extra bonus for ADVANCED
passed pawns (6th/7th rank). The v10 base is restored exactly — the
v11 qsearch futility-margin change (200 -> 0) is REVERTED, because
v11 lost nearly every game on TIME in grading (trial 0 Elo 705,
trial 1 Elo 0) and its gate was mathematically dead.

## The change

In `v()`, the passed-pawn loop (v10's `+2*r^2` bonus) now adds
+30 centipawns when the passed pawn is advanced — white on the 6th
or 7th rank (`r>=5`), black on the 6th or 7th rank from its side
(`7-r>=5`, i.e. `r<=2`):

```python
if max(B[f:f+3])<=r:x+=2*r*r+30*(r>4)
elif min(W[f:f+3])>=r:x-=2*(7-r)*(7-r)+30*(r<3)
```

Why: v10's bottleneck is trial 1 vs the STRONG baseline (1464, lead
0). v10's games show gradual endgame losses and FAILED CONVERSIONS
(up a pawn / up B+P drawn by repetition). The v10 passed-pawn term
already helped trial 1 (+68 vs v9) and trial 2 (+210); the advanced
rank is where conversion actually happens, so the extra +30cp
concentrates the bonus there. King safety is PROVEN not the
bottleneck (v9 pawn-shield did nothing vs strong), and a mobility
term would need move generation inside eval (called at every
qsearch node) — not time-safe. This change costs ZERO extra
computation (pure arithmetic in the existing pawn loop) and 18
source bytes.

Qsearch futility margin RESTORED to 200 (v10 value). LMR,
null-move, history, PSTs UNCHANGED from v10.

## Files

- `golf12.py` — readable engine source (v10 + advanced-pawn bonus).
- `main.py` — lzma+base85 packed `golf12.py`, 4,087 bytes (cap 4,096),
  decompresses byte-exact.
- `pack_final.py` — packs `golf12.py` into `main.py`.
- `perft12.py` — perft (18 checks) + PST identity vs v9/v10 + eval
  deltas vs v10 + U2 checks.
- `CHANGES-v12.md` — full change log, verification, grading.
- `LICENSE` — MIT.

## Verification

- Perft: 18 checks (6 standard positions, depths 1-3) ALL OK.
- PSTs (P and Eg) byte-identical to v9/v10; V and M identical.
- Eval deltas v12-vs-v10 are EXACTLY the advanced-pawn bonus
  (10 sanity cases + 4 boundary cases, all OK).
- Deterministic: same position + fresh engine state -> same move
  (3 positions, 2 runs each).
- `main.py` (4,087 bytes) decompresses byte-exact to `golf12.py`.
- Smoke test: exits 0 with no env server.

## Grading

See CHANGES-v12.md. Baselines: trial 0 = 1326, trial 1 = 1464,
trial 2 = 1290. Gate: mean lead > 0 with the 90% paired t-interval
(n=3, t=2.92) above 0.
