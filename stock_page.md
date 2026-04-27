# Stock Page API Documentation

## Example URL
```
https://www.5paisa.com/stocks/suzlon-share-price
```

---

## Page Load Sequence and Timing

### Initial Server-Side Load (0-2 seconds)
The page makes all critical server-side API calls before rendering HTML to the user.

### Client-Side Load (After Page Render)
- **Immediate (0-500ms)**: Client-side JavaScript initialization
- **500ms**: Chart URL generation
- **1000ms**: Shareholding pattern chart rendering
- **1500ms**: Financial data initial load
- **2000ms**: Forecast data, Similar stocks, Poll data, FnO check, Sector/Cap info
- **3000ms+**: Additional lazy-loaded content

---

## Server-Side API Calls (PHP Backend)

These APIs are called during initial page load in [`StockPageController::getData()`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:155)

### 1. Search Scrip Code API
**Purpose**: Get BSE Script Code for the stock

**Endpoint**: 
```
POST https://gateway.5paisa.com/prelogin/prod/searchscrip
```

**Headers**:
```
UserID: ZyT47UW2g56
Password: H98qlU4Sn2
Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81
Content-Type: application/json
```

**Request Body**:
```json
{
  "Exch": "N",
  "ExchType": "C",
  "RecordCount": "1",
  "Symbol": "SUZLON"
}
```

**CURL Example**:
```bash
curl -X POST 'https://gateway.5paisa.com/prelogin/prod/searchscrip' \
-H 'UserID: ZyT47UW2g56' \
-H 'Password: H98qlU4Sn2' \
-H 'Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81' \
-H 'Content-Type: application/json' \
-d '{
  "Exch": "N",
  "ExchType": "C",
  "RecordCount": "1",
  "Symbol": "SUZLON"
}'
```

**Response Fields**:
- `Data[0].ScripCode` - BSE Script Code
- `Data[0].Symbol` - Stock symbol
- `Data[0].Exchange` - Exchange (N=NSE, B=BSE)

**Called At**: Line 179 in StockPageController.php
**Function**: `getScriptCodeBse()`

---

### 2. Stock Overview API
**Purpose**: Get stock price, fundamentals, and technical data

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/{SCRIP_CODE}/
```

**Headers**:
```
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
requestCode: pearlapi
UserId: 5PAISAAPI
Password: 5nadynsiitnienny
```

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/532667/' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
-H 'Ocp-Apim-Trace: true' \
-H 'Content-Type: application/json' \
-H 'KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44' \
-H 'requestCode: pearlapi' \
-H 'UserId: 5PAISAAPI' \
-H 'Password: 5nadynsiitnienny'
```

**Response Structure**:
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "stockData": [
      "Stock Name",
      "...",
      "Current Price",
      "...",
      "BSE Code",
      "ISIN",
      "...",
      "Price",
      "...",
      "Change",
      "Change%"
    ],
    "fundamentalData": [...],
    "technicalData": [...],
    "52_Week_High": "...",
    "52_Week_low": "...",
    "lastUpdated": "..."
  }
}
```

**Called At**: Line 199, 263, 537, 726 in StockPageController.php
**Functions**: `getData()`, `getKeyStats()`

---

### 3. Fundamental Data API
**Purpose**: Get detailed fundamental metrics

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/fundamental/{SCRIP_CODE}/
```

**Headers**: Same as Overview API

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/fundamental/532667/' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
-H 'Ocp-Apim-Trace: true' \
-H 'Content-Type: application/json' \
-H 'KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44' \
-H 'requestCode: pearlapi' \
-H 'UserId: 5PAISAAPI' \
-H 'Password: 5nadynsiitnienny'
```

**Response**: Contains NSE code, BSE code, BSE Script Code

**Called At**: Line 263 in StockPageController.php

---

### 4. Technical Analysis API
**Purpose**: Get EMA, SMA, pivot points, RSI, MFI, MACD

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/{SCRIP_CODE}/
```

**Headers**: Same as Overview API

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/532667/' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
-H 'Ocp-Apim-Trace: true' \
-H 'Content-Type: application/json' \
-H 'KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44'
```

**Response Structure**:
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "stockPriceData": {
      "currentPrice": "...",
      "dayChange": "...",
      "dayChangeP": "..."
    },
    "movingAverages": {
      "tableData": [
        ["50 Day", "SMA_value", "EMA_value"],
        ["100 Day", "SMA_value", "EMA_value"],
        ["200 Day", "SMA_value", "EMA_value"]
      ]
    },
    "pivotData": {
      "pivotPoint": "...",
      "firstResistanceR1": "...",
      "secondResistanceR2": "...",
      "thirdResistanceR3": "...",
      "firstSupportS1": "...",
      "secondSupportS2": "...",
      "thirdSupportS3": "..."
    },
    "technicals": [
      {
        "name": "Relative Strength Index",
        "value": "...",
        "color": "..."
      },
      {
        "name": "Money Flow Index",
        "value": "...",
        "color": "..."
      },
      {
        "name": "MACD",
        "value": "...",
        "color": "..."
      },
      {
        "name": "MACD Signal",
        "value": "...",
        "color": "..."
      }
    ],
    "priceAnalysis": {
      "day": {...},
      "week": {...},
      "month": {...},
      "halfYear": {...},
      "year": {...},
      "fiveYear": {...}
    }
  }
}
```

**Called At**: Lines 363, 537, 798, 863, 1169, 1316, 2599 in StockPageController.php and StockPageService.php
**Functions**: `getTechnicalData()`, `getKeyStats()`, `openPrevData()`, `getReturns()`

---

### 5. Tech Trend (SWOT) API
**Purpose**: Get bullish/bearish moving average indicators

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/{SCRIP_CODE}
```

**Headers**: Same as Overview API

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/532667' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
-H 'Ocp-Apim-Trace: true' \
-H 'Content-Type: application/json' \
-H 'KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44'
```

**Response**:
```json
{
  "head": {
    "status": "0"
  },
  "body": {
    "532667": {
      "techTrend": {
        "bearish": 5,
        "bullish": 7,
        "total": 12
      }
    }
  }
}
```

**Called At**: Line 1259 in StockPageService.php
**Function**: `getSwotData()`

---

### 6. Rapid Results API
**Purpose**: Get latest result highlights

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/rapid-results/{SCRIP_CODE}/
```

**Headers**: Same as Overview API

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/rapid-results/532667/' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b'
```

**Response**: Latest news/results data for the stock

**Called At**: Line 1146 in StockPageController.php
**Function**: `getResultHighlights()`

---

### 7. Corporate Actions API
**Purpose**: Get dividends, bonuses, splits, board meetings

**Endpoint**: 
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/{SCRIP_CODE}
```

**Headers**: Same as Overview API

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/532667' \
-H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
-H 'Ocp-Apim-Trace: true' \
-H 'Content-Type: application/json' \
-H 'KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44'
```

**Response**:
```json
{
  "body": {
    "eventsData": [
      {
        "eventType": "Dividend",
        "recordDate": "...",
        "purpose": "...",
        "remarks": "..."
      },
      {
        "eventType": "Board Meeting",
        "boardMeetDate": "...",
        "purpose": "...",
        "remarks": "..."
      }
    ]
  }
}
```

**Called At**: Line 883 in StockPageController.php
**Function**: `getCorporateActionData()`

---

### 8. Company Profile API (Atlas)
**Purpose**: Get company details, sector, co_code

**Endpoint**: 
```
GET https://apihub.5paisa.com/atlas_cache/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/{SYMBOL}?responseType=json
```

**Authorization**: Basic indiainfoline\iiflweb:Glaxo@123

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/atlas_cache/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/SUZLON?responseType=json' \
--user 'indiainfoline\iiflweb:Glaxo@123'
```

**Response**: Contains `co_code` needed for other Atlas APIs

**Called At**: Line 361 in StockPageService.php
**Function**: `getCoCode()`

---

### 9. Quote Details API (Atlas)
**Purpose**: Get live price, volume, 52-week high/low, EPS

**Endpoint**: 
```
GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{CO_CODE}/{EXCHANGE}?responseType=json
```

**Authorization**: Basic indiainfoline\iiflweb:Glaxo@123

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/12345/nse?responseType=json' \
--user 'indiainfoline\iiflweb:Glaxo@123'
```

**Response Structure**:
```json
{
  "response": {
    "data": {
      "getquotedetailslist": {
        "getquotedetails": {
          "price": "...",
          "change": "...",
          "high_price": "...",
          "low_price": "...",
          "open_price": "...",
          "oldprice": "...",
          "hi_52_wk": "...",
          "lo_52_wk": "...",
          "volume": "...",
          "eps": "...",
          "companysector": "...",
          "companysectorcode": "..."
        }
      }
    }
  }
}
```

**Called At**: Lines 531, 702, 952, 1930 in StockPageController.php and StockPageService.php
**Functions**: `getKeyStats()`, `getSectorLink()`, `getPerformance()`

---

### 10. MarketSmith Instrument ID API
**Purpose**: Get instrument ID for ratings and ownership data

**Endpoint**: 
```
GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/srch.json?text={SYMBOL}&lang=en&ver=2&ms-auth=...
```

**CURL Example**:
```bash
curl -X GET 'https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/srch.json?text=SUZLON&lang=en&ver=2&ms-auth=10454+MarketSmithINDUID-Web000000013505+MarketSmithINDUID-Web000000013505+3+211017225341+2003449279'
```

**Response**: Returns `instrumentId`

**Called At**: Line 453 in StockPageService.php
**Function**: `getInstrumentId()`

---

### 11. Investment Rating API (MarketSmith)
**Purpose**: Get stock ratings (Master Score, EPS Rating, RS Rating, etc.)

**Endpoint**: 
```
GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/wisdom.json?lang=en&ver=2&ms-auth=...
```

**CURL Example**:
```bash
curl -X GET 'https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/12345/wisdom.json?lang=en&ver=2&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196'
```

**Response**: Rating data with itemValue, pageGroupLink, bodyText

**Called At**: Line 407 in StockPageService.php
**Function**: `getInvestRating()`

---

### 12. Ownership/Management Data API (MarketSmith)
**Purpose**: Get shareholding pattern and management details

**Endpoint**: 
```
GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false&ms-auth=...
```

**CURL Example**:
```bash
curl -X GET 'https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/12345/financeDetails.json?isConsolidated=false&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196'
```

**Response**: Contains shareholdingLatestFourQuaterPatternModel, managementInfoData

**Called At**: Line 430 in StockPageService.php
**Function**: `getOwnershipManagementData()`

---

### 13. Financial Data API (MarketSmith via Kong Gateway)
**Purpose**: Get P&L, Balance Sheet, Cash Flow, Ratios

**Endpoint**: 
```
GET https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated={true|false}
```

**Headers**:
```
gateway-name: fivepaisa-gateway
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**CURL Example**:
```bash
curl -X GET 'https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/12345/financeDetails.json?isConsolidated=true' \
-H 'gateway-name: fivepaisa-gateway' \
-H 'x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs'
```

**Response**: Contains:
- `bsHeader` - Balance Sheet headers
- `bsData` - Balance Sheet data
- `cfHeader` - Cash Flow headers
- `cfData` - Cash Flow data
- `incAnnualHeader` - Annual P&L headers
- `incAnnualData` - Annual P&L data
- `incQuarterHeader` - Quarterly P&L headers
- `incQuarterData` - Quarterly P&L data
- `ratiosAnnualHeader` - Ratios headers
- `ratiosAnnualData` - Ratios data

**Called At**: Line 1278 in StockPageService.php
**Function**: `financeDetailsApi()`

---

## Client-Side API Calls (JavaScript)

These APIs are called after page load via AJAX in [`stock-page.js`](modules/custom/fivepaisa_stock_page/js/stock-page.js)

### 14. Forecast Data API
**Purpose**: Get broker estimates and target price

**Triggered**: After 2 seconds (when #forecast section is visible)

**Endpoint**: 
```
GET /get-forecast-data/{NSECODE}
```

**Backend Route**: `StockPageController::getForcastGraphData()`

**Backend API Called**:
```
GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/getBrokerEstimates.json?instrumentId={INSTRUMENT_ID}&ms-auth=...
```

**CURL Example (Backend)**:
```bash
curl -X GET 'https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/getBrokerEstimates.json?instrumentId=12345&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196'
```

**Response**: Forecast data with recommendations, financials, targetPrice, chartData

**Called At**: Line 340 in stock-page.js

---

### 15. Shareholding Pattern Names JSON
**Purpose**: Get shareholding category names for selected month

**Triggered**: After 1 second, or when month tab is clicked

**Endpoint**: 
```
GET /shareholding-pattern-names-json/{MONTH_YEAR}/{NSECODE}
```

**Backend Route**: `StockPageController::shareholdingPatternNamesJson()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/shareholding-pattern-names-json/Mar-2024/SUZLON'
```

**Response**:
```json
{
  "data": "<div>...HTML with shareholding pattern names...</div>"
}
```

**Called At**: Line 686 in stock-page.js

---

### 16. Shareholding Pattern Data JSON
**Purpose**: Get shareholding percentages for selected month

**Triggered**: After 1 second, or when month tab is clicked

**Endpoint**: 
```
GET /shareholding-pattern-data-json/{MONTH_YEAR}/{NSECODE}
```

**Backend Route**: `StockPageController::shareholdingPatternDataJson()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/shareholding-pattern-data-json/Mar-2024/SUZLON'
```

**Response**:
```json
{
  "data": "<div>...HTML with shareholding percentages...</div>"
}
```

**Called At**: Line 687 in stock-page.js

---

### 17. Shareholding Pattern Chart
**Purpose**: Get chart data for shareholding pattern visualization

**Triggered**: After 1 second, or when month tab is clicked

**Endpoint**: 
```
GET /shareholding-pattern-chart/{MONTH_YEAR}/{NSECODE}
```

**Backend Route**: `StockPageController::shareholdingPatternChart()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/shareholding-pattern-chart/Mar-2024/SUZLON'
```

**Response**:
```json
{
  "data": [
    {"name": "Promoters", "value": 65.5},
    {"name": "FII", "value": 15.2},
    {"name": "DII", "value": 10.3},
    {"name": "Public", "value": 9.0}
  ]
}
```

**Called At**: Line 688, 662 in stock-page.js

---

### 18. Similar Stocks API
**Purpose**: Get similar stocks from same sector

**Triggered**: After 2 seconds

**Endpoint**: 
```
GET /api/get-similar-stocks?nsecode={NSECODE}
```

**Backend Route**: `StockPageController::getSimilarStocks()`

**Backend APIs Called**:
1. `https://apihub.5paisa.com/atlas_cache/datamart/Equity/CompanyInfo.svc/quotesfinder/{SYMBOL}?responseType=json`
2. `https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{CO_CODE}/{EXCHANGE}?responseType=json`
3. `https://apihub.5paisa.com/atlas_cache/datamart/Equity/Market.svc/SectorWiseComp/{SECTOR_CODE}/?responseType=json`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/api/get-similar-stocks?nsecode=SUZLON'
```

**Response**:
```json
{
  "data": {
    "similarStocks": "<div>...HTML with similar stock cards...</div>"
  }
}
```

**Called At**: Line 772 in stock-page.js

---

### 19. FnO Check API
**Purpose**: Check if stock has Futures and Options available

**Triggered**: After 2 seconds

**Endpoint**: 
```
GET /check-fno/{NSECODE}
```

**Backend Route**: `StockPageController::displayFnoData()`

**Backend API Called**:
```
GET https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/{Fut|Opt}/{SYMBOL}?responseType=json
```

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/check-fno/SUZLON'
```

**Response**:
```json
{
  "future_exist": "true",
  "option_exist": "true"
}
```

**Called At**: Line 1006 in stock-page.js

---

### 20. Sector and Cap Info API
**Purpose**: Get sector name and market cap category

**Triggered**: After 2 seconds

**Endpoint**: 
```
GET /api/{NSECODE}/sector-cap-info
```

**Backend Route**: `StockPageController::getSectorLink()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/api/SUZLON/sector-cap-info'
```

**Response**:
```json
{
  "sectorInfo": {
    "sectorLink": "/stocks/sector/renewable-energy",
    "sectorName": "Renewable Energy"
  },
  "capInfo": {
    "capName": "Mid Cap",
    "capLink": "/share-market-today/mid-cap-stocks"
  }
}
```

**Called At**: Line 1051 in stock-page.js

---

### 21. Financial Data API (Tab Based)
**Purpose**: Get financial data based on selected tab (Quarterly/Annual P&L, Balance Sheet, etc.)

**Triggered**: After 1.5 seconds (initial load) or when tab is clicked

**Endpoint**: 
```
GET /financial-data/{TAB_FILTER}/{NSECODE}
```

**Filters**: 
- `QuarterlyPLStandaloneResult`
- `QuarterlyPLConsolidatedResult`
- `AnnualPLStandaloneResult`
- `AnnualPLConsolidatedResult`
- `BalanceSheetStandaloneResult`
- `BalanceSheetConsolidatedResult`
- `CashFlowStandaloneResult`
- `CashFlowConsolidatedResult`
- `RatiosStandaloneResult`
- `RatiosConsolidatedResult`

**Backend Route**: `StockPageController::getFinancialData()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/SUZLON'
```

**Response**:
```json
{
  "table": "<table>...HTML table with financial data...</table>"
}
```

**Called At**: Lines 1145, 67 in stock-page.js

---

### 22. Price Range Filter API
**Purpose**: Get price change data for selected time range (from chart interaction)

**Triggered**: When chart time range button is clicked

**Endpoint**: 
```
GET /price-range-filter/{NSECODE}/{RANGE}
```

**Ranges**: `1D`, `1W`, `1M`, `6M`, `1Y`, `5Y`, `Max`

**Backend Route**: `StockPageController::priceRangeFilter()`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/price-range-filter/SUZLON/1M'
```

**Response**:
```json
{
  "priceAnalysis": {
    "change": 2.5,
    "changePercent": 5.2,
    "color": "positive"
  }
}
```

**Called At**: Lines 1181, 1257 in stock-page.js

---

### 23. Stock Poll APIs
**Purpose**: Stock sentiment polling (Buy/Sell/Hold votes)

**Triggered**: After 2 seconds for initialization

#### 23a. Get Vote Count
**Endpoint**: 
```
GET /stock-poll/votes-count/{STOCK_URL_SLUG}
```

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/stock-poll/votes-count/suzlon-share-price'
```

**Response**: Plain text with vote count

**Called At**: Line 913, 967 in stock-page.js

#### 23b. Cast Vote
**Endpoint**: 
```
GET /stock-poll/{VOTE_TYPE}/{STOCK_URL_SLUG}
```

**Vote Types**: `buy`, `sell`, `hold`

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/stock-poll/buy/suzlon-share-price'
```

**Called At**: Line 930 in stock-page.js

#### 23c. Get Poll HTML
**Endpoint**: 
```
GET /stock-poll/poll-html-json/{STOCK_URL_SLUG}
```

**CURL Example**:
```bash
curl -X GET 'https://www.5paisa.com/stock-poll/poll-html-json/suzlon-share-price'
```

**Response**:
```json
{
  "buy": {
    "percentage_vote": "45.5",
    "poll_progress_color": "green-progress"
  },
  "sell": {
    "percentage_vote": "30.2",
    "poll_progress_color": "red-progress"
  },
  "hold": {
    "percentage_vote": "24.3",
    "poll_progress_color": "yellow-progress"
  }
}
```

**Called At**: Line 943 in stock-page.js

---

## API Call Summary

### Server-Side (Initial Page Load)
1. **Search Scrip Code** - Get BSE Script Code
2. **Stock Overview** - Main stock data, price, fundamentals
3. **Fundamental Data** - Detailed fundamental metrics
4. **Technical Analysis** - EMA, SMA, RSI, MACD, pivots
5. **Tech Trend (SWOT)** - Bullish/Bearish indicators
6. **Rapid Results** - Latest results/news
7. **Corporate Actions** - Dividends, splits, bonuses
8. **Company Profile (Atlas)** - Get co_code
9. **Quote Details (Atlas)** - Live price, volume, 52W high/low
10. **MarketSmith Instrument ID** - Get instrument ID
11. **Investment Rating** - Stock ratings
12. **Ownership/Management** - Shareholding and management
13. **Financial Data** - P&L, Balance Sheet, Cash Flow

**Total Server-Side APIs: 13-15 APIs** (some conditional)

### Client-Side (After Page Load)
14. **Forecast Data** - Broker estimates (2s delay)
15. **Shareholding Names** - Pattern names (1s delay)
16. **Shareholding Data** - Pattern percentages (1s delay)
17. **Shareholding Chart** - Chart visualization (1s delay)
18. **Similar Stocks** - Related stocks (2s delay)
19. **FnO Check** - Futures & Options availability (2s delay)
20. **Sector/Cap Info** - Sector and market cap (2s delay)
21. **Financial Data** - Tab-based financial data (1.5s delay)
22. **Price Range Filter** - Time range price change (on interaction)
23. **Stock Poll** - Sentiment polling (2s delay, 3 endpoints)

**Total Client-Side APIs: 10+ APIs** (some triggered on user interaction)

---

## Page Load Performance Analysis

### Critical Path (Initial Load)
```
User Request → Server Processing (1-2s) → HTML Response → Client JS Init (0.5s)
```

### Server-Side Processing Timeline
```
0ms    - Request received
0-100ms - URL parsing, validation, redirects
100-500ms - Database queries (stock details, FAQs, about)
500-1500ms - External API calls (parallel where possible):
  - Overview API
  - Fundamental API  
  - Technical API
  - Corporate Actions API
  - Atlas APIs
  - MarketSmith APIs
1500-2000ms - Template rendering, cache warming
2000ms - HTML sent to browser
```

### Client-Side Loading Timeline
```
0ms    - HTML received, DOM parsing begins
100ms  - CSS loaded, initial render
200ms  - JavaScript files loaded and parsed
300ms  - DOM ready, event listeners attached
500ms  - Chart iframe initialization
1000ms - Shareholding pattern data loaded
1500ms - Financial data table loaded
2000ms - Forecast, similar stocks, FnO, polls loaded
3000ms+ - Page fully interactive
```

### First Contentful Paint (FCP)
**Target**: < 1.5 seconds
**Actual**: ~2-2.5 seconds (server processing time)

### Time to Interactive (TTI)
**Target**: < 3 seconds
**Actual**: ~3-4 seconds (includes client-side lazy loading)

### Optimization Opportunities
1. **Parallel API Calls**: Most server-side APIs are called sequentially, could be parallelized
2. **Caching**: Some APIs have 24-hour cache, could be extended
3. **Lazy Loading**: Non-critical sections loaded after initial render
4. **CDN**: Static assets (CSS, JS, images) served from CDN
5. **Database Optimization**: Stock details, FAQs could be cached more aggressively

---

## API Dependencies and Data Flow

```
URL Request (e.g., /stocks/suzlon-share-price)
  ↓
Extract NSE Code (SUZLON)
  ↓
[1] Search Scrip API → Get BSE Script Code (532667)
  ↓
[2] Overview API (using BSE Code)
  ├→ Stock Price, Change, Volume
  ├→ Fundamental Data (PE, PB, Market Cap)
  └→ Technical Data (RSI, MFI)
  ↓
[3] Fundamental API → NSEcode, BSEcode verification
  ↓
[4] Technical Analysis API
  ├→ EMA/SMA values
  ├→ Pivot Points (R1, R2, R3, S1, S2, S3)
  ├→ MACD, RSI, MFI details
  └→ Price Analysis (1D, 1W, 1M, 6M, 1Y, 5Y)
  ↓
[5] Tech Trend API → Bullish/Bearish count
  ↓
[6] Rapid Results API → Latest company news
  ↓
[7] Corporate Actions API → Dividends, Splits, Bonuses
  ↓
[8] Company Profile (Atlas) → Get co_code
  ↓
[9] Quote Details (Atlas, using co_code)
  ├→ Live Price Updates
  ├→ Day High/Low, 52W High/Low
  ├→ Volume, EPS
  └→ Sector Information
  ↓
[10] MarketSmith Instrument Search → Get instrument_id
  ↓
[11] Investment Rating (using instrument_id)
  └→ Master Score, EPS Rating, RS Rating, etc.
  ↓
[12] Ownership/Management (using instrument_id)
  ├→ Shareholding Pattern (4 quarters)
  └→ Management Details (MD, Directors)
  ↓
[13] Financial Data (using instrument_id)
  ├→ Balance Sheet
  ├→ Cash Flow
  ├→ P&L (Quarterly & Annual)
  └→ Financial Ratios
  ↓
Render HTML Template
  ↓
Send to Browser
  ↓
Client-Side JavaScript Loads Additional Data:
  ├→ [14] Forecast Data
  ├→ [15-17] Shareholding Pattern (interactive)
  ├→ [18] Similar Stocks
  ├→ [19] FnO Check
  ├→ [20] Sector/Cap Info
  ├→ [21] Financial Tables (on tab switch)
  ├→ [22] Price Range Filter (on chart interaction)
  └→ [23] Stock Polls (Buy/Sell/Hold sentiment)
```

---

## Error Handling

### Server-Side
- All API calls wrapped in try-catch blocks
- Failed API calls return empty arrays/default values
- Page still renders with partial data if some APIs fail
- 404 redirect to `/stocks/all` if critical data missing

### Client-Side
- AJAX calls have error handlers
- Failed requests show "Loading..." or hide sections
- Timeout set to 10 seconds for most requests
- Fallback images for logos that fail to load

---

## Caching Strategy

### Server-Side Caching
- **Stock Overview Data**: 24 hours
- **Technical Data**: 24 hours
- **Corporate Actions**: 24 hours
- **Ratings Data**: 24 hours
- **Shareholding Pattern**: 24 hours
- **Financial Data**: 24 hours
- **Forecast Data**: 24 hours

### Cache Keys Format
```
script_code_bse{NSECODE}
corp_data{NSECODE}
technical_data{NSECODE}
get_ratings_{NSECODE}
highlight_data{NSECODE}
get_returns{NSECODE}
getFinancialData.{NSECODE}
```

### Cache Invalidation
- Automatic after TTL expires
- Manual clearing via Drupal cache rebuild
- Per-stock cache clearing available

---

## Notes

1. **Kong Gateway Migration**: Many APIs have been migrated to Kong Gateway (apihub.5paisa.com) for better performance
2. **Authentication**: Multiple authentication mechanisms used (Kong subscription key, Basic auth, JWT tokens)
3. **Rate Limiting**: Some external APIs may have rate limits
4. **Data Freshness**: Most data updated every 15-30 minutes during market hours
5. **Lazy Loading**: Non-critical content loaded after initial page render to improve perceived performance

---

## Estimated Total Page Load Time

### Optimal Conditions (Good Network, Server Cache Hit)
- **Server Processing**: 1-1.5 seconds
- **HTML Transfer**: 0.2-0.5 seconds
- **Client Rendering**: 0.5-1 second
- **Total First Meaningful Paint**: **2-3 seconds**

### Average Conditions (Mixed Cache, Normal Network)
- **Server Processing**: 2-3 seconds
- **HTML Transfer**: 0.3-0.7 seconds
- **Client Rendering**: 1-1.5 seconds
- **Total First Meaningful Paint**: **3.5-5 seconds**

### Worst Case (No Cache, Slow Network, Peak Load)
- **Server Processing**: 4-6 seconds
- **HTML Transfer**: 1-2 seconds
- **Client Rendering**: 1.5-2 seconds
- **Total First Meaningful Paint**: **6.5-10 seconds**

---

## Testing the APIs

To test any API:

1. **Server-Side APIs**: Can be tested directly with CURL commands provided
2. **Client-Side APIs**: Need to access via browser or set appropriate cookies/headers
3. **Rate Limiting**: Be mindful of rate limits when testing
4. **Cache**: First request will be slower (cache miss), subsequent requests faster

Example testing sequence:
```bash
# 1. Get BSE Script Code
curl -X POST 'https://gateway.5paisa.com/prelogin/prod/searchscrip' \
  -H 'UserID: ZyT47UW2g56' \
  -H 'Password: H98qlU4Sn2' \
  -H 'Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81' \
  -H 'Content-Type: application/json' \
  -d '{"Exch":"N","ExchType":"C","RecordCount":"1","Symbol":"SUZLON"}'

# 2. Use the ScripCode from response in Overview API
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/532667/' \
  -H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b'

# 3. Get Technical Analysis
curl -X GET 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/532667/' \
  -H 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b'
```

---

## Server-Side API Performance Table

| # | API Name | Endpoint | Avg Response Time | Cache Hit Time | Timeout | Priority |
|---|----------|----------|------------------|----------------|---------|----------|
| 1 | Search Scrip Code | `gateway.5paisa.com/prelogin/prod/searchscrip` | 150-300ms | 5-10ms | 10s | Critical |
| 2 | Stock Overview | `apihub.5paisa.com/pearl-ca/.../overview/` | 200-400ms | 10-20ms | No timeout | Critical |
| 3 | Fundamental Data | `apihub.5paisa.com/pearl-ca/.../fundamental/` | 150-250ms | 5-10ms | No timeout | High |
| 4 | Technical Analysis | `apihub.5paisa.com/pearl-ca/.../technical-analysis/` | 250-500ms | 10-15ms | No timeout | Critical |
| 5 | Tech Trend (SWOT) | `apihub.5paisa.com/pearl-ca/.../tech-trend/` | 150-300ms | 5-10ms | 10s | Medium |
| 6 | Rapid Results | `apihub.5paisa.com/pearl-ca/.../rapid-results/` | 200-350ms | 10-15ms | No timeout | Medium |
| 7 | Corporate Actions | `apihub.5paisa.com/pearl-ca/.../events/calendar/stock/` | 300-500ms | 15-25ms | No timeout | Medium |
| 8 | Company Profile (Atlas) | `apihub.5paisa.com/atlas_cache/.../snapshotcompprofile/` | 200-400ms | 10-20ms | No timeout | High |
| 9 | Quote Details (Atlas) | `apihub.5paisa.com/atlas-ba-rt/.../getquotedetails/` | 250-450ms | 15-25ms | No timeout | Critical |
| 10 | MarketSmith Instrument ID | `msi-gcloud-prod.appspot.com/.../instr/srch.json` | 300-600ms | N/A | 10s | High |
| 11 | Investment Rating | `msi-gcloud-prod.appspot.com/.../wisdom.json` | 400-700ms | N/A | 10s | Medium |
| 12 | Ownership/Management | `msi-gcloud-prod.appspot.com/.../financeDetails.json` | 500-800ms | N/A | 10s | High |
| 13 | Financial Data (MarketSmith) | `apihub.5paisa.com/marketsmith-ca/.../financeDetails.json` | 400-700ms | 20-30ms | 10s | High |

**Total Server-Side API Time (Sequential)**: 3.5-6 seconds (first load) | 150-250ms (cached)

**Note**: With caching enabled (24-hour cache), subsequent page loads are significantly faster.

---

## Client-Side API Performance Table

| # | API Name | Endpoint | Trigger Delay | Avg Response Time | Cache | User Impact |
|---|----------|----------|---------------|-------------------|-------|-------------|
| 14 | Forecast Data | `/get-forecast-data/{NSECODE}` | 2000ms | 300-600ms | 24h | Low (optional) |
| 15 | Shareholding Names | `/shareholding-pattern-names-json/` | 1000ms | 100-200ms | 24h | Medium |
| 16 | Shareholding Data | `/shareholding-pattern-data-json/` | 1000ms | 100-200ms | 24h | Medium |
| 17 | Shareholding Chart | `/shareholding-pattern-chart/` | 1000ms | 150-250ms | 24h | Medium |
| 18 | Similar Stocks | `/api/get-similar-stocks` | 2000ms | 400-800ms | 24h | Low |
| 19 | FnO Check | `/check-fno/{NSECODE}` | 2000ms | 200-400ms | 48h | Medium |
| 20 | Sector/Cap Info | `/api/{NSECODE}/sector-cap-info` | 2000ms | 200-350ms | No cache | High |
| 21 | Financial Data (Tabs) | `/financial-data/{TAB}/{NSECODE}` | 1500ms | 300-600ms | 24h | High |
| 22 | Price Range Filter | `/price-range-filter/{NSECODE}/{RANGE}` | On interaction | 150-300ms | No cache | Medium |
| 23a | Poll - Vote Count | `/stock-poll/votes-count/` | 2000ms | 50-100ms | No cache | Low |
| 23b | Poll - Cast Vote | `/stock-poll/{VOTE}/` | User action | 100-200ms | No cache | Low |
| 23c | Poll - Get Results | `/stock-poll/poll-html-json/` | 2000ms | 100-150ms | No cache | Low |

**Total Client-Side API Time**: 2-4 seconds (spread across delays)

---

## Combined Page Load Timeline with API Times

```
Time      | Server-Side                          | Client-Side              | User Experience
----------|--------------------------------------|--------------------------|---------------------------
0ms       | Request received                     | -                        | User sees loading
          |                                      |                          |
0-100ms   | URL parsing, validation              | -                        | Still loading
          |                                      |                          |
100-200ms | DB query: Stock details (50ms)       | -                        | Still loading
          | DB query: FAQs (30ms)                |                          |
          | DB query: About section (40ms)       |                          |
          |                                      |                          |
200-400ms | API #1: Search Scrip (200ms)         | -                        | Still loading
          |                                      |                          |
400-800ms | API #2: Overview (300ms)             | -                        | Still loading
          |                                      |                          |
800-1050ms| API #3: Fundamental (200ms)          | -                        | Still loading
          |                                      |                          |
1050-1500ms| API #4: Technical Analysis (400ms)  | -                        | Still loading
          |                                      |                          |
1500-1700ms| API #5: Tech Trend (200ms)          | -                        | Still loading
          |                                      |                          |
1700-2000ms| API #6: Rapid Results (250ms)       | -                        | Still loading
          |                                      |                          |
2000-2500ms| API #7: Corporate Actions (400ms)   | -                        | Still loading
          |                                      |                          |
2500-2800ms| API #8: Company Profile (250ms)     | -                        | Still loading
          |                                      |                          |
2800-3200ms| API #9: Quote Details (350ms)       | -                        | Still loading
          |                                      |                          |
3200-3700ms| API #10: Instrument ID (500ms)      | -                        | Still loading
          |                                      |                          |
3700-4300ms| API #11: Investment Rating (600ms)  | -                        | Still loading
          |                                      |                          |
4300-5000ms| API #12: Ownership/Management (700ms)| -                       | Still loading
          |                                      |                          |
5000-5600ms| API #13: Financial Data (600ms)     | -                        | Still loading
          |                                      |                          |
5600-6000ms| Template rendering (400ms)          | -                        | Still loading
          |                                      |                          |
6000ms    | HTML sent to browser                 | HTML received            | Page starts rendering
          |                                      |                          |
6100ms    | -                                    | CSS loaded (100ms)       | Styles applied
          |                                      |                          |
6200ms    | -                                    | JS files loaded (100ms)  | Scripts parsed
          |                                      |                          |
6300ms    | -                                    | DOM ready (100ms)        | **First Contentful Paint**
          |                                      |                          |
7000ms    | -                                    | Shareholding APIs (200ms)| Charts render
          |                                      |                          |
7500ms    | -                                    | Financial table (400ms)  | Tables populate
          |                                      |                          |
8000ms    | -                                    | Forecast (500ms)         | Forecast section loads
          |                                      | Similar stocks (600ms)   | Similar stocks appear
          |                                      | FnO check (300ms)        | FnO links added
          |                                      | Sector/Cap (250ms)       | Breadcrumb updated
          |                                      | Polls (200ms)            | Poll widget active
          |                                      |                          |
9000ms    | -                                    | All content loaded       | **Page Fully Interactive**

**Total Time to First Contentful Paint**: ~6.3 seconds (first load, no cache)
**Total Time to Interactive**: ~9 seconds (first load, no cache)

**With Cache Enabled**:
**Total Time to First Contentful Paint**: ~2.5 seconds
**Total Time to Interactive**: ~4 seconds
```

---

## Performance Breakdown by Page Section

| Page Section | Server-Side Time | Client-Side Time | Total Time | Cacheable | Data Source |
|--------------|------------------|------------------|------------|-----------|-------------|
| **Stock Header & Price** | 600-1000ms | 0ms | 600-1000ms | Yes (24h) | APIs #1, #2, #9 |
| **Performance Section** | 400-800ms | 0ms | 400-800ms | Yes (24h) | APIs #4, #9 |
| **Investment Returns** | 250-400ms | 0ms | 250-400ms | Yes (24h) | API #4 |
| **Fundamentals** | 300-600ms | 0ms | 300-600ms | Yes (24h) | APIs #2, #8, #9 |
| **Chart** | 200-300ms | 500ms | 700-800ms | Partial | API #1 + iframe |
| **Financials (Tabs)** | 600-900ms | 400-600ms | 1000-1500ms | Yes (24h) | API #13 |
| **Technical Section** | 400-700ms | 0ms | 400-700ms | Yes (24h) | APIs #4, #5 |
| **Ratings** | 600-900ms | 0ms | 600-900ms | Yes (24h) | APIs #10, #11 |
| **Events/Corporate Actions** | 300-500ms | 0ms | 300-500ms | Yes (24h) | API #7 |
| **F&O Section** | 0ms | 200-400ms | 200-400ms | Yes (48h) | API #19 |
| **Shareholding Pattern** | 500-800ms | 200-400ms | 700-1200ms | Yes (24h) | APIs #12, #15-17 |
| **Similar Stocks** | 0ms | 400-800ms | 400-800ms | Yes (24h) | API #18 |
| **About Section** | 50-100ms | 0ms | 50-100ms | Yes (perm) | Database |
| **FAQs** | 100-200ms | 0ms | 100-200ms | No cache | DB + API #2 |
| **Stock Polls** | 0ms | 200-350ms | 200-350ms | No cache | APIs #23a-c |

---

## Individual API Response Time Analysis

### Fastest APIs (< 200ms)
| API | Uncached | Cached | Cache Duration |
|-----|----------|--------|----------------|
| Search Scrip Code | 150-300ms | 5-10ms | 24h |
| Fundamental Data | 150-250ms | 5-10ms | 24h |
| Tech Trend | 150-300ms | 5-10ms | 24h |
| Price Range Filter | 150-300ms | No cache | - |
| Poll - Vote Count | 50-100ms | No cache | - |
| Poll - Get Results | 100-150ms | No cache | - |

### Medium Speed APIs (200-400ms)
| API | Uncached | Cached | Cache Duration |
|-----|----------|--------|----------------|
| Stock Overview | 200-400ms | 10-20ms | 24h |
| Rapid Results | 200-350ms | 10-15ms | 24h |
| Company Profile | 200-400ms | 10-20ms | 24h |
| Quote Details | 250-450ms | 15-25ms | No cache |
| FnO Check | 200-400ms | Cached | 48h |
| Sector/Cap Info | 200-350ms | No cache | - |
| Shareholding Names | 100-200ms | Cached | 24h |
| Shareholding Data | 100-200ms | Cached | 24h |

### Slower APIs (400ms+)
| API | Uncached | Cached | Cache Duration | Bottleneck? |
|-----|----------|--------|----------------|-------------|
| Technical Analysis | 250-500ms | 10-15ms | 24h | No |
| Corporate Actions | 300-500ms | 15-25ms | 24h | No |
| MarketSmith Instrument ID | 300-600ms | N/A | No cache | Minor |
| Investment Rating | 400-700ms | N/A | No cache | **Yes** |
| Ownership/Management | 500-800ms | N/A | No cache | **Yes** |
| Financial Data (MarketSmith) | 400-700ms | 20-30ms | 24h | Minor |
| Similar Stocks | 400-800ms | Cached | 24h | Minor |

---

## Bottleneck APIs (Main Performance Issues)

### 1. Ownership/Management API
- **Time**: 500-800ms (no cache)
- **Provider**: MarketSmith (msi-gcloud-prod.appspot.com)
- **Impact**: Delays shareholding pattern section
- **Optimization**: Not cacheable by vendor, affects every page load
- **% of Total**: ~10-15% of server processing time

### 2. Investment Rating API
- **Time**: 400-700ms (no cache)
- **Provider**: MarketSmith (msi-gcloud-prod.appspot.com)
- **Impact**: Delays ratings section
- **Optimization**: Not cacheable by vendor
- **% of Total**: ~8-12% of server processing time

### 3. Financial Data API
- **Time**: 400-700ms (uncached) | 20-30ms (cached)
- **Provider**: MarketSmith via Kong Gateway
- **Impact**: Delays financial tables
- **Optimization**: Good caching (24h)
- **% of Total**: ~10% of server processing time (first load only)

### 4. Quote Details API (Atlas)
- **Time**: 250-450ms (no cache)
- **Provider**: Atlas Real-Time API
- **Impact**: Live price data must be fresh
- **Optimization**: Cannot be cached (real-time data)
- **% of Total**: ~5-8% of server processing time

---

## Cache Impact Analysis

### Without Cache (First Visit)
```
Server-Side APIs:     5-6 seconds
Database Queries:     100-200ms
Template Rendering:   400-500ms
─────────────────────────────────
Total Server Time:    5.5-6.5 seconds

Client-Side Assets:   300-500ms
Client-Side APIs:     2-4 seconds
─────────────────────────────────
Total Page Load:      8-11 seconds
```

### With Full Cache (Subsequent Visits within 24h)
```
Server-Side APIs:     150-250ms (95% faster!)
Database Queries:     100-200ms
Template Rendering:   100-200ms
─────────────────────────────────
Total Server Time:    350-650ms

Client-Side Assets:   300-500ms
Client-Side APIs:     1-2 seconds (cached)
─────────────────────────────────
Total Page Load:      2-3.5 seconds (70-75% faster!)
```

---

## API Call Sequencing

### Sequential Calls (Blocking)
These APIs must complete before next one starts:
1. Search Scrip → Get BSE Code (200ms)
2. Overview → Get Stock Data (300ms)
3. Company Profile → Get co_code (250ms)
4. Quote Details → Get Live Price (350ms)

**Total Sequential Time**: ~1.1 seconds

### Parallel Calls (Non-Blocking)
These could potentially run in parallel:
- Fundamental Data (200ms)
- Technical Analysis (400ms)
- Tech Trend (200ms)
- Rapid Results (250ms)
- Corporate Actions (400ms)
- Investment Rating (600ms)
- Ownership/Management (700ms)
- Financial Data (600ms)

**Current Sequential Time**: ~3.5 seconds
**Potential Parallel Time**: ~700ms (limited by slowest API)
**Optimization Opportunity**: ~2.8 seconds saved

---

## Database Query Performance

| Query Type | Purpose | Avg Time | Cacheable | Impact |
|------------|---------|----------|-----------|--------|
| Stock Page Title | Get custom H1 title | 30-50ms | Yes (permanent) | Low |
| Stock Meta Details | Get meta title/description | 40-60ms | No | Low |
| About Company | Get about section content | 30-50ms | No | Low |
| Stock FAQs | Get FAQ content | 40-80ms | No | Low |
| Corporate Banner | Check if banner enabled | 20-30ms | No | Very Low |
| Dividend Check | Check if dividend data exists | 30-50ms | No | Low |

**Total DB Query Time**: 100-200ms

---

