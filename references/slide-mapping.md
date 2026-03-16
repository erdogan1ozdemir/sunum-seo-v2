# Slide Mapping — Per-Slide Data & Output Specification

> This document contains the exact data requirements, table structures, chart specifications, and insight templates for each slide in the SEO Monthly Report.
> Variables: `[brand]`, `[domain]`, `[Ay]`, `[Yıl]`, `[Yıl-1]`, `[Yıl-2]`, `[Ay-1]`, `[month]`

---

## Table of Contents

- [Slide 1: Kapak](#slide-1-kapak)
- [Slide 2: İçindekiler](#slide-2-içindekiler)
- [Slide 3: Section Divider](#slide-3-section-divider-01)
- [Slide 4: Brand Arama Hacmi](#slide-4-brand-arama-hacmi)
- [Slide 5: Brand + Category Hacmi](#slide-5-brand--category-hacmi)
- [Slide 6: Non-Branded Hacim](#slide-6-non-branded-hacim)
- [Slide 7: Segment Kırılım](#slide-7-segment-kırılım)
- [Slide 8: Rekabet](#slide-8-rekabet)
- [Slide 9-12: Section Dividers](#slide-9-12-section-dividers)
- [Slide 10: GA4 Kanal Performansı](#slide-10-ga4-kanal-performansı)
- [Slide 11: GA4 Organik Detay](#slide-11-ga4-organik-detay)
- [Slide 13: GSC Brand & Non-Brand](#slide-13-gsc-brand--non-brand)
- [Slide 14: GSC Trend](#slide-14-gsc-trend)
- [Slide 15: Competitor Visibility](#slide-15-competitor-visibility)
- [Slide 16: Share of Clicks & AI SoV](#slide-16-share-of-clicks--ai-sov)
- [Slide 17: Kategori Visibility](#slide-17-kategori-visibility)
- [Slide 18-19: SEOmonitor Keywords](#slide-18-19-seomonitor-keywords)
- [Slide 20: AI Referral](#slide-20-ai-referral)
- [Slide 21: GSC Keyword Performansı](#slide-21-gsc-keyword-performansı)
- [Slide 22: GSC Sayfa Performansı](#slide-22-gsc-sayfa-performansı)
- [Slide 23: Kapanış](#slide-23-kapanış)

---

## Slide 1: Kapak

**Type:** KAPAK | **Data:** None
**Content:** `[brand] | Aylık SEO Değerlendirme` + `[Ay]  [Yıl]`

---

## Slide 2: İçindekiler

**Type:** İÇİNDEKİLER | **Data:** None
**Content:** 3 bölüm listesi:
```
01  Marka, Kategori Arama Hacimleri & Rekabet
02  [domain] - Trafik, Sıralama, Visibility
03  [domain] - GSC Kelime & Sayfa Performansı
```

---

## Slide 3: Section Divider 01

**Type:** SECTION DIVIDER | **Watermark:** 01
**Title:** Arama Hacmi & Rekabet

---

## Slide 4: Brand Arama Hacmi

**Header:** `Marka Arama Hacimleri | [brand] Marka Hacmi`
**API:** Google Ads API (Keyword Planner) — Keyword Planner search volume
**Query:** `[brand]` keyword, location Turkey, 36 months

### Data Required
- Monthly search volume for brand keyword: 36 months ([Yıl-2] Jan → [Yıl] [Ay])
- YoY % = ([Yıl][Ay] - [Yıl-1][Ay]) / [Yıl-1][Ay] × 100
- MoM % = ([Yıl][Ay] - [Yıl][Ay-1]) / [Yıl][Ay-1] × 100

### Chart Spec
- **Type:** Grouped Bar Chart (2 series side by side)
- **Title text on chart:** `"[brand]" Search Volume`
- **X axis:** Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec
- **Y axis:** Search volume (auto-scaled, gridlines)
- **Series 1:** [Yıl-1] data → Blue `#4472C4`
- **Series 2:** [Yıl] data → Red `#FF0000` (only filled up to [Ay], rest empty)
- **Data labels:** On top of each bar, value in K format (e.g., "33.1K")
- **Legend:** Bottom center: `[Yıl-1]` (blue square) `[Yıl]` (red square)

### Table Spec
- **Columns:** `[brand]` | Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec
- **Rows:** [Yıl-2], [Yıl-1], [Yıl]
- **[Yıl] row:** Only [Ay] column has data, rest are empty cells
- **Header row:** Dark grey `#313131` bg, white text
- **Cell values:** K/M format (33.1K)
- **Conditional coloring:** Each cell in [Yıl-1] row: compare to [Yıl-2] same month → up = light green bg, down = light red bg. Same logic for [Yıl] row vs [Yıl-1].

### Insight Template
```
[brand] only brand arama hacmi [Ay] ayında geçtiğimiz yıla göre {YoY_durum}, MoM olarak kıyaslandığında {MoM_pct} {MoM_durum}.
```
- `{YoY_durum}`: =0 → "sabit kalırken" | >0 → "%X artarken" | <0 → "%X azalırken" → **RED color**
- `{MoM_pct}`: absolute value "**%X.X**" → **BOLD + RED**
- `{MoM_durum}`: "artmıştır" / "azalmıştır" / "sabit kalmıştır"
- **Placement:** Large text above chart (Bricolage ExtraBold ~52pt)

### Footnote
`Source: Keyword Planner` (coral badge, bottom-left)

---

## Slide 5: Brand + Category Hacmi

**Header:** `Marka Arama Hacimleri | [brand] Marka + Category Hacmi`
**API:** Google Ads API (Keyword Planner)
**Query:** All brand+category keywords SUM, 36 months

### Structure
Identical to Slide 4 except:
- Chart title: `"[brand]" + "Category" Branded Search Volume`
- Table first column: empty (no brand name label)
- Values are typically higher (72.8K range instead of 33.1K)

### Insight Template
```
[brand] + keyword arama hacmi YoY kıyasladığımızda {YoY_pct} {YoY_yön}, MoM kıyasladığımızda {MoM_pct} {MoM_yön}.
```

---

## Slide 6: Non-Branded Hacim

**Header:** `[brand] İlgili Arama Hacimleri | Non-Branded Arama Hacmi`
**API:** Google Ads API (Keyword Planner)
**Query:** All non-brand keywords SUM, 36 months

### Structure
Identical to Slide 4 except:
- Values are in M range (1.8M, 2.4M)

### Insight Template
```
[brand] için takip ettiğimiz Non-branded kelimelerin aramalarını incelediğimizde [Ay] ayı arama hacmi geçen yılın {YoY_pct} {YoY_konum}, MoM kıyasladığımızda ise {MoM_durum}.
```
- `{YoY_konum}`: >0 → "üzerinde kalırken" | <0 → "altında kalırken"

---

## Slide 7: Segment Kırılım

**Header:** `Arama Hacimleri | [brand] Marka + Category & Non Brand Hacimler`
**API:** Google Ads API (Keyword Planner)
**Query:** Per-segment keyword groups, [Yıl-1][Ay], [Yıl][Ay], [Yıl][Ay-1] averages

### Layout: 2×2 Table Grid (left 65%) + Insight Panel (right 35%)

### Table 1 — NonBrand YoY (top-left)
| NonBrand Aranma Hacimleri | [Yıl-1][Ay] Avg. | [Yıl][Ay] Avg. | YoY Change |
|---------------------------|-------------------|-----------------|------------|
| [Segment 1]               | 1,018,450         | 838,550         | -18%       |
| [Segment 2]               | ...               | ...             | ...        |
| **Grand Total**            | **5,192,960**     | **3,788,960**   | **-25%**   |

### Table 2 — NonBrand MoM (top-right)
Same structure: [Yıl][Ay-1] Avg | [Yıl][Ay] Avg | MoM Change

### Table 3 — Brand+Cat YoY (bottom-left)
Same structure with brand+category keyword group volumes

### Table 4 — Brand+Cat MoM (bottom-right)
Same structure

### Table Formatting
- **Header:** Dark green `#10332F` bg, white bold text
- **Values:** Comma-separated numbers (1,018,450)
- **Change column:** Conditional color gradient
- **Grand Total row:** Bold

### Insight Panel (right side, 35% width)
Dash-bullet list with bold keywords and colored percentages:

```
- Non-branded kelimelerin aranma hacmine baktığımızda tüm segmentlerde YoY düşüş görülmektedir.

- En yüksek düşüş {worst_segment} kategorisindedir ({worst_yoy}).

- MoM kıyasladığımızda ise daha düşük oranlarda seyretmekle birlikte tüm kategorilerde düşüş gözlenmiştir.

- Brand+category kelimelerin aranma hacmine baktığımızda YoY kıyaslamada {exception_segment} ({exception_pct}) hariç tüm kategorilerde düşüş görülmektedir.

- En yüksek düşüş yaşanan segment {worst_bc_segment} ({worst_bc_pct}) ve {second_worst_bc_segment} ({second_worst_bc_pct}).

- MoM kıyaslamada ise {mom_exception_segments} dışında artış yaşanmıştır. En yüksek düşüş {mom_worst_segments} kategorilerindedir.
```

**Bold:** Segment names, "Non-branded kelimelerin", "Brand+category kelimelerin"
**Color:** Percentages → red if negative, green if positive. Category names mentioned in context of decline → red color.

---

## Slide 8: Rekabet

**Header:** `Rekabet | Arama Hacmi Değişimi`
**Title:** `[brand] & Rakip Arama Hacmi Değişimi`
**API:** Google Ads API (Keyword Planner)
**Query:** Each competitor brand keyword, 13 months

### Table Spec
| Brand | [Yıl-1]Jan | [Yıl-1]Feb | ... | [Yıl-1]Dec | [Yıl][Ay] | YoY | MoM |
|-------|-----------|-----------|-----|-----------|----------|-----|-----|
| trendyol | 11.1M | 9.1M | ... | 11.1M | 11.1M | 0% | 0% |
| [brand] | **33.1K** | ... | ... | **40.5K** | **33.1K** | **0%** | **-18%** |

- **Sort:** By latest month volume, descending
- **Header:** Dark green `#10332F` bg, white text
- **[brand] row:** Bold to stand out
- **YoY/MoM columns:** Conditional color gradient (green/red)
- **Values:** K/M suffix format

### Insight (below table)
```
- Rakip arama hacimleri YoY kıyasladığında {artan_rakipler} artış yakalarken, diğer rakiplerin sabit kaldığı gözlemlenmiştir.
- MoM kıyaslamada ise {istisna} dışında dikey markalarda düşüş görülmektedir.
```
**Bold:** Competitor names. **Color:** Percentages in parens → green/red.

---

## Slide 9-12: Section Dividers

- **Slide 9:** Watermark "02", Title: "[domain] - GA4 Trafik"
- **Slide 12:** Watermark "03", Title: "[domain] - GSC Trafik, Sıralama, Visibility"

---

## Slide 10: GA4 Kanal Performansı

**Header:** `[domain] | Organik Trafik Ölçümlemesi`
**Title:** `Google Analytics [Ay] [Yıl] Verileri`
**API:** GA4 Data API

### Main Table (left 60%)
| # | Session default channel group | Sessions | Sessions (%) | Bounce rate | Avg session duration | Total users | Transactions | Total revenue |
|---|------------------------------|----------|-------------|-------------|---------------------|-------------|--------------|---------------|
| 1. | Organic Search | 168.7K | 56.46% | 35.84% | 04:48 | 74.2K | 67 | ₺565.08K |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |
| | **Grand total** | **298.7K** | **100%** | **36.95%** | **04:31** | **130.9K** | **145** | **₺1.25M** |

- **Header:** Dark green bg, white text
- **Row 1 (Organic):** Green text highlight
- **Revenue:** ₺ prefix with K/M suffix

### Mini Table 1 (right top): Organic Sessions - MoM
| | [Ay-1] [Yıl] | [Ay] [Yıl] | % Change |
|---|---|---|---|
| Sessions | 193.0K | 167.7K | -13% |

### Mini Table 2 (right middle): Organic Sessions - YoY
Same structure with [Yıl-1][Ay] vs [Yıl][Ay]

### Insight (right bottom)
```
[Ay] ayında gelen toplam trafiğin ({total_sessions}) %{organic_pct}'sını ({organic_sessions}) organik kanal oluşturmaktadır.

Organik session'lar MoM kıyaslandığında %{mom_pct}'lük {mom_yön} gösterirken, YoY karşılaştırmada %{yoy_pct} {yoy_durum} kaydetmiştir.
```

---

## Slide 11: GA4 Organik Detay

**Header:** `[domain] | Organik Trafik Ölçümlemesi`
**Title:** `Google Analytics [Ay] [Yıl] Verileri`
**API:** GA4 Data API — 13 months organic data

### Combo Chart (left 60%)
- **Type:** Stacked Bar + Line overlay
- **X axis:** Last 13 months (Jan [Yıl-1] → [Ay] [Yıl])
- **Left Y axis:** Revenue/Sessions volume (bar height)
- **Right Y axis:** Transaction count (line, 0-100 range)
- **Bar Series 1:** Organic Revenue → Grey `#B7B7B7`
- **Bar Series 2:** Organic Sessions → Yellow `#FFD666` (stacked on top of revenue)
- **Line Series:** Organic Transactions → Red `#E74C3C` line with dot markers
- **Data labels:** Session values on bars (65.1K), transaction counts on line points (75)
- **Legend:** Bottom → "Organic Revenue" (grey), "Organic Session" (yellow), "Organic Transaction" (red dot)

### Mini Table 1 (right top): Revenue YoY
| | [Yıl-1][Ay] | [Yıl][Ay] | % Change |
|---|---|---|---|
| Total Revenue | 961.9K | 1.2M | 29.8% |
| Organic Revenue | 575.6K | 565.1K | -1.8% |
| Organic/Total % | 59.8% | 45.3% | -14.6% |

### Mini Table 2 (right middle): Transaction YoY
Same structure: Total Transaction, Organic Transaction, Organic/Total %

### Insight (below chart, left-aligned bullets)
```
- [Ay] ayında YoY kıyaslamada Total Revenue %{total_rev_yoy} {yön} {total_rev_value} seviyesine ulaşmıştır.
- Organik gelir ise %{org_rev_yoy} {yön} göstermiş ve organik kanalın toplam gelir içindeki payı %{org_rev_pct}'e {yön}.
- Toplam işlem sayısı %{total_trx_yoy} {yön} {total_trx_value}'e yükselmiştir.
- Organik işlemler %{org_trx_yoy} {yön}.
- Organik kanalın toplam işlem içindeki payı %{org_trx_pct}'ye {yön}.
```

---

## Slide 13: GSC Brand & Non-Brand

**Header:** `[domain] | Brand & Non-Brand`
**Title:** `Google Search Console Verileri`
**API:** Search Console API

### KPI Card Panel (upper-left, 3 rows × 5 cards)

**Row 1 — Total:** `[Grey label "Total"]`
Each card: Metric name (small grey) / Big value (bold) / MoM change (green↑ or red↓)
Cards: Impressions | Clicks | Average Position | Site CTR | Query Count

**Row 2 — Branded Queries (All Devices):** Same 5 cards
**Row 3 — Non-Branded Queries (All Devices):** Same 5 cards

### YoY Table (bottom)
| | Impressions-YoY | | | Clicks-YoY | | | CTR-YoY | | |
|---|---|---|---|---|---|---|---|---|---|
| | [Yıl-1][Ay] | [Yıl][Ay] | % Change | [Yıl-1][Ay] | [Yıl][Ay] | % Change | [Yıl-1][Ay] | [Yıl][Ay] | % Change |
| Total | 4.2M | 3.9M | -6.1% | 196.6K | 181.3K | -7.8% | 4.7% | 4.6% | -0.1% |
| Branded | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| Non-Branded | ... | ... | ... | ... | ... | ... | ... | ... | ... |

### Insight (right panel)
```
Google Search Console verilerine göre,
- [Ay] ayında [domain] {impressions} gösterim almıştır, geçen aya göre %{imp_mom} azalmıştır.
- CTR geçen aya göre %{ctr_mom}, tıklama ise %{click_mom} azalmıştır.
- Brand Impression'lar geçen aya göre %{brand_imp_mom}, tıklamalar %{brand_click_mom}, CTR ise %{brand_ctr_mom} azalmıştır.
- Non-brand kelimelerde Impression %{nb_imp_mom}, tıklamalar ise %{nb_click_mom} azalmıştır.
```

---

## Slide 14: GSC Trend

**Header:** `[domain] | Organik Trafik Ölçümlemesi`
**Title:** `Google Search Console Verileri`
**API:** Search Console API — 36 months monthly aggregate

### Chart 1 (top half): Impression Trend
- **Type:** Grouped Bar (3 series)
- **Title on chart:** `[domain] Total Impression Trend`
- **Series:** [Yıl-2] → Light grey `#B7B7B7`, [Yıl-1] → Dark `#2D2D2D`, [Yıl] → Orange `#FFD666`
- **Data labels:** M format on bars (4.6M)

### Chart 2 (bottom half): Click Trend
- Same structure, title: `[domain] Total Click Trend`
- **Data labels:** K format (181.3K)

### Insight (right side)
```
- [Ay] ayında gösterimler geçtiğimiz yıla göre %{imp_yoy} {yön} göstermektedir.
- Toplam {clicks} tıklama, {impressions} gösterim gerçekleşmiştir.
- [Ay] ayı tıklamaları geçtiğimiz yıla göre %{click_yoy} {yön} göstermektedir.
```

---

## Slide 15: Competitor Visibility

**Header:** `[domain] | Competitor Visibility Comparison`
**API:** SEOmonitor API

### Table/List
Each row: `[site_logo] [site_name] | [📱 mob_vis%] [mom_badge] | [💻 desk_vis%] [mom_badge]`
Sorted low to high by visibility.
MoM badge: green `+X%` or red `-X%`

### Insight (right panel)
Template covers: tracked keyword count, mobil/desktop trends, market-wide patterns, marketplace positions.

---

## Slide 16: Share of Clicks & AI SoV

**Header:** `[domain] | Total Click Share`
**API:** SEOmonitor API

### 2 Pie Charts side by side
**Pie 1:** Google SoC — competitor click share %
**Pie 2:** AI Search SoV — AI answer mention/link share %
Large slices labeled: `trendyol.com: 30.1%`, `[domain]: 9%`

### Insight (below charts)
```
- SEOmonitor üzerinden takip ettiğimiz {kw_count} civarı kelimede en çok tıklama alan marka %{top1_pct} oranla {top1_name} olmuştur.
- {top2_name} %{top2_pct} oranla 2. marka olmuştur. [domain] %{brand_pct} oranıyla tıklamalarda {rank}. en büyük paya sahip markadır.
- Marketplace'ler AI SOV tarafında en büyük paya sahipken; [domain] ise %{ai_pct} oranıyla en yüksek SoV'ye sahip markalar olarak öne çıkmaktadır.
```

---

## Slide 17: Kategori Visibility

**Header:** `[domain] | Anahtar Kelime Sıralamaları & Visibility`
**API:** SEOmonitor API — keyword group visibility

### List/Card Layout (left 55%)
Each category row:
`[↑↓ arrow] [rank] [icon] [Category Name] [kw_count → volume] [vis% badge] [mom% badge]`

### Insight (right 45%)
Category-level visibility trend analysis, top risers/fallers.

---

## Slide 18-19: SEOmonitor Keywords

**Slide 18:** Top Impact Keywords — screenshot or table from SEOmonitor
**Slide 19:** 2 panel layout: "En Yüksek Arama Hacimli" (left) + "En İyi Artış Yaşayan" (right)

---

## Slide 20: AI Referral

**Header:** `[domain] | GA4 AI Referral Trafik`
**API:** GA4 Data API — sessionSource filter for AI platforms

### Insight
```
YoY kıyaslamada [Ay] ayında AI referral oturumları %{sessions_yoy} artmıştır. Kullanıcılar ise %{users_yoy} oranında artmıştır.
MoM bakıldığında [Ay-1] ayına kıyasla kullanıcılar %{users_mom} {yön} gösterirken; oturumlar %{sessions_mom} {yön}.
```

---

## Slide 21: GSC Keyword Performansı

**Header:** `[domain] | GSC Anahtar Kelime Performansı`
**Title:** `En Çok İyileşen & Kötüleşen Sorgular (YoY)`
**API:** Search Console API — query dimension, YoY comparison

### 2 Tables side by side

**Table 1 (left) — En Çok İyileşen (✅ green header icon):**
| Keyword | Clicks [Yıl-1] | Clicks [Yıl] | Change % | Impressions [Yıl] | CTR | Position |
|---------|----------------|---------------|----------|-------------------|-----|----------|
| 10 rows sorted by click change DESC (positive) |

**Table 2 (right) — En Çok Kötüleşen (❌ red header icon):**
Same columns, 10 rows sorted by click change ASC (negative)

- **Filter:** Non-brand only (exclude brand regex matches)
- **Header:** Dark green bg, white text
- **Change %:** Green (positive table) / Red (negative table)

---

## Slide 22: GSC Sayfa Performansı

**Header:** `[domain] | GSC Sayfa Performansı`
**Title:** `En Çok İyileşen & Kötüleşen Sayfalar (YoY)`
**API:** Search Console API — page dimension, YoY comparison

### Structure
Identical to Slide 22 but with page URLs instead of keywords.
- URL displayed as path only (strip domain)
- e.g., `/banyo-mobilyalari` instead of `https://vitra.com.tr/banyo-mobilyalari`

---

## Slide 23: Kapanış

**Type:** KAPANIŞ | **Data:** None
**Content:** "Teşekkürler" centered, coral background
