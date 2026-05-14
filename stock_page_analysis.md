# TCS Stock Page API Documentation

## Overview
This document provides comprehensive details about all APIs used when accessing `https://www.5paisa.com/stocks/tcs-share-price`

---

## Table of Contents
1. [Server-Side APIs (Initial Page Load)](#server-side-apis-initial-page-load)
2. [Client-Side APIs (Dynamic Loading)](#client-side-apis-dynamic-loading)
3. [API to Section Mapping](#api-to-section-mapping)

---

## Server-Side APIs (Initial Page Load)

These APIs are called from the PHP backend ([`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php)) when the page initially loads.

### 1. **Search Scrip API** (BSE Code Lookup)
**Purpose:** Get BSE script code from NSE code

**Endpoint:**
```
POST https://gateway.5paisa.com/prelogin/prod/searchscrip
```

**Headers:**
```bash
UserID: ZyT47UW2g56
Password: H98qlU4Sn2
Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81
Content-Type: application/json
```

**Request Body:**
```json
{
  "Exch": "N",
  "ExchType": "C",
  "RecordCount": "1",
  "Symbol": "TCS"
}
```

**cURL Example:**
```bash
curl --location 'https://gateway.5paisa.com/prelogin/prod/searchscrip' \
--header 'UserID: ZyT47UW2g56' \
--header 'Password: H98qlU4Sn2' \
--header 'Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81' \
--header 'Content-Type: application/json' \
--data '{
  "Exch": "N",
  "ExchType": "C",
  "RecordCount": "1",
  "Symbol": "TCS"
}'
```

**Response:**
```json
{
  "Data": [
    {
      "ScripCode": "532540",
      "Series": "EQ",
      "Exchange": "N",
      "NSEcode": "TCS",
      "BSEcode": "532540"
    }
  ]
}
```

**Used For:** Stock Price Header, Basic Stock Info

---

### 2. **Stock Overview API**
**Purpose:** Get stock price, fundamentals, and technical data

**Endpoint:**
```
GET /overview/{bseCode or nseCode}/
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/overview/TCS/
```

**Headers:**
```bash
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: <TREND_KEY>
requestCode: pearlapi
UserId: 5PAISAAPI
password: 5nadynsiitnienny
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/overview/TCS/' \
--header 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
--header 'Ocp-Apim-Trace: true' \
--header 'Content-Type: application/json' \
--header 'KEY: <TREND_KEY>' \
--header 'requestCode: pearlapi' \
--header 'UserId: 5PAISAAPI' \
--header 'password: 5nadynsiitnienny'
```

**Response:**
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "stockData": [
      "Tata Consultancy Services Ltd.",
      "TCS",
      "N",
      "EQ",
      "532540",
      "INE467B01029",
      "",
      "4250.50",
      "4300.00",
      "4200.00",
      "25.30",
      "0.60"
    ],
    "lastUpdated": "14-May-2026 14:30:00",
    "fundamentalData": [
      {"name": "Market Cap Cr", "value": 1543210.50},
      {"name": "Price to Earnings", "value": 32.45},
      {"name": "Price to Book Value Adjusted", "value": 12.34},
      {"name": "Dividend Yield", "value": 1.23},
      {"name": "PE to Growth", "value": 1.5}
    ],
    "technicalData": [
      {"name": "Money Flow Index", "value": 65.32},
      {"name": "Relative Strength Index", "value": 58.45}
    ],
    "52_Week_High": 4500.00,
    "52_Week_low": 3200.00
  }
}
```

**Used For:** 
- Stock Price Header
- Key Statistics Section
- 52-Week High/Low
- Market Cap, PE Ratio, etc.

---

### 3. **Fundamental Data API**
**Purpose:** Get detailed fundamental stock information

**Endpoint:**
```
GET /fundamental/{bseCode or nseCode}/
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/fundamental/TCS/
```

**Headers:** Same as Overview API

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/fundamental/TCS/' \
--header 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
--header 'Content-Type: application/json'
```

**Response:**
```json
{
  "body": {
    "stockData": {
      "NSEcode": "TCS",
      "BSEcode": "532540",
      "BSEScriptCode": "tcs",
      "companyName": "Tata Consultancy Services Ltd."
    }
  }
}
```

**Used For:** Stock Redirect Logic, Company Name

---

### 4. **Corporate Actions API** (Events Calendar)
**Purpose:** Get dividend, bonus, split, board meeting data

**Endpoint:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/{nseCode}
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS
```

**Headers:**
```bash
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: <TREND_KEY>
requestCode: pearlapi
UserId: 5PAISAAPI
password: 5nadynsiitnienny
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS' \
--header 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b' \
--header 'Ocp-Apim-Trace: true' \
--header 'Content-Type: application/json'
```

**Response:**
```json
{
  "body": {
    "eventsData": [
      {
        "eventType": "Dividend",
        "recordDate": "2026-03-15",
        "purpose": "Interim Dividend",
        "remarks": "Rs.10.00 per share"
      },
      {
        "eventType": "Board Meeting",
        "boardMeetDate": "2026-04-10",
        "purpose": "To consider quarterly results"
      },
      {
        "eventType": "Announcements",
        "recordDate": "2026-05-01",
        "purpose": "Annual General Meeting"
      }
    ]
  }
}
```

**Used For:** Corporate Actions Section (Dividends, Bonus, Splits, Board Meetings)

---

### 5. **Rapid Results API** (Result Highlights)
**Purpose:** Get latest quarterly result highlights

**Endpoint:**
```
GET /rapid-results/{nseCode}/
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/rapid-results/TCS/
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/rapid-results/TCS/'
```

**Response:**
```json
{
  "body": {
    "newsList": [
      {
        "stockName": "TCS",
        "NSEcode": "TCS",
        "quarterlyResults": {
          "revenue": 61000,
          "netProfit": 12200,
          "eps": 33.5
        }
      }
    ]
  }
}
```

**Used For:** Result Highlights Section

---

### 6. **Technical Analysis API**
**Purpose:** Get technical indicators (EMA, SMA, RSI, MACD, Pivot Points)

**Endpoint:**
```
GET /technical-analysis/{nseCode}/
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/technical-analysis/TCS/
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/technical-analysis/TCS/'
```

**Response:**
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "stockPriceData": {
      "currentPrice": 4250.50,
      "dayChange": 25.30,
      "dayChangeP": 0.60
    },
    "movingAverages": {
      "tableData": [
        ["5 Day", 4240.25, 4245.30],
        ["10 Day", 4230.15, 4235.20],
        ["20 Day", 4200.50, 4210.75],
        ["50 Day", 4150.25, 4165.50],
        ["100 Day", 4100.00, 4120.30],
        ["200 Day", 4050.75, 4075.25]
      ]
    },
    "technicals": [
      {"name": "Relative Strength Index", "value": 58.45, "color": "green"},
      {"name": "Money Flow Index", "value": 65.32, "color": "green"},
      {"name": "MACD", "value": 12.50, "color": "positive"},
      {"name": "MACD Signal", "value": 10.25, "color": "positive"}
    ],
    "pivotData": {
      "pivot": 4250.00,
      "r1": 4280.00,
      "r2": 4310.00,
      "s1": 4220.00,
      "s2": 4190.00
    },
    "priceAnalysis": {
      "day": {"changePercent": 0.60, "color": "positive"},
      "week": {"changePercent": 2.35, "color": "positive"},
      "month": {"changePercent": 5.20, "color": "positive"},
      "quarter": {"changePercent": 8.50, "color": "positive"},
      "halfYear": {"changePercent": 12.30, "color": "positive"},
      "year": {"changePercent": 18.75, "color": "positive"}
    }
  }
}
```

**Used For:** 
- Technical Analysis Section (EMA/SMA Tables)
- RSI, MFI, MACD Indicators
- Pivot Points
- Share Returns Section

---

### 7. **SWOT Data API** (Tech Trend)
**Purpose:** Get bullish/bearish technical trend data

**Endpoint:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/{nseCode}
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/TCS
```

**Headers:** Same as Overview API

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/TCS' \
--header 'Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b'
```

**Response:**
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "TCS": {
      "techTrend": {
        "total": 20,
        "bearish": 5,
        "bullish": 15
      }
    }
  }
}
```

**Used For:** SWOT Analysis Graph (Bullish vs Bearish percentage)

---

### 8. **FnO Overview API** (Futures & Options Check)
**Purpose:** Check if stock has Futures/Options available

**Endpoint:**
```
GET https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/{type}/{symbol}?responseType=json
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json
```

**Headers:**
```bash
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json' \
--header 'Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw=='
```

**Response:**
```json
{
  "response": {
    "data": {
      "companylist": {
        "company": [
          {
            "symbol": "TCS",
            "hasFutures": true,
            "hasOptions": true
          }
        ]
      }
    }
  }
}
```

**Used For:** FnO Section (to show Futures/Options links)

---

## Client-Side APIs (Dynamic Loading)

These APIs are called from JavaScript ([`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js)) after the page loads.

### 9. **Sector & Cap Info API**
**Purpose:** Get sector link and market cap classification

**Endpoint:**
```
GET /api/{nseCode}/sector-cap-info
```

**Full URL Example:**
```
GET https://www.5paisa.com/api/TCS/sector-cap-info
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/api/TCS/sector-cap-info'
```

**Response:**
```json
{
  "sectorInfo": {
    "sectorLink": "/stocks/sector/information-technology",
    "sectorName": "Information Technology"
  },
  "capInfo": {
    "capName": "Large Cap",
    "capLink": "/share-market-today/large-cap-stocks"
  }
}
```

**Used For:** Stock Header Section (Sector and Market Cap links)

---

### 10. **Similar Stocks API**
**Purpose:** Get similar stocks from the same sector

**Endpoint:**
```
GET /api/get-similar-stocks?nsecode={nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/api/get-similar-stocks?nsecode=TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/api/get-similar-stocks?nsecode=TCS'
```

**Response:**
```json
{
  "data": {
    "similarStocks": "<div class='stock-page__similarwrapper'>...</div>"
  }
}
```

**Used For:** Similar Stocks Section

---

### 11. **Shareholding Pattern Months API**
**Purpose:** Get available months for shareholding pattern data

**Endpoint:**
```
GET /shareholding-pattern-months-json/{nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/shareholding-pattern-months-json/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/shareholding-pattern-months-json/TCS'
```

**Response:**
```json
{
  "data": "<ul><li><a href='#Mar-2026'>Mar 2026</a></li><li><a href='#Dec-2025'>Dec 2025</a></li></ul>"
}
```

**Used For:** Shareholding Pattern Tabs

---

### 12. **Shareholding Pattern Names API**
**Purpose:** Get shareholder category names for a specific month

**Endpoint:**
```
GET /shareholding-pattern-names-json/{monthYear}/{nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/shareholding-pattern-names-json/Mar-2026/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/shareholding-pattern-names-json/Mar-2026/TCS'
```

**Response:**
```json
{
  "data": "<div>Promoter Holdings: 72.05%</div><div>FII Holdings: 15.20%</div>"
}
```

**Used For:** Shareholding Pattern Chart Labels

---

### 13. **Shareholding Pattern Data API**
**Purpose:** Get shareholding percentage data for chart

**Endpoint:**
```
GET /shareholding-pattern-data-json/{monthYear}/{nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/shareholding-pattern-data-json/Mar-2026/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/shareholding-pattern-data-json/Mar-2026/TCS'
```

**Response:**
```json
{
  "data": "<canvas id='shareholding-chart'>...</canvas>"
}
```

**Used For:** Shareholding Pattern Chart

---

### 14. **Financial Data API**
**Purpose:** Get quarterly/annual financial statements

**Endpoint:**
```
GET /financial-data/{tabFilter}/{nseCode}
```

**Tab Filters:**
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

**Full URL Example:**
```
GET https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/TCS'
```

**Response:**
```json
{
  "table": "<table class='table stock-table'><thead><tr><th>Indicator</th><th>Mar 2026</th></tr></thead><tbody>...</tbody></table>"
}
```

**Used For:** Financials Section (P&L, Balance Sheet, Cash Flow, Ratios)

---

### 15. **FnO Check API**
**Purpose:** Check if Futures/Options exist for the stock

**Endpoint:**
```
GET /check-fno/{nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/check-fno/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/check-fno/TCS'
```

**Response:**
```json
{
  "future_exist": "true",
  "option_exist": "true"
}
```

**Used For:** Show/Hide FnO Section

---

### 16. **Forecast Graph Data API**
**Purpose:** Get analyst forecast and price targets

**Endpoint:**
```
GET /get-forecast-data/{nseCode}
```

**Full URL Example:**
```
GET https://www.5paisa.com/get-forecast-data/TCS
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/get-forecast-data/TCS'
```

**Response:**
```json
{
  "forecast": {
    "consensusRating": "Buy",
    "recommendations": [
      {"symbol": "TCS", "rating": "Buy", "numOfRating": 25}
    ],
    "targetPrice": {
      "meanTarget": 4500,
      "highTarget": 4800,
      "lowTarget": 4200
    },
    "financials": [
      {"year": 2026, "value": 35.5}
    ]
  }
}
```

**Used For:** Analyst Forecast Section

---

### 17. **Price Range Filter API**
**Purpose:** Get price change for different time ranges (triggered by chart interaction)

**Endpoint:**
```
GET /price-range-filter/{nseCode}/{rangeFilter}
```

**Range Filters:** `1D`, `1W`, `1M`, `6M`, `1Y`, `5Y`, `Max`

**Full URL Example:**
```
GET https://www.5paisa.com/price-range-filter/TCS/1M
```

**cURL Example:**
```bash
curl --location 'https://www.5paisa.com/price-range-filter/TCS/1M'
```

**Response:**
```json
{
  "priceAnalysis": {
    "change": 220.50,
    "changePercent": 5.20,
    "color": "positive"
  }
}
```

**Used For:** Update stock price header when user changes chart timeframe

---

## External APIs (Third-Party)

### 18. **Kong Gateway APIs** (Quote Details)
**Purpose:** Get detailed company information

**Endpoint:**
```
GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{coCode}/{exchange}?responseType=json
```

**Full URL Example:**
```
GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/3045/nse?responseType=json
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/3045/nse?responseType=json'
```

**Response:**
```json
{
  "response": {
    "data": {
      "getquotedetailslist": {
        "getquotedetails": {
          "companysectorcode": "1001",
          "companysector": "Information Technology",
          "complname": "Tata Consultancy Services Ltd.",
          "symbol": "TCS",
          "price": 4250.50
        }
      }
    }
  }
}
```

**Used For:** Sector Information

---

### 19. **Quote Finder API**
**Purpose:** Search and get company code

**Endpoint:**
```
GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/quotesfinder/{symbol}?responseType=json
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/quotesfinder/TCS?responseType=json'
```

**Response:**
```json
{
  "response": {
    "data": {
      "getquoteslist": {
        "@recordcount": 1,
        "getquotes": {
          "co_code": "3045",
          "symbol": "TCS",
          "exchange": "nse"
        }
      }
    }
  }
}
```

**Used For:** Similar Stocks Lookup

---

### 20. **Sector-wise Companies API**
**Purpose:** Get all companies in a sector

**Endpoint:**
```
GET https://apihub.5paisa.com/atlas-req1-rt/datamart/Equity/Market.svc/SectorWiseComp/{sectorCode}/?responseType=json
```

**cURL Example:**
```bash
curl --location 'https://apihub.5paisa.com/atlas-req1-rt/datamart/Equity/Market.svc/SectorWiseComp/1001/?responseType=json'
```

**Response:**
```json
{
  "response": {
    "data": {
      "CompanyList": {
        "@recordcount": 50,
        "Company": [
          {
            "co_code": "3045",
            "sc_code": "TCS",
            "exchange_nse": "Yes"
          }
        ]
      }
    }
  }
}
```

**Used For:** Similar Stocks Section

---

## API to Section Mapping

### Stock Price Header
- **APIs Used:**
  1. Search Scrip API (BSE Code)
  2. Stock Overview API (Price Data)
  3. Technical Analysis API (Price Changes)
  4. Sector & Cap Info API (Sector/Cap Links)

### Key Statistics
- **APIs Used:**
  1. Stock Overview API (Market Cap, PE, PB, Dividend Yield)
  2. Technical Analysis API (RSI, MFI)

### About Company
- **Source:** CMS (Drupal Content)
- **APIs:** None (from database)

### Share Returns
- **APIs Used:**
  1. Technical Analysis API (Price Analysis for different periods)

### Technicals (EMA/SMA)
- **APIs Used:**
  1. Technical Analysis API (Moving Averages, Pivot Points)

### SWOT Analysis
- **APIs Used:**
  1. SWOT Data API (Tech Trend - Bullish/Bearish)

### Corporate Actions
- **APIs Used:**
  1. Corporate Actions API (Dividends, Bonus, Splits, Board Meetings)

### Result Highlights
- **APIs Used:**
  1. Rapid Results API (Quarterly Results)

### Financials
- **APIs Used:**
  1. Financial Data API (P&L, Balance Sheet, Cash Flow, Ratios)

### Shareholding Pattern
- **APIs Used:**
  1. Shareholding Pattern Months API (Available Months)
  2. Shareholding Pattern Names API (Shareholder Categories)
  3. Shareholding Pattern Data API (Chart Data)

### FnO Section
- **APIs Used:**
  1. FnO Overview API (Check if Futures/Options exist)
  2. FnO Check API (Client-side verification)

### Similar Stocks
- **APIs Used:**
  1. Similar Stocks API
  2. Quote Finder API
  3. Sector-wise Companies API
  4. Kong Gateway Quote Details API

### Analyst Forecast
- **APIs Used:**
  1. Forecast Graph Data API

### Chart
- **Source:** Embedded iframe from TradingView/5paisa charts
- **Dynamic Updates:** Price Range Filter API

---

## Loading Sequence

### Initial Page Load (Server-Side)
1. **Search Scrip API** → Get BSE code
2. **Stock Overview API** → Get basic stock data
3. **Fundamental Data API** → Get company details
4. **Technical Analysis API** → Get technical indicators
5. **SWOT Data API** → Get bullish/bearish trend
6. **Corporate Actions API** → Get events
7. **Rapid Results API** → Get result highlights
8. **FnO Overview API** → Check futures/options

### After Page Load (Client-Side - Progressive)
1. **Sector & Cap Info API** (Immediate)
2. **Chart Rendering** (On scroll to chart section)
3. **Similar Stocks API** (When section is visible - IntersectionObserver)
4. **FnO Check API** (When FnO section is visible)
5. **Financial Data API** (When Financials section is visible)
6. **Shareholding Pattern APIs** (When tab is clicked)
7. **Price Range Filter API** (When user interacts with chart)
8. **Forecast Graph Data API** (When Forecast section is visible)

---

## Notes

### Performance Optimizations
- **Caching:** Most APIs implement 24-hour cache (`84600` seconds)
- **Lazy Loading:** Client-side APIs use IntersectionObserver for on-demand loading
- **Progressive Enhancement:** Page renders with server-side data, then enhances with client-side data

### Error Handling
- All APIs have try-catch blocks
- Failed API calls don't break the page
- Fallback content or hidden sections on API failure

### API Authentication
- **Kong Gateway:** Uses `Ocp-Apim-Subscription-Key`
- **Pearl API:** Uses `KEY`, `UserId`, `password`, `requestCode`
- **Search Scrip:** Uses `UserID`, `Password`, `Ocp-Apim-Subscription-Key`
- **FnO API:** Uses Basic Authentication

---

## Testing the APIs

To test these APIs for the TCS stock page:

1. Replace `{nseCode}` with `TCS`
2. Replace `{bseCode}` with `532540`
3. Use the provided cURL commands
4. Check the response format

**Example Test:**
```bash
# Test Stock Overview API
curl --location 'https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/overview/TCS/' \
--header 'Content-Type: application/json'
```

---

## Conclusion

The TCS stock page uses **20+ APIs** that work together to provide:
- Real-time stock prices
- Technical and fundamental analysis
- Corporate actions and events
- Financial statements
- Shareholding patterns
- Analyst forecasts
- Similar stock recommendations

The architecture follows a **progressive enhancement** pattern where:
1. Server-side APIs provide critical data for initial render
2. Client-side APIs enhance the experience with additional data
3. Lazy loading ensures optimal performance
4. Caching reduces API calls and improves speed
