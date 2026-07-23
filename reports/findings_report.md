# Findings Report: Market Sentiment and Trader Performance on Hyperliquid

## 1. Objective

Explore the relationship between Bitcoin market sentiment (Fear/Greed Index) and trader performance using historical trade-level data from Hyperliquid, in order to uncover patterns that could inform smarter trading strategies.

## 2. Data Overview

- **Sentiment data**: 2,644 daily records (2018-02-01 to 2025-05-02), classified into 5 categories: Extreme Fear, Fear, Neutral, Greed, Extreme Greed.
- **Trades data**: 211,224 individual trade records (2023-05-01 to 2025-05-01) across 32 accounts and 246 coins, with execution price, size, side, direction, closed PnL, and fees.
- After merging trades to sentiment by calendar date, 211,218 rows retained (6 rows on a single date with no sentiment record were dropped — negligible impact).

## 3. Methodology

1. **Cleaning & validation**: verified data types, checked for duplicates (none found), confirmed the merge did not fan out (211,224 trades in → 211,224 out before the 6-row drop).
2. **Trade classification**: trades were split into opening (`Open Long`, `Open Short`, `Buy`, `Sell`) and closing (`Close Long`, `Close Short`, `Short > Long`, `Long > Short`, liquidations, settlement) based on the `Direction` field, since `Closed PnL` is only meaningful for closing trades (84.5% of opening trades had PnL = 0, vs 0.08% of closing trades — confirming this split was correct).
3. **Aggregate analysis**: grouped closing trades by sentiment category to compute trade count, total/average/median PnL, win rate, and average position size.
4. **Statistical testing**: used a Kruskal-Wallis test (non-parametric, appropriate given the heavily skewed PnL distribution) to test whether PnL differs significantly across sentiment groups.
5. **Robustness checks**: examined per-account performance to test whether the aggregate pattern reflects broad trader behavior or is concentrated in a few accounts; examined trading day counts per sentiment category to catch small-sample distortions.

## 4. Key Findings

### 4.1 Aggregate performance by sentiment

| Sentiment | Trades | Total PnL | Avg PnL | Median PnL | Win Rate | Avg Size (USD) |
|---|---|---|---|---|---|---|
| Fear | 26,513 | $3,367,722 | $127.02 | $7.11 | 88.6% | $8,871 |
| Extreme Fear | 9,369 | $879,803 | $93.91 | $8.05 | 80.0% | $5,916 |
| Greed | 19,369 | $1,383,789 | $71.44 | $3.98 | 76.1% | $6,637 |
| Neutral | 15,870 | $1,082,913 | $68.24 | $4.40 | 83.0% | $6,120 |
| Extreme Greed | 13,701 | $633,511 | $46.24 | $7.10 | 87.4% | $3,492 |

Average PnL and win rate are both notably higher during **Fear** than **Greed**. A Kruskal-Wallis test confirms this difference is statistically significant (H = 735.62, p < 0.001) — far too strong to be attributable to chance.

### 4.2 This pattern is not universal — concentration and consistency checks

Two checks were run to stress-test the aggregate result:

- **Concentration**: the top 5 accounts (of 32) generate **62.6%** of total platform PnL. The aggregate numbers are disproportionately influenced by a small number of high-volume/high-skill traders.
- **Consistency**: only **14 of 31 accounts (45%)** with both Fear and Greed trades actually performed better during Fear. This is close to a coin flip — it means the "Fear outperforms Greed" pattern, while real in aggregate, is **not a behavior most individual traders share**.

**Interpretation**: the aggregate contrarian signal appears to be driven by a subset of traders who perform well specifically during high-fear conditions (possibly buying dips or exploiting volatility), rather than reflecting a platform-wide behavioral edge available to any trader.

### 4.3 Position sizing also shifts with sentiment

Average trade size is highest during Fear ($7,816) — nearly 60% larger than during Greed ($5,737) or Extreme Greed ($3,112). This means part of the higher absolute PnL during Fear reflects traders **sizing up**, not purely a higher win rate. Win rate alone (88.6% Fear vs 76.1% Greed) still shows a real edge, but the magnitude of the PnL gap is partly a sizing effect.

### 4.4 Extreme Fear results are low-sample and event-driven

Extreme Fear conditions occurred on only **14 of 480 trading days** in the dataset, and just 2 of those days (2025-04-09 and 2025-03-11) account for ~34% of all Extreme Fear trade volume. This suggests Extreme Fear metrics are driven by a small number of acute market events rather than a stable, repeatable pattern, and should be treated as directional/anecdotal rather than a robust trend — unlike Fear, Greed, and Neutral, which each have 67–193 days of support.

## 5. Limitations

- Dataset covers only 32 accounts, all from a single platform (Hyperliquid) — findings may not generalize to the broader market or retail trader population.
- PnL concentration among a few accounts means aggregate statistics are sensitive to a small number of large traders.
- Sentiment is measured at the daily/market level, not trader-specific — it does not capture whether an individual trader was aware of or reacting to that sentiment.
- Extreme Fear findings are based on a small number of trading days and should not be treated with the same confidence as Fear/Greed/Neutral results.

## 6. Recommendations for Trading Strategy

1. **Do not treat "buy the Fear" as a universal rule.** Only 45% of accounts benefit from it — applying it blindly without evidence of a trader's own historical edge in Fear conditions could reduce, not improve, performance.
2. **Position-sizing discipline matters more during Greed regimes**, where win rates are lower (76.1%) — reducing size or tightening risk controls during Greed periods may reduce downside more effectively than trying to "beat" the regime.
3. **Any Fear-based strategy should be validated per-trader**, not applied platform-wide — the accounts that reliably outperform in Fear (e.g., accounts with the largest positive Fear-minus-Greed gap) could be studied further as a model for a specific tactical approach, rather than generalized to all traders.
4. **Treat Extreme Fear signals cautiously** given the thin sample (14 days) — a strategy calibrated on Extreme Fear data risks overfitting to two large market events rather than a repeatable pattern.
