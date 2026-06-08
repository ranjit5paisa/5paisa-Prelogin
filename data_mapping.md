# Stock Page — API Data Source Mapping

| Data Point | Source API |
|---|---|
| `symbol` | Trendlyne `fundamental/` |
| `scriptCode` | Gateway Prelogin `searchscrip` |
| `exchange` | Gateway Prelogin `searchscrip` |
| `exchangeName` | Gateway Prelogin `searchscrip` |
| `stockName` | Trendlyne `overview/` |
| `currentPrice` | Trendlyne `overview/` |
| `priceChange.value` | Trendlyne `overview/` |
| `priceChange.percentage` | Trendlyne `overview/` |
| `priceChange.display` | Trendlyne `overview/` |
| `lastUpdated` | Trendlyne `overview/` |
| `ISIN` | Trendlyne `overview/` |
| `co_code` | Atlas `snapshotcompprofile-version2/` |
| `dayRange.high` | Atlas `getquotedetails/` |
| `dayRange.low` | Atlas `getquotedetails/` |
| `dayRange.progress` | Atlas `getquotedetails/` (calculated) |
| `fiftyTwoWeekRange.high` | Atlas `getquotedetails/` |
| `fiftyTwoWeekRange.low` | Atlas `getquotedetails/` |
| `fiftyTwoWeekRange.progress` | Atlas `getquotedetails/` (calculated) |
| `priceDetails.openPrice` | Atlas `getquotedetails/` |
| `priceDetails.previousClose` | Atlas `getquotedetails/` |
| `priceDetails.volume` | Atlas `getquotedetails/` |
| `EMA List` (5/10/20/50/100/200 Day) | Trendlyne `technical-analysis/` |
| `SMA List` (5/10/20/50/100/200 Day) | Trendlyne `technical-analysis/` |
| `oneMonth.value` | Trendlyne `technical-analysis/` |
| `oneMonth.color` | Trendlyne `technical-analysis/` |
| `threeMonth.value` | Trendlyne `technical-analysis/` |
| `threeMonth.color` | Trendlyne `technical-analysis/` |
| `sixMonth.value` | Trendlyne `technical-analysis/` |
| `sixMonth.color` | Trendlyne `technical-analysis/` |
| `oneYear.value` | Trendlyne `technical-analysis/` |
| `oneYear.color` | Trendlyne `technical-analysis/` |
| `pivotPoint` | Trendlyne `technical-analysis/` |
| `firstResistanceR1` | Trendlyne `technical-analysis/` |
| `firstSupportS1` | Trendlyne `technical-analysis/` |
| `secondResistanceR2` | Trendlyne `technical-analysis/` |
| `secondSupportS2` | Trendlyne `technical-analysis/` |
| `thirdResistanceR3` | Trendlyne `technical-analysis/` |
| `thirdSupportS3` | Trendlyne `technical-analysis/` |
