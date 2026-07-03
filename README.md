# Olist — Customer Churn & Lifetime Value

**Predicting repeat purchase and estimating customer lifetime value on a Brazilian
e-commerce marketplace where 97% of customers buy only once.**

Portfolio project #2 · customer & loyalty analytics · [Kaggle notebook](#) · [LinkedIn](#)

---

## The problem

Olist wants to grow loyalty: who churns, who is worth keeping, and where should
retention spend go? The catch is that **only approx. 2.2% of customers ever buy twice.** On a
market this one-and-done, the textbook churn model ("flag anyone inactive for N days")
scores 97% accuracy by predicting *everyone* leaves — and teaches nothing.

So the first deliverable here isn't a model. It's a **defensible re-definition of the
problem**, built from evidence, before a single line of model code.

## Headline findings

| # | Finding | Number |
|---|---------|--------|
| 1 | Repeat purchase is only **weakly predictable** from a customer's first order | CV ROC-AUC ≈ **0.56** |
| 2 | **What you sell beats how you serve** — repeat rate swings approx. 4× by category | 2.3% (furniture/fashion) vs 0.6% (electronics/food) |
| 3 | **Failed delivery** is the clearest killer of repeat; speed barely matters | 0.16% vs 1.32% (delivered) |
| 4 | Value is **highly concentrated** in a thin repeat layer | top 10% hold **approx. 66%** of cohort CLV |

**Strategic read:** you cannot *target* your way to loyalty on Olist. The lever is
engineering the second purchase through **catalogue mix and delivery reliability**, then
concentrating retention economics on the small, high-value tier that CLV identifies.

## How the churn definition was derived (the hard part)

Because most customers are one-time buyers, "churn" had to be defined carefully rather
than assumed. Four evidence-based steps:

1. **The join trap** — `customer_id` is regenerated per order; only `customer_unique_id`
   identifies the real person. The wrong key reports *zero* repeat customers.
2. **Horizon from data** — measured how long real repeaters took to return; approx. 69% come back
   within 90 days (one quarter), which becomes the churn line.
3. **Phantom repeats** — approx. 850 "repeats" were same-checkout orders split across sellers
   (median gap: 0.0 minutes). Collapsed same-day fragments into true purchase events.
4. **Censoring** — customers whose first order fell in the final 90 days had no chance to
   return; dropped from the labelled base (approx. 9.5%) rather than mislabelled.

**Final definition:** churn = no second *purchase event* (distinct order-day) within 90
days of the first, on a labelable base of approx. 86,900 customers.

## Method

- **Repeat-propensity model** — LightGBM, features drawn **strictly from the first order**
  (leakage-safe), evaluated on **cross-validated out-of-fold** predictions and judged on
  **PR-AUC / lift**, never accuracy (approx. 76:1 class imbalance).
- **Drivers** — read as raw conditional repeat rates (a deliberately weak model's
  importances aren't trustworthy).
- **CLV** — BG/NBD + Gamma-Gamma (`lifetimes`) applied **only** to the approx. 2,150 true
  repeaters, since both models require `frequency > 0`.

## Results

- ROC-AUC 0.56 → an honest "mostly unpredictable" verdict, treated as a *finding*.
- Category and delivery-failure effects quantified as conditional rates.
- 12-month CLV scored per repeat customer; value concentration is the robust headline
  (mean CLV is unstable due to a low-frequency cohort — see LEARNINGS.md).

## Files

| File | What |
|------|------|
| `olist-churn-clv.ipynb` | Full analysis, narrative, and charts |
| `olist_clv_repeat_cohort.csv` | Scored repeat cohort (frequency, recency, p_alive, CLV) |
| `LEARNINGS.md` | Methodological notes and pitfalls |

## Data

[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
— approx. 100k orders, 2016–2018, 9 relational tables.

## Stack

Python · pandas · LightGBM · scikit-learn · lifetimes (BG/NBD, Gamma-Gamma) · matplotlib
