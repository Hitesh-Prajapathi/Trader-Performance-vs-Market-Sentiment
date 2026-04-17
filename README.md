# Trader-Performance-vs-Market-Sentiment

Analysis of how the Bitcoin Fear & Greed Index relates to trader behavior and performance on Hyperliquid, plus a segment-aware risk model.

## 1. Objective

- Link **market sentiment (Fear/Greed)** with:
  - Trader **performance**: PnL, win rate.
  - Trader **behavior**: trade frequency, size, BUY/SELL activity.
  - **Segments**: frequent vs infrequent, high vs low win-rate.
- Build a **classification model** to predict whether an account–day is profitable.

## 2. Data

- **Fear/Greed index**  
  - Daily: `timestamp`, `value`, `classification`, `date`.
- **Hyperliquid trades**  
  - Per trade: `Account`, `Side`, `Size USD`, `Closed PnL`, `Timestamp IST`, etc.

Both datasets are aligned at daily `date` level.

## 3. Approach (very brief)

1. **Prep & merge**
   - Parse `Timestamp IST` → `date`, align with sentiment `date`.
   - Build **market-level** table `daily` (PnL, trades, avg size, BUY/SELL + sentiment).
   - Build **account-level** table `account_daily` (account_daily_pnl, trades_per_day, avg_trade_size_usd, win_rate + sentiment).

2. **Segmentation & EDA**
   - Segments:
     - Activity: `frequent` vs `infrequent` (median trades/day).
     - Performance: `high_win_rate` vs `low_win_rate` (≥ 0.6 vs < 0.6).
   - Compare PnL, trades/day, win-rate across:
     - sentiment buckets,
     - activity segments,
     - performance segments.

3. **Model**
   - Target: `is_profitable` (account_daily_pnl > 0).
   - Features per account–day:
     - `trades_per_day`, `avg_trade_size_usd`, `win_rate`,
     - `BUY`, `SELL`, `buy_sell_ratio`,
     - one-hot segments + sentiment.
   - Model: **RandomForestClassifier** (200 trees, basic regularization).
   - 80/20 train–validation split (stratified).

## 4. Key Findings

- **Fear vs Greed**
  - Extreme Fear / Fear days have the **highest average daily PnL**.
  - Greed days have the **weakest PnL**, despite positive sentiment.
- **Segments**
  - Frequent and high-win-rate traders are profitable across all sentiments (especially in Fear).
  - Low-win-rate traders lose money in every regime, with largest losses on Greed/Neutral days.
- **Model**
  - Random Forest reaches ~**96% accuracy** and **ROC-AUC ≈ 0.99** on validation.
  - Most important features: `win_rate`, segment flags, BUY/SELL activity, trades/day.  
    Sentiment adds secondary but non-zero signal.

## 5. Strategy Ideas (high level)

- Use Fear regimes as opportunity for **skilled, frequent** traders, with controlled sizing.
- De-risk **low-win-rate** traders on Greed/Neutral days (caps on trades and size).
- Apply **dynamic limits** based on segment + sentiment (max trades, max size, daily loss).

## 6. How to Run

1. Open `Bitcoin_trade_sentiment.ipynb` in Jupyter/Colab.
2. Place `fear_greed_index.csv` and `historical_data.csv` in the same environment.
3. Run cells from top to bottom.
