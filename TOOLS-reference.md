# TOOLS.md — Investment Brain 工具清單

## ⚠️ 報告產出規範（強制）
**在產出任何分析報告之前，必須先讀取報告指南：**
```
read REPORT-GUIDELINE.md
```
指南路徑：`/Users/peterting/.openclaw/workspace-inv-brain/REPORT-GUIDELINE.md`
- 報告必須嚴格遵循 8 大章節結構
- **所有 MOPS Scraper 資料（22 項）必須全部讀取分析**
- 每個數據點必須附上 📎 來源引用
- 不同分析面向（基本面/技術面/籌碼面/消息面）必須使用對應工具

---

## 已安裝的 Skills（可直接使用）

### 1. Stock Data Skill 📈
即時報價與技術分析（台股 + 美股）

```bash
# 即時報價
skills/stock-data/scripts/quote.sh 2330      # 台積電
skills/stock-data/scripts/quote.sh AAPL      # Apple
skills/stock-data/scripts/quote.sh --json NVDA  # JSON 輸出

# 技術分析
skills/stock-data/scripts/technical.sh AAPL ma 5,20,60    # 移動平均線
skills/stock-data/scripts/technical.sh AAPL rsi 14         # RSI
skills/stock-data/scripts/technical.sh AAPL macd 12,26,9   # MACD

# 綜合分析（含買賣訊號）
skills/stock-data/scripts/analysis.sh 2330

# 批量查詢
skills/stock-data/scripts/portfolio.sh 2330,AAPL,NVDA,TSLA
```

**資料來源：**
- 台股: TWSE Open API（即時、免費、官方）
- 美股: Yahoo Finance（延遲 15 分、免費）

---

### 2. MOPS Scraper — 公開資訊觀測站資料抓取工具

自動抓取上市櫃公司 22 項完整公開資料（財報、營收、股東結構等）。

```bash
# 基本用法（抓取 + 自動產出 crawl-manifest.json）
cd /Users/peterting/.openclaw/workspace/tools/mops-scraper
python3 -m mops_scraper --stock 2382 -v

# 多股票
python3 -m mops_scraper --stock 2382,2330,2454 -v

# 指定年份範圍（預設 2 年）
python3 -m mops_scraper --stock 2382 --years 3 -v

# ⭐ 解析季報 JSON → 乾淨的財務指標（不重新爬取）
python3 -m mops_scraper --stock 2330 --parse -v

# 指定特定類別
python3 -m mops_scraper --stock 2330 --category quarterly-reports,monthly-revenue -v
```

**抓取的 22 項資料：**

| 類別 | 資料項 | 存放路徑 |
|------|--------|---------|
| 基本資料 | 精華版3.0 | `basic-info/company_snapshot.json` |
| 基本資料 | 公司基本資料 | `basic-info/company_profile.json` |
| 基本資料 | 被投資控股公司 | `basic-info/invested_companies.json` |
| 電子書 | 財務報告書 | `financial-reports/*.pdf` |
| 電子書 | 財務預測書 | `financial-forecasts/*.pdf` |
| 電子書 | 公開說明書 | `prospectus/*.pdf` |
| 電子書 | 年報及股東會 | `annual-reports/*.pdf` |
| 電子書 | 自結財務資訊 | `self-compiled-financials/*.json` |
| 電子書 | 關係企業三書表 | `related-company-reports/*.json` |
| 股東 | 董監大股東持股/質押/轉讓 | `shareholders/directors_holdings.json` |
| 股東 | 買回存託憑證 | `shareholders/tdr_buyback.json` |
| 股東 | 股權分散表 | `shareholders/share_distribution.json` |
| 股東 | 僑外及大陸投資人持股 | `shareholders/foreign_holdings.json` |
| 變更 | 歷年變更登記 | `registration-changes/registration_changes.json` |
| 有價證券 | 國內海外轉換情形 | `securities/convertible_securities.json` |
| 有價證券 | 國內有價證券 | `securities/domestic_securities.json` |
| 有價證券 | 海外有價證券 | `securities/overseas_securities.json` |
| 有價證券 | 員工認股權 | `securities/employee_stock_options.json` |
| 有價證券 | 限制員工權利新股 | `securities/restricted_stock.json` |
| 有價證券 | 外國發行人股票/TDR/債券 | `securities/foreign_issuer_info.json` |
| 財務 | 月營收 | `monthly-revenue/revenue_{from}_{to}.json` |
| 財務 | 季報（BS+IS+CF）| `quarterly-reports/{year}Q{quarter}_季報.json` |
| 重訊 | 重大訊息 | `major-news/major_news.json` |

**資料存放路徑：**
```
memory/company-data/{stock_code}_{company_name}/
├── crawl-manifest.json    ← ⭐ 爬取清單（每項資料的狀態）
├── metadata.json          ← 抓取紀錄
├── annual-reports/        ← 年報 PDFs
├── basic-info/            ← 公司基本資料
├── financial-reports/     ← 財務報告書 PDFs
├── monthly-revenue/       ← 月營收
├── quarterly-reports/     ← 季報原始 JSON + 解析後指標
│   ├── 2025Q3_季報.json   ← MOPS 原始（multi-index，難讀）
│   └── 2025Q3_parsed.json ← ⭐ 解析後（乾淨 JSON，直接用）
├── shareholders/          ← 股東資料
├── securities/            ← 有價證券
└── major-news/            ← 重大訊息
```

### ⭐ crawl-manifest.json（V2 新增 — 必讀）
每次爬取後自動產出，記錄每項資料的狀態：
- `success`：成功抓到資料
- `failed`：爬蟲出錯（HTTP error、反爬蟲等）
- `not_applicable`：該公司確實沒有這筆資料

**分析前必須先讀 crawl-manifest.json**，確認哪些資料可用、哪些缺失。

### ⭐ 季報 Parsed JSON（V2 新增 — 基本面分析核心）
`--parse` 產出的 `*_parsed.json` 包含乾淨的財務指標：

```json
{
  "income_statement": {
    "revenue": 3809054272,     // 營業收入（千元）
    "gross_profit": 2281293979,
    "operating_profit": 1936091677,
    "net_income": 1715396780,
    "eps": 66.26,
    "gross_margin_pct": 59.89, // 毛利率 %
    "operating_margin_pct": 50.83,
    "net_margin_pct": 45.03
  },
  "balance_sheet": {
    "total_assets": 7933023878,
    "current_ratio_pct": 261.8,
    "quick_ratio_pct": 242.04,
    "inventory": 288109485,
    "cash": 2767856402
  },
  "cash_flow": {
    "operating": 2274975625,
    "capex": -1272410529,
    "free_cash_flow": 1002565096
  },
  "ratios": {
    "roe_pct": 35.89,
    "roa_pct": 21.63,
    "debt_ratio_pct": 31.16,
    "inventory_turnover": 21.21,
    "earnings_quality": 1.33  // 營業現金流/稅後淨利
  }
}
```

**分析時如何使用：**
1. **基本面（⭐優先用 parsed JSON）**：讀取 `quarterly-reports/*_parsed.json`，直接取得毛利率、ROE、FCF 等
2. **營收追蹤**：讀取 `monthly-revenue/` 比較 MoM/YoY 成長
3. **公司概況**：讀取 `basic-info/company_snapshot.json`
4. **籌碼面**：讀取 `shareholders/share_distribution.json`（股權分散）+ `directors_holdings.json`（董監持股）
5. **消息面**：讀取 `major-news/major_news.json`
6. **深度研究**：讀取年報 PDF 做產業和營運分析
7. **資料品質確認**：先讀 `crawl-manifest.json`，確認資料完整度

---

### 3. Polymarket — 預測市場數據 🔮

查詢 Polymarket 預測市場的賠率、趨勢和價格動量。

```bash
# 熱門市場
python3 skills/polymarketodds/scripts/polymarket.py trending

# 搜尋特定主題
python3 skills/polymarketodds/scripts/polymarket.py search "taiwan"
python3 skills/polymarketodds/scripts/polymarket.py search "fed rate"

# 最大漲跌幅
python3 skills/polymarketodds/scripts/polymarket.py movers

# 即將結算的市場
python3 skills/polymarketodds/scripts/polymarket.py calendar
```

**投資應用：** 地緣政治風險評估、聯準會政策預期、重大事件機率。

---

### 4. Technical Analyst Skill 📊
圖表技術分析（需提供 K 線圖截圖）

用法：提供週線圖 → 辨識趨勢、支撐/壓力、型態，給出情境分析和機率評估。
適用於：台股、美股、加密貨幣、外匯。

---

## 可用的 Web 工具（無需安裝）

### 5. web_search + web_fetch
直接用搜尋引擎 + 網頁抓取做即時研究。

**常用資料源：**
```
# 台股
https://openapi.twse.com.tw/                    # 證交所 Open API（Swagger）
https://mops.twse.com.tw/                        # 公開資訊觀測站
https://www.tdcc.com.tw/portal/zh/smWeb/qryStock # 集保戶股權分散表
https://smart.tdcc.com.tw/opendata/getOD.ashx?id=1-5  # TDCC 開放資料 API

# 美股
https://finance.yahoo.com/quote/{SYMBOL}         # Yahoo Finance
https://finviz.com/quote.ashx?t={SYMBOL}         # Finviz（基本面+技術面快覽）

# 總經
https://fred.stlouisfed.org/                     # FRED（GDP/CPI/利率/就業）
https://tradingeconomics.com/                     # 各國總經指標

# 市場情緒
https://feargreedmeter.com/                      # CNN Fear & Greed Index
https://alternative.me/crypto/fear-and-greed-index/  # Crypto Fear & Greed

# 計算工具
https://kirilloid.ru/travian/                     # （Peter 的其他用途）
```

---

## 可安裝的 Python 套件（目前環境已有 pandas, numpy, requests）

以下套件可視需要用 `pip3 install` 安裝：

### 6. yfinance — Yahoo Finance Python API
```bash
pip3 install yfinance
```
```python
import yfinance as yf

# 美股完整資料
ticker = yf.Ticker("AAPL")
ticker.info          # 基本面（P/E, P/B, EPS, 市值...）
ticker.financials    # 年度損益表
ticker.balance_sheet # 資產負債表
ticker.cashflow      # 現金流量表
ticker.history(period="1y")  # 歷史股價

# 台股也支援（代號加 .TW）
tw = yf.Ticker("2330.TW")
tw.info['trailingPE']  # 本益比
```
**優勢：** 美股最完整的免費基本面數據，含分析師評級、ESG 評分。
**限制：** 非官方 API，可能不穩定；台股資料較少。

### 7. FinMind — 台股專用 50+ 資料集
```bash
pip3 install finmind
```
```python
import requests
url = "https://api.finmindtrade.com/api/v4/data"
params = {
    "dataset": "TaiwanStockPrice",
    "data_id": "2330",
    "start_date": "2025-01-01",
}
data = requests.get(url, params=params).json()
```
**可用資料集（50+）：**
- 技術面：日K、即時報價、PER/PBR、每5秒委託成交、加權指數、當沖標的
- 基本面：綜合損益表、現金流量表、資產負債表、股利政策、除權息、月營收
- 籌碼面：外資持股、股權分散表、融資融券、三大法人買賣、借券明細
- 其他：信用額度餘額、分點資料、八大行庫買賣

**優勢：** 台股最完整的結構化免費 API。
**注意：** 免費版有 API call 限制，重度使用需註冊 token。

### 8. twstock — 台股輕量級工具
```bash
pip3 install twstock
```
```python
from twstock import Stock, BestFourPoint

stock = Stock('2330')
stock.price  # 收盤價序列
stock.date   # 日期序列

bfp = BestFourPoint(stock)
bfp.best_four_point_to_buy()   # 四大買點判斷
bfp.best_four_point_to_sell()  # 四大賣點判斷
bfp.best_four_point()          # 綜合判斷
```
**優勢：** 純 Python、輕量、內建四大買賣點判斷。
**限制：** 資料僅來自 TWSE，無基本面資料。

### 9. fredapi — 美國聯準會經濟數據
```bash
pip3 install fredapi
```
```python
from fredapi import Fred
fred = Fred(api_key='YOUR_KEY')  # 免費申請: https://fred.stlouisfed.org/docs/api/api_key.html

# 常用指標
fed_rate = fred.get_series('FEDFUNDS')      # 聯邦基金利率
cpi = fred.get_series('CPIAUCSL')           # CPI
gdp = fred.get_series('GDP')                # GDP
unemployment = fred.get_series('UNRATE')    # 失業率
yield_10y = fred.get_series('DGS10')        # 10年期公債殖利率
yield_2y = fred.get_series('DGS2')          # 2年期公債殖利率
# 殖利率曲線倒掛 = DGS2 > DGS10 → 衰退訊號
```
**優勢：** 美國總經數據金標準，800,000+ 時間序列。
**需要：** 免費 API Key（申請即得）。

### 10. Alpha Vantage — 全球股票/外匯/加密貨幣
```bash
pip3 install alpha_vantage
```
**提供：** 即時/歷史報價、50+ 技術指標、基本面、外匯、加密貨幣、新聞情緒。
**免費限制：** 25 requests/day（付費版無限制）。
**申請 Key：** https://www.alphavantage.co/support/#api-key

---

## 分析工作流建議

### 台股個股深度分析
```
1. MOPS Scraper → 抓取財報、營收、股東資料
2. Stock Data Skill → 即時報價 + 技術指標
3. FinMind API → 法人買賣超、融資融券、分點資料
4. TDCC 開放資料 → 集保戶股權分散表變化
5. web_search → 產業新聞、法說會重點
```

### 美股個股深度分析
```
1. yfinance → 基本面（財報、估值、分析師評級）
2. Stock Data Skill → 技術分析（MA/RSI/MACD）
3. Finviz (web_fetch) → 快速總覽 + Sector 比較
4. FRED → 相關總經指標
5. Polymarket → 相關事件風險評估
```

### 總經環境掃描
```
1. FRED → 利率、CPI、GDP、就業
2. Fear & Greed Index → 市場情緒
3. Polymarket → 政策/地緣政治預期
4. web_search → 央行會議紀要、最新經濟數據
```

---

## 注意事項
- TWSE 有 rate limit，大量抓取時有被暫時封鎖的風險（scraper 有 retry）
- Yahoo Finance 非官方 API，偶爾會改格式導致 yfinance 壞掉
- 免費 API 都有呼叫次數限制，密集分析時注意間隔
- MOPS 的 `metadata.json` 記錄了成功/失敗項目，可用來判斷資料完整度


### 11. QA Auditor (防偷懶稽核腳本)
在產出 HTML 報告後，發布前必須撰寫並執行 `qa_auditor.py`，透過 Regex 掃描 `<h2>1.` 到 `<h2>9.` 是否完整存在，以及免責聲明中的 EPS 數字是否與本文表格對齊。如果報錯 (AssertionError)，絕對不可發布。

### 12. Factbook Generator (數據強制鎖定工具)
**指令：** `python3 tools/cfa-auditor/generate_raw_factbook.py <stock_code>`
**說明：** 在開始寫任何報告之前，**必須**第一步執行此腳本。它會強制從 `yfinance` 與 `FinMind` 抓取最原始的股價、EPS、融資餘額、三大法人買賣超，並鎖定寫入一個唯讀的 `memory/company-data/<stock>_factbook.json` 中。
**限制：** 報告中的任何數據都必須與 Factbook 的數字一字不差。這是一道物理防線，徹底斬斷大腦「憑空捏造數字」的可能。


## 🌟 CFA V8.0 官方指定查證資料庫 (Zero-Trust Data Sources)
在撰寫報告或交叉驗證數據時，**必須優先使用 `web_search` 針對以下高可信度網域進行定向搜尋 (例如 `site:winvest.tw 1514 EPS`)**：

### 1. 台股精確 EPS 與股數查證 (取代 yfinance TTM 誤差)
- **Win 投資 (winvest.tw)**：用於查證最精確的「單季 EPS」、「累計 EPS」與「全年 EPS」，避免 TTM 誤差。
- **玩股網 (wantgoo.com) / 鉅亨網 (cnyes.com)**：用於即時技術面交叉驗證與財報數據雙重確認。
- **Yahoo 股市 (tw.stock.yahoo.com)**：用於查證精確的「流通股數 (Shares Outstanding)」。

### 2. 絕對估值與歷史 PE Band (WACC / DCF / PE 區間)
- **GuruFocus (gurufocus.com)**：查詢長線「歷史 PE 區間 (PE Band)」與中位數，以及美股/ADR 的 DCF 基準。
- **AlphaSpread (alphaspread.com)**：查詢精確的 WACC (加權平均資本成本) 估算區間與 Base Case DCF 模型。
- **Damodaran (pages.stern.nyu.edu/~adamodar)**：查詢最新的台灣國家風險溢酬 (Country Risk Premium) 與 ERP。
- **Trading Economics (tradingeconomics.com)**：查詢台灣 10 年期公債殖利率 (無風險利率 Rf)。

### 3. 法說會與前瞻指引 (CapEx / 毛利率預測)
- **公司官方 IR (如 investor.tsmc.com)**：直接搜尋官方 Earnings Release 與逐字稿 (Transcript)。
- **散戶嘴鼓 (poorstock.com)**：查詢台股中小型公司的法說會重點摘要。
- **TrendForce / Futurum Group**：查詢資本支出 (CapEx) 預測與產業鏈動態（如 AI 伺服器、半導體產能）。

### 4. 操作指令規範
當需要抓取最新共識時，嚴禁自行捏造。應使用：
`web_search(query="<股票代號> EPS 2026 共識 site:yahoo.com OR site:cnyes.com")`
來取得真實的券商預估中位數。
