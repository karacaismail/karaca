# APE-EAP — EA Platform UI Tasarım ve Varyant Geliştirme

Bu depo, **AI First EA (APE-EAP)** ürün ailesinin (EA Platform, EBP, EOP, EBM, ERX)
frontend/UI tasarım sistemi çalışmalarını barındırır. Yerel proje klasörü karşılığı:
`frontend/claudeui`.

## İçerik

### `docs/ui-variant-plan/` — Çok-Varyantlı UI Geliştirme Planı

Card, form elemanları, data table ve destekleyici bileşenler için **[A,B,C,D,E,F]**
yaklaşımıyla 6 ince-taneli UI varyantı geliştirme planı. Varyantların genel tasarım
felsefesi ortaktır (Flat 2.0 temeli + bağlamsal kartlar, sabit renk/font/token seti);
farklılıklar 12 mikro-detay ekseninde tanımlıdır. Bu bir A/B testi değil, çok-kollu
ince ayrıntı karşılaştırmasıdır.

| Dosya | İçerik |
|---|---|
| [00-genel-plan.md](docs/ui-variant-plan/00-genel-plan.md) | Ana plan: fazlar (P0–P5), zaman çizelgesi, riskler, karar kapıları |
| [01-varyant-cercevesi.md](docs/ui-variant-plan/01-varyant-cercevesi.md) | Değişmezler, 12 mikro-eksen, A–F varyant tanımları, mühendislik/Figma modeli |
| [02-card-varyantlari.md](docs/ui-variant-plan/02-card-varyantlari.md) | Card ailesi × A–F spesifikasyonları |
| [03-form-varyantlari.md](docs/ui-variant-plan/03-form-varyantlari.md) | Form elemanları × A–F, validation modeli |
| [04-table-varyantlari.md](docs/ui-variant-plan/04-table-varyantlari.md) | Data table/grid × A–F, mobil stratejiler, performans |
| [05-bilesen-varyantlari.md](docs/ui-variant-plan/05-bilesen-varyantlari.md) | Button, badge, tabs, paneller, overlay bileşenleri × A–F |
| [06-figma-mcp-promptlari.md](docs/ui-variant-plan/06-figma-mcp-promptlari.md) | Figma MCP prompt kataloğu (variables, component set, karşılaştırma canvas'ı) |
| [07-storybook-mcp-promptlari.md](docs/ui-variant-plan/07-storybook-mcp-promptlari.md) | Storybook MCP prompt kataloğu (matrix story, testler, CI kapıları) |
| [08-degerlendirme-protokolu.md](docs/ui-variant-plan/08-degerlendirme-protokolu.md) | Varyant seçim protokolü: skor kartı, eleme, domain eşleme, freeze |
| [09-frontend-katman-mimarisi.md](docs/ui-variant-plan/09-frontend-katman-mimarisi.md) | Uzlaşılmış katman mimarisi: SurfaceContract, App Core, renderer katmanları, karar kaydı (K1–K10) |

## Sabit tasarım kararları (özet)

- **Font:** Roboto (min weight 400, min boyut 1rem), script fallback Noto Sans.
- **Renkler:** Primary `#FFB900` (üstünde metin daima `#080616`), secondary Parlement Mavisi
  `#003399` (dark modda yalnız yüzey/border; metin accent'i `#93A8F4`), dark canvas `#080616`.
- **Radius:** yüzey/kontrol tavanı 8px (0.5rem; mutlak üst limit 12px), **input alanları muaf**.
- **Yaklaşım:** Mobile-native first (320px), dark/light, data-dense, WCAG 2.2 AA,
  i18n/RTL hazır, glass yalnız chrome/overlay.

## Araç zinciri

Figma MCP (tasarım + variables + Code Connect) ↔ Storybook MCP (kod + story + test + CI).
Prompt katalogları 06 ve 07 numaralı dosyalardadır.
