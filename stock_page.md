# Stock Page cURL Documentation

## Table of Contents
1. [Main Page Access](#main-page-access)
2. [External API Endpoints (Kong Gateway)](#external-api-endpoints-kong-gateway)
3. [External API Endpoints (Atlas)](#external-api-endpoints-atlas)
4. [Internal Drupal API Routes](#internal-drupal-api-routes)
5. [Authentication & Headers](#authentication--headers)
6. [Common Parameters](#common-parameters)

---

## Main Page Access

### Stock Page URL
Access any stock page using the following pattern:
```bash
curl -X GET "https://www.5paisa.com/stocks/tcs-share-price" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Accept: text/html"
```

**URL Pattern:** `/stocks/{stock-symbol}-share-price`

**Examples:**
- `/stocks/tcs-share-price`
- `/stocks/reliance-share-price`
- `/stocks/infosys-share-price`

---

## External API Endpoints (Kong Gateway)

### 1. Stock Overview Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:199)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

**Response includes:** Stock price, fundamentals, technicals, 52-week high/low

---

### 2. Fundamental Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:263)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/fundamental/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

---

### 3. Technical Analysis Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1169)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

**Response includes:** EMA, SMA, RSI, MACD, pivot points, moving averages

---

### 4. Corporate Actions (Events)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:881-903)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

**Response includes:** Board meetings, dividends, bonus, splits, announcements

---

### 5. Technical Trend (SWOT - Bearish/Bullish)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1259)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/TCS" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

**Response includes:** Bearish count, bullish count, technical trend percentage

---

### 6. Rapid Results (Stock Highlights)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1146)

```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/rapid-results/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

**Response includes:** Latest quarterly results, earnings highlights

---

## External API Endpoints (Atlas)

### 7. Script Code Search (BSE/NSE)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1050-1075)

```bash
curl -X POST "https://gateway.5paisa.com/prelogin/prod/searchscrip" \
  -H "UserID: ZyT47UW2g56" \
  -H "Password: H98qlU4Sn2" \
  -H "Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81" \
  -H "Content-Type: application/json" \
  -d '{
    "Exch": "N",
    "ExchType": "C",
    "RecordCount": "1",
    "Symbol": "TCS"
  }' \
  --insecure
```

**Response includes:** ScripCode, Exchange, Symbol for chart generation

---

### 8. Company Quote Details
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:702)

```bash
curl -X GET "https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{CO_CODE}/nse?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  --insecure
```

Replace `{CO_CODE}` with company code obtained from snapshot API.

**Response includes:** EPS, sector, market cap, company details

---

### 9. Company Snapshot Profile
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:361)

```bash
curl -X GET "https://apihub.5paisa.com/atlas_cache/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  --insecure
```

**Response includes:** co_code for further API calls

---

### 10. Sector Companies List
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1961)

```bash
curl -X GET "https://apihub.5paisa.com/atlas_cache/datamart/Equity/Market.svc/SectorWiseComp/{SECTOR_CODE}/?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  --insecure
```

Replace `{SECTOR_CODE}` with sector code.

**Used for:** Similar stocks feature

---

### 11. F&O Overview (Futures & Options)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2126)

```bash
# For Futures
curl -X GET "https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  --insecure

# For Options
curl -X GET "https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Opt/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  --insecure
```

---

## External API Endpoints (MarketSmith India - via Kong Gateway)

### 12. Instrument Search (for Financial Data)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:1230-1259)

```bash
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instrument?symbol=TCS" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  --insecure
```

**Response includes:** instrumentId for finance details

---

### 13. Finance Details (Consolidated/Standalone)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:1269-1300)

```bash
# Consolidated (isConsolidated=true)
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=true" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  --insecure

# Standalone (isConsolidated=false)
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  --insecure
```

Replace `{INSTRUMENT_ID}` with the ID from instrument search.

**Response includes:** Balance sheet, P&L, cash flow, ratios data

---

### 14. Investment Ratings & Wisdom
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:403-418)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/wisdom.json?lang=en&ver=2&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  --insecure
```

**Response includes:** Master rating, EPS rating, price strength, buyer demand

---

### 15. Broker Estimates (Forecasts)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:381-395)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/getBrokerEstimates.json?instrumentId={INSTRUMENT_ID}&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  --insecure
```

**Response includes:** Target price, revenue estimates, EPS forecasts

---

### 16. Ownership & Management Data
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:426-442)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  --insecure
```

**Response includes:** Shareholding pattern, management info

---

## Internal Drupal API Routes

### 17. Financial Data (AJAX)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:560-613)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:737)

```bash
curl -X GET "https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Available Filters:**
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

---

### 18. Sector Link Information
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:691-738)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:680-702)

```bash
curl -X GET "https://www.5paisa.com/api/TCS/sector-cap-info" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Response includes:** Sector name, sector link, market cap category (Small/Mid/Large Cap)

---

### 19. Similar Stocks
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1916-2024)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:579)

```bash
curl -X GET "https://www.5paisa.com/api/get-similar-stocks?nsecode=TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Response includes:** HTML with 4 similar stocks from same sector

---

### 20. FnO Check (Futures & Options Availability)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2097-2114)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:641-677)

```bash
curl -X GET "https://www.5paisa.com/check-fno/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Response:**
```json
{
  "future_exist": "true",
  "option_exist": "true"
}
```

---

### 21. Shareholding Pattern - Months
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:647-652)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-months/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

---

### 22. Shareholding Pattern - Names
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:679-684)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:535)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-names-json/Dec-2024/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

---

### 23. Shareholding Pattern - Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:663-668)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:536)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-data-json/Dec-2024/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

---

### 24. Forecast Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:812-865)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:477)

```bash
curl -X GET "https://www.5paisa.com/get-forecast-data/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Response includes:** Consensus rating, target price, analyst recommendations

---

### 25. Price Range Filter (Chart Filter)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2596-2629)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:767-807)

```bash
curl -X GET "https://www.5paisa.com/price-range-filter/TCS/1W" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0"
```

**Valid Range Filters:** `1D`, `1W`, `1M`, `6M`, `1Y`, `5Y`, `Max`


## Authentication & Headers

### Kong Gateway Headers (Pearl APIs)
```
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
requestCode: pearlapi
UserId: 5PAISAAPI
password: 5nadynsiitnienny
```

### Atlas Gateway Headers
```
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

### MarketSmith Headers
```
gateway-name: fivepaisa-gateway
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs
```

### Gateway Search Headers
```
UserID: ZyT47UW2g56
Password: H98qlU4Sn2
Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81
Content-Type: application/json
```

---

## Common Parameters

### Stock Identifiers
- **NSE Code**: NSE symbol (e.g., `TCS`, `RELIANCE`, `INFY`)
- **BSE Code**: BSE script code (numeric)
- **co_code**: Company code from snapshot API
- **instrumentId**: MarketSmith instrument ID

### Date Formats
- Shareholding: `MMM-YYYY` (e.g., `Dec-2024`)
- Events: `DD/MM/YYYY`

---

## API Call Execution Pattern

### Sequential vs Parallel Execution

Based on the code analysis, the stock page uses a **hybrid approach**:

#### Sequential Calls (Server-Side - PHP)
These APIs are called **sequentially** when the page loads on the server:

1. **Script Code Lookup** → Get BSE/NSE codes
2. **Overview API** → Get basic stock data
3. **Fundamental API** → Get fundamental metrics
4. **Technical Analysis API** → Get technical indicators
5. **Corporate Actions API** → Get events data

**Source:** [`StockPageController.php::getData()`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:155-454)

**Execution Flow:**
```php
// 1. Get script codes first
$checkData = $this->getScriptCodeBse($nsecode);

// 2. Then get overview (depends on script code)
$stockResp = $this->stockPageService->getClient()->get('overview/' . $bseNsecode);

// 3. Then get fundamentals
$stock_fundamental_api = $this->stockPageService->getClient()->get('fundamental/' . $bseNsecode);

// 4. Then get technical data
$build['#tech'] = $this->getTechnicalData($nsecode, $bseNsecode);

// 5. Then get corporate actions
$build['#corp_action'] = $this->getCorporateActionData($nsecode, $bseNsecode);
```

#### Parallel Calls (Client-Side - JavaScript)
These APIs are called **in parallel** when user scrolls/interacts:

**Lazy-Loaded on Scroll:**
- Similar Stocks API
- Financial Data API
- Chart iframe loading
- Shareholding pattern data

**Source:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js)

**Execution Pattern:**
```javascript
// Multiple APIs triggered on scroll (parallel)
const observer = new IntersectionObserver((entries) => {
  if (entry.isIntersecting) {
    // These can be called in parallel
    renderSimilarStocks(nsecode);    // API 1
    getFinancialData(filter, code);  // API 2
    loadChart();                      // API 3
  }
});
```

### Recommendation for Testing

#### 1. **Initial Page Load (Sequential)**
Execute these in order for accurate page load simulation:

```bash
# Step 1: Get script codes
curl -X POST "https://gateway.5paisa.com/prelogin/prod/searchscrip" \
  -H "UserID: ZyT47UW2g56" \
  -H "Password: H98qlU4Sn2" \
  -d '{"Symbol":"TCS"}' > scripcode.json

# Step 2: Get overview
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" > overview.json

# Step 3: Get technical analysis
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" > technical.json

# Step 4: Get corporate actions
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" > events.json
```

#### 2. **User Interaction (Parallel)**
These can be executed simultaneously:

```bash
# Execute all in parallel (use & for background jobs)
curl -X GET "https://www.5paisa.com/api/get-similar-stocks?nsecode=TCS" > similar.json &
curl -X GET "https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/TCS" > financials.json &
curl -X GET "https://www.5paisa.com/check-fno/TCS" > fno.json &
curl -X GET "https://www.5paisa.com/api/TCS/sector-cap-info" > sector.json &

# Wait for all background jobs
wait
```

### Performance Considerations

| API Type | Execution | Cache | Priority |
|----------|-----------|-------|----------|
| Overview | Sequential | 24h | High |
| Technical | Sequential | 24h | High |
| Fundamental | Sequential | 24h | High |
| Corporate Actions | Sequential | 24h | Medium |
| Similar Stocks | Parallel (lazy) | None | Low |
| Financial Data | Parallel (lazy) | 24h | Low |
| Shareholding | Parallel (lazy) | None | Low |
| Chart | Parallel (lazy) | None | Low |

**Caching Strategy:**
- Server-side caching: 24 hours (86400 seconds)
- Client-side: Based on browser cache
- Lazy-loaded content: Only fetched when visible

---

## Complete Example: Fetching TCS Stock Page Data

### Step 1: Get Overview Data (Sequential - Required First)
```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  -H "KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44" \
  -H "requestCode: pearlapi" \
  -H "UserId: 5PAISAAPI" \
  -H "password: 5nadynsiitnienny" \
  --insecure
```

### Step 2: Get Technical Analysis
```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Ocp-Apim-Trace: true" \
  -H "Content-Type: application/json" \
  --insecure
```

### Step 3: Get Corporate Actions
```bash
curl -X GET "https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS" \
  -H "Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b" \
  -H "Content-Type: application/json" \
  --insecure
```

### Step 4: Get Financial Data
```bash
# First get instrument ID
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instrument?symbol=TCS" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  --insecure

# Then get financial details
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  --insecure
```

### Step 5: Get F&O Data
```bash
curl -X GET "https://www.5paisa.com/check-fno/TCS" \
  -H "Accept: application/json"
```

---

## Notes

1. **SSL Verification**: Most APIs use `--insecure` flag as they have self-signed certificates
2. **Rate Limiting**: Kong Gateway APIs may have rate limits
3. **Caching**: Data is cached for 24 hours (86400 seconds) on the server side
4. **Error Handling**: Always check HTTP status codes (200, 403, 404, etc.)
5. **Stock Code Format**: NSE codes are uppercase (TCS, RELIANCE), URL slugs are lowercase (tcs-share-price)
6. **Special Characters**: Stocks with `&` in names (M&M) use clean URLs (mm-share-price)

---

## Troubleshooting

### 403 Forbidden
- Check authentication headers
- Verify subscription keys are valid
- Ensure correct API endpoint

### 404 Not Found
- Verify stock symbol exists
- Check NSE/BSE code format
- Ensure proper URL structure

### Empty Data
- Stock may not have F&O available
- Financial data may not exist for the period
- Corporate actions may not be announced

---

## API Response Examples

### Overview API Response Structure
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "stockData": [/* price, change, volume, etc. */],
    "fundamentalData": [/* market cap, PE, PB, etc. */],
    "technicalData": [/* RSI, MFI, MACD, etc. */],
    "52_Week_High": "3500.00",
    "52_Week_low": "2800.00",
    "priceInsightData": [/* circuit hits, etc. */]
  }
}
```

### Financial Data Response Structure
```json
{
  "bsHeader": [/* balance sheet headers */],
  "bsData": [/* balance sheet data */],
  "incQuarterHeader": [/* P&L headers */],
  "incQuarterData": [/* quarterly P&L data */],
  "ratiosAnnualHeader": [/* ratios headers */],
  "ratiosAnnualData": [/* ratios data */]
}
```

---

**Last Updated:** April 24, 2026  
**Documentation Version:** 1.0  
**Base URL:** `https://www.5paisa.com`
