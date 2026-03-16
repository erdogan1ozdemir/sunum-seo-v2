# API Reference — Veri Çekme Detayları

> Her slide için hangi API, hangi endpoint, hangi parametreler, response'dan ne parse edilir, nereye yazılır.

---

## Genel Bilgiler

| API | Auth | Base URL / Method |
|-----|------|-------------------|
| Google Ads (Keyword Planner) | OAuth2 + developer token | `googleads.v17.services.KeywordPlanIdeaService.GenerateHistoricalMetrics` |
| GA4 Data API | OAuth2 | `POST https://analyticsdata.googleapis.com/v1beta/properties/{id}:runReport` |
| Search Console API | OAuth2 | `POST https://www.googleapis.com/webmasters/v3/sites/{siteUrl}/searchAnalytics/query` |
| SEOmonitor API | API Key (header) | `https://api.seomonitor.com/v3/` |

---

## 1. Google Ads API — Keyword Planner (Slide 4–8)

### Endpoint
```
KeywordPlanIdeaService.GenerateHistoricalMetrics
```

### Slide 4: Brand Keyword Hacmi

**Request:**
```json
{
  "keyword_plan_network": "GOOGLE_SEARCH",
  "keywords": ["vitra"],
  "historical_metrics_options": {
    "year_month_range": {
      "start": {"year": 2024, "month": 1},
      "end": {"year": 2026, "month": "[current_month_num]"}
    }
  },
  "geo_target_constants": ["geoTargetConstants/2792"],
  "language": "languageConstants/1017"
}
```

**Response parse:**
```python
for result in response.results:
    keyword = result.text
    for monthly in result.keyword_metrics.monthly_search_volumes:
        year = monthly.year
        month = monthly.month  # JANUARY=1, FEBRUARY=2...
        volume = monthly.monthly_searches
        # → matrix[year][month] = volume
```

**Output yapısı (slide_data.json):**
```json
{
  "slide_4": {
    "keyword": "vitra",
    "matrix": {
      "2024": {"Jan": 33100, "Feb": 33100, ...},
      "2025": {"Jan": 33100, "Feb": 27100, ...},
      "2026": {"Jan": 33100}
    },
    "yoy_pct": 0.0,
    "mom_pct": -18.2,
    "chart_data": {
      "months": ["Jan","Feb",...,"Dec"],
      "series_prev": [33100, 27100, 33100, ...],
      "series_curr": [33100, null, null, ...]
    }
  }
}
```

### Slide 5: Brand + Category Toplam Hacmi

**Request:** Aynı endpoint, keyword listesi brand+category:
```json
{
  "keywords": ["vitra banyo", "vitra klozet", "vitra lavabo", "vitra armatür", ...]
}
```

**Post-processing:**
```python
# Her keyword'ün aylık hacmini al, aylar bazında topla
monthly_totals = {}
for kw_result in response.results:
    for monthly in kw_result.keyword_metrics.monthly_search_volumes:
        key = f"{monthly.year}_{monthly.month}"
        monthly_totals[key] = monthly_totals.get(key, 0) + monthly.monthly_searches
```

### Slide 6: Non-Brand Toplam Hacmi

Slide 5 ile aynı yapı, keyword listesi non-brand:
```json
{
  "keywords": ["klozet", "lavabo", "banyo dolabı", "duşakabin", ...]
}
```

### Slide 7: Segment Kırılım

Her segment için ayrı çağrı veya tek çağrıda gruplu parse:
```python
segments = brand_profile["segments"]  # {"Armatürler": {"nonbrand": [...], "brand_cat": [...]}, ...}

for segment_name, kw_groups in segments.items():
    # NonBrand grubu
    nb_response = keyword_planner.generate_historical_metrics(keywords=kw_groups["nonbrand"], ...)
    nb_totals = sum_monthly(nb_response)
    
    # Brand+Cat grubu  
    bc_response = keyword_planner.generate_historical_metrics(keywords=kw_groups["brand_cat"], ...)
    bc_totals = sum_monthly(bc_response)
    
    # Her segment için: prev_year_month_avg, curr_year_month_avg, prev_month_avg
    # YoY ve MoM hesapla
```

**Output yapısı:**
```json
{
  "slide_7": {
    "nonbrand_yoy": [
      {"segment": "Armatürler", "prev_avg": 1018450, "curr_avg": 838550, "yoy_change": -17.7},
      ...
      {"segment": "Grand Total", "prev_avg": 5192960, "curr_avg": 3788960, "yoy_change": -27.0}
    ],
    "nonbrand_mom": [...],
    "brand_cat_yoy": [...],
    "brand_cat_mom": [...]
  }
}
```

### Slide 8: Rakip Brand Hacimleri

```python
competitors = brand_profile["competitors"]  # [{"name": "trendyol", "keyword": "trendyol"}, ...]
all_keywords = [c["keyword"] for c in competitors]

response = keyword_planner.generate_historical_metrics(
    keywords=all_keywords,
    year_month_range={"start": {"year": prev_year, "month": report_month_num}, 
                      "end": {"year": curr_year, "month": report_month_num}},
    geo_target_constants=["geoTargetConstants/2792"]
)

# Her rakip × 13 ay matris + YoY/MoM hesapla
```

---

## 2. GA4 Data API (Slide 10, 11, 20)

### Endpoint
```
POST https://analyticsdata.googleapis.com/v1beta/properties/{property_id}:runReport
```

### Slide 10: Kanal Performansı

**Request:**
```json
{
  "dateRanges": [{"startDate": "2026-01-01", "endDate": "2026-01-31"}],
  "dimensions": [{"name": "sessionDefaultChannelGroup"}],
  "metrics": [
    {"name": "sessions"},
    {"name": "bounceRate"},
    {"name": "averageSessionDuration"},
    {"name": "totalUsers"},
    {"name": "transactions"},
    {"name": "totalRevenue"}
  ],
  "orderBys": [{"metric": {"metricName": "sessions"}, "desc": true}]
}
```

**Response parse:**
```python
channels = []
for row in response.rows:
    channels.append({
        "channel": row.dimension_values[0].value,
        "sessions": int(row.metric_values[0].value),
        "bounce_rate": float(row.metric_values[1].value),
        "avg_duration": float(row.metric_values[2].value),  # saniye → mm:ss
        "users": int(row.metric_values[3].value),
        "transactions": int(row.metric_values[4].value),
        "revenue": float(row.metric_values[5].value)
    })
# Grand total hesapla
# Organic Search satırını ayır → MoM/YoY için ek çağrılar
```

**Organic MoM için ek çağrı** (aynı endpoint, önceki ay date range):
```json
{
  "dateRanges": [{"startDate": "2025-12-01", "endDate": "2025-12-31"}],
  "dimensionFilter": {
    "filter": {
      "fieldName": "sessionDefaultChannelGroup",
      "stringFilter": {"value": "Organic Search", "matchType": "EXACT"}
    }
  },
  "metrics": [{"name": "sessions"}]
}
```

**Organic YoY için ek çağrı:** Aynı yapı, `startDate/endDate` geçen yıl aynı ay.

### Slide 11: Organik Detay (13 aylık)

**Request:** 13 ayrı aylık çağrı veya tek çağrıda date dimension:
```json
{
  "dateRanges": [{"startDate": "2025-01-01", "endDate": "2026-01-31"}],
  "dimensions": [
    {"name": "sessionDefaultChannelGroup"},
    {"name": "yearMonth"}
  ],
  "dimensionFilter": {
    "filter": {
      "fieldName": "sessionDefaultChannelGroup",
      "stringFilter": {"value": "Organic Search", "matchType": "EXACT"}
    }
  },
  "metrics": [
    {"name": "sessions"},
    {"name": "transactions"},
    {"name": "totalRevenue"}
  ]
}
```

**Ayrıca total (filtresiz) çağrı:** Organic/Total oranı hesaplamak için aynı çağrı dimensionFilter olmadan.

### Slide 20: AI Referral

**Request:**
```json
{
  "dateRanges": [{"startDate": "2026-01-01", "endDate": "2026-01-31"}],
  "dimensions": [{"name": "sessionSource"}],
  "dimensionFilter": {
    "filter": {
      "fieldName": "sessionSource",
      "inListFilter": {
        "values": ["chatgpt.com", "gemini.google.com", "perplexity.ai", "copilot.microsoft.com", "claude.ai"]
      }
    }
  },
  "metrics": [
    {"name": "sessions"},
    {"name": "totalUsers"}
  ]
}
```

**MoM ve YoY:** Aynı çağrıyı [Ay-1] ve [Yıl-1][Ay] date range'leriyle tekrarla.

---

## 3. Search Console API (Slide 13, 14, 21, 22)

### Endpoint
```
POST https://www.googleapis.com/webmasters/v3/sites/{siteUrl}/searchAnalytics/query
```
`siteUrl`: URL-encoded, ör: `sc-domain%3Avitra.com.tr`

### Slide 13: Brand & Non-Brand

**Request (Total — current month):**
```json
{
  "startDate": "2026-01-01",
  "endDate": "2026-01-31",
  "dimensions": ["query"],
  "rowLimit": 25000,
  "aggregationType": "auto"
}
```

**Post-processing:**
```python
brand_regex = re.compile(brand_profile["brand_regex"], re.IGNORECASE)

total = {"impressions": 0, "clicks": 0, "ctr_sum": 0, "position_sum": 0, "count": 0}
branded = {"impressions": 0, "clicks": 0, ...}
nonbranded = {"impressions": 0, "clicks": 0, ...}

for row in response["rows"]:
    query = row["keys"][0]
    imp = row["impressions"]
    clk = row["clicks"]
    pos = row["position"]
    
    total["impressions"] += imp
    total["clicks"] += clk
    total["count"] += 1
    
    if brand_regex.search(query):
        branded["impressions"] += imp
        branded["clicks"] += clk
        branded["count"] += 1
    else:
        nonbranded["impressions"] += imp
        nonbranded["clicks"] += clk
        nonbranded["count"] += 1

# CTR = clicks / impressions
# Avg Position = weighted average (position × impressions) / total impressions
# Query Count = unique query count
```

**MoM:** Aynı çağrıyı [Ay-1] date range ile tekrarla.
**YoY:** Aynı çağrıyı [Yıl-1][Ay] date range ile tekrarla.

### Slide 14: Impression/Click Trend (36 aylık)

**Request:**
```json
{
  "startDate": "2024-01-01",
  "endDate": "2026-01-31",
  "dimensions": ["date"],
  "rowLimit": 25000,
  "aggregationType": "auto"
}
```

**Post-processing:**
```python
# Günlük veriyi aylık aggregate'e dönüştür
monthly = {}
for row in response["rows"]:
    date = row["keys"][0]  # "2024-01-15"
    year_month = date[:7]  # "2024-01"
    if year_month not in monthly:
        monthly[year_month] = {"impressions": 0, "clicks": 0}
    monthly[year_month]["impressions"] += row["impressions"]
    monthly[year_month]["clicks"] += row["clicks"]

# 3 yıllık matris oluştur: year → [Jan, Feb, ..., Dec]
```

### Slide 21: Keyword Gainers/Losers

**Request (current period):**
```json
{
  "startDate": "2026-01-01",
  "endDate": "2026-01-31",
  "dimensions": ["query"],
  "rowLimit": 5000,
  "aggregationType": "auto"
}
```

**Request (previous year same month):**
```json
{
  "startDate": "2025-01-01",
  "endDate": "2025-01-31",
  "dimensions": ["query"],
  "rowLimit": 5000
}
```

**Post-processing:**
```python
# İki dönemi keyword bazında join et
curr_dict = {row["keys"][0]: row for row in curr_response["rows"]}
prev_dict = {row["keys"][0]: row for row in prev_response["rows"]}

all_keywords = set(curr_dict.keys()) | set(prev_dict.keys())
changes = []
for kw in all_keywords:
    if brand_regex.search(kw):
        continue  # Brand keyword'leri hariç tut
    curr_clicks = curr_dict.get(kw, {}).get("clicks", 0)
    prev_clicks = prev_dict.get(kw, {}).get("clicks", 0)
    click_change = curr_clicks - prev_clicks
    click_change_pct = (click_change / prev_clicks * 100) if prev_clicks > 0 else None
    changes.append({
        "keyword": kw,
        "clicks_prev": prev_clicks,
        "clicks_curr": curr_clicks,
        "change": click_change,
        "change_pct": click_change_pct,
        "impressions_curr": curr_dict.get(kw, {}).get("impressions", 0),
        "ctr_curr": curr_dict.get(kw, {}).get("ctr", 0),
        "position_curr": curr_dict.get(kw, {}).get("position", 0)
    })

# Top 10 artış (change desc) + Top 10 düşüş (change asc)
gainers = sorted([c for c in changes if c["change"] > 0], key=lambda x: -x["change"])[:10]
losers = sorted([c for c in changes if c["change"] < 0], key=lambda x: x["change"])[:10]
```

### Slide 22: Page Gainers/Losers

Slide 21 ile aynı yapı, `dimensions: ["page"]` olarak değişir.
URL kısaltma: `url.replace(f"https://{domain}", "").replace(f"http://{domain}", "")`

---

## 4. SEOmonitor API (Slide 15–19)

### Auth
```
Header: Authorization: Bearer {api_key}
```

### Slide 15: Competitor Visibility

**Endpoint:** `GET /v3/rank-tracker/visibility`
```
?campaign_id={campaign_id}&start_date=2026-01-01&end_date=2026-01-31
```

**Response parse:**
```python
for competitor in response["data"]:
    site = competitor["domain"]
    mobile_vis = competitor["visibility"]["mobile"]
    desktop_vis = competitor["visibility"]["desktop"]
    mobile_change = competitor["visibility_change"]["mobile"]  # MoM
    desktop_change = competitor["visibility_change"]["desktop"]
```

### Slide 16: Share of Clicks & AI SoV

**Endpoint:** `GET /v3/rank-tracker/share-of-clicks`
```
?campaign_id={campaign_id}&start_date=2026-01-01&end_date=2026-01-31
```

**Response parse:**
```python
soc_data = {}
sov_data = {}
for item in response["data"]["share_of_clicks"]:
    soc_data[item["domain"]] = item["percentage"]
for item in response["data"]["ai_share_of_voice"]:
    sov_data[item["domain"]] = item["percentage"]
```

### Slide 17: Kategori Visibility

**Endpoint:** `GET /v3/rank-tracker/groups`
```
?campaign_id={campaign_id}&start_date=2026-01-01&end_date=2026-01-31
```

**Response parse:**
```python
for group in response["data"]:
    category = group["name"]
    keyword_count = group["keyword_count"]
    total_volume = group["total_search_volume"]
    visibility = group["visibility"]["mobile"]
    visibility_change = group["visibility_change"]["mobile"]  # MoM
```

### Slide 18-19: Top Keywords

**Endpoint:** `GET /v3/rank-tracker/keywords`
```
?campaign_id={campaign_id}&order_by=visibility_impact&order=desc&limit=20
```

**Rising keywords:**
```
?campaign_id={campaign_id}&order_by=position_change&order=desc&limit=15
```

---

## Veri Akış Şeması

```
[User Input: brand + month/year]
         │
         ▼
[Step 0: Config Load]  ──→  brand-profiles/{brand}.json
         │
         ▼
[Step 1: Data Collection]
    ├── Google Ads API ──→ raw/kp_brand.json
    │                  ──→ raw/kp_brand_cat.json
    │                  ──→ raw/kp_nonbrand.json
    │                  ──→ raw/kp_segment.json
    │                  ──→ raw/kp_competitors.json
    ├── GA4 Data API   ──→ raw/ga4_channel.json
    │                  ──→ raw/ga4_organic_13m.json
    │                  ──→ raw/ga4_ai_referral.json
    ├── GSC API        ──→ raw/gsc_brand_nonbrand.json (×3 periods)
    │                  ──→ raw/gsc_trend_36m.json
    │                  ──→ raw/gsc_keywords_curr.json + prev.json
    │                  ──→ raw/gsc_pages_curr.json + prev.json
    └── SEOmonitor API ──→ raw/seom_visibility.json
                       ──→ raw/seom_soc.json
                       ──→ raw/seom_groups.json
                       ──→ raw/seom_keywords.json
         │
         ▼
[Step 2: Processing]   ──→ processed/slide_data.json (tüm slide verileri)
         │
         ▼
[Step 3: Charts]       ──→ charts/chart_slide04_brand_volume.png
                       ──→ charts/chart_slide05_brand_cat_volume.png
                       ──→ charts/chart_slide06_nonbrand_volume.png
                       ──→ charts/chart_slide11_organic_combo.png
                       ──→ charts/chart_slide14_impression_trend.png
                       ──→ charts/chart_slide14_click_trend.png
                       ──→ charts/chart_slide16_soc_sov_pie.png
         │
         ▼
[Step 4: Insights]     ──→ processed/slide_insights.json
         │
         ▼
[Step 5: PPTX Build]   ──→ {brand}_SEO_Degerlendirme_{Ay}_{Yil}.pptx
         │
         ▼
[Step 6: QA]           ──→ Final PPTX
```
