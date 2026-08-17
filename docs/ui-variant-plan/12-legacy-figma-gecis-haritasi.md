# 12 — Legacy Figma Geçiş Haritası ("Variants" sistemi → yeni token mimarisi)

Bu doküman, beş yıllık Figma "Variants" sisteminin (Border/Shadow/Spacing/
Margin&Padding/Size/Radius/Pseudo classes/Style colors kümeleri) yeni token
mimarisine ([01](./01-varyant-cercevesi.md), [10](./10-frontend-katman-mimarisi.md))
geçiş matrisidir. Karar dili: **KEEP / RENAME / SPLIT / MERGE / RETIRE / UNKNOWN**.
UNKNOWN kalemleri yalnız sahip (İsmail) kapatır. Eski Figma dosyası çöpe atılmaz;
bu matris kapanana kadar referans arşividir.

Bağlantılı dosyalar: [01-varyant-cercevesi.md](./01-varyant-cercevesi.md) ·
[10-frontend-katman-mimarisi.md](./10-frontend-katman-mimarisi.md) ·
[11-vibecoding-gorev-paketi.md](./11-vibecoding-gorev-paketi.md)

## 1. Geçiş matrisi

| Eski küme | Eski değerler | Karar | Yeni karşılık |
|---|---|---|---|
| Border (1–6) | 1,2,3,4,5,6 px | **SPLIT + RETIRE** | `border/width`: 0 / 1 / 2. 1→default, 2→focus ring & durum şeridi kalınlığı. 3–6 RETIRE (hiçbir spesifikasyonda kullanılmıyor). Semantik roller: subtle/default/strong/focus |
| Shadow (XS–XP) | 6 adsız kademe | **MERGE + RETIRE** | `elevation`: none / raised (y=2 blur=8 %10) / overlay (y=4 blur=16 %12) — yalnız Variant F + overlay katmanı; dark'ta ton+1px border ikamesi. 6 kademe RETIRE: 3 kademe yeterli |
| Spacing (12–240) | 12,24,32,48,64,72,96,128,160,240 | **MERGE** | Tek primitive ölçek `space/1..7` = 4,8,12,16,24,32,48. 64/96 yalnız R3 bölüm boşluğu olarak ÖNERİ (space/8=64, space/9=96 — sahip onayı bekler). 72,128,160,240 RETIRE |
| Margin & Padding (4–120) | 4,6,8,12,16,24,36,60,92,120 | **MERGE** | Ayrı token ailesi OLMAZ: margin/padding/gap aynı spacing ölçeğini tüketir. 6,36,60,92,120 RETIRE (ölçek dışı) |
| Size (XS–XP) | 6 adsız kademe | **SPLIT** | "Neyin boyutu?" ayrışır: control-height = density (36/44/52); icon (20/24); avatar ölçeği; container max-width; viewport bantları (320/480/768/1024/1440). Tek genel "size" ölçeği RETIRE |
| Radius (XS–XP) | 6 adsız kademe | **RENAME** | `radius`: 0 / 2 / 4 / 6 / 8 / pill. Tavan 8px; pill yalnız kapsül bileşenler + Variant F input ([01] muafiyet kuralı). Bileşen eşlemesi varyant başına 01'de |
| Pseudo classes (Active/DeActive × Default/Hover/Focus/Disabled/Error/Collapse) | 2 sütun × 6 durum | **SPLIT** | X5 Interaction State Grammar'a dağılır: Default/Hover/Focus → native pseudo (`:hover`, `:focus-visible`, hover-guard ile); Disabled → etkileşim durumu (`:disabled`); Error → validation (`:user-invalid` / `[aria-invalid]`); Collapse → `[aria-expanded="false"]` (pseudo-class değil ARIA state). "Active/DeActive" ekseni RETIRE — seçim/basılılık `[aria-selected]` / `[aria-pressed]` / `[aria-current]` ile ifade edilir |
| Style colors (Succes/Info/Warning/Danger/Disabled × Light/Dark) | 5 durum × 2 tema | **SPLIT** | Success/Info/Warning → semantic status renkleri (light/dark mode'lu, ikon+metin eşliğinde). **Danger ikiye ayrılır:** danger-aksiyon tonu (yıkıcı buton) ≠ error-validation durumu. **Disabled renk DEĞİLDİR** → etkileşim durumu (opaklık/etkisizleştirme token'ı). Light/Dark iki sütunluk tablo değil, TÜM semantic koleksiyonun mode'udur. Yazım düzeltmesi: "Succes" → success |
| "cihaza göre / değil" notları | Frame başına serbest not | **RENAME** | Ölçülebilir responsive sözleşme: min viewport 320px (eski iOS4 tarayıcı uyumluluğu HEDEFLENMEZ, yalnız genişlik sözleşmesidir); shell=media query, bileşen=container query; hover yalnız `(hover:hover) and (pointer:fine)` |

## 2. Geçişin genel ilkeleri

1. Eski sistemde **isim var, anlam yok** (XS–XP): yeni sistemde her token'ın rolü ve
   kullanım eşlemesi vardır; adsız kademe taşınmaz.
2. Eski sistem **durum türlerini karıştırıyor**: etkileşim durumu (hover/disabled),
   validation durumu (error), ARIA durumu (expanded/selected) ve ürün durumu
   (loading/empty) ayrı türlerdir — X5 grameri bu ayrımı zorunlu kılar.
3. Eski sistemde **mükerrerlik** var (Spacing ↔ Margin&Padding): yeni sistemde tek
   ölçek, çok tüketici.
4. Her RETIRE kararı geri alınabilir: eski değer arşiv Figma'sında durur; ihtiyaç
   kanıtlanırsa token önerisi olarak geri gelir (doğrudan kod içine dönemez).

## 3. Açık kalemler (sahip onayı bekleyen)

- [ ] 64/96 bölüm boşluğu token'ları (space/8, space/9) eklensin mi?
- [ ] Avatar ve container max-width ölçeklerinin değerleri (eski "Size" kümesinden
      hangileri taşınacak — UNKNOWN: eski kademelerin px karşılıkları dosyadan okunmalı).
- [ ] Danger aksiyon tonunun hex'i (error/600 #DC2626'dan ayrışacak mı, aynı mı kalacak?).

## Kabul kriterleri

- [ ] Matristeki her satırın kararı Foundation Contract'a işlendi; UNKNOWN kalmadı.
- [ ] Yeni token kaynağında eski adlarla (XS–XP, Active/DeActive) hiçbir token yok.
- [ ] Storybook 00 Foundations kümesinde "Legacy Migration Map" sayfası bu matrisi
      JSON kaynağından render ediyor.
- [ ] Disabled hiçbir yerde renk token'ı olarak tanımlı değil; Danger-aksiyon ile
      error-validation ayrımı token adlarında görünür.
