# Sunum Design System

> Renkler, fontlar, layout kuralları, asset konumları. Tüm koordinatlar inch cinsinden (pptxgenjs/python-pptx).
> Slide boyutu: 26.67" × 15.00" (EMU: 24387175 × 13716000) — Custom Widescreen 16:9

---

## Renk Paleti

| Rol | HEX | RGB | CSS |
|-----|-----|-----|-----|
| Coral (Primary) | `#FF7B52` | 255,123,82 | Kapak/kapanış bg, breadcrumb, source badge, vurgulu rakamlar |
| Koyu Yeşil | `#10332F` | 16,51,47 | Section divider bg, tablo header bg |
| Yarı-saydam Yeşil | `#254E49` | 37,78,73 | Section divider watermark numara |
| Kırık Beyaz | `#FAF8F5` | 250,248,245 | İçerik slide arka planı |
| Siyah | `#000000` | 0,0,0 | Ana gövde metin |
| Koyu Gri | `#313131` | 49,49,49 | Tablo header (data slide'lar) |
| Pozitif Yeşil | `#27AE60` | 39,174,96 | Artış yüzde metni |
| Negatif Kırmızı | `#E74C3C` | 231,76,60 | Düşüş yüzde metni |
| Chart Mavi | `#4472C4` | 68,114,196 | Bar seri: önceki yıl |
| Chart Kırmızı | `#FF0000` | 255,0,0 | Bar seri: güncel yıl |
| Chart Gri | `#B7B7B7` | 183,183,183 | Bar seri: revenue / 2 yıl önce |
| Chart Koyu | `#2D2D2D` | 45,45,45 | Bar seri: önceki yıl (3-seri chart) |
| Chart Sarı | `#FFD666` | 255,214,102 | Bar seri: güncel yıl (3-seri) / sessions |
| Chart Line Kırmızı | `#E74C3C` | 231,76,60 | Line seri: transactions |

### Conditional Formatting Gradient

| Aralık | Renk | HEX |
|--------|------|-----|
| Güçlü pozitif (>20%) | Koyu yeşil | `#57BB8A` |
| Orta pozitif (5-20%) | Orta yeşil | `#9DD8BB` |
| Hafif pozitif (0-5%) | Açık yeşil | `#C5E8D7` |
| Nötr (0%) | Beyaz | `#FFFFFF` |
| Hafif negatif (-5% - 0%) | Açık kırmızı | `#F2BCB7` |
| Orta negatif (-20% - -5%) | Orta kırmızı | `#EA948D` |
| Güçlü negatif (<-20%) | Koyu kırmızı | `#E67C73` |

---

## Fontlar

| Font | Ağırlık | Kullanım | Boyut |
|------|---------|----------|-------|
| Bricolage Grotesque | ExtraBold | Slide başlıkları, KPI sayılar | 52-75pt |
| Bricolage Grotesque | SemiBold | Breadcrumb | 24-30pt |
| Bricolage Grotesque | Regular | Insight paragraf, alt başlık | 21-29pt |
| Outfit | SemiBold | Section divider watermark numara | ~400pt |
| Outfit | ExtraLight | Kapak başlık, kapanış "Teşekkürler" | 86-96pt |
| Outfit | Regular | Kapak alt başlık "[Ay] [Yıl]" | 86pt |
| Calibri | Regular | Tablo verileri, gövde metin | 14-20pt |
| Inter Medium | Medium | KPI kart etiketleri | 14-16pt |
| Ubuntu | Regular | Source badge metin | 14pt |

> **Not:** Bu fontlar Google Slides'tan geldiği için sistemde olmayabilir. Fallback: Arial.
> pptxgenjs ile kullanırken `fontFace: "Bricolage Grotesque"` şeklinde belirt — font yoksa Arial'a düşer.

---

## Asset Konumları ve Boyutları

### Dosya Konumları
```
assets/
├── logos/
│   └── inbound_wordmark_white.png    → Beyaz wordmark logo (kapak + TOC)
├── icons/
│   └── inbound_icon.png              → Daire ikon (tüm içerik slide sol alt)
└── decorative/
    └── circle_motif.png              → Dekoratif daire/yay motifi (kapak + kapanış)
```

### Kapak Slide (Slide 1) — Asset Yerleşimi

```
┌────────────────────────────────────────────────────────┐
│  BG: Coral #FF7B52 (tüm slide)                        │
│                                                        │
│  circle_motif.png                                      │
│  x: -1.0" y: 0.5" w: 10" h: 13"                      │
│  Opacity: %15-20 (beyaz, arka plan üstünde yarı-saydam)│
│  Flip: none (sol tarafta, yay sağa açılır)            │
│                                                        │
│  "[brand] | Aylık SEO Değerlendirme"                   │
│  x: 2" y: 5" w: 22" h: 3"                            │
│  Font: Outfit ExtraLight, 95pt, white, center          │
│                                                        │
│  "[Ay]  [Yıl]"                                         │
│  x: 2" y: 8" w: 22" h: 2"                            │
│  Font: Outfit Regular, 86pt, white, center             │
│                                                        │
│  inbound_wordmark_white.png                            │
│  x: 11.5" y: 13.5" w: 3.5" h: 0.7"                  │
│  (alt orta, beyaz logo)                                │
└────────────────────────────────────────────────────────┘
```

### İçindekiler Slide (Slide 2) — Asset Yerleşimi

```
┌──────────────────────┬─────────────────────────────────┐
│ SOL YARI             │ SAĞ YARI                        │
│ BG: Coral #FF7B52    │ BG: Kırık Beyaz #FAF8F5         │
│ w: 12" (x: 0-12)    │ w: 14.67" (x: 12-26.67)        │
│                      │                                 │
│ circle_motif.png     │ "01" (Outfit, 40pt, #434343)    │
│ x: -1" y: -1"       │ "Marka, Kategori Arama..."      │
│ w: 8" h: 10"         │ (Bricolage SemiBold, 20pt, #10332F)│
│ Opacity: %15         │                                 │
│                      │ "02" ...                        │
│ "SUNUM AKIŞI"        │ "03" ...                        │
│ x: 1" y: 6" w: 10"  │                                 │
│ Font: Outfit XLight  │                                 │
│ 96pt, white          │                                 │
│                      │                                 │
│ inbound_icon.png     │                                 │
│ x: 0.5" y: 13.5"    │                                 │
│ w: 0.8" h: 0.8"     │                                 │
└──────────────────────┴─────────────────────────────────┘
```

### Section Divider (Slide 3, 9, 12) — Asset Yerleşimi

```
┌────────────────────────────────────────────────────────┐
│  BG: Koyu Yeşil #10332F                               │
│                                                        │
│  Coral Dash (üst)                                      │
│  x: 12.5" y: 5.5" w: 1.5" h: 0.15"                  │
│  Shape: roundRect, fill: #FF7B52                       │
│                                                        │
│  "Arama Hacmi & Rekabet"                               │
│  x: 3" y: 6" w: 20" h: 3"                            │
│  Font: Bricolage ExtraBold, 74pt, white, center        │
│                                                        │
│  Coral Dash (alt)                                      │
│  x: 12.5" y: 9.5" w: 1.5" h: 0.15"                  │
│  Shape: roundRect, fill: #FF7B52                       │
│                                                        │
│  "02" (Watermark numara)                               │
│  x: -0.5" y: 9" w: 6" h: 5.5"                       │
│  Font: Outfit SemiBold, ~400pt, #254E49 (yarı-saydam) │
└────────────────────────────────────────────────────────┘
```

### İçerik Slide (Tüm Data Slide'ları) — Ortak Elemanlar

```
┌────────────────────────────────────────────────────────┐
│  BG: Kırık Beyaz #FAF8F5                              │
│                                                        │
│  BREADCRUMB                                            │
│  x: 0.5" y: 0.3" w: 20" h: 0.6"                     │
│  Font: Bricolage SemiBold, 24pt, #FF7B52              │
│  Metin: "Sayfa Kategorisi | Alt Kategori"             │
│                                                        │
│  BAŞLIK                                                │
│  x: 0.5" y: 1.0" w: 25" h: 2.5"                     │
│  Font: Bricolage ExtraBold, 52pt, #000000             │
│  (Vurgulu kelimeler: #FF7B52 veya #E74C3C)            │
│                                                        │
│  [İÇERİK ALANI: x: 0.5" y: 3.5" → y: 13.5"]         │
│                                                        │
│  inbound_icon.png                                      │
│  x: 0.4" y: 14.0" w: 0.6" h: 0.6"                   │
│                                                        │
│  SOURCE BADGE                                          │
│  x: 1.2" y: 14.0" w: 3.5" h: 0.6"                   │
│  Shape: roundRect, fill: #FF7B52, radius: 0.15"       │
│  Metin: "Source: Keyword Planner"                      │
│  Font: Ubuntu, 14pt, white, center                     │
└────────────────────────────────────────────────────────┘
```

### Kapanış Slide (Slide 23) — Asset Yerleşimi

```
┌────────────────────────────────────────────────────────┐
│  BG: Coral #FF7B52                                     │
│                                                        │
│  circle_motif.png                                      │
│  x: 17" y: 0.5" w: 10" h: 13"                        │
│  Opacity: %15 (SAĞ tarafta — kapak'ın aynası)         │
│  Flip: horizontal                                      │
│                                                        │
│  Beyaz Dash (üst)                                      │
│  x: 12.5" y: 6.0" w: 1.5" h: 0.1"                   │
│  Shape: roundRect, fill: #FFFFFF                       │
│                                                        │
│  "Teşekkürler"                                         │
│  x: 3" y: 6.5" w: 20" h: 3"                          │
│  Font: Outfit ExtraLight, 86pt, white, center          │
│                                                        │
│  Beyaz Dash (alt)                                      │
│  x: 12.5" y: 9.5" w: 1.5" h: 0.1"                   │
│  Shape: roundRect, fill: #FFFFFF                       │
└────────────────────────────────────────────────────────┘
```

---

## Slide Akışı (23 Slide)

```
 1  KAPAK
 2  İÇİNDEKİLER
 3  SECTION DIVIDER — 01 Arama Hacmi & Rekabet
 4  Brand Arama Hacmi (Grouped Bar + Tablo)
 5  Brand + Category Hacmi (Grouped Bar + Tablo)
 6  Non-Branded Hacim (Grouped Bar + Tablo)
 7  Segment Kırılım (4 Tablo + Insight)
 8  Rekabet Tablosu (Tablo + Insight)
 9  SECTION DIVIDER — 02 GA4 Trafik
10  GA4 Kanal Performansı (Tablo + Mini Tablo + Insight)
11  GA4 Organik Detay (Combo Chart + Mini Tablo + Insight)
12  SECTION DIVIDER — 03 GSC Trafik, Sıralama, Visibility
13  GSC Brand & Non-Brand (KPI Kartları + YoY Tablo + Insight)
14  GSC Impression/Click Trend (2 Grouped Bar + Insight)
15  Competitor Visibility (Tablo/Liste + Insight)
16  Share of Clicks & AI SoV (2 Pie Chart + Insight)
17  Kategori Bazında Visibility (Liste + Insight)
18  Visibility Top Keywords (Tablo)
19  Top & Rising Keywords (2 Panel Tablo)
20  AI Referral Trafik (Tablo + Insight)
21  GSC Keyword Performansı — İyileşen/Kötüleşen (2 Tablo)
22  GSC Sayfa Performansı — İyileşen/Kötüleşen (2 Tablo)
23  KAPANIŞ
```
