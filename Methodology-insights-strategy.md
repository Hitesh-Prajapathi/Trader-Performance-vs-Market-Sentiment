## Methodology, Insights, and Strategy (Summary)

### Methodology

1. **Data preparation**
   - Parsed Hyperliquid trade timestamps (`Timestamp IST`) to dates and aligned them with the Fear & Greed index dates.
   - Built:
     - `daily` (market-level): daily PnL, number of trades, average trade size, BUY/SELL counts, sentiment classification.
     - `account_daily` (account-level): account_daily_pnl, trades_per_day, avg_trade_size_usd, wins, losses, win_rate, sentiment.

2. **Segmentation**
   - Activity segment: `frequent` vs `infrequent` traders (above vs below median trades_per_day).
   - Performance segment: `high_win_rate` vs `low_win_rate` (win_rate ≥ 0.6 vs < 0.6).

3. **Analysis**
   - Compared PnL, trades_per_day, avg_trade_size_usd, BUY/SELL counts across sentiment buckets.
   - Compared PnL and win_rate across sentiment × activity segment and sentiment × performance segment.

4. **Modeling**
   - Target: `is_profitable` (1 if account_daily_pnl > 0, else 0).
   - Features: trades_per_day, avg_trade_size_usd, win_rate, BUY, SELL, buy_sell_ratio, segment dummies, sentiment one‑hot.
   - Model: RandomForestClassifier (200 trees), 80/20 train–validation split, stratified.

---

### Key Insights

1. **Performance vs sentiment**
   - Extreme Fear and Fear days show the highest average daily PnL; Greed days have the weakest profitability.

2. **Behavior vs sentiment**
   - Trade counts are highest in Fear/Extreme Fear regimes, while average trade size is smallest on Extreme Fear and largest on Neutral/Greed/Fear days.

3. **Segments**
   - Frequent traders and high‑win‑rate traders are consistently profitable across all sentiments, especially in Fear.
   - Low‑win‑rate traders are loss‑making in every regime, with largest losses on Greed and Neutral days.

4. **Model**
   - Random Forest achieves ~96% accuracy and ROC‑AUC ≈ 0.99.
   - Win_rate and segment features dominate feature importance; sentiment is informative but secondary.

---

### Strategy Recommendations

1. **Fear regimes as opportunity (for skilled traders)**
   - Allow active/high‑win‑rate traders to stay active or slightly increase activity in Fear/Extreme Fear, but with moderate position sizes and strict loss limits.
   - Advise low‑win‑rate/infrequent traders to avoid impulsive trading and use smaller size during Extreme Fear.

2. **De‑risk low‑win‑rate traders on Greed/Neutral days**
   - Impose tighter caps on trade count and position size for low‑win‑rate traders when sentiment is Greed/Neutral to avoid overtrading in optimistic environments.

3. **Segment‑aware risk controls**
   - Continuously classify accounts into (frequent/infrequent, high/low win‑rate) and adjust max trades, max size, and daily loss limits dynamically based on both segment and current sentiment regime.
