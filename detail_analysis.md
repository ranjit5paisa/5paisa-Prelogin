# Stock Page - Complete API Flows with Request/Response Examples

**Module:** `fivepaisa_stock_page`  
**Purpose:** Complete API call chains showing dependencies, requests, and responses

---

## API Call Chains by Section

### 1. STOCK PROFILE SECTION - API Flow

#### Step 1: Get Stock Overview Data
**API:** Overview API  
**Endpoint:** `GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/overview/TCS/`

**Request Headers:**
```http
GET /pearl-ca/clientapi/pearlapi/stock/overview/TCS/ HTTP/1.1
Host: apihub.5paisa.com
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
requestCode: pearlapi
UserId: 5PAISAAPI
Password: 5nadynsiitnienny
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
      "Tata Consultancy Services Ltd.",  // [0] Stock Name
      "TCS",                              // [1] NSE Code
      "",                                 // [2]
      "",                                 // [3]
      "532540",                           // [4] BSE Code
      "INE467B01029",                     // [5] ISIN
      "",                                 // [6]
      2264.00,                            // [7] Current Price
      "",                                 // [8]
      "",                                 // [9]
      18.00,                              // [10] Price Change
      0.8                                 // [11] Price Change %
    ],
    "lastUpdated": "15 May, 2026 3:57 PM (IST)",
    "fundamentalData": [
      {
        "name": "Market Cap Cr",
        "value": 819135
      },
      {
        "name": "Price to Earnings",
        "value": 16.6
      },
      {
        "name": "PE to Growth",
        "value": 12.3
      },
      {
        "name": "Price to Book Value Adjusted",
        "value": 7
      },
      {
        "name": "Dividend Yield",
        "value": 4.8
      }
    ],
    "technicalData": [
      {
        "name": "Money Flow Index",
        "value": 32.09
      },
      {
        "name": "Relative Strength Index",
        "value": 27.41
      }
    ],
    "52_Week_High": 3600,
    "52_Week_low": 2206
  }
}
```

**Data Extracted:**
- Stock Name: "Tata Consultancy Services Ltd."
- NSE Code: "TCS"
- BSE Code: "532540"
- Current Price: 2264.00
- Price Change: +18.00 (0.8%)
- Last Updated: "15 May, 2026 3:57 PM (IST)"

**Service Method:** [`StockPageService.php:940`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:940) - `getPerformance()`

---

#### Step 2: Get Company Code (co_code)
**API:** Company Profile API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-req1-rt/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/TCS?responseType=json`

**Request Headers:**
```http
GET /atlas-req1-rt/datamart/Equity/CompanyInfo.svc/snapshotcompprofile-version2/TCS?responseType=json HTTP/1.1
Host: apihub.5paisa.com
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**Request Parameters:**
- `nsecode`: TCS
- `responseType`: json

**Response:**
```json
{
  "response": {
    "@type": "success",
    "data": {
      "snapshotcompprofilelist": {
        "snapshotcompprofile": {
          "co_code": "18564",
          "comp_name": "Tata Consultancy Services Ltd.",
          "industry": "IT - Software"
        }
      }
    }
  }
}
```

**Data Extracted:**
- **co_code: "18564"** ← REQUIRED for next API

**Service Method:** [`StockPageService.php:358`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:358) - `getCoCode()`

---

#### Step 3: Get Quote Details (for Sector Info)
**API:** Quote Details API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/18564/nse?responseType=json`

**Request Headers:**
```http
GET /atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/18564/nse?responseType=json HTTP/1.1
Host: apihub.5paisa.com
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**Request Parameters:**
- `co_code`: 18564 (from Step 2)
- `exchange`: nse
- `responseType`: json

**Response:**
```json
{
  "response": {
    "data": {
      "getquotedetailslist": {
        "@recordcount": "1",
        "getquotedetails": {
          "symbol": "TCS",
          "complname": "Tata Consultancy Services Ltd.",
          "companysector": "IT - Software",
          "companysectorcode": "15",
          "high_price": 2305.00,
          "low_price": 2252.00,
          "price": 2264.00,
          "hi_52_wk": 3600.00,
          "lo_52_wk": 2206.00,
          "open_price": 2252.00,
          "oldprice": 2246.00,
          "volume": 3410637,
          "eps": 144.66
        }
      }
    }
  }
}
```

**Data Extracted:**
- **companysector:** "IT - Software"
- **companysectorcode:** "15"
- Sector Link: `/stocks/sector/it-software`
- Cap Category: Large Cap (based on Market Cap > 20000 Cr)
- Cap Link: `/share-market-today/large-cap-stocks`

**Service Method:** [`StockPageController.php:696`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:696) - `getSectorLink()`

---

### 2. PERFORMANCE SECTION - API Flow

#### API Used: Quote Details (Same as Step 3 above)

**Data Points Extracted from Same Response:**

```php
// Day High/Low
$day_high = 2305.00;
$day_low = 2252.00;
$day_progress = (($current_price - $day_low) / ($day_high - $day_low)) * 100;
// = ((2264 - 2252) / (2305 - 2252)) * 100 = 22.68%

// 52 Week High/Low
$fiftytwo_high = 3600.00;
$fiftytwo_low = 2206.00;
$fiftytwo_progress = (($current_price - $fiftytwo_low) / ($fiftytwo_high - $fiftytwo_low)) * 100;
// = ((2264 - 2206) / (3600 - 2206)) * 100 = 4.13%

// Open/Previous/Volume
$open_price = 2252.00;
$prev_close = 2246.00;
$volume = 3410637;
```

---

#### Get Moving Averages (DMA)
**API:** Technical Analysis API  
**Endpoint:** `GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/`

**Request Headers:**
```http
GET /pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/ HTTP/1.1
Host: apihub.5paisa.com
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
requestCode: pearlapi
UserId: 5PAISAAPI
Password: 5nadynsiitnienny
```

**Response:**
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "movingAverages": {
      "tableData": [
        ["20 Day", 2447.42, 2410.08],
        ["50 Day", 2467.98, 2517.46],
        ["100 Day", 2773.76, 2681.88],
        ["200 Day", 2925.16, 2912.17]
      ]
    },
    "stockPriceData": {
      "currentPrice": 2264.00,
      "dayChange": 18.00,
      "dayChangeP": 0.8
    }
  }
}
```

**Data Extracted:**
```php
// Format: ["Period", SMA, EMA]
$dma_50 = 2517.46;  // EMA (index [2])
$dma_100 = 2681.88; // EMA
$dma_200 = 2912.17; // EMA
```

**Service Method:** [`StockPageService.php:789`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:789) - `openPrevData()`

---

### 3. CHART SECTION - API Flow

#### Get Scrip Code
**API:** Search Scrip API  
**Endpoint:** `POST https://gateway.5paisa.com/prelogin/prod/searchscrip`

**Request Headers:**
```http
POST /prelogin/prod/searchscrip HTTP/1.1
Host: gateway.5paisa.com
UserID: ZyT47UW2g56
Password: H98qlU4Sn2
Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81
Content-Type: application/json
```

**Request Body:**
```json
{
  "Symbol": "TCS",
  "RecordCount": "1"
}
```

**Response:**
```json
{
  "Message": "Success",
  "Status": 1,
  "Data": [
    {
      "Exch": "N",
      "ExchType": "C",
      "ScripCode": 11536,
      "ScripName": "TCS",
      "Symbol": "TCS",
      "Exchange": "N",
      "Series": "EQ"
    },
    {
      "Exch": "B",
      "ExchType": "C",
      "ScripCode": 532540,
      "ScripName": "TCS",
      "Symbol": "TCS",
      "Exchange": "B",
      "Series": "A"
    }
  ]
}
```

**Data Extracted:**
```php
// Filter by matching Symbol === 'TCS'
$scripCode = 11536;
$symbol = "TCS";
$exchange = "N"; // N=NSE, B=BSE
$exchange_name = ($exchange == 'N') ? 'nse' : 'bse'; // nse
```

**Generated Chart URL:**
```
https://tradechart.5paisa.com/5pWebChart/?type=overview&exchange=nse&exchType=C&symbol=TCS&scripCode=11536&osName=TTWEB&source=5pWebsite&theme=Light
```

**Service Method:** [`StockPageService.php:1013`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:1013) - `getGraph()`

---

### 4. INVESTMENT RETURNS - API Flow

#### API: Technical Analysis (Price Analysis)
**Endpoint:** `GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/technical-analysis/TCS/`

**Same API as DMA**, different data extracted:

**Response Section:**
```json
{
  "body": {
    "priceAnalysis": {
      "month": {
        "changePercent": -8.32,
        "color": "negative"
      },
      "quarter": {
        "changePercent": -15.79,
        "color": "negative"
      },
      "halfYear": {
        "changePercent": -27.01,
        "color": "negative"
      },
      "year": {
        "changePercent": -36.12,
        "color": "negative"
      }
    }
  }
}
```

**Data Extracted:**
```php
// 1 Month Return
$month_return = "-8.32%";
$month_color = "stock--down"; // negative

// 3 Month Return
$quarter_return = "-15.79%";
$quarter_color = "stock--down";

// 6 Month Return
$halfyear_return = "-27.01%";
$halfyear_color = "stock--down";

// 1 Year Return
$year_return = "-36.12%";
$year_color = "stock--down";
```

**Service Method:** [`StockPageController.php:1313`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1313) - `getReturns()`

---

### 5. FUNDAMENTALS SECTION - API Flow

#### APIs Already Called:
1. Overview API (Step 1) - Gets P/E, PEG, Market Cap, P/B, Dividend Yield, MFI, RSI
2. Quote Details API (Step 3) - Gets EPS
3. Technical Analysis API (Performance) - Gets MACD Signal, ATR

**Complete Data Mapping:**

From **Overview API** (`fundamentalData`):
```json
{
  "fundamentalData": [
    {"name": "Price to Earnings", "value": 16.6},        // P/E Ratio
    {"name": "PE to Growth", "value": 12.3},             // PEG Ratio
    {"name": "Market Cap Cr", "value": 819135},          // Market Cap
    {"name": "Price to Book Value Adjusted", "value": 7}, // P/B Ratio
    {"name": "Dividend Yield", "value": 4.8}             // Dividend Yield
  ],
  "technicalData": [
    {"name": "Money Flow Index", "value": 32.09},        // MFI
    {"name": "Relative Strength Index", "value": 27.41} // RSI
  ]
}
```

From **Quote Details API** (`getquotedetails`):
```json
{
  "eps": 144.66
}
```

From **Technical Analysis API** (`technicals`):
```json
{
  "technicals": [
    {"name": "MACD Signal", "value": -38.06},
    {"name": "Average True Range", "value": 61.43}
  ]
}
```

**Final Output:**
```html
<ul>
  <li>P/E Ratio</li>
  <li>16.6</li>
</ul>
<ul>
  <li>PEG Ratio</li>
  <li>12.3</li>
</ul>
<ul>
  <li>Market Cap Cr</li>
  <li>819,135</li>
</ul>
<ul>
  <li>P/B Ratio</li>
  <li>7</li>
</ul>
<ul>
  <li>Average True Range</li>
  <li>61.43</li>
</ul>
<ul>
  <li>EPS</li>
  <li>144.66</li>
</ul>
<ul>
  <li>Dividend Yield</li>
  <li>4.8</li>
</ul>
<ul>
  <li>MACD Signal</li>
  <li>-38.06</li>
</ul>
<ul>
  <li>RSI</li>
  <li>27.41</li>
</ul>
<ul>
  <li>MFI</li>
  <li>32.09</li>
</ul>
```

**Service Method:** [`StockPageService.php:520`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:520) - `getKeyStats()`

---

### 6. FINANCIAL SECTION - Complete API Flow

#### Step 1: Get Instrument ID
**API:** Instrument Search API  
**Endpoint:** `GET https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instrument?symbol=TCS`

**Request Headers:**
```http
GET /marketsmith-ca/gateway/broker/instrument?symbol=TCS HTTP/1.1
Host: apihub.5paisa.com
gateway-name: fivepaisa-gateway
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs
```

**Response:**
```json
{
  "response": {
    "results": {
      "instrumentId": "12345",
      "symbol": "TCS",
      "industryName": "IT - Software"
    }
  }
}
```

**Data Extracted:**
- **instrumentId: "12345"** ← REQUIRED for next API
- industryName: "IT - Software" ← determines which data headers to use

**Service Method:** [`StockPageService.php:1230`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:1230) - `getInstrumentIdFinancial()`

---

#### Step 2: Get Financial Data (Consolidated)
**API:** Finance Details API  
**Endpoint:** `GET https://apihub.5paisa.com/marketsmith-ca/gateway/broker/instr/0/12345/financeDetails.json?isConsolidated=true`

**Request Headers:**
```http
GET /marketsmith-ca/gateway/broker/instr/0/12345/financeDetails.json?isConsolidated=true HTTP/1.1
Host: apihub.5paisa.com
gateway-name: fivepaisa-gateway
x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJicm9rZXJOYW1lIjoiRml2ZSBQYWlzYSIsImlhdCI6MTY0NjkyNDI1MX0.ZbzfGQcoUImEYL0YpyMnzJxJxMb6dWzJKQCpDgXnqf9Fs
```

**Request Parameters:**
- `instrumentId`: 12345 (from Step 1)
- `isConsolidated`: true (or false for Standalone)

**Response:**
```json
{
  "incQuarterHeader": [
    {"id": "c1", "displayName": "Revenue"},
    {"id": "c3", "displayName": "Operating Profit"},
    {"id": "c17", "displayName": "Interest"},
    {"id": "c19", "displayName": "Depreciation"},
    {"id": "c23", "displayName": "Profit before tax"},
    {"id": "c25", "displayName": "Net Profit"}
  ],
  "incQuarterData": [
    {
      "c0": "Mar-26",
      "c1": 68254.00,
      "c3": 24512.00,
      "c17": 145.00,
      "c19": 1823.00,
      "c23": 22544.00,
      "c25": 17254.00
    },
    {
      "c0": "Dec-25",
      "c1": 65487.00,
      "c3": 23654.00,
      "c17": 138.00,
      "c19": 1785.00,
      "c23": 21731.00,
      "c25": 16589.00
    }
  ],
  "incAnnualHeader": [...],
  "incAnnualData": [...],
  "bsHeader": [...],
  "bsData": [...],
  "cfHeader": [...],
  "cfData": [...],
  "ratiosAnnualHeader": [...],
  "ratiosAnnualData": [...]
}
```

**Data Processing:**

1. **Filter Headers** (based on industry):
   ```php
   // For IT-Software industry
   $QuarterlyPL_headers = ["c1", "c3", "c17", "c19", "c23", "c25"];
   ```

2. **Map Headers to Display Names:**
   ```php
   $displayNames = [
     "c1" => "Revenue",
     "c3" => "Operating Profit",
     "c17" => "Interest",
     "c19" => "Depreciation",
     "c23" => "Profit before tax",
     "c25" => "Net Profit"
   ];
   ```

3. **Transform Data:**
   ```php
   $result = [
     "Mar 26" => [
       "Revenue" => 68254.00,
       "Operating Profit" => 24512.00,
       "Interest" => 145.00,
       "Depreciation" => 1823.00,
       "Profit before tax" => 22544.00,
       "Net Profit" => 17254.00
     ],
     "Dec 25" => [
       "Revenue" => 65487.00,
       "Operating Profit" => 23654.00,
       "Interest" => 138.00,
       "Depreciation" => 1785.00,
       "Profit before tax" => 21731.00,
       "Net Profit" => 16589.00
     ]
   ];
   ```

4. **Generate HTML Table:**
   ```html
   <table class="table stock-table stock-table-number">
     <thead>
       <tr>
         <th>Indicator</th>
         <th>Mar 26</th>
         <th>Dec 25</th>
       </tr>
     </thead>
     <tbody>
       <tr>
         <td>Revenue</td>
         <td>68,254.00</td>
         <td>65,487.00</td>
       </tr>
       <!-- ... more rows -->
     </tbody>
   </table>
   ```

**Service Methods:**
- `StockPageService.php:1269` - `financeDetailsApi()`
- `StockPageService.php:1400` - `getFinancialData()` (Consolidated)
- `StockPageService.php:1488` - `getFinancialDataStandalone()`

---

#### Step 3: AJAX Load Financial Tab
**Internal API:** `/financial-data/{tabFilter}/{nseCode}`  
**Example:** `GET /financial-data/QuarterlyPLConsolidatedResult/TCS`

**JavaScript Request:**
```javascript
const response = await fetch('/financial-data/QuarterlyPLConsolidatedResult/TCS');
const data = await response.json();
```

**Response:**
```json
{
  "table": "<table class='table stock-table stock-table-number'>...</table>"
}
```

**Controller Method:** [`StockPageController.php:565`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:565) - `getFinancialData()`

---

### 7. TECHNICAL SECTION - API Flow

#### Sub-section 7.1: EMA & SMA

**API:** Technical Analysis (Same as Performance Section)

**Response Section:**
```json
{
  "body": {
    "movingAverages": {
      "tableData": [
        ["20 Day", 2447.42, 2410.08],
        ["50 Day", 2467.98, 2517.46],
        ["100 Day", 2773.76, 2681.88],
        ["200 Day", 2925.16, 2912.17]
      ]
    }
  }
}
```

**Data Mapping:**
```php
// Format: ["Period", SMA (index 1), EMA (index 2)]
$movingAvg = [
  "20 Day" => ["SMA" => 2447.42, "EMA" => 2410.08],
  "50 Day" => ["SMA" => 2467.98, "EMA" => 2517.46],
  "100 Day" => ["SMA" => 2773.76, "EMA" => 2681.88],
  "200 Day" => ["SMA" => 2925.16, "EMA" => 2912.17]
];
```

---

#### Sub-section 7.2: Tech Trend (Bullish/Bearish)

**API:** Tech Trend API  
**Endpoint:** `GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/stock/tech-trend/TCS`

**Request Headers:**
```http
GET /pearl-ca/clientapi/pearlapi/stock/tech-trend/TCS HTTP/1.1
Host: apihub.5paisa.com
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
UserId: 5PAISAAPI
password: 5nadynsiitnienny
requestCode: pearlapi
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
        "total": 16,
        "bearish": 16,
        "bullish": 0
      }
    }
  }
}
```

**Data Calculation:**
```php
$total = 16;
$bearish = 16;
$bullish = 0;
$bullish_percentage = ($bullish / $total) * 100; // = 0%
```

**HTML Output:**
```html
<div class="stock-page__bullishbg">
  <ul class="bullish-box">
    <li><!-- Bear icon --></li>
    <li>
      Bearish Moving Average
      <span class="number__text">16</span>
    </li>
  </ul>
  <ul class="bearish-box">
    <li><!-- Bull icon --></li>
    <li>
      Bullish Moving Average
      <span class="number__text">0</span>
    </li>
  </ul>
</div>
```

**Service Method:** [`StockPageController.php:1258`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1258) - `getSwotData()`

---

#### Sub-section 7.3: Pivot Points (Resistance & Support)

**API:** Technical Analysis (Same API)

**Response Section:**
```json
{
  "body": {
    "pivotData": {
      "pivot": 2239.13,
      "r1": 2271.87,
      "r2": 2297.73,
      "r3": 2330.47,
      "s1": 2213.27,
      "s2": 2180.53,
      "s3": 2154.67
    }
  }
}
```

**HTML Output:**
```html
<div class="resistance-support__speedo">
  <span class="resistance-support__speedotext centre">2239.13</span>
  <img src=".../pointer.svg" alt="Pivot Speed">
</div>
<div class="resistance-support__value">
  <div class="resistance-support__valueleft">
    <ul>
      <li><span>R3</span>2,330.47</li>
      <li><span>R2</span>2,297.73</li>
      <li><span>R1</span>2,271.87</li>
    </ul>
  </div>
  <div class="resistance-support__valueright">
    <ul>
      <li><span>S1</span>2,213.27</li>
      <li><span>S2</span>2,180.53</li>
      <li><span>S3</span>2,154.67</li>
    </ul>
  </div>
</div>
```

---

#### Sub-section 7.4: Ratings

##### Step 1: Get Instrument ID
**API:** Instrument Search API (MarketSmith Legacy)  
**Endpoint:** `GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/srch.json?text=TCS&lang=en&ver=2&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196`

**Response:**
```json
{
  "response": {
    "responseHeader": {
      "error": false
    },
    "results": [
      {
        "instrumentId": "54321",
        "symbol": "TCS",
        "exchange": "NSE"
      }
    ]
  }
}
```

**Data Extracted:**
- **instrumentId: "54321"** ← REQUIRED for ratings API

**Service Method:** [`StockPageService.php:449`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:449) - `getInstrumentId()`

---

##### Step 2: Get Investment Rating
**API:** Investment Rating API  
**Endpoint:** `GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/54321/wisdom.json?lang=en&ver=2&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196`

**Request Parameters:**
- `instrumentId`: 54321 (from Step 1)
- `lang`: en
- `ver`: 2
- `ms-auth`: authentication token

**Response:**
```json
{
  "response": {
    "responseHeader": {
      "error": false
    },
    "results": [
      {
        "name": "Master Score",
        "pageGroupLink": 1,
        "itemValue": "C",
        "bodyText": "Tata Consultancy Services Ltd. (TCS) is a global IT services, consulting, and business solutions leader..."
      },
      {
        "name": "EPS Rating",
        "itemValue": "45"
      },
      {
        "name": "RS Rating",
        "itemValue": "38"
      },
      {
        "name": "A/D Rating",
        "itemValue": "B"
      },
      {
        "name": "Industry Group Rank",
        "itemValue": "15 of 197"
      }
    ]
  }
}
```

**Data Processing:**

1. **Master Score → Stars:**
   ```php
   $grade = "C";
   $star_count = ($grade == 'A') ? 1 : (($grade == 'B') ? 2 : (($grade == 'C') ? 3 : 4));
   // C = 3 stars
   ```

2. **EPS Strength → Stars:**
   ```php
   $eps_value = 45; // 0-100 scale
   if ($eps_value >= 90) $stars = 5;
   elseif ($eps_value >= 80) $stars = 4;
   elseif ($eps_value >= 60) $stars = 3;
   else $stars = 2;
   // 45 = 2 stars
   ```

3. **Price Strength (RS Rating) → Stars:**
   ```php
   $rs_value = 38;
   // Same logic as EPS
   // 38 = 2 stars
   ```

4. **Buyer Demand (A/D Rating) → Stars:**
   ```php
   $ad_rating = "B";
   // A-level = 5★, B-level = 4★, C-level = 3★, D/E = 2★
   // B = 4 stars
   ```

5. **Group Rank → Stars:**
   ```php
   $rank_text = "15 of 197";
   $rank = 15; // Extract number before "of"
   if ($rank >= 1 && $rank <= 20) $stars = 5;
   elseif ($rank >= 21 && $rank <= 40) $stars = 4;
   elseif ($rank >= 41 && $rank <= 60) $stars = 3;
   else $stars = 2;
   // 15 = 5 stars
   ```

**HTML Output:**
```html
<div class="stock-page__count">
  <div class="stock-page__starbox active">
    <p>Master Rating</p>
    <div class="stock__reviewwrapper">
      <i class="star"></i>
      <i class="star"></i>
      <i class="star"></i>
      <i class="star-no"></i>
      <i class="star-no"></i>
    </div>
  </div>
  <div class="stock-page__starbox">
    <p>EPS Strength</p>
    <div class="stock__reviewwrapper">
      <i class="star"></i>
      <i class="star"></i>
      <i class="star-no"></i>
      <i class="star-no"></i>
      <i class="star-no"></i>
    </div>
  </div>
  <!-- Similar for other ratings -->
</div>
```

**Service Method:** [`StockPageService.php:403`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:403) - `getInvestRating()`  
**Controller Method:** [`StockPageController.php:2327`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2327) - `getRatingsData()`

---

### 8. EVENTS/CORPORATE ACTIONS - API Flow

#### Get Events Calendar
**API:** Events Calendar API  
**Endpoint:** `GET https://apihub.5paisa.com/pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS`

**Request Headers:**
```http
GET /pearl-ca/clientapi/pearlapi/events/calendar/stock/TCS HTTP/1.1
Host: apihub.5paisa.com
Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b
Ocp-Apim-Trace: true
Content-Type: application/json
KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44
requestCode: pearlapi
UserId: 5PAISAAPI
password: 5nadynsiitnienny
```

**Response:**
```json
{
  "head": {
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "eventsData": [
      {
        "eventType": "Board Meeting",
        "boardMeetDate": "2026-04-09",
        "purpose": "Audited Results & Final Dividend",
        "remarks": ""
      },
      {
        "eventType": "Dividend",
        "recordDate": "2026-05-25",
        "purpose": "FINAL",
        "remarks": "Rs.31.00 per share(3100%)Final Dividend"
      },
      {
        "eventType": "Dividend",
        "recordDate": "2026-01-17",
        "purpose": "SPECIAL",
        "remarks": "Rs.46.00 per share(4600%)Special Dividend"
      },
      {
        "eventType": "Bonus",
        "recordDate": "2025-12-15",
        "purpose": "Bonus Issue",
        "remarks": "1:1 Bonus"
      },
      {
        "eventType": "Split",
        "recordDate": "2024-06-10",
        "purpose": "Stock Split",
        "remarks": "Face Value Split from Rs.10 to Rs.1"
      }
    ]
  }
}
```

**Data Processing:**

1. **Filter by Event Type:**
   ```php
   $board_meetings = [];
   $dividends = [];
   $bonus = [];
   $splits = [];
   
   foreach ($eventsData as $event) {
     if ($event['eventType'] == 'Board Meeting') {
       $board_meetings[] = $event;
     }
     elseif ($event['eventType'] == 'Dividend') {
       $dividends[] = $event;
     }
     // ... etc
   }
   ```

2. **Sort by Date (Descending):**
   ```php
   usort($dividends, function($a, $b) {
     return strtotime($b['recordDate']) - strtotime($a['recordDate']);
   });
   ```

3. **Limit to 5 Records:**
   ```php
   $dividends = array_slice($dividends, 0, 5);
   ```

4. **Format Amount in Remarks:**
   ```php
   $remarks = "Rs.31.00 per share(3100%)Final Dividend";
   // Extract Rs.31.00
   if (strpos($remarks, 'Rs.') !== false) {
     $float = floatval(str_replace("Rs.", "", explode(" ", $remarks)[0]));
     $formatted_remarks = "Rs." . number_format($float, 2) . " per share...";
   }
   ```

**HTML Output (Board Meetings):**
```html
<table class="table stock-table corporate-table">
  <thead>
    <tr>
      <th>Date</th>
      <th>Purpose</th>
      <th>Remarks</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2026-04-09</td>
      <td>Audited Results & Final Dividend</td>
      <td></td>
    </tr>
    <!-- ... more rows -->
  </tbody>
</table>
```

**HTML Output (Dividends):**
```html
<table class="table stock-table corporate-table">
  <thead>
    <tr>
      <th>Date</th>
      <th>Purpose</th>
      <th>Remarks</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>2026-05-25</td>
      <td>FINAL</td>
      <td>Rs.31.00 per share(3100%)Final Dividend</td>
    </tr>
    <tr>
      <td>2026-01-17</td>
      <td>SPECIAL</td>
      <td>Rs.46.00 per share(4600%)Special Dividend</td>
    </tr>
    <!-- ... more rows -->
  </tbody>
</table>
```

**Controller Method:** [`StockPageController.php:878`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:878) - `getCorporateActionData()`

---

### 9. F&O SECTION - API Flow

#### Check F&O Availability
**API:** FutOpt Overview API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-req1-rt/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json`

**Request Headers:**
```http
GET /atlas-req1-rt/datamart/FNO/FutOpt.svc/FutOptOverview/Fut/TCS?responseType=json HTTP/1.1
Host: apihub.5paisa.com
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**Request Parameters:**
- `type`: Fut (for Futures) or Opt (for Options)
- `nsecode`: TCS
- `responseType`: json

**Response (Futures Available):**
```json
{
  "response": {
    "data": {
      "companylist": {
        "@recordcount": "1",
        "company": [
          {
            "symbol": "TCS",
            "series": "FUTIDX"
          }
        ]
      }
    }
  }
}
```

**Response (No F&O):**
```json
{
  "response": {
    "data": {
      "companylist": {
        "@recordcount": "0"
      }
    }
  }
}
```

**Internal API Response:**
```json
{
  "future_exist": "true",
  "option_exist": "true"
}
```

**JavaScript Processing:**
```javascript
const response = await fetch('/check-fno/TCS');
const data = await response.json();

if (data.future_exist === "true") {
  // Add Futures link
  fnoWrapper.innerHTML += `
    <a href="/derivatives/tcs-futures">
      <img src=".../fno-futures.svg">
      TCS Futures
    </a>
  `;
  
  // Add Option Chain link
  fnoWrapper.innerHTML += `
    <a href="/derivatives/tcs-option-chain">
      <img src=".../fno-chain.svg">
      TCS Option Chain
    </a>
  `;
}
```

**Service Method:** [`StockPageController.php:2126`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2126) - `getFutOptOverview()`  
**AJAX Endpoint:** [`StockPageController.php:2102`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:2102) - `displayFnoData()`

---

### 10. SHAREHOLDING PATTERN - Complete API Flow

#### Step 1: Get Instrument ID (MarketSmith)
**API:** Same as Ratings Section  
**instrumentId:** 54321

---

#### Step 2: Get Finance Details (Shareholding Data)
**API:** Finance Details API  
**Endpoint:** `GET https://msi-gcloud-prod.appspot.com/gateway/simple-api/ms-india/instr/0/54321/financeDetails.json?isConsolidated=false&ms-auth=3990+MarketSmithINDUID-Web0000000000+MarketSmithINDUID-Web0000000000+0+210921231441+159745196`

**Request Parameters:**
- `instrumentId`: 54321
- `isConsolidated`: false
- `ms-auth`: authentication token

**Response:**
```json
{
  "responseHeader": {
    "error": false
  },
  "shareholdingLatestFourQuaterPatternModel": [
    {
      "month": "Mar-26",
      "name": "Promoters",
      "value": 71.77,
      "displayOrder": 1
    },
    {
      "month": "Mar-26",
      "name": "Mutual Funds",
      "value": 5.77,
      "displayOrder": 2
    },
    {
      "month": "Mar-26",
      "name": "Insurance  Companies",
      "value": 6.69,
      "displayOrder": 3
    },
    {
      "month": "Mar-26",
      "name": "Foreign Portfolio Investors",
      "value": 9.66,
      "displayOrder": 4
    },
    {
      "month": "Mar-26",
      "name": "Financial Institutions/ Banks",
      "value": 0.01,
      "displayOrder": 5
    },
    {
      "month": "Mar-26",
      "name": "Individual Investors",
      "value": 4.52,
      "displayOrder": 6
    },
    {
      "month": "Mar-26",
      "name": "Others",
      "value": 1.58,
      "displayOrder": 7
    },
    {
      "month": "Dec-25",
      "name": "Promoters",
      "value": 71.75,
      "displayOrder": 1
    }
    // ... more quarters
  ]
}
```

**Data Processing:**

1. **Group by Month:**
   ```php
   $grouped = [];
   foreach ($shareholdingData as $entry) {
     $month = $entry['month'];
     if (!isset($grouped[$month])) {
       $grouped[$month] = [];
     }
     $grouped[$month][] = [
       'name' => $entry['name'],
       'value' => $entry['value'],
       'displayOrder' => $entry['displayOrder']
     ];
   }
   ```

2. **Extract Unique Months (Tabs):**
   ```php
   $months = array_keys($grouped); // ["Mar-26", "Dec-25", "Sep-25", "Jun-25"]
   ```

3. **Generate Tabs HTML:**
   ```html
   <ul class="stockpagenew__tabs" id="shareholding_pattern_tabs">
     <li><a href="#Mar-26" class="active">Mar 26</a></li>
     <li><a href="#Dec-25">Dec 25</a></li>
     <li><a href="#Sep-25">Sep 25</a></li>
     <li><a href="#Jun-25">Jun 25</a></li>
   </ul>
   ```

4. **Generate Category Names (Left Side):**
   ```html
   <div class="shareholding__data-left">
     <ul>
       <li><span class="data-color data-1"></span>Promoters</li>
       <li><span class="data-color data-2"></span>Mutual Funds</li>
       <li><span class="data-color data-3"></span>Insurance  Companies</li>
       <li><span class="data-color data-4"></span>Foreign Portfolio Investors</li>
       <li><span class="data-color data-5"></span>Financial Institutions/ Banks</li>
       <li><span class="data-color data-6"></span>Individual Investors</li>
       <li><span class="data-color data-7"></span>Others</li>
     </ul>
   </div>
   ```

5. **Generate Progress Bars (Right Side):**
   ```html
   <div class="shareholding__data-right">
     <div class="data-progress">
       <div class="data-1" style="width:71.77%"></div>
       <span>71.77%</span>
     </div>
     <div class="data-progress">
       <div class="data-2" style="width:5.77%"></div>
       <span>5.77%</span>
     </div>
     <!-- ... more categories -->
   </div>
   ```

**Service Method:** [`StockPageService.php:1059`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:1059) - `getShareholdingPatternApi()`

---

#### Step 3: AJAX Tab Change

**JavaScript Event:**
```javascript
document.querySelector('#shareholding_pattern_tabs a[href="#Dec-25"]').click();
```

**AJAX Request 1 (Names):**
```http
GET /shareholding-pattern-names-json/Dec-25/TCS HTTP/1.1
```

**Response:**
```json
{
  "data": "<div class='shareholding__data-left'>...</div>"
}
```

**AJAX Request 2 (Data):**
```http
GET /shareholding-pattern-data-json/Dec-25/TCS HTTP/1.1
```

**Response:**
```json
{
  "data": "<div class='shareholding__data-right'>...</div>"
}
```

**Controller Methods:**
- [`StockPageController.php:684`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:684) - `shareholdingPatternNamesJson()`
- [`StockPageController.php:668`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:668) - `shareholdingPatternDataJson()`

---

### 11. MORE SECTION - Similar Stocks API Flow

#### Step 1: Get Company Basic Info
**API:** Quotes Finder API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/quotesfinder/TCS?responseType=json`

**Request Headers:**
```http
GET /atlas-ba-rt/datamart/Equity/CompanyInfo.svc/quotesfinder/TCS?responseType=json HTTP/1.1
Host: apihub.5paisa.com
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**Response:**
```json
{
  "response": {
    "data": {
      "getquoteslist": {
        "@recordcount": "1",
        "getquotes": {
          "co_code": "18564",
          "symbol": "TCS",
          "exchange": "nse"
        }
      }
    }
  }
}
```

**Data Extracted:**
- **co_code: "18564"**
- exchange: "nse"

---

#### Step 2: Get Stock Details (for Sector Code)
**API:** Quote Details API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/18564/nse?responseType=json`

**Response:**
```json
{
  "response": {
    "data": {
      "getquotedetailslist": {
        "getquotedetails": {
          "companysectorcode": "15"
        }
      }
    }
  }
}
```

**Data Extracted:**
- **companysectorcode: "15"** ← REQUIRED for next API

---

#### Step 3: Get Sector Companies
**API:** Sector Companies API  
**Endpoint:** `GET https://apihub.5paisa.com/atlas-req1-rt/datamart/Equity/Market.svc/SectorWiseComp/15/?responseType=json`

**Request Headers:**
```http
GET /atlas-req1-rt/datamart/Equity/Market.svc/SectorWiseComp/15/?responseType=json HTTP/1.1
Host: apihub.5paisa.com
Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==
```

**Request Parameters:**
- `sectorCode`: 15 (from Step 2)
- `responseType`: json

**Response:**
```json
{
  "response": {
    "data": {
      "CompanyList": {
        "@recordcount": "50",
        "Company": [
          {
            "co_code": "11536",
            "sc_code": "532540",
            "exchange_nse": "Yes",
            "comp_name": "Infosys Ltd."
          },
          {
            "co_code": "18765",
            "sc_code": "500209",
            "exchange_nse": "Yes",
            "comp_name": "Wipro Ltd."
          },
          {
            "co_code": "19234",
            "sc_code": "507685",
            "exchange_nse": "Yes",
            "comp_name": "HCL Technologies Ltd."
          }
          // ... more companies
        ]
      }
    }
  }
}
```

**Data Processing:**
```php
// Shuffle for randomization
shuffle($companies);

// Limit to 4 companies
$similar_stocks = array_slice($companies, 0, 4);

// Exclude the current stock (TCS)
$similar_stocks = array_filter($similar_stocks, function($company) {
  return $company['comp_name'] !== 'Tata Consultancy Services Ltd.';
});
```

---

#### Step 4: Get Details for Each Similar Stock
**API:** Quote Details API (Loop)

**For each company in $similar_stocks:**

**Endpoint:** `GET https://apihub.5paisa.com/atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/{co_code}/nse?responseType=json`

**Example Request:**
```http
GET /atlas-ba-rt/datamart/Equity/CompanyInfo.svc/getquotedetails/11536/nse?responseType=json HTTP/1.1
```

**Response:**
```json
{
  "response": {
    "data": {
      "getquotedetailslist": {
        "getquotedetails": {
          "symbol": "INFY",
          "complname": "Infosys Limited",
          "hi_52_wk": 1950.00,
          "price": 1485.50,
          "change": -0.5,
          "trd_qty": 5234567
        }
      }
    }
  }
}
```

**Generate HTML for Each Stock:**
```html
<div class="stock-page__similarbox">
  <div class="stock-page__simheader">
    <img class="stock_logo" src="https://images.5paisa.com/MarketIcons/INFY.png" width="55" height="55">
    <div class="stock-page__compname">
      <a href="/stocks/infy-share-price">Infosys Limited</a>
    </div>
  </div>
  <div class="stock-page__simlist">
    <ul>
      <li>52 week high</li>
      <li>1,950.00</li>
    </ul>
    <ul>
      <li>Market price</li>
      <li>1,485.50 <span class="red--color">(-0.50%)</span></li>
    </ul>
    <ul>
      <li>Volume</li>
      <li>5,234,567</li>
    </ul>
  </div>
</div>
```

**Final AJAX Response:**
```json
{
  "data": {
    "similarStocks": "<div class='stock-page__similarwrapper'><div class='mobile-scroll'>...4 similar stock cards...</div></div>"
  }
}
```

**Controller Method:** [`StockPageController.php:1921`](modules/custom/fivepaisa_stock_page/src/Controller/StockPageController.php:1921) - `getSimilarStocks()`

---

### 12. FAQs SECTION - Data Sources

#### Source 1: Dynamic FAQs (API-generated)

**Data Input (from previous APIs):**
```php
$stocksextras = [
  "TCS",                    // [0] Page Title
  "2264",                   // [1] Current Price
  $fundamentalData,         // [2] Array of fundamentals
  "3600",                   // [3] 52 Week High
  "2206"                    // [4] 52 Week Low
];
```

**FAQ Generation:**
```php
$faqse = [
  [
    'q' => 'What is Share Price of TCS?',
    'a' => 'TCS share price is ₹2,264 As on 15 May, 2026 | 16:20'
  ],
  [
    'q' => 'What is the P/E ratio of TCS?',
    'a' => 'The P/E ratio of TCS is 16.6 As on 15 May, 2026 | 16:20'
  ],
  [
    'q' => 'What is the PB ratio of TCS?',
    'a' => 'The PB ratio of TCS is 7 As on 15 May, 2026 | 16:20'
  ],
  [
    'q' => 'What is the Market Cap of TCS?',
    'a' => 'The Market Cap of TCS is ₹819135 Cr As on 15 May, 2026 | 16:20'
  ],
  [
    'q' => 'What is the 52 week High/Low of TCS?',
    'a' => 'The 52 week High & Low of TCS is ₹3600/₹2206 respectively As on 15 May, 2026 | 16:20'
  ]
];
```

**Service Method:** [`StockPageService.php:605`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:605) - `getdynamicFaqs()`

---

#### Source 2: CMS FAQs

**Drupal Query:**
```php
$query = $entityTypeManager->getStorage('node')->getQuery()
  ->condition('status', 1)
  ->condition('type', 'stock_faq')
  ->condition('field_stock_nse_code', 'TCS')
  ->notExists('field_stock_type')
  ->accessCheck(false)
  ->pager(10);

$nids = $query->execute();
```

**Example CMS Data:**
```php
Node ID: 12345
Type: stock_faq
field_stock_nse_code: TCS
field_questions_and_answers: [
  Paragraph 1:
    field_question: "What is the full form of TCS?"
    field_answer: "TCS is an abbreviation for Tata Consultancy Services..."
  
  Paragraph 2:
    field_question: "When did TCS have its IPO?"
    field_answer: "TCS issued its initial public offering (IPO) in July 2004..."
]
```

---

#### Merged FAQs Output

**HTML Generated:**
```html
<div class="paisabase-faqs-accordian stock__holder" id="FAQs">
  <div class="paisabase-faqs-accordian__wrapper">
    <h2 class="base-block__heading">TCS FAQs</h2>
    <div class="faqs-accordian" id="accordionFaqsClosed">
      
      <!-- Dynamic FAQ #1 -->
      <div class="faqs-accordian__content">
        <div class="faqs-accordian__click" id="faqs-heading0">
          <h3 class="accordian-title">
            <a data-toggle="collapse" href="#collapseFaqsClose0">
              What is Share Price of TCS?
            </a>
          </h3>
        </div>
        <div id="collapseFaqsClose0" class="faqs-accordian__open collapse">
          <p>TCS share price is ₹2,264 As on 15 May, 2026 | 16:20</p>
        </div>
      </div>
      
      <!-- Dynamic FAQ #2-5 -->
      <!-- ... -->
      
      <!-- CMS FAQ #1 -->
      <div class="faqs-accordian__content">
        <div class="faqs-accordian__click" id="faqs-heading5">
          <h4 class="accordian-title">
            <a data-toggle="collapse" href="#collapseFaqsClose5">
              What is the full form of TCS?
            </a>
          </h4>
        </div>
        <div id="collapseFaqsClose5" class="faqs-accordian__open collapse">
          <p>TCS is an abbreviation for Tata Consultancy Services...</p>
        </div>
      </div>
      
      <!-- CMS FAQ #2-N -->
      <!-- ... -->
      
    </div>
  </div>
</div>
```

**Service Method:** [`StockPageService.php:641`](modules/custom/fivepaisa_stock_page/src/StockPageService.php:641) - `getFaqs()`

---

## Complete API Dependency Tree

```
Main Stock Page Load (Server-Side)
│
├── 1. Overview API
│   ├── Returns: Stock Name, Price, Change, Fundamentals, Technicals
│   └── Used in: Profile, Fundamentals sections
│
├── 2. Company Profile API
│   ├── Requires: nsecode (TCS)
│   ├── Returns: co_code (18564)
│   └── Used by: Quote Details API
│
├── 3. Quote Details API
│   ├── Requires: co_code (18564) from API #2
│   ├── Returns: Performance metrics, EPS, Sector info
│   └── Used in: Performance, Fundamentals, Sector Link
│
├── 4. Technical Analysis API
│   ├── Requires: nsecode (TCS)
│   ├── Returns: Moving Averages, Pivot Points, Price Analysis
│   └── Used in: Performance (DMA), Technicals, Returns sections
│
├── 5. Tech Trend API
│   ├── Requires: nsecode (TCS)
│   ├── Returns: Bullish/Bearish signals
│   └── Used in: Technicals (SWOT)
│
├── 6. Instrument Search API (MarketSmith)
│   ├── Requires: nsecode (TCS)
│   ├── Returns: instrumentId (54321)
│   └── Used by: Finance Details API, Investment Rating API
│
├── 7. Finance Details API
│   ├── Requires: instrumentId (54321) from API #6
│   ├── Returns: Financial statements, Shareholding pattern
│   └── Used in: Financial tabs, Shareholding section
│
├── 8. Investment Rating API
│   ├── Requires: instrumentId (54321) from API #6
│   ├── Returns: Ratings (Master, EPS, RS, A/D, Group Rank)
│   └── Used in: Technicals (Ratings subsection)
│
├── 9. Events Calendar API
│   ├── Requires: nsecode (TCS)
│   ├── Returns: Board meetings, Dividends, Bonus, Splits
│   └── Used in: Events/Corporate Actions section
│
├── 10. Search Scrip API
│   ├── Requires: nsecode (TCS)
│   ├── Returns: scripCode (11536), exchange
│   ├── Used to generate: Chart iframe URL
│   └── Used in: Chart section
│
└── 11. CMS Queries
    ├── About Company content
    ├── FAQs content
    └── Meta tags (title, description)

Client-Side AJAX Calls (on user interaction)
│
├── 12. Sector & Cap Info API (on page load)
│   ├── Calls APIs: #2, #3
│   └── Returns: Sector link, Cap link
│
├── 13. Financial Data API (on tab click)
│   ├── Uses cached data from API #7
│   └── Returns: HTML table for selected tab
│
├── 14. Shareholding Pattern APIs (on tab click)
│   ├── Uses cached data from API #7
│   ├── Returns: Names HTML, Data HTML
│   └── Three endpoints: names-json, data-json, chart
│
├── 15. F&O Check API (when section becomes visible)
│   ├── Calls: FutOpt Overview API
│   └── Returns: future_exist, option_exist flags
│
├── 16. Similar Stocks API (when section becomes visible)
│   ├── Calls APIs: Quotes Finder → Quote Details → Sector Companies → Quote Details (loop)
│   └── Returns: HTML cards for 4 similar stocks
│
└── 17. Price Range Filter API (from chart iframe message)
    ├── Uses cached data from API #4
    └── Returns: Price analysis for selected range
```

---

## Complete Authentication Matrix

| API Base Domain | Auth Type | Headers/Credentials | Used For |
|-----------------|-----------|---------------------|----------|
| `apihub.5paisa.com/pearl-ca` | Kong Gateway + Legacy | `Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b`<br>`KEY: 5260c06e20fb53c4521b8cf1f2eb0ba616634e44`<br>`UserId: 5PAISAAPI`<br>`Password: 5nadynsiitnienny` | Overview, Technical Analysis, Tech Trend, Events Calendar, Rapid Results |
| `apihub.5paisa.com/atlas-ba-rt`<br>`apihub.5paisa.com/atlas-req1-rt` | Basic Auth | `Authorization: Basic aW5kaWFpbmZvbGluZVxpaWZsd2ViOkdsYXhvQDEyMw==`<br>(indiainfoline\iiflweb:Glaxo@123) | Company Profile, Quote Details, Quotes Finder, Sector Companies, FutOpt Overview |
| `apihub.5paisa.com/marketsmith-ca` | Gateway Token | `gateway-name: fivepaisa-gateway`<br>`x-access-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` | Instrument Search, Finance Details |
| `gateway.5paisa.com/tradeapi` | Kong Gateway | `Ocp-Apim-Subscription-Key: a4af51382266497bb5464d95fbb2017b` | Scoreboard |
| `gateway.5paisa.com/prelogin` | Custom Headers | `UserID: ZyT47UW2g56`<br>`Password: H98qlU4Sn2`<br>`Ocp-Apim-Subscription-Key: ad5445f4018348c38e3b5d6a68a39c81` | Search Scrip |
| `msi-gcloud-prod.appspot.com` | Query Param | `ms-auth=3990+MarketSmithINDUID-Web0000000000+...` | Legacy MarketSmith (Instrument Search, Investment Rating) |

---

## Summary

This document provides complete API call chains with:
- ✅ Step-by-step request/response flows
- ✅ Exact API endpoints with full URLs
- ✅ Complete request headers for each API
- ✅ Sample request bodies where applicable
- ✅ Full response JSON structures
- ✅ Data extraction and processing logic
- ✅ API dependency chains (which API needs data from which)
- ✅ Service method references with line numbers
- ✅ Controller method references with line numbers
- ✅ HTML output examples
- ✅ Complete dependency tree visualization
- ✅ Authentication matrix with decoded credentials

Every API call is documented with its prerequisites, request format, response format, and how the data flows through the system.
