# 10 — Frontend Katman Mimarisi (Uzlaşılmış Nihai Tanım, Revizyon R1)

Bu doküman, çok-taraflı mimari tartışmanın (katman listesi → headless eleştirisi → renderer
mimarisi → nihai uzlaşma) bağlayıcı sonucudur: frontend'in katmanlarını, her katmanın
sahipliğini, bağımlılık kurallarını ve A–F varyant programının bu katmanlara nasıl
oturduğunu tanımlar. Çelişki durumunda bu dosya, katmanlama konusunda diğer tartışma
kayıtlarından üstündür; varyant değerlerinde ise [01-varyant-cercevesi.md](./01-varyant-cercevesi.md)
üstün kalır.

Kanonik kaynak kuralı: bu doküman setinin kanonik kaynağı `karacaismail/karaca`
reposundaki `docs/ui-variant-plan/` ağacıdır; yerel kopyalar (ör. makinedeki
`frontend/ui-variant-plan/` klasörü) türevdir ve repoya senkronlanır. Yerel ağaçta
üretilmiş ve repoda olmayan dosyalar (bileşen envanteri, antdesign notları) repoya
alınırken numara çakışması yaşanmaması için **09 numarası bileşen envanteri dosyasına
rezerve edilmiştir**; bu dosya bu nedenle 10 numaradadır. (İlk yayın 09 numarasıyla ve
MK kararları "K" önekiyle yapılmıştı; 08'deki K1–K8 kriter etiketleriyle çakıştığı için
Revizyon R1'de MK-* önekine ve 10 numarasına taşındı.)

Bağlantılı dosyalar: [00-genel-plan.md](./00-genel-plan.md) ·
[01-varyant-cercevesi.md](./01-varyant-cercevesi.md) · [04-table-varyantlari.md](./04-table-varyantlari.md) ·
[07-storybook-mcp-promptlari.md](./07-storybook-mcp-promptlari.md) ·
[08-degerlendirme-protokolu.md](./08-degerlendirme-protokolu.md)

## 1. Karar kaydı (uzlaşılan maddeler)

| # | Karar | Gerekçe |
|---|---|---|
| MK-1 | İki headless seviyesi vardır: **SurfaceContract** (yüzeyin NE olduğu) + **renderer-içi davranış motoru** (etkileşimin NASIL çalıştığı) | SurfaceContract klavye/focus/seçim yönetmez; renderer'lar yönetir. İkisi birbirinin yerine geçmez |
| MK-2 | Davranış sahipliği **bileşen ailesi başına tek sahiptir**; tek global kütüphane zorunlu değildir | TanStack Table grid'in sahibi; form/overlay için ilk aday React Aria (Base UI dar spike ile koşullu). Yeni sahip eklemek karar kapısından geçer |
| MK-3 | Router = **TanStack Router** | Taşıyıcı özellik tip güvenli `validateSearch` (URL-sahipli grid state); Query/Table ekosistem uyumu. React Router Data Mode SPA'da loader/action sunar — elenme nedeni bu değil, typed search param zayıflığıdır |
| MK-4 | **AntD = varsayılan CRUD renderer** (`core-mono` temasıyla); **custom AEP renderer** = gelişmiş yüzeyler | Frappe modeli: kutudan çıkan çalışır UI + estetik bağımsızlık theme → template → renderer üçlüsüyle |
| MK-5 | **A–F varyant programı tam sadakatle yalnız custom AEP renderer'da yaşar**; AntD'de theme adapter üzerinden yaklaşıklama | Stripe şeridi, inset ring, input muafiyeti, F yükselti-ikame kuralları AntD anatomisiyle ifade edilemez |
| MK-6 | **Renderer Registry ikinci gerçek renderer'a kadar ertelenir**; kontrat saflığı ilk günden CI kuralıdır | Abstraction'ı ikinci somut kullanıcıdan önce inşa etmemek; ama kontrata UI tipi sızmasını sonradan temizlemek imkânsıza yakındır |
| MK-7 | Kontrat saflığı: SurfaceContract ve Generated SDK **ReactNode, AntD tipi, CSS class, renderer importu taşımaz** | Aynı kontratın iki renderer'da çalışması kabul testidir |
| MK-8 | Ad yetkisi ikiye ayrılır: saf teknik adları MASTER belirler; **proje sözlüğünden türeyen adlar önce sözlük onayından geçer** (İsmail) | Generated SDK adları binlerce kez üretir; sözlük (atonota taksonomisi) çözülmeden SDK üretimi başlamaz. Jenerik DDD terimleri identifier'lara giremez |
| MK-9 | `core-mono` = platform default teması; AEP marka teması (#FFB900 / #003399 / #080616) onun üstünde bir temadır | Planın "renk değişmez" kuralı marka temasının İÇİNDE geçerlidir; platform default'una uygulanmaz |
| MK-10 | A11y + test kapıları renderer'dan bağımsızdır ve her katmanın "bitti" tanımıdır | AntD'nin default davranışı axe fail-blocking / 44px hedef / focus-visible kapılarının yerine geçmez |
| MK-11 | Kontrat topolojisi ayrışır: Kernel sözleşmesi → (a) transport DTO + Generated SDK, (b) UI projeksiyonu → **SurfaceDefinition**; adapter, SDK yanıtını **SurfaceData**'ya çevirir | SDK backend'in tel biçimini taşır, SurfaceDefinition UI'ın ne göstereceğini tanımlar; `Kernel → SDK → SurfaceContract` tek doğrusal zinciri ikisini karıştırır |
| MK-12 | SurfaceDefinition'ın kanonik biçimi **JSON Schema 2020-12**; TypeScript tipleri ve Zod şemaları üretilmiş tüketicilerdir; OpenAPI yalnız transport sınırında | Çok-dilli ve agent-okur tüketiciler (2030 hedefi) şema-kanonik kaynak ister; TS-kanonik olsaydı agent'lar ikinci elden okurdu |
| MK-13 | Form state sahibi **React Hook Form + Zod**; AntD Form yalnız görsel renderer olarak kullanılır — ikinci bir validation otoritesi kurulmaz | Renderer bağımsızlığı (MK-5/MK-7 ile tutarlı); validation tek kaynaktan türeyip her iki renderer'da aynı çalışır |
| MK-14 | Mock altyapısı **MSW + adapter sınırı**; aynı mock'lar geliştirme, Storybook ve testte kullanılır | Tek fixture kaynağı; mock-gerçek geçişi adapter'da kalır (E4→E5) |
| MK-15 | TanStack Router **file-based routing** + `validateSearch` | Resmî önerilen mod; route ağacı ve search şemaları tipli üretilir |
| MK-16 | İlk teslimat tek **golden slice**'tır: liste/DataGrid + URL'de filtre/sort/page + form veya drawer + 5 durum (loading/empty/error/permission/zero-results) + tr ve ar-RTL + 3 density — AYNI SurfaceDefinition ile hem AntD hem custom AEP render | Bilinmeyen bilinmeyenleri (AntD global CSS/portal sızıntıları, React Aria–AntD focus/dismiss farkları, URL–Query key senkronu, RTL uzaması, virtualization) geniş üretimden ÖNCE ortaya çıkarır |
| MK-17 | Compact density (36px satır) yalnız fine-pointer masaüstü bağlamında önerilir; touch bağlamda standard/comfortable varsayılır; her durumda interaktif hücre hit-area ≥44px kuralı geçerli kalır | 36px satır – 44px dokunma hedefi gerilimi cihaz koşuluyla çözülür ([04](./04-table-varyantlari.md) hit-area kuralı korunur) |
| MK-18 | Theme bir **ThemeProfile**'dır: renderer'ın token konfigürasyon profili — bağımsız bir kompozisyon katmanı değildir. A–F overlay yalnız X1 kesen ekseninde yaşar; D1'deki üç-seviye ifadesi ThemeProfile yığını olarak okunur | D1/X4 (tema modları) ve D1/X1 (A–F) mükerrerliği kapanır; tek konum kuralı |

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
│  (Registry: bugün basit eşleme dosyası — MK-6)                      │
│   ├── react-antd renderer (varsayılan CRUD)                       │
│   └── custom AEP renderer (gelişmiş yüzeyler, A–F tam sadakat)    │
│                                                                   │
│  Her renderer'ın İÇ katmanları (custom AEP'de tamamı elle;        │
│  AntD'de R4–R7'yi AntD sağlar):                                   │
│   R1 Design token'ları (primitive/semantic/density/variant-overlay)│
│   R2 CSS temeli (reset, font, tema modları, logical props/RTL)    │
│   R3 Grid & layout (container-query öncelikli)                    │
│   R4 Görsel primitive'ler (Box, Text, Icon, Portal, VisuallyHidden)│
│   R5 Davranış katmanı (aile başına TEK sahip — MK-2)                │
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
| A3 SurfaceContract | İkiye ayrışır (MK-11): **SurfaceDefinition** = alanlar, kolonlar, aksiyonlar, validation/görünürlük kuralları, durum sözlüğü, i18n anahtarları, a11y semantiği, `comparisonPriority`/sortable/filterable view metadata'sı (kanonik biçim JSON Schema 2020-12, MK-12). **SurfaceData** = adapter'ın SDK yanıtından ürettiği ekran verisi | `ReactNode`, AntD tipleri, CSS class, renderer importu (MK-7 — CI kuralı) |

Kontrat topolojisi (MK-11):

```text
Kernel sözleşmesi
    ├── transport projeksiyonu → OpenAPI/DTO → Generated SDK ─┐
    │                                                         ├─ Adapter → SurfaceData
    └── UI projeksiyonu → SurfaceDefinition ──────────────────┘
                                                              ↓
                                                   Renderer + Controller (R5)
```

SDK ve SurfaceDefinition birbirinin yerine geçmez: SDK tel biçimini, SurfaceDefinition
UI projeksiyonunu taşır; ikisini adapter birleştirir.

Adlandırma: A1'deki kavram adları proje sözlüğüne (atonota taksonomisi) tabidir; sözlükte
statüsü çözülmemiş terimler (ör. "Domain"in seviye adı olarak kullanımı) identifier'a
girmez, SDK üretimi sözlük onayını bekler (MK-8).

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
| R5 Davranış katmanı | Aile başına TEK sahip: grid → TanStack Table (+Virtual); form/overlay → React Aria (aday, MK-2); sahiplik tablosu bu dosyada güncellenir | Klavye-tam play testleri; sahip başına tek implementasyon |
| R6 Bileşenler | R5 davranışı + R1 token'ı birleştirir; **varyant-kör** — yalnız `data-variant` token'ı okur, harf bilmez | Matrix story (varyant × tema × density) + axe yeşil |
| R7 Durum bileşenleri | Skeleton (gerçek içerik geometrisi + **density'ye tepkili**, layout shift yok), Empty, Error, Offline, Permission-denied | Her sayfa şablonunda 5 durumun story'si |
| R8 Birleşik pattern'ler | DataGrid, arama+filtre, form bölümü, file upload, tarih aralığı | Play + i18n (de/tr/ar-RTL) story'leri |

### D. Kompozisyon katmanları

| Katman | İçerik |
|---|---|
| D1 Theme | Üç seviye: `core-mono` (platform default'u; nötr, semantik durum renkleri ve focus korunur) → AEP marka teması (sabit palet, MK-9) → A–F variant overlay (yalnız custom renderer'da tam) |
| D2 UI Template | PageTemplate slot kompozisyonu: Header / PrimaryActions / FilterBar / Content / Aside / FooterActions / Overlays. Aynı renderer ile farklı kompozisyonlar üretir |

Renderer seçimi (Registry): bugün basit bir eşleme dosyasıdır
(`surface-tipi → renderer`, app veya route bazında RendererProfile); ikinci gerçek
renderer üretime girdiğinde registry altyapıya terfi eder (MK-6). Gelişmiş DataGrid,
AntD sayfası içinde kontrollü bir capability island olarak yaşayabilir — tablo
davranışının sahibi o adada TanStack'tir, iç içe iki tablo motoru kurulmaz.

MK-18 uyarınca D1 bir ThemeProfile yığınıdır (renderer token konfigürasyonu): tema
modları/density X4'te, A–F overlay X1'de çözülür; D1 bunların profil olarak
paketlenmesidir, ayrı bir stil kaynağı değildir.

### E. Ürün katmanları

| Katman | İçerik | Backend gerekir mi? |
|---|---|---|
| E1 App Shell | Sidebar/header/breadcrumb, mobil navigasyon (bottom nav ≤5 öğe) | Hayır |
| E2 Sayfa şablonları | Liste, detay, create/edit, wizard, ayarlar — SurfaceContract tüketir | Hayır |
| E3 Feature ekranları | İş dilini bilen ekranlar (ör. teşvik başvuru formu) | Kısmen |
| E4 Mock API | Kontratla AYNI şemayı kullanan mock implementasyon; adapter sınırından takılır | Hayır |
| E5 Entegrasyon | Gerçek API, auth, retry, cache invalidation, optimistic update | Evet |

Kontrat tipleri E4'te değil A katmanında, en başta tanımlanır; E4 yalnız implementasyondur
(MSW + adapter sınırı; aynı mock'lar dev/Storybook/test — MK-14).

E5 iyimser "adapter'ı değiştir" adımı DEĞİLDİR; kendi protokol kapısıyla gelir:
pagination davranışı, auth/token yenileme, hata zarfı, optimistic concurrency, upload
ve yetki modeli bu kapıda gerçek backend'e karşı ayrıca doğrulanır.

## 4. Bağımlılık kuralları (yasaklar)

- Her katman yalnız kendinden ALTTAKİ katmana bağımlıdır; yatay ve yukarı bağımlılık yasak.
- A katmanı hiçbir UI tipini import edemez (MK-7 — CI'da `ReactNode`/antd/css importu taraması).
- B katmanı renderer bilmez; renderer seçimi D'de yapılır.
- R6 bileşenleri Router/Query import edemez; veri ve navigasyon callback'leri sınırdan
  (props/context) girer.
- R6 bileşenleri varyant harfini bilemez; yalnız R1 token'ı tüketir (X1).
- R4 primitive'leri ürün anlamı taşıyamaz; E3 feature bileşenleri R1–R5'e doğrudan inemez
  (R6/R8 üzerinden geçer).
- Aynı bileşen ailesinde ikinci bir davranış sahibi eklemek karar kapısına tabidir (MK-2).

## 5. P0–P5 ve backend-öncesi sınır eşleşmesi

| Faz | Katmanlar |
|---|---|
| P0 | R1 + R2 (+ AntD ThemeConfig adapter iskeleti); A3 kontrat tipleri taslağı |
| P1 | R3 + R4 + R5 (davranış sahiplik tablosu dahil) |
| P1→P2 kapısı | **MK-16 golden slice**: aynı SurfaceDefinition ile AntD + custom AEP'de liste/DataGrid + URL state + form/drawer + 5 durum + tr/ar-RTL + 3 density; bu dilim geçmeden geniş bileşen/A–F üretimine girilmez |
| P2 | R6 + R7 + R8 × A–F (custom renderer); AntD renderer'da eşdeğer CRUD yüzeyi |
| P3 | D2 + E1 + E2 (kompozisyon ekranları, 6-lı karşılaştırma) |
| P4–P5 | 08 protokolü → kazanan varyant(lar) D1'e terfi; E3 feature ekranları kazananla başlar |
| Backend sonrası | E5 (E4 mock, adapter sınırından gerçek SDK istemcisiyle değiştirilir) |

Backend başlamadan tamamlanabilir sınır: A3 (tipler) + B (mock'la) + C + D + E1–E4.
Bu sınır "ürün bitti" değil, **entegrasyona hazır frontend** demektir.

## Kabul kriterleri

- [ ] SurfaceContract ve SDK'da UI tipi importu yok; CI taraması kurulu (MK-7).
- [ ] Davranış sahiplik tablosu dolu: her bileşen ailesinin tek sahibi yazılı; sahipsiz aile yok (MK-2).
- [ ] Router yalnız URL state'in, Query yalnız uzak verinin sahibi; bileşen içinde Router/Query importu yok.
- [ ] R6 bileşenlerinde varyant harfi geçmiyor; varyant yalnız `data-variant` + token overlay ile.
- [ ] AntD yüzeylerinde A–F sadakati İDDİA EDİLMİYOR; yaklaşıklama sınırı dokümante.
- [ ] `core-mono` temasında semantik durum renkleri ve focus göstergesi mevcut (MK-9/MK-10).
- [ ] Skeleton'lar density moduna tepkili; yükleme→içerik geçişinde layout shift yok.
- [ ] Kernel/SDK identifier'ları sözlük onayından geçti; jenerik DDD terimi identifier'da yok (MK-8).
- [ ] Aynı SurfaceDefinition örneği hem react-antd hem custom AEP renderer'da render edilebiliyor (MK-7 kabul testi — registry terfisinin ön koşulu); test yalnız "render oldu" değil: import sınırı, aynı intent çıktıları, klavye davranışı, a11y ve görsel geometri de doğrulanır.
- [ ] SurfaceDefinition kanoniği JSON Schema 2020-12; TS/Zod tüketicileri şemadan üretiliyor, elle çatallanmıyor (MK-12).
- [ ] MK-16 golden slice P2 başlamadan iki renderer'da da yeşil.
- [ ] Form validation tek otoriteden (RHF+Zod) türüyor; AntD Form kuralları ikinci kaynak değil (MK-13).
