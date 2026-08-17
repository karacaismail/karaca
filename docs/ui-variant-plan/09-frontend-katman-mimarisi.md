# 09 — Frontend Katman Mimarisi (Uzlaşılmış Nihai Tanım)

Bu doküman, çok-taraflı mimari tartışmanın (katman listesi → headless eleştirisi → renderer
mimarisi → nihai uzlaşma) bağlayıcı sonucudur: frontend'in katmanlarını, her katmanın
sahipliğini, bağımlılık kurallarını ve A–F varyant programının bu katmanlara nasıl
oturduğunu tanımlar. Çelişki durumunda bu dosya, katmanlama konusunda diğer tartışma
kayıtlarından üstündür; varyant değerlerinde ise [01-varyant-cercevesi.md](./01-varyant-cercevesi.md)
üstün kalır.

Bağlantılı dosyalar: [00-genel-plan.md](./00-genel-plan.md) ·
[01-varyant-cercevesi.md](./01-varyant-cercevesi.md) · [04-table-varyantlari.md](./04-table-varyantlari.md) ·
[07-storybook-mcp-promptlari.md](./07-storybook-mcp-promptlari.md) ·
[08-degerlendirme-protokolu.md](./08-degerlendirme-protokolu.md)

## 1. Karar kaydı (uzlaşılan maddeler)

| # | Karar | Gerekçe |
|---|---|---|
| K1 | İki headless seviyesi vardır: **SurfaceContract** (yüzeyin NE olduğu) + **renderer-içi davranış motoru** (etkileşimin NASIL çalıştığı) | SurfaceContract klavye/focus/seçim yönetmez; renderer'lar yönetir. İkisi birbirinin yerine geçmez |
| K2 | Davranış sahipliği **bileşen ailesi başına tek sahiptir**; tek global kütüphane zorunlu değildir | TanStack Table grid'in sahibi; form/overlay için ilk aday React Aria (Base UI dar spike ile koşullu). Yeni sahip eklemek karar kapısından geçer |
| K3 | Router = **TanStack Router** | Taşıyıcı özellik tip güvenli `validateSearch` (URL-sahipli grid state); Query/Table ekosistem uyumu. React Router Data Mode SPA'da loader/action sunar — elenme nedeni bu değil, typed search param zayıflığıdır |
| K4 | **AntD = varsayılan CRUD renderer** (`core-mono` temasıyla); **custom AEP renderer** = gelişmiş yüzeyler | Frappe modeli: kutudan çıkan çalışır UI + estetik bağımsızlık theme → template → renderer üçlüsüyle |
| K5 | **A–F varyant programı tam sadakatle yalnız custom AEP renderer'da yaşar**; AntD'de theme adapter üzerinden yaklaşıklama | Stripe şeridi, inset ring, input muafiyeti, F yükselti-ikame kuralları AntD anatomisiyle ifade edilemez |
| K6 | **Renderer Registry ikinci gerçek renderer'a kadar ertelenir**; kontrat saflığı ilk günden CI kuralıdır | Abstraction'ı ikinci somut kullanıcıdan önce inşa etmemek; ama kontrata UI tipi sızmasını sonradan temizlemek imkânsıza yakındır |
| K7 | Kontrat saflığı: SurfaceContract ve Generated SDK **ReactNode, AntD tipi, CSS class, renderer importu taşımaz** | Aynı kontratın iki renderer'da çalışması kabul testidir |
| K8 | Ad yetkisi ikiye ayrılır: saf teknik adları MASTER belirler; **proje sözlüğünden türeyen adlar önce sözlük onayından geçer** (İsmail) | Generated SDK adları binlerce kez üretir; sözlük (atonota taksonomisi) çözülmeden SDK üretimi başlamaz. Jenerik DDD terimleri identifier'lara giremez |
| K9 | `core-mono` = platform default teması; AEP marka teması (#FFB900 / #003399 / #080616) onun üstünde bir temadır | Planın "renk değişmez" kuralı marka temasının İÇİNDE geçerlidir; platform default'una uygulanmaz |
| K10 | A11y + test kapıları renderer'dan bağımsızdır ve her katmanın "bitti" tanımıdır | AntD'nin default davranışı axe fail-blocking / 44px hedef / focus-visible kapılarının yerine geçmez |

## 2. Katman haritası

```text
┌─ A. SÖZLEŞME KATMANLARI (UI bilmez) ──────────────────────────────┐
│  A1 Kernel sözleşmeleri                                           │
│  A2 Generated SDK          A3 Headless SurfaceContract            │
└──────────────┬───────────────────────────┬────────────────────────┘
               ↓                           ↓
┌─ B. APP CORE (renderer bilmez) ───────────────────────────────────┐
│  B1 TanStack Router   B2 TanStack Query   B3 Auth/Session/Flags   │
│  B4 i18n çekirdeği (locale, Intl/CLDR, dir)   B5 Telemetry        │
└──────────────┬────────────────────────────────────────────────────┘
               ↓
┌─ C. RENDERER SEÇİMİ ──────────────────────────────────────────────┐
│  (Registry: bugün basit eşleme dosyası — K6)                      │
│   ├── react-antd renderer (varsayılan CRUD)                       │
│   └── custom AEP renderer (gelişmiş yüzeyler, A–F tam sadakat)    │
│                                                                   │
│  Her renderer'ın İÇ katmanları (custom AEP'de tamamı elle;        │
│  AntD'de R4–R7'yi AntD sağlar):                                   │
│   R1 Design token'ları (primitive/semantic/density/variant-overlay)│
│   R2 CSS temeli (reset, font, tema modları, logical props/RTL)    │
│   R3 Grid & layout (container-query öncelikli)                    │
│   R4 Görsel primitive'ler (Box, Text, Icon, Portal, VisuallyHidden)│
│   R5 Davranış katmanı (aile başına TEK sahip — K2)                │
│   R6 Bileşenler (davranış + token; varyant-kör)                   │
│   R7 Durum bileşenleri (Skeleton/Empty/Error/Offline/Permission)  │
│   R8 Birleşik pattern'ler (DataGrid, arama+filtre, form bölümü)   │
└──────────────┬────────────────────────────────────────────────────┘
               ↓
┌─ D. KOMPOZİSYON KATMANLARI ───────────────────────────────────────┐
│  D1 Theme (core-mono default → AEP marka teması → A–F overlay)    │
│  D2 UI Template (PageTemplate slot'ları: Header/Filter/Content/…) │
└──────────────┬────────────────────────────────────────────────────┘
               ↓
┌─ E. ÜRÜN KATMANLARI ──────────────────────────────────────────────┐
│  E1 App Shell + navigasyon                                        │
│  E2 Sayfa şablonları (liste/detay/form/wizard/ayarlar)            │
│  E3 Feature ekranları (iş dilini bilir)                           │
│  E4 Mock API implementasyonu (kontrat tipleri A'da zaten erken)   │
│  E5 Gerçek backend entegrasyonu                                   │
└───────────────────────────────────────────────────────────────────┘

KESEN EKSENLER (katman değil; ilgili katmanları dikine keser):
  X1 Variant overlay A–F  → R1–R8 (custom renderer'da tam, AntD'de yaklaşık)
  X2 A11y + test kapıları → her katmanın "bitti" tanımı (story+play+axe+görsel regresyon)
  X3 i18n/RTL             → B4 (locale/Intl) + R2 (logical properties) ÇİFT yerleşim
  X4 Tema modları + density → token seviyesinde (R1) çözülür, bileşene sızmaz
```

## 3. Katman tanımları

### A. Sözleşme katmanları

| Katman | İçerik | Bilmediği şeyler |
|---|---|---|
| A1 Kernel sözleşmeleri | İş kuralları, izinler, aksiyonlar, invariant'lar, audit | React, JSX, herhangi bir UI kütüphanesi |
| A2 Generated SDK | Tipli query/command istemcisi, hata modeli, pagination modeli | UI teknolojisi |
| A3 SurfaceContract | Alanlar, kolonlar, aksiyonlar, validation/görünürlük kuralları, loading/empty/error/permission durumları, i18n anahtarları, a11y semantiği, `comparisonPriority`/sortable/filterable view metadata'sı | `ReactNode`, AntD tipleri, CSS class, renderer importu (K7 — CI kuralı) |

Adlandırma: A1'deki kavram adları proje sözlüğüne (atonota taksonomisi) tabidir; sözlükte
statüsü çözülmemiş terimler (ör. "Domain"in seviye adı olarak kullanımı) identifier'a
girmez, SDK üretimi sözlük onayını bekler (K8).

### B. App Core

| Katman | Sahiplik | Not |
|---|---|---|
| B1 TanStack Router | URL state: route, page, sort, paylaşılabilir filtre (`validateSearch` şemalı) | Router bileşenlerin içine sızmaz |
| B2 TanStack Query | Uzak veri + cache, route-seviyesi prefetch, invalidation | Query bileşenlerin içine sızmaz; veri props/context sınırından girer |
| B3 Auth/Session, feature flags | Oturum görünümü, yetki bağlamı | İzin KARARI Kernel'dedir; burada yalnız görünümü |
| B4 i18n çekirdeği | Locale çözümü, Intl/CLDR formatlama, mesaj katalogları, `dir` yönetimi | RTL'in CSS ayağı R2'dedir (X3) |
| B5 Telemetry | Bileşen kullanım/etkileşim ölçümü | 08 protokolünün veri kaynağı |

State sahiplik tablosu (bağlayıcı):

| State | Sahibi |
|---|---|
| Route, page, sort, paylaşılabilir filtre | TanStack Router |
| Uzak veri ve cache | TanStack Query |
| Form state | Form kütüphanesi / headless form (E2-R8 sınırında) |
| Bileşen-yerel UI state (açık/kapalı, hover…) | Bileşenin kendisi |
| Oturum/yetki görünümü | App Core (B3) |

### C. Renderer-içi katmanlar (R1–R8)

Custom AEP renderer'da tamamı kurulur; AntD renderer'da R4–R7'yi AntD sağlar ve R1'den
yalnız theme adapter (vendor-neutral token → `ThemeConfig`) beslenir.

| Katman | İçerik | "Bitti" tanımı (X2) |
|---|---|---|
| R1 Token'lar | 4 koleksiyon: primitive / semantic (light+dark) / density (3 mod) / variant-overlay (a–f). Style Dictionary → CSS variables + AntD ThemeConfig adapter çıktısı | Kontrast matrisi AA; token drift CI'da |
| R2 CSS temeli | Reset, self-host Roboto+Noto fallback, tema modları, **logical properties (start/end)**, `prefers-reduced-motion` | RTL smoke testi; dark/light geçişi |
| R3 Grid & layout | Container-query öncelikli; Stack/Flex/Split; 320px-first bantlar | 320/768/1440 story'leri |
| R4 Görsel primitive'ler | Box, Text, Icon (Phosphor), Portal, VisuallyHidden — ürün anlamı taşımaz | Story + axe |
| R5 Davranış katmanı | Aile başına TEK sahip: grid → TanStack Table (+Virtual); form/overlay → React Aria (aday, K2); sahiplik tablosu bu dosyada güncellenir | Klavye-tam play testleri; sahip başına tek implementasyon |
| R6 Bileşenler | R5 davranışı + R1 token'ı birleştirir; **varyant-kör** — yalnız `data-variant` token'ı okur, harf bilmez | Matrix story (varyant × tema × density) + axe yeşil |
| R7 Durum bileşenleri | Skeleton (gerçek içerik geometrisi + **density'ye tepkili**, layout shift yok), Empty, Error, Offline, Permission-denied | Her sayfa şablonunda 5 durumun story'si |
| R8 Birleşik pattern'ler | DataGrid, arama+filtre, form bölümü, file upload, tarih aralığı | Play + i18n (de/tr/ar-RTL) story'leri |

### D. Kompozisyon katmanları

| Katman | İçerik |
|---|---|
| D1 Theme | Üç seviye: `core-mono` (platform default'u; nötr, semantik durum renkleri ve focus korunur) → AEP marka teması (sabit palet, K9) → A–F variant overlay (yalnız custom renderer'da tam) |
| D2 UI Template | PageTemplate slot kompozisyonu: Header / PrimaryActions / FilterBar / Content / Aside / FooterActions / Overlays. Aynı renderer ile farklı kompozisyonlar üretir |

Renderer seçimi (Registry): bugün basit bir eşleme dosyasıdır
(`surface-tipi → renderer`); ikinci gerçek renderer üretime girdiğinde registry
altyapıya terfi eder (K6).

### E. Ürün katmanları

| Katman | İçerik | Backend gerekir mi? |
|---|---|---|
| E1 App Shell | Sidebar/header/breadcrumb, mobil navigasyon (bottom nav ≤5 öğe) | Hayır |
| E2 Sayfa şablonları | Liste, detay, create/edit, wizard, ayarlar — SurfaceContract tüketir | Hayır |
| E3 Feature ekranları | İş dilini bilen ekranlar (ör. teşvik başvuru formu) | Kısmen |
| E4 Mock API | Kontratla AYNI şemayı kullanan mock implementasyon; adapter sınırından takılır | Hayır |
| E5 Entegrasyon | Gerçek API, auth, retry, cache invalidation, optimistic update | Evet |

Kontrat tipleri E4'te değil A katmanında, en başta tanımlanır; E4 yalnız implementasyondur.

## 4. Bağımlılık kuralları (yasaklar)

- Her katman yalnız kendinden ALTTAKİ katmana bağımlıdır; yatay ve yukarı bağımlılık yasak.
- A katmanı hiçbir UI tipini import edemez (K7 — CI'da `ReactNode`/antd/css importu taraması).
- B katmanı renderer bilmez; renderer seçimi D'de yapılır.
- R6 bileşenleri Router/Query import edemez; veri ve navigasyon callback'leri sınırdan
  (props/context) girer.
- R6 bileşenleri varyant harfini bilemez; yalnız R1 token'ı tüketir (X1).
- R4 primitive'leri ürün anlamı taşıyamaz; E3 feature bileşenleri R1–R5'e doğrudan inemez
  (R6/R8 üzerinden geçer).
- Aynı bileşen ailesinde ikinci bir davranış sahibi eklemek karar kapısına tabidir (K2).

## 5. P0–P5 ve backend-öncesi sınır eşleşmesi

| Faz | Katmanlar |
|---|---|
| P0 | R1 + R2 (+ AntD ThemeConfig adapter iskeleti); A3 kontrat tipleri taslağı |
| P1 | R3 + R4 + R5 (davranış sahiplik tablosu dahil) |
| P2 | R6 + R7 + R8 × A–F (custom renderer); AntD renderer'da eşdeğer CRUD yüzeyi |
| P3 | D2 + E1 + E2 (kompozisyon ekranları, 6-lı karşılaştırma) |
| P4–P5 | 08 protokolü → kazanan varyant(lar) D1'e terfi; E3 feature ekranları kazananla başlar |
| Backend sonrası | E5 (E4 mock, adapter sınırından gerçek SDK istemcisiyle değiştirilir) |

Backend başlamadan tamamlanabilir sınır: A3 (tipler) + B (mock'la) + C + D + E1–E4.
Bu sınır "ürün bitti" değil, **entegrasyona hazır frontend** demektir.

## Kabul kriterleri

- [ ] SurfaceContract ve SDK'da UI tipi importu yok; CI taraması kurulu (K7).
- [ ] Davranış sahiplik tablosu dolu: her bileşen ailesinin tek sahibi yazılı; sahipsiz aile yok (K2).
- [ ] Router yalnız URL state'in, Query yalnız uzak verinin sahibi; bileşen içinde Router/Query importu yok.
- [ ] R6 bileşenlerinde varyant harfi geçmiyor; varyant yalnız `data-variant` + token overlay ile.
- [ ] AntD yüzeylerinde A–F sadakati İDDİA EDİLMİYOR; yaklaşıklama sınırı dokümante.
- [ ] `core-mono` temasında semantik durum renkleri ve focus göstergesi mevcut (K9/K10).
- [ ] Skeleton'lar density moduna tepkili; yükleme→içerik geçişinde layout shift yok.
- [ ] Kernel/SDK identifier'ları sözlük onayından geçti; jenerik DDD terimi identifier'da yok (K8).
- [ ] Aynı SurfaceContract örneği hem react-antd hem custom AEP renderer'da render edilebiliyor (K7 kabul testi — registry terfisinin ön koşulu).
