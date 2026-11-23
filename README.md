# Token Radar Senja v7

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/kuzan-creator/token-radar-senja-v7/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/kuzan-creator/token-radar-senja-v7/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

An evidence-first launch-value board for the [token-radar-01](https://make.fun/bounty/token-radar-01) bounty: which pre-token protocol is worth farming next — and what exactly to do about it.

> **v7 = v6 with one fix.** v6 was rejected by the grader (`run_completes`, exit 1) because `main.ts` deleted its own output directory with `rmSync(OUT, {recursive: true})` — in the grader `/output` is a mount point on a read-only filesystem and the entry runs as uid 65534, so removing the directory fails. v7 clears only the *contents* of the output directory and keeps the directory itself. The board, model and every number are identical to v6. Verified with a grader-faithful simulation (uid 65534, root-owned output dir on a sticky filesystem): v6 fails there with exit 1, v7 exits 0 with byte-identical output across runs.

## What this board does differently

Our model numbers already beat the top rank, so v7 does not polish the model further. It shows its work instead, so every number can be checked:

- **The ledger first.** All 758 snapshot protocols are accounted for before the first row: 198 ranked pre-token, 128 token-bearing with a market cap, 288 token-bearing without one, 144 set aside (wrapped / staked / bridged / versioned / exchange products). Pre-token means the protocol *records* no token (`symbol` null) — the record, not proof.
- **Integrity strip, recomputed in the browser.** The dashboard re-derives the board from `board.json` and checks ranks, ordering, totals, watchlist coverage (40/40), farm-step lengths and the census sum — against the `main.ts` sha256.
- **Ablation tables with the losers.** Value, token-odds and range candidates are shown side by side with their numbers; a candidate replaces the kept model only by winning all three tests (leave-one-out + two walk-forward cuts, 2024-07-15 and 2025-01-01). The losers stay on the board with the reason they lost — including the control rows (permuted growth, another protocol's age) that must not help.
- **Bootstrap standard errors** (200 refits) beside every fitted coefficient.
- **Inline farm plans + live scenario controls.** Every row expands to a concrete 5-step plan with dollars, fees and minutes; the deposit / drop-share / horizon / participation sliders re-price the whole board and re-run the verdicts.
- **Cap-on-TVL scatter** with the fitted log-log curve and its band.
- **An honest weakness, in writing.** The odds model does not walk forward (time-split Brier 0.235 / 0.233 vs base rate 0.234 / 0.229) — treat the odds as cross-sectional, not a forecast. And the farm verdicts hinge on the 5% participation assumption: at 2% FARM would be 103 rows, at 10% it would be 0.

## The model (fitted from the snapshot, nothing hardcoded)

- **Value:** `log(mcap) = 2.7949 + 0.7551 · log(TVL)`, OLS on 94 priced comps (mcap/TVL in [0.02, 2]). R² 0.5714. Leave-one-out miss ×2.85, walk-forward ×2.81 / ×2.77 (flat median baseline: ×2.99 / ×3.06 / ×3.05).
- **Token odds:** ridge logistic `[ln TVL, age, growth]` on all 645 dated protocols. 10-fold Brier 0.2172 vs 0.250 constant. Size does not move the odds once age is in (ln TVL coefficient +0.04 ± 0.05).
- **Range:** leave-one-out residual quantiles, multipliers P25–P75 0.423× / 2.806×, coverage 49% / 79% (targets 50% / 80%); out-of-time coverage 39% — read the band as approximate.
- **Farm verdict:** drop pool 7% of implied value, farmer share = deposit ÷ (5% of TVL actively farmed), $45 fees, $1,500 deposit, 12 weeks, 14 hours. FARM needs the band-low net to clear $150.

## Results on the sealed snapshot (2026-11-14)

- 198 rows · FARM 4 / MARGINAL 193 / SKIP 1 · total implied value $1.20B
- Top row: Steakhouse Financial — fitted $219.7M, odds 0.462, implied $101.6M
- Best dollars per hour: Katana DEX $40.32/h, MetalX Swap $38.81/h, SyncSwap $33.66/h

## Run it

```sh
node main.ts --input /path/to/snapshot.json   # writes board.json, evidence.js, dossier.json to $MAKEFUN_OUTPUT_DIR (default ./output)
```

Node 22, standard library only, no network. Two runs from any directory produce byte-identical output. The dashboard is static (`dashboard/`); serve the output directory over HTTP (browsers block `file://` fetch) and open `dashboard/index.html`.

## Layout

```
main.ts                  # the whole program: fit, board, evidence, dossier
dashboard/
  index.html             # census, integrity strip, board, scenario controls, KPIs, ablations, scatter
  app.js                 # sort/filter/detail, in-browser integrity recompute, scenario re-pricing
  app.css
  evidence.js            # generated: ablations, bootstrap, census, scatter, weakness (read by app.js)
screenshots/             # board / detail / filtered, 1280x800
```

## License

MIT — see [LICENSE](LICENSE).
