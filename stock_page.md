# Stock Page cURL Documentation

## Table of Contents
1. [Main Page Access](#main-page-access)
2. [Measuring API Response Times](#measuring-api-response-times)
3. [External API Endpoints (Kong Gateway)](#external-api-endpoints-kong-gateway)
4. [External API Endpoints (Atlas)](#external-api-endpoints-atlas)
5. [Internal Drupal API Routes](#internal-drupal-api-routes)
6. [Authentication & Headers](#authentication--headers)
7. [Common Parameters](#common-parameters)
8. [Performance Benchmarks](#performance-benchmarks)

---

## Measuring API Response Times

### Using cURL with Timing Output

Add the `-w` (write-out) flag to measure response times:

```bash
curl -X GET "https://www.5paisa.com/stocks/tcs-share-price" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Accept: text/html" \
  -w "\n\n⏱️  Response Times:\n-------------------\nDNS Lookup:        %{time_namelookup}s\nTCP Connection:    %{time_connect}s\nTLS Handshake:     %{time_appconnect}s\nServer Processing: %{time_starttransfer}s\nTotal Time:        %{time_total}s\nHTTP Code:         %{http_code}\n" \
  -o /dev/null -s
```

### Full Timing Breakdown Format

Use this comprehensive timing template for all API calls:

```bash
curl -X GET "API_URL_HERE" \
  -H "HEADERS_HERE" \
  -w "@curl-format.txt" \
  -o response.json -s
```

**Create `curl-format.txt` with:**
```
\n
=====================================
⏱️  TIMING BREAKDOWN
=====================================
DNS Lookup Time:      %{time_namelookup}s
TCP Connect Time:     %{time_connect}s
TLS Handshake Time:   %{time_appconnect}s
Pre-Transfer Time:    %{time_pretransfer}s
Redirect Time:        %{time_redirect}s
Start Transfer Time:  %{time_starttransfer}s
TOTAL TIME:           %{time_total}s
-------------------------------------
Download Speed:       %{speed_download} bytes/sec
Upload Speed:         %{speed_upload} bytes/sec
Size Downloaded:      %{size_download} bytes
Size Uploaded:        %{size_upload} bytes
HTTP Status Code:     %{http_code}
=====================================
```

### Quick Timing (One-Liner)

```bash
# Simple one-liner for quick timing
curl -X GET "API_URL" -H "HEADERS" -w "\nTotal: %{time_total}s | HTTP: %{http_code}\n" -o /dev/null -s
```

---

## Main Page Access

### Stock Page URL
Access any stock page using the following pattern:
```bash
curl -X GET "https://www.5paisa.com/stocks/tcs-share-price" \
  -H "User-Agent: Mozilla/5.0" \
  -H "Accept: text/html" \
  -w "\nTotal Time: %{time_total}s | HTTP Code: %{http_code}\n" \
  -o response.html
```

**URL Pattern:** `/stocks/{stock-symbol}-share-price`

**Examples:**
- `/stocks/tcs-share-price`
- `/stocks/reliance-share-price`
- `/stocks/infosys-share-price`

**Expected Response Time:** 3.0s - 4.0s (includes multiple sequential backend API calls)

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Stock price, fundamentals, technicals, 52-week high/low  
**Expected Response Time:** 0.2-0.5s  
**Timeout Configured:** 10s (Source: Code defaults)

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Expected Response Time:** 0.2-0.4s  
**Timeout Configured:** 10s

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** EMA, SMA, RSI, MACD, pivot points, moving averages  
**Expected Response Time:** 0.3-0.6s  
**Timeout Configured:** 10s

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Board meetings, dividends, bonus, splits, announcements  
**Expected Response Time:** 0.2-0.5s  
**Timeout Configured:** None (default)

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Bearish count, bullish count, technical trend percentage  
**Expected Response Time:** 0.2-0.4s  
**Timeout Configured:** 10s

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Latest quarterly results, earnings highlights  
**Expected Response Time:** 0.2-0.4s  
**Timeout Configured:** None (default)

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
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** ScripCode, Exchange, Symbol for chart generation  
**Expected Response Time:** 0.1-0.3s  
**Timeout Configured:** 10s (connect + response)

---

### 8. Company Quote Details
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:702)

```bash
curl -X GET "https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{CO_CODE}/nse?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

Replace `{CO_CODE}` with company code obtained from snapshot API.

**Response includes:** EPS, sector, market cap, company details  
**Expected Response Time:** 0.2-0.4s

---

### 9. Company Snapshot Profile
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:361)

```bash
curl -X GET "https://apihub.5paisa.com/atlas_cache/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** co_code for further API calls  
**Expected Response Time:** 0.2-0.4s

---

### 10. Sector Companies List
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1961)

```bash
curl -X GET "https://apihub.5paisa.com/atlas_cache/datamart/Equity/Market.svc/SectorWiseComp/{SECTOR_CODE}/?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

Replace `{SECTOR_CODE}` with sector code.

**Used for:** Similar stocks feature  
**Expected Response Time:** 0.3-0.8s

---

### 11. F&O Overview (Futures & Options)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2126)

```bash
# For Futures
curl -X GET "https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure

# For Options
curl -X GET "https://apihub.5paisa.com/atlas-ca/datamart/FNO/FutOpt.svc/FutOptOverview/Opt/TCS?responseType=json" \
  -H "Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Expected Response Time:** 0.2-0.5s each  
**Timeout Configured:** 10s

---

## External API Endpoints (MarketSmith India - via Kong Gateway)

### 12. Instrument Search (for Financial Data)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:1230-1259)

```bash
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instrument?symbol=TCS" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** instrumentId for finance details  
**Expected Response Time:** 0.3-0.6s  
**Timeout Configured:** 10s

---

### 13. Finance Details (Consolidated/Standalone)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:1269-1300)

```bash
# Consolidated (isConsolidated=true)
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=true" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure

# Standalone (isConsolidated=false)
curl -X GET "https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false" \
  -H "gateway-name: fivepaisa-gateway" \
  -H "x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

Replace `{INSTRUMENT_ID}` with the ID from instrument search.

**Response includes:** Balance sheet, P&L, cash flow, ratios data  
**Expected Response Time:** 0.5-1.2s  
**Timeout Configured:** 10s  
**Cache Duration:** 24 hours

---

### 14. Investment Ratings & Wisdom
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:403-418)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/wisdom.json?lang=en&ver=2&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Master rating, EPS rating, price strength, buyer demand  
**Expected Response Time:** 0.3-0.7s  
**Timeout Configured:** 10s  
**Cache Duration:** 24 hours

---

### 15. Broker Estimates (Forecasts)
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:381-395)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/getBrokerEstimates.json?instrumentId={INSTRUMENT_ID}&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Target price, revenue estimates, EPS forecasts  
**Expected Response Time:** 0.4-0.8s  
**Timeout Configured:** 10s

---

### 16. Ownership & Management Data
**Source:** [`StockPageService.php`](../modules/custom/fivepaisa_stock_page/src/StockPageService.php:426-442)

```bash
curl -X GET "https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/{INSTRUMENT_ID}/financeDetails.json?isConsolidated=false&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n" \
  --insecure
```

**Response includes:** Shareholding pattern, management info  
**Expected Response Time:** 0.4-0.9s  
**Timeout Configured:** 10s

---

## Internal Drupal API Routes

### 17. Financial Data (AJAX)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:560-613)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:737)

```bash
curl -X GET "https://www.5paisa.com/financial-data/QuarterlyPLStandaloneResult/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
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

**Expected Response Time:** 0.3-0.8s  
**Cache Duration:** 24 hours

---

### 18. Sector Link Information
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:691-738)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:680-702)

```bash
curl -X GET "https://www.5paisa.com/api/TCS/sector-cap-info" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Response includes:** Sector name, sector link, market cap category (Small/Mid/Large Cap)  
**Expected Response Time:** 0.2-0.5s

---

### 19. Similar Stocks
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1916-2024)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:579)

```bash
curl -X GET "https://www.5paisa.com/api/get-similar-stocks?nsecode=TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Response includes:** HTML with 4 similar stocks from same sector  
**Expected Response Time:** 0.5-1.5s (makes multiple external API calls)

---

### 20. FnO Check (Futures & Options Availability)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2097-2114)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:641-677)

```bash
curl -X GET "https://www.5paisa.com/check-fno/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Response:**
```json
{
  "future_exist": "true",
  "option_exist": "true"
}
```

**Expected Response Time:** 0.1-0.3s  
**Cache Duration:** 2 days

---

### 21. Shareholding Pattern - Months
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:647-652)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-months/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Expected Response Time:** 0.3-0.7s

---

### 22. Shareholding Pattern - Names
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:679-684)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:535)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-names-json/Dec-2024/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Expected Response Time:** 0.2-0.5s

---

### 23. Shareholding Pattern - Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:663-668)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:536)

```bash
curl -X GET "https://www.5paisa.com/shareholding-pattern-data-json/Dec-2024/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Expected Response Time:** 0.2-0.5s

---

### 24. Forecast Data
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:812-865)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:477)

```bash
curl -X GET "https://www.5paisa.com/get-forecast-data/TCS" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Response includes:** Consensus rating, target price, analyst recommendations  
**Expected Response Time:** 0.4-0.8s

---

### 25. Price Range Filter (Chart Filter)
**Source:** [`StockPageController.php`](../modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2596-2629)
**JavaScript:** [`stock-detail.js`](../modules/custom/fivepaisa_stock_page/js/stock-detail.js:767-807)

```bash
curl -X GET "https://www.5paisa.com/price-range-filter/TCS/1W" \
  -H "Accept: application/json" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\nTotal Time: %{time_total}s | HTTP: %{http_code}\n"
```

**Valid Range Filters:** `1D`, `1W`, `1M`, `6M`, `1Y`, `5Y`, `Max`  
**Expected Response Time:** 0.2-0.4s


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

---

## Performance Benchmarks

### API Response Time Summary

| API Endpoint | Expected Time | Timeout | Cache | Priority |
|--------------|---------------|---------|-------|----------|
| **Kong Gateway APIs (Pearl)** |
| Stock Overview | 0.2-0.5s | 10s | 24h | Critical |
| Fundamental Data | 0.2-0.4s | 10s | 24h | High |
| Technical Analysis | 0.3-0.6s | 10s | 24h | High |
| Corporate Actions | 0.2-0.5s | Default | 24h | Medium |
| Tech Trend (SWOT) | 0.2-0.4s | 10s | None | Medium |
| Rapid Results | 0.2-0.4s | Default | 24h | Medium |
| **Atlas Gateway APIs** |
| Script Code Search | 0.1-0.3s | 10s | 24h | Critical |
| Company Quote Details | 0.2-0.4s | Default | None | High |
| Company Snapshot | 0.2-0.4s | Default | None | High |
| Sector Companies | 0.3-0.8s | Default | None | Low |
| F&O Overview | 0.2-0.5s | 10s | 2 days | Medium |
| **MarketSmith APIs** |
| Instrument Search | 0.3-0.6s | 10s | None | High |
| Finance Details | 0.5-1.2s | 10s | 24h | Medium |
| Investment Ratings | 0.3-0.7s | 10s | 24h | Medium |
| Broker Estimates | 0.4-0.8s | 10s | None | Low |
| Ownership/Management | 0.4-0.9s | 10s | None | Low |
| **Internal Drupal APIs** |
| Financial Data | 0.3-0.8s | Default | 24h | Medium |
| Similar Stocks | 0.5-1.5s | Default | None | Low |
| Sector/Cap Info | 0.2-0.5s | Default | None | Medium |
| FnO Check | 0.1-0.3s | Default | 2 days | Low |
| Shareholding Pattern | 0.3-0.7s | Default | None | Low |
| Price Range Filter | 0.2-0.4s | Default | None | Low |

### Total Page Load Time Breakdown

**Sequential API Calls (Server-Side):**
1. Script Code Search: ~0.2s
2. Overview API: ~0.4s
3. Fundamental API: ~0.3s
4. Technical Analysis: ~0.5s
5. Corporate Actions: ~0.4s

**Total Backend API Time:** ~3.5s  

**Lazy-Loaded APIs (Client-Side):**
- Triggered after scroll/interaction
- Execute in parallel
- Combined time: ~1.0-2.5s (parallel execution)

### Measuring Complete Page Load

```bash
# Measure total page load with all timing details
curl -X GET "https://www.5paisa.com/stocks/tcs-share-price" \
  -H "User-Agent: Mozilla/5.0" \
  -w "\n
=== PAGE LOAD TIMING ===
DNS Lookup:      %{time_namelookup}s
TCP Connect:     %{time_connect}s
TLS Handshake:   %{time_appconnect}s
Start Transfer:  %{time_starttransfer}s
TOTAL TIME:      %{time_total}s
HTTP Code:       %{http_code}
Size Downloaded: %{size_download} bytes
=========================\n" \
  -o /dev/null -s
```

**Expected Output:**
```
=== PAGE LOAD TIMING ===
DNS Lookup:      0.012s
TCP Connect:     0.135s
TLS Handshake:   0.267s
Start Transfer:  2.145s
TOTAL TIME:      2.456s
HTTP Code:       200
Size Downloaded: 45678 bytes
=========================
```
