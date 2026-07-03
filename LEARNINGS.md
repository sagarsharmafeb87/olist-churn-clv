# Learnings — Olist Churn & CLV

Method notes and traps, kept honest for my own reference and for anyone reading the repo.

## Data / definition traps

- **`customer_id` is per-order, not per-person.** Join on `customer_unique_id` or you will
  conclude nobody ever repeats. This single key choice makes or breaks the project.
- **Same-day order fragments masquerade as repeat purchases.** Multi-seller checkouts are
  logged as separate `order_id`s with a approx. 0-minute gap. approx. 850 of approx. 3,000 "repeaters" were
  this artifact. Count distinct *order-days*, not order_ids.
- **Right-censoring contaminates naive churn labels.** A customer who bought near the end
  of the observation window had no runway to return. Drop them from the labelable base
  instead of labelling them "churned."
- **A degenerate base rate makes accuracy meaningless.** At approx. 1.3% positives, a "predict
  churn for everyone" dummy scores approx. 97%. Judge on PR-AUC and lift vs base, not accuracy.

## Modelling

- **Leakage rule:** every feature must be knowable from the FIRST order only, because that
  is all you have at real scoring time. First-order delivery/review are fair game; anything
  from the second order is cheating.
- **Aggressive class weighting hurt ranking.** `scale_pos_weight ≈ 76` chased a handful of
  positives and overfit noise; the regularised, unweighted CV model ranked better.
- **Early-stopping on the same split you report is subtle peeking.** Use out-of-fold CV to
  read the honest ceiling.
- **A weak model's feature importances are not trustworthy.** With ROC-AUC 0.56, drivers
  were read as raw conditional rates instead — undeniable and model-free.
- **A modest, honest AUC beats a suspiciously high one.** 0.56 with a clear interpretation
  ("repeat is mostly unpredictable from order one") is a real finding, not a failure.

## CLV (BG/NBD + Gamma-Gamma)

- **Scope to the repeat cohort.** Both models need `frequency > 0`; forcing them onto
  one-time buyers degenerates to naive average-order-value.
- **Watch the Gamma-Gamma shape parameter.** With a small, low-frequency cohort, `q < 1`
  makes the fitted *mean* CLV unstable and long-tailed (mean ≫ median). Headline the
  **concentration** of value (robust), not the absolute mean.
- **`lifetimes` is unmaintained but works** for classic MLE fits. If it fails to install,
  `btyd` is a near-drop-in replacement (`from btyd import BetaGeoFitter`).

## Communication

- **Action titles sell the finding.** "Repeat buyers return fast — and rarely after one
  quarter" beats "Days to second order." The chart is proof; the headline is the point.
- **One visual identity across resume, LinkedIn, and notebook** (shared navy `#0C2345`)
  reads as a coherent brand without the reader knowing why.
