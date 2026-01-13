# Black–Scholes–Merton Option Pricing: An Empirical Comparison with Market Prices

## Overview
This project empirically evaluates the performance of the Black–Scholes–Merton (BSM) option pricing model for short-maturity U.S. equity call options. Using real market data, the analysis compares observed option prices with theoretical BSM prices and examines the impact of incorporating dividend adjustments on pricing accuracy.

This project is intended as an applied finance and data analysis exercise, demonstrating end-to-end data collection, model implementation, and empirical evaluation.

---

## Data
- **Source:** Yahoo Finance
- **Assets:** 9 U.S. equities (AAPL, MSFT, NVDA, JPM, BAC, WFC, MRK, PFE, UNH)
- **Instruments:** Call options
- **Sample:** 13 trading days
- **Frequency:** Daily option chain snapshots

Each CSV file corresponds to one trading day and contains observed market prices, implied volatility, strike prices, maturities, and dividend-related information and more potentially useful features.

---

## Methodology
1. Collected daily option chain data from Yahoo Finance.
2. Computed theoretical call option prices using:
   - Black–Scholes model (no dividends)
   - Black–Scholes–Merton model (with dividends)
3. Used market-implied volatility as the volatility input.
4. Compared model prices to observed market prices.
5. Evaluated pricing accuracy using relative pricing errors and mean absolute errors.
6. Analyzed how dividend adjustments affect pricing performance across stocks.

---

## Key Findings
- Dividend-adjusted Black–Scholes–Merton prices closely track observed market prices for short-maturity call options.
- Ignoring dividends leads to larger pricing errors for dividend-paying stocks, particularly in financials and healthcare.
- Improvements from dividend adjustments are heterogeneous across stocks and depend on dividend yield levels.
- Significant pricing errors remain for some contracts, reflecting simplifying assumptions such as constant volatility and European-style pricing.

---
