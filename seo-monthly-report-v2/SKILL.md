---
name: seo-monthly-report
description: >
  Generate a complete monthly SEO performance presentation (PPTX) for any brand in the Inbound portfolio.
  Use this skill whenever the user wants to create, generate, or prepare a monthly SEO report, SEO
  presentation, aylık SEO sunumu, SEO değerlendirme sunumu, or any variant of "prepare the monthly
  SEO deck for [brand]". Also trigger when the user says things like "[brand] için [ay] sunumu hazırla",
  "SEO raporu oluştur", "aylık sunum", "monthly SEO deck", or references preparing slide decks with
  SEO metrics (search volume, GSC, GA4, visibility, rankings). This skill handles the full pipeline:
  data collection via Google APIs (GA4 Data API, Search Console API, Google Ads API) and
  SEOmonitor API, data processing, chart generation, insight writing, and PPTX creation.
  Works for all Inbound client brands.
---

# SEO Monthly Report Generator

End-to-end monthly SEO performance presentation builder for Inbound client brands.

## Before You Start

**Read these reference files in order:**

1. `references/slide-mapping.md` — Per-slide spec: data, tables, charts, insight templates
2. `references/design-system.md` — Colors, fonts, layout coordinates, **asset placement with exact x/y/w/h**
3. `references/api-reference.md` — **API endpoints, request/response examples, data flow diagram**
4. `references/chart-specs.md` — matplotlib chart rendering code (4 chart types)
5. `/mnt/skills/public/pptx/SKILL.md` — PPTX creation mechanics (pptxgenjs or python-pptx)

**Referans sunum:** `references/vitra_referans_sunum.pptx` — Orijinal VitrA Ocak 2026 sunumu. Üretilen PPTX bu sunumla birebir aynı tonda olmalı. Emin olmadığın durumlarda bu dosyayı unpack edip slide yapısını incele.

---

## Required Assets

Sunum oluşturmak için bu dosyalar gereklidir. `assets/` klasöründe bulunur:

| Asset | Dosya Yolu | Boyut | Kullanım |
|-------|-----------|-------|----------|
| Inbound Wordmark Logo (beyaz) | `assets/logos/inbound_wordmark_white.png` | ~230×48px | Kapak slide alt orta |
| Inbound Circle Icon | `assets/icons/inbound_icon.png` | ~80×80px | Tüm içerik slide'larının sol alt köşesi |
| Dekoratif Daire Motifi | `assets/decorative/circle_motif.png` | ~490×1080px | Kapak sol, kapanış sağ (flip horizontal) |

> **Önemli:** Logolar siyah arka plan üzerinde beyaz olarak görünür. Sunumda coral (#FF7B52) arka plan üzerinde kullanıldığında beyaz olarak render edilir. circle_motif.png koyu renkte — coral bg üzerinde %15 opacity ile yarı-saydam şekilde yerleştirilir.

---

## Minimum Required Input

| Parameter | Required | Example |
|-----------|----------|---------|
| **Brand name** | ✅ | "VitrA" |
| **Report month & year** | ✅ | "Şubat 2026" |

### Derived Parameters
```
[Ay]    = Türkçe ay adı (Ocak, Şubat, Mart, Nisan, Mayıs, Haziran, Temmuz, Ağustos, Eylül, Ekim, Kasım, Aralık)
[Yıl]   = 2026
[Yıl-1] = 2025
[Yıl-2] = 2024
[Ay-1]  = Önceki ay Türkçe (Ocak → Aralık, yıl geri alınır)
[month] = English month (January, February...) — API query'lerinde kullanılır
```

---

## Step 0: Configuration

### 0a. Brand Profile

Load `references/brand-profiles/[brand].json`. İçeriği:
```json
{
  "brand": "VitrA",
  "domain": "vitra.com.tr",
  "brand_regex": "vitra|vitrA|VitrA",
  "ga4_property_id": "properties/XXXXXXXXX",
  "seomonitor_campaign_id": "XXXXX",
  "gsc_site_url": "sc-domain:vitra.com.tr",
  "location_code": 2792,
  "segments": ["Armatürler", "Banyo Aksesuarları", ...],
  "competitors": [{"name": "trendyol", "keyword": "trendyol"}, ...],
  "ai_referral_sources": ["chatgpt.com", "gemini.google.com", ...]
}
```

Profil yoksa kullanıcıdan topla ve `_template.json`'dan oluştur.

### 0b. Keyword Data Source

**Bu, kullanıcıya sorulması gereken ilk soru.** Slide 4–7 keyword hacim verisine ihtiyaç duyar:

> "Keyword verileri için hangi kaynağı kullanayım?"
> 1. **SEOmonitor gruplarından çek** (brand+category ve non-brand grupları tanımlı)
> 2. **CSV dosyası yükle** (keyword | segment | type | aylık hacimler)
> 3. **WHC platform verisi**

Segment kırılımı (Slide 7) için de aynı soru geçerli — segment grupları SEOmonitor'den mi gelecek, CSV'den mi?

### 0c. Competitor List

Slide 8 için rakip listesi gerekli. Öncelik: brand profile → SEOmonitor kampanya → kullanıcıya sor.

---

## Step 1: Data Collection

Tüm API çağrılarını toplu yap, her birini ayrı ayrı anlatma. Detaylı endpoint bilgileri için `references/api-reference.md` dosyasını oku.

### API Çağrı Matrisi

| Slide | API | Endpoint | Çıktı |
|-------|-----|----------|-------|
| 4 | Google Ads | KeywordPlanIdeaService.GenerateHistoricalMetrics | raw/kp_brand.json |
| 5 | Google Ads | (aynı endpoint, brand+cat keyword listesi) | raw/kp_brand_cat.json |
| 6 | Google Ads | (aynı endpoint, non-brand keyword listesi) | raw/kp_nonbrand.json |
| 7 | Google Ads | (aynı endpoint, segment bazlı gruplar) | raw/kp_segment.json |
| 8 | Google Ads | (aynı endpoint, rakip keyword'ler) | raw/kp_competitors.json |
| 10 | GA4 | properties/{id}:runReport — channel group | raw/ga4_channel.json |
| 11 | GA4 | properties/{id}:runReport — organic 13 ay | raw/ga4_organic_13m.json |
| 13 | GSC | searchAnalytics.query — query dim, 3 period | raw/gsc_brand_nonbrand.json |
| 14 | GSC | searchAnalytics.query — date dim, 36 ay | raw/gsc_trend_36m.json |
| 15 | SEOmonitor | /v3/rank-tracker/visibility | raw/seom_visibility.json |
| 16 | SEOmonitor | /v3/rank-tracker/share-of-clicks | raw/seom_soc.json |
| 17 | SEOmonitor | /v3/rank-tracker/groups | raw/seom_groups.json |
| 18-19 | SEOmonitor | /v3/rank-tracker/keywords | raw/seom_keywords.json |
| 20 | GA4 | properties/{id}:runReport — AI source filter | raw/ga4_ai_referral.json |
| 21 | GSC | searchAnalytics.query — query dim, 2 period | raw/gsc_keywords.json |
| 22 | GSC | searchAnalytics.query — page dim, 2 period | raw/gsc_pages.json |

**Hata yönetimi:** Çağrı başarısız → 1 kez retry → hâlâ başarısız → slide'ı "veri çekilemedi" placeholder ile oluştur, kullanıcıya raporla.

---

## Step 2: Data Processing

Her slide için ham veriyi sunum-hazır formata dönüştür. Detaylar `references/slide-mapping.md`'de.

### Evrensel Hesaplamalar
```
YoY % = (current - last_year_same) / last_year_same × 100
MoM % = (current - previous_month) / previous_month × 100
```

### Sayı Formatlama
| Aralık | Format | Örnek |
|--------|--------|-------|
| ≥ 1M | X.XM | 2.4M |
| ≥ 1K | X.XK | 33.1K |
| < 1K | Ham sayı | 587 |
| Yüzde | %X.X | %18.2 |
| Para (TRY) | ₺X.XXK | ₺565.08K |

### Conditional Formatting
Pozitif → yeşil gradient (`#57BB8A` → `#C5E8D7`), negatif → kırmızı gradient (`#E67C73` → `#F2BCB7`). Detay `references/design-system.md`'de.

---

## Step 3: Chart Generation

Python matplotlib ile PNG olarak üret. `references/chart-specs.md` dosyasında 4 chart tipi için hazır kod var:

| Slide | Chart Tipi | Dosya Adı |
|-------|-----------|-----------|
| 4 | Grouped Bar 2 seri | chart_slide04_brand_volume.png |
| 5 | Grouped Bar 2 seri | chart_slide05_brand_cat_volume.png |
| 6 | Grouped Bar 2 seri | chart_slide06_nonbrand_volume.png |
| 11 | Combo Stacked Bar + Line | chart_slide11_organic_combo.png |
| 14 | Grouped Bar 3 seri (×2) | chart_slide14_impression.png + chart_slide14_click.png |
| 16 | Dual Pie | chart_slide16_soc_sov.png |

Chart'ları 300 DPI, beyaz arka plan, `bbox_inches='tight'` ile kaydet.

---

## Step 4: Insight Generation

Her data slide için `references/slide-mapping.md`'deki insight template'larını kullan. Değişkenleri hesaplanmış verilerle doldur.

### Kurallar
- **Dil:** Türkçe, formal, analitik ton
- **Vurgulama:** Artış yüzdesi → yeşil `#27AE60`, düşüş → kırmızı `#E74C3C`, metrik adları → bold
- **Uzunluk:** Slide başına 2-5 bullet, her biri 1-2 cümle max
- **Yön kelimeleri:**
  - Cümle ortası: >0 → "artarken" | <0 → "azalırken" | =0 → "sabit kalırken"
  - Cümle sonu: >0 → "artmıştır" | <0 → "azalmıştır" | =0 → "sabit kalmıştır"

---

## Step 5: PPTX Generation

`/mnt/skills/public/pptx/SKILL.md` dosyasını oku. pptxgenjs veya python-pptx kullan.

### Slide Boyutu
```javascript
// pptxgenjs
pres.defineLayout({ name: 'INBOUND_WIDE', width: 26.67, height: 15.0 });
pres.layout = 'INBOUND_WIDE';
```

### 23 Slide Build Sırası

Her slide tipi için tasarım detayları `references/design-system.md`'de (koordinatlar, asset konumları dahil).

```
Slide  1: KAPAK           assets: wordmark logo + circle motif
Slide  2: İÇİNDEKİLER     assets: circle motif + inbound icon
Slide  3: SECTION DIVIDER  "01 - Arama Hacmi & Rekabet"
Slide  4: DATA + CHART     chart_slide04 + tablo + insight
Slide  5: DATA + CHART     chart_slide05 + tablo + insight
Slide  6: DATA + CHART     chart_slide06 + tablo + insight
Slide  7: DATA + 4 TABLO   4 tablo + insight panel
Slide  8: DATA + TABLO     büyük tablo + insight
Slide  9: SECTION DIVIDER  "02 - GA4 Trafik"
Slide 10: DATA + TABLO     ana tablo + mini tablolar + insight
Slide 11: DATA + CHART     chart_slide11 + mini tablolar + insight
Slide 12: SECTION DIVIDER  "03 - GSC Trafik, Sıralama, Visibility"
Slide 13: DATA + KPI       KPI kartları + YoY tablo + insight
Slide 14: DATA + 2 CHART   chart_slide14 ×2 + insight
Slide 15: DATA + TABLO     visibility tablo/liste + insight
Slide 16: DATA + 2 PIE     chart_slide16 + insight
Slide 17: DATA + LİSTE     kategori liste + insight
Slide 18: DATA + TABLO     top keywords tablo
Slide 19: DATA + 2 TABLO   top + rising keywords
Slide 20: DATA + TABLO     AI referral tablo + insight
Slide 21: DATA + 2 TABLO   keyword gainers + losers
Slide 22: DATA + 2 TABLO   page gainers + losers
Slide 23: KAPANIŞ          assets: circle motif (flipped)
```

### Ortak Elemanlar (her data slide'da)
1. **BG:** `#FAF8F5`
2. **Breadcrumb:** x:0.5 y:0.3, coral `#FF7B52`, Bricolage SemiBold 24pt
3. **Inbound icon:** x:0.4 y:14.0, w:0.6 h:0.6 → `assets/icons/inbound_icon.png`
4. **Source badge:** x:1.2 y:14.0, w:3.5 h:0.6, roundRect `#FF7B52`, Ubuntu 14pt white

---

## Step 6: QA

```bash
# PPTX → PDF → JPG
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
rm -f slide-*.jpg
pdftoppm -jpeg -r 150 output.pdf slide
ls -1 "$PWD"/slide-*.jpg
```

Her slide thumbnail'ını kontrol et: text overflow, eksik veri, chart hizalama, tablo formatı, asset konumları.

---

## Output

1. `[brand]_SEO_Degerlendirme_[Ay]_[Yıl].pptx`
2. Başarılı/başarısız slide özeti
3. Veri anomalileri notu (varsa)
