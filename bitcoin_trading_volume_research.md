# Bitcoin Trading Volume Research

Date prepared: 2026-06-28

## Executive summary

- On 2026-06-28, CoinGecko showed Bitcoin 24-hour trading volume around **$15.2 billion**, equivalent to roughly **253,000 BTC/day**, **10,600 BTC/hour**, or **176 BTC/minute** at a spot price near $60,050.
- Across the 19 daily observations from 2026-06-09 through 2026-06-27, reported Bitcoin volume averaged about **471,000 BTC/day**, or **19,600 BTC/hour** and **327 BTC/minute**.
- The same sample ranged from about **262,000 BTC/day** to **723,000 BTC/day**. The population standard deviation was about **136,000 BTC/day**, with a coefficient of variation near **29%**.
- CME Bitcoin futures and monthly options settle to the CME CF Bitcoin Reference Rate (BRR) at 4:00 p.m. London time on the last Friday of the contract month. Deribit Bitcoin futures and options generally expire at 08:00 UTC and use a 30-minute TWAP of the Deribit index from 07:30 to 08:00 UTC.

## Scope and method

This note covers **Bitcoin only**. Volume estimates use reported exchange trading volume and should be treated as market-activity indicators, not a perfect measure of unique investor turnover.

Reported crypto volume is usually a rolling 24-hour figure. BTC-denominated volume is estimated as:

```text
BTC traded per day = reported USD volume / BTC USD price
BTC traded per hour = BTC traded per day / 24
BTC traded per minute = BTC traded per day / 1,440
```

## Current Bitcoin volume snapshot

Using the CoinGecko Bitcoin page observed on 2026-06-28:

| Metric | Value |
| --- | ---: |
| BTC price used | $60,050.67 |
| 24-hour reported trading volume | $15,212,163,698 |
| Estimated BTC traded per day | 253,322 BTC |
| Estimated BTC traded per hour | 10,555 BTC |
| Estimated BTC traded per minute | 176 BTC |

This snapshot is lower than several immediately preceding days, so it should be treated as a point-in-time value rather than a stable baseline.

## Recent daily volume conversion

The table below converts recent reported daily USD volume into BTC terms using each day's close price.

| Date | Reported volume (USD) | Close price (USD/BTC) | Est. BTC/day | Est. BTC/hour | Est. BTC/minute |
| --- | ---: | ---: | ---: | ---: | ---: |
| 2026-06-27 | $39,391,565,258 | $59,943 | 657,150 | 27,381 | 456 |
| 2026-06-26 | $40,411,370,315 | $59,982 | 673,725 | 28,072 | 468 |
| 2026-06-25 | $43,197,491,544 | $59,713 | 723,419 | 30,142 | 502 |
| 2026-06-24 | $29,785,354,921 | $60,909 | 489,014 | 20,376 | 340 |
| 2026-06-23 | $26,433,062,359 | $62,652 | 421,903 | 17,579 | 293 |
| 2026-06-22 | $16,735,692,357 | $63,957 | 261,671 | 10,903 | 182 |
| 2026-06-21 | $17,171,401,473 | $63,232 | 271,562 | 11,315 | 189 |
| 2026-06-20 | $23,172,898,934 | $64,240 | 360,724 | 15,030 | 251 |
| 2026-06-19 | $31,279,738,052 | $63,514 | 492,486 | 20,520 | 342 |
| 2026-06-18 | $32,345,181,753 | $62,900 | 514,232 | 21,426 | 357 |
| 2026-06-17 | $25,567,698,998 | $64,423 | 396,872 | 16,536 | 276 |
| 2026-06-16 | $33,915,183,708 | $65,599 | 517,008 | 21,542 | 359 |
| 2026-06-15 | $22,887,575,560 | $66,301 | 345,207 | 14,384 | 240 |
| 2026-06-14 | $17,522,802,429 | $65,714 | 266,653 | 11,110 | 185 |
| 2026-06-13 | $28,141,339,067 | $64,378 | 437,127 | 18,214 | 304 |
| 2026-06-12 | $30,075,017,169 | $63,538 | 473,339 | 19,722 | 329 |
| 2026-06-11 | $27,715,292,455 | $63,552 | 436,104 | 18,171 | 303 |
| 2026-06-10 | $40,990,944,564 | $61,493 | 666,595 | 27,775 | 463 |
| 2026-06-09 | $34,027,346,086 | $61,658 | 551,872 | 22,995 | 383 |

## Volume variance and dispersion

For the 19-day sample above:

| Metric | USD volume/day | BTC/day | BTC/hour | BTC/minute |
| --- | ---: | ---: | ---: | ---: |
| Mean | $29.51 billion | 471,403 | 19,642 | 327 |
| Minimum | $16.74 billion | 261,671 | 10,903 | 182 |
| Maximum | $43.20 billion | 723,419 | 30,142 | 502 |
| Range | $26.46 billion | 461,748 | 19,239 | 321 |
| Population variance | 6.11e19 USD^2 | 18.56 billion BTC^2 | 32.22 million BTC^2 | 8,951 BTC^2 |
| Population standard deviation | $7.81 billion | 136,236 | 5,676 | 95 |
| Coefficient of variation | 26.5% | 28.9% | 28.9% | 28.9% |

Practical reading:

- A normal recent day in this sample was near **471,000 BTC of reported turnover**.
- A high-volume day exceeded **700,000 BTC**, or more than **500 BTC per minute** on a simple 24-hour average.
- A low-volume day was closer to **260,000 BTC**, or about **180 BTC per minute**.
- Real intraday flow is uneven. Activity tends to cluster around macro announcements, US market hours, Asia/Europe overlap, liquidations, ETF or treasury-related news, and derivatives expiry windows.

## Why Bitcoin volume changes through the day

1. **Time-zone overlap:** Volume often rises when Europe and the US are both active.
2. **Macro data and risk sentiment:** Inflation data, central-bank communications, employment reports, dollar moves, and Treasury yields can affect BTC order flow.
3. **Leverage and liquidations:** Perpetual swaps and futures can amplify spot moves through forced liquidations and margin adjustments.
4. **ETF, treasury, and institutional flows:** Spot ETF creations/redemptions, treasury accumulation, and institutional rebalancing can affect exchange liquidity.
5. **Derivatives expiry and options positioning:** Dealers hedging expiring options may buy or sell spot/futures as BTC approaches large strike concentrations.
6. **Exchange and stablecoin liquidity:** BTC volume is often paired against USDT, USDC, USD, and other quote currencies.

## Bitcoin futures expiry conditions

### CME Bitcoin futures

- **Final trading time:** Trading in expiring futures terminates at **4:00 p.m. London time on the last Friday of the contract month** if that day is a business day in either the UK or the US.
- **Holiday adjustment:** If the last Friday is not a business day in both the UK and the US, trading terminates on the preceding day that is a business day in either the UK or the US.
- **Settlement type:** Cash settlement.
- **Final settlement price:** The **CME CF Bitcoin Reference Rate (BRR)** published at 4:00 p.m. London time on the last trade date.
- **Disruption handling:** If the BRR is not publishable or not published on the termination day, CME rules allow the exchange to defer or postpone final settlement for up to 14 consecutive calendar days.

### Deribit Bitcoin futures

- **Daily futures:** Expire every day at **08:00 UTC**.
- **Weekly futures:** Expire every Friday at **08:00 UTC**.
- **Monthly futures:** Expire on the last Friday of the month at **08:00 UTC**.
- **Final settlement:** Cash-settled at the official delivery price.
- **Delivery price:** 30-minute TWAP of the relevant Deribit index from **07:30 to 08:00 UTC**.

## Bitcoin options expiry conditions

### CME options on Bitcoin futures

- **Style:** European exercise.
- **Underlying:** One CME Bitcoin futures contract.
- **Monthly expiry:** Last Friday of the contract month.
- **Trading termination:** 4:00 p.m. London time on the last Friday of the contract month, subject to the same UK/US business-day adjustment.
- **Settlement at expiration:** In-the-money options are automatically exercised into the underlying expiring cash-settled futures contract, which settles to the CME CF BRR.
- **Final settlement reference:** CME CF BRR at 4:00 p.m. London time on expiration day.

### Deribit Bitcoin options

- **Style:** European.
- **Daily options:** Expire daily at **08:00 UTC**.
- **Weekly options:** Expire each Friday at **08:00 UTC**.
- **Monthly options:** Expire on the last Friday of each calendar month at **08:00 UTC**.
- **Quarterly options:** Expire on the last Friday of each calendar quarter at **08:00 UTC**.
- **Delivery price:** 30-minute TWAP of the relevant Deribit index from **07:30 to 08:00 UTC**.
- **Settlement:** Cash-settled economics. In-the-money options are settled based on intrinsic value against the delivery price; out-of-the-money options expire worthless.

## Research limitations

- The volume calculations use reported exchange volume and closing prices, so they are estimates.
- The table does not separate spot volume from derivatives volume. For exact segmentation, use a provider that separates spot, perpetual swaps, dated futures, and options by venue.
- BTC traded per minute and per hour are simple averages derived from daily totals. They do not represent actual minute-by-minute tape data.
- Different venues can have different settlement calendars, holiday adjustments, contract multipliers, margin currencies, and fee rules.

## Sources

- CoinGecko Bitcoin page and historical data: https://www.coingecko.com/en/coins/bitcoin
- Glassnode Bitcoin spot volume definition: https://studio.glassnode.com/charts/market.SpotVolumeDailySumAll?a=BTC
- CME Bitcoin futures rulebook, Chapter 350: https://www.cmegroup.com/rulebook/CME/IV/350/350.pdf
- CME options on Bitcoin futures FAQ: https://www.cmegroup.com/trading/cryptocurrency-indices/cme-options-bitcoin-futures-frequently-asked-questions.html
- Deribit contract introduction policy: https://support.deribit.com/hc/en-us/articles/25944688876957-Contract-Introduction-Policy
- Deribit settlement policy: https://support.deribit.com/hc/en-us/articles/29734325712413-Settlement
