# Dividend Adjustment Effects on Black–Scholes–Merton Option Pricing Accuracy

The project compares two otherwise identical Black–Scholes–Merton specifications:

- **BSM with `q = 0`**: dividends are ignored;
- **BSM with `q > 0`**: the observed continuous dividend yield is included.

## Main interpretation

The theoretical direction of the dividend effect is not the empirical contribution: for a positive dividend yield, the dividend-adjusted call value is lower than the otherwise identical zero-dividend value. The empirical questions are **how large that omission effect is** and **whether correcting it moves model prices closer to observed market prices**.

## Data

### Option snapshots

The empirical sample contains **13 option-chain snapshots** covering:

- August 29, 2025;
- September 2, 3, 4, 8, 9, 10, 11, 12, 15, 16, 17, and 18, 2025.

All contracts share the September 19, 2025 expiration used in the study.

The nine underlyings are:

```text
AAPL, BAC, JPM, MRK, MSFT, NVDA, PFE, UNH, WFC
```

The option-chain data were collected from Yahoo Finance and frozen as CSV snapshots so the replication analysis does not depend on later changes to live web data.

### Historical underlying prices

`historical_closes_presample.csv` contains the pre-sample daily closing prices used to estimate historical volatility independently from option premiums.

The primary specification uses the annualized standard deviation of the final **20 pre-sample daily log returns (HV20)**. HV10 and HV15 are reported as robustness checks.

## Primary empirical specification

### Observed market premium

The primary observed option price is the positive two-sided bid–ask midpoint:

```text
market_mid = (bid + ask) / 2
```

The last transaction price is not the primary market-price proxy; it is retained only in a robustness specification.

### Uniform cleaning rules

A contract-date observation is retained in the primary sample only when:

1. bid and ask are non-missing and strictly positive;
2. `ask >= bid`;
3. stock price `S > 0` and strike `K > 0`;
4. the midpoint is positive and satisfies the call upper bound `midpoint <= S`;
5. dividend yield `q >= 0`;
6. time to expiry is positive;
7. `lastTradeDate` is available;
8. the last trade is no more than **one business session** before the option snapshot.

The rule is applied uniformly across all tickers. It was not designed specifically to remove NVDA observations.

### Time to maturity

The revised analysis uses ACT/ACT for the 2025 sample:

```text
T = actual calendar days to 2025-09-19 / 365
```

### Risk-free rate

The fixed risk-free rate is:

```text
r = 0.0441
```

or **4.41% annualized**, matching the original computational workflow.

### Volatility

The primary volatility input is independent of the option premium:

```text
sigma_HV20 = std(last 20 pre-sample daily log returns) * sqrt(252)
```

Yahoo Finance implied volatility is retained only as a diagnostic/robustness input and is not used as independent evidence of model fit.

## Pricing formulas

For a European call with continuous dividend yield `q`,

```text
C(q) = S exp(-qT) N(d1) - K exp(-rT) N(d2)
```

with

```text
d1 = [ln(S/K) + (r - q + 0.5 sigma^2)T] / [sigma sqrt(T)]
d2 = d1 - sigma sqrt(T)
```

The no-dividend specification sets `q = 0` while holding all other inputs fixed.

The call's dividend-yield sensitivity is

```text
dC/dq = -T S exp(-qT) N(d1)
```

which gives the first-order dividend-omission approximation

```text
C(0) - C(q) ≈ q T S exp(-qT) N(d1).
```

## Error measures

The same error definition is used for both specifications.

Absolute error:

```text
AE = |model price - market midpoint|
```

Ticker-level mean absolute error:

```text
MAE = mean(AE)
```

Normalized MAE for cross-stock comparison:

```text
NMAE = sum(AE) / sum(market midpoint)
```

Positive `MAE improvement %` means that the dividend-adjusted specification has lower MAE:

```text
100 * (MAE_q0 - MAE_q) / MAE_q0
```

## Statistical tests

The inferential analysis does not treat every strike in an option chain as an independent observation. For each ticker, absolute errors are first aggregated to the **ticker × trading-day** level, producing 13 paired daily MAEs.

The notebook reports:

- paired t-test;
- Wilcoxon signed-rank test;
- number of days on which the dividend specification performs better/worse;
- Holm-adjusted p-values across the nine ticker-level tests.

The manuscript interprets these tests conservatively and distinguishes statistical significance from economic magnitude.

## American-style option diagnostic

The observed U.S. single-stock calls are American-style while the closed-form BSM formula prices European calls. To quantify this exercise-style issue, the notebook includes a **100-step Cox–Ross–Rubinstein tree** using the same HV20 and dividend inputs.

The CRR results are a diagnostic of the possible early-exercise component; they are not used to redefine the primary BSM experiment.

## Yahoo implied-volatility diagnostic

The notebook also computes model values using Yahoo's reported implied volatility. These values reproduce observed option prices extremely closely, which illustrates why this specification is not treated as independent validation: implied volatility is inferred from option prices in the first place.


## Reproducibility reference values

A clean run should report:

```text
Raw observations:          9,959
Primary cleaned sample:    5,115
Risk-free rate:            0.0441
Expiry:                    2025-09-19
Activity threshold:        <= 1 business session
Approximation correlation: ~0.999996339
Mean abs. approximation error: ~0.000085666
```

Primary retained sample sizes by ticker are:

| Ticker | N |
|---|---:|
| AAPL | 654 |
| BAC | 388 |
| JPM | 465 |
| MRK | 270 |
| MSFT | 786 |
| NVDA | 1,336 |
| PFE | 189 |
| UNH | 700 |
| WFC | 327 |

If these counts change unexpectedly, first confirm that the 13 option CSVs and `historical_closes_presample.csv` are the same frozen data version used for the revision.

## Key primary pricing result

Using the cleaned sample and independent HV20 volatility, the estimated percentage change in MAE from adding the dividend yield is:

| Ticker | MAE improvement % |
|---|---:|
| AAPL | -0.151 |
| BAC | 3.889 |
| JPM | -16.876 |
| MRK | -12.742 |
| MSFT | -7.619 |
| NVDA | -0.074 |
| PFE | -4.108 |
| UNH | 4.583 |
| WFC | -12.685 |

Positive values indicate lower MAE after the dividend adjustment; negative values indicate higher MAE. This result is why the revised paper does **not** claim that adding dividends systematically improves empirical pricing accuracy.


## Interpretation and limitations

The analysis deliberately separates two questions:

1. **Mechanical model effect:** How much does setting `q=0` change the European BSM call price?
2. **Empirical pricing accuracy:** Does correcting `q` move that model value closer to the observed option market midpoint?

The first is analytically predictable. The second depends on other model misspecifications and market features and is therefore empirical.

Important limitations remain:

- the sample contains only 13 trading days and nine underlyings;
- continuous dividend yield approximates discrete cash-dividend timing;
- HV20 is deliberately independent of option prices but is not a full stochastic-volatility model;
- observed single-stock options are American-style, whereas the primary closed-form BSM comparison is European;
- bid–ask midpoint is a practical valuation proxy rather than an executable transaction price;
- the fixed 4.41% risk-free rate is retained from the original design rather than using a maturity-specific daily term structure.

These limitations should be kept in mind when interpreting the cross-sectional results.

## Data provenance

The option snapshots and underlying-market information used in the project were originally collected from Yahoo Finance. The repository uses frozen local files for reproducibility rather than querying live endpoints during replication.

Please consult Yahoo Finance's applicable terms and data policies before redistributing market data beyond the scope permitted to you.

## Paper

**Dividend Adjustment Effects on Black–Scholes–Merton Option Pricing Accuracy for Short Maturity Equity Call Options**

Authors:

- Abdoulaye Maiga
- Oluwasegun M. Ibrahim
