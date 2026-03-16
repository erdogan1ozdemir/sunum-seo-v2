# SEO Monthly Report — Claude Skill

Marka adı ve ay verin, gerisini bu skill halleder: API'lerden veri çekme, hesaplama, chart üretimi, insight yazımı ve PPTX sunum oluşturma.

Inbound dijital pazarlama ajansının tüm markaları için çalışır.

---

## Ne İşe Yarar?

Her ay müşteri markalar için hazırlanan **23 slide'lık Aylık SEO Değerlendirme Sunumu**nu otomatik oluşturur.

- **Google Ads API** → Keyword arama hacimleri (brand, category, non-brand, rakipler)
- **GA4 Data API** → Organik trafik, kanal performansı, AI referral trafik
- **Search Console API** → Impression, click, CTR, keyword/sayfa performansı
- **SEOmonitor API** → Visibility, share of clicks, AI SoV, keyword ranking

YoY/MoM hesaplar, chart üretir, Türkçe insight yazar, Inbound tasarımında PPTX oluşturur.

---

## Kurulum (Claude Code)

### 1. Repo'yu klonla
```bash
git clone https://github.com/erdogan1ozdemir/seo-monthly-report.git
```

### 2. Skill'i Claude Code'a kur
```bash
mkdir -p ~/.claude/skills
cp -r seo-monthly-report/ ~/.claude/skills/seo-monthly-report/
```

### 3. Marka profili oluştur
```bash
cp references/brand-profiles/_template.json references/brand-profiles/yeni-marka.json
# Düzenle: domain, GA4 Property ID, SEOmonitor Campaign ID, rakipler, segmentler
```

---

## Kullanım

```
VitrA için Şubat 2026 sunumu hazırla
```

Skill sorar: "Keyword verileri için hangi kaynağı kullanayım?" (SEOmonitor / CSV / WHC)

Sonra tüm API çağrılarını yapar ve PPTX oluşturur.

---

## Dosya Yapısı

```
seo-monthly-report/
├── SKILL.md                                    # Ana workflow (6 adım)
├── README.md                                   # Bu dosya
├── LICENSE
├── .gitignore
├── references/
│   ├── slide-mapping.md                        # 23 slide × tablo/chart/insight detayı
│   ├── design-system.md                        # Renkler, fontlar, asset koordinatları (x/y/w/h)
│   ├── api-reference.md                        # API endpoint, request/response, veri akış şeması
│   ├── chart-specs.md                          # matplotlib chart kodu (4 chart tipi)
│   ├── vitra_referans_sunum.pptx               # Orijinal referans sunum (tasarım/ton kontrolü)
│   └── brand-profiles/
│       ├── vitra.json                          # VitrA örnek profili
│       └── _template.json                      # Boş şablon
├── assets/
│   ├── logos/
│   │   └── inbound_wordmark_white.png          # Beyaz wordmark (kapak + TOC)
│   ├── icons/
│   │   └── inbound_icon.png                    # Daire ikon (tüm data slide sol alt)
│   └── decorative/
│       └── circle_motif.png                    # Daire/yay motifi (kapak sol, kapanış sağ)
└── docs/
    ├── vitra_referans_sunum.pptx               # İnsan referansı kopya
    └── SEO_Sunum_Otomasyon_Mapping.xlsx        # Slide mapping Excel (insan referansı)
```

### Önemli Dosyalar

| Dosya | Ne İşe Yarar |
|-------|-------------|
| `api-reference.md` | Her API çağrısının endpoint'i, request body, response parse kodu, output JSON yapısı |
| `design-system.md` | Her asset'in slide üzerindeki tam koordinatları (x, y, width, height inch cinsinden) |
| `slide-mapping.md` | Her slide'ın tablo kolon/satır yapısı, chart renk/eksen/seri detayı, insight cümlesi şablonu |
| `vitra_referans_sunum.pptx` | Skill'in ürettiği sunumun birebir eşleşmesi gereken orijinal sunum |

---

## API Gereksinimleri

| API | Kullanılan Slide'lar | Kimlik Doğrulama |
|-----|---------------------|-----------------|
| Google Ads (Keyword Planner) | 4–8 | OAuth2 + developer token |
| GA4 Data API | 10, 11, 20 | OAuth2 |
| Search Console API | 13, 14, 21, 22 | OAuth2 |
| SEOmonitor API | 15–19 | API Key (header) |

---

## Marka Profili

`references/brand-profiles/_template.json` dosyasını kopyalayıp doldur:

```json
{
  "brand": "Marka Adı",
  "domain": "marka.com.tr",
  "brand_regex": "marka|Marka",
  "ga4_property_id": "properties/XXXXXXXXX",
  "seomonitor_campaign_id": "XXXXX",
  "gsc_site_url": "sc-domain:marka.com.tr",
  "location_code": 2792,
  "segments": ["Kategori 1", "Kategori 2"],
  "competitors": [{"name": "rakip1", "keyword": "rakip1"}],
  "ai_referral_sources": ["chatgpt.com", "gemini.google.com", "perplexity.ai"]
}
```

---

## Sunum Tasarımı

| Element | Değer |
|---------|-------|
| Coral (accent) | `#FF7B52` |
| Koyu Yeşil (header) | `#10332F` |
| Kırık Beyaz (bg) | `#FAF8F5` |
| Başlık fontu | Bricolage Grotesque ExtraBold |
| Gövde fontu | Calibri |
| Slide boyutu | 26.67 × 15.00 inch |

---

## Lisans

MIT
