# 13 — AiCommandCard Spesifikasyonu (Genişleyen AI-First Kart)

Bu doküman, admin panel ve frontpage'lerde ORTAK kullanılacak `AiCommandCard`
bileşeninin bağlayıcı spesifikasyonudur: kullanıcı hikâyesi, iki vibecoding yasası
(Morph Yasası + Mikro-Animasyon Sözleşmesi), anatomi/adlandırma, durum makinesi,
koreografi ve makine-denetlenebilir kabul testleri. LLM worker'lar bu dosyayı prose
olarak değil KONTRAT olarak okur: her kural bir Storybook play testine bağlıdır.

Bağlantılı dosyalar: [01-varyant-cercevesi.md](./01-varyant-cercevesi.md) ·
[05-bilesen-varyantlari.md](./05-bilesen-varyantlari.md) ·
[10-frontend-katman-mimarisi.md](./10-frontend-katman-mimarisi.md) ·
[11-vibecoding-gorev-paketi.md](./11-vibecoding-gorev-paketi.md)

## 1. Kullanıcı hikâyesi

**Kısa hikâye:** Siteyi gezen bir kullanıcı olarak, arama çubuğu gibi görünen
AI-first karta tıkladığımda kartın KENDİSİNİN — border'larından itibaren — yanlara ve
aşağıya doğru esneyerek büyümesini istiyorum; böylece 12'li menü kartlarını görebilir,
arama yapabilir, aynı input alanında AI sorgusu yazabilir ve mevcut sayfayı kontrol
edebilirim.

**Kapalı durumda görünenler (yalnız ikon):** logo işareti, breadcrumb'ın yalnız son
ucu (current page), nabız atan AI işareti, bildirimler işareti.

**Açık durumda ortaya çıkanlar:** tam breadcrumb izi, AI arama alanı, AI sorgu önerisi
pill'leri, 12'li navigasyon kart grid'i, bildirimler ve profil butonları.

### Kabul senaryoları (Given / When / Then)

| # | Senaryo |
|---|---|
| S1 | KAPALI kart görünürken KULLANICI karta tıkladığında (veya Enter/Space) AYNI kutu genişler; yeni bir modal/popover/drawer AÇILMAZ |
| S2 | Genişleme sırasında kapalı durumdaki 4 öğe (logo, crumb, AI işareti, bildirim) kaybolup yeniden belirmez; yeni konumlarına SÜREREK taşınır (shared-element süreklilik) |
| S3 | İçerik, kutu büyüdükçe SIRAYLA ortaya çıkar (arama alanı → pill'ler → kart grid'i); hepsi aynı anda belirmez |
| S4 | AÇIK durumda Esc veya kapatma butonu kartı AYNI kutuya geri daraltır; odak tetikleyiciye döner |
| S5 | Genişleyince odak AI arama alanına gider; klavye ile tüm iç öğeler gezilebilir |
| S6 | `prefers-reduced-motion` etkinse morph animasyonsuz (150ms crossfade) tamamlanır; AI nabzı durur; işlev kaybı olmaz |
| S7 | 320px ekranda kapalı kart tam genişliktir; açık durum viewport'u kaplar; yatay taşma yoktur |
| S8 | Sayfa içeriği genişleme sırasında REFLOW OLMAZ: kartın kapalı ayak izi yerinde kalır, açık panel üstünde yüzer |

## 2. İki Vibecoding Yasası

LLM'lerin "expand" isteğini modal açarak çözmesinin nedeni, isteğin doğal dilde kalması
ve doğal dilin en yaygın pattern'e (overlay bileşeni) çökmesidir. Çözüm: yasayı
invariant + otomatik test olarak yazmak.

### Yasa 1 — Morph Yasası (aynı-düğüm genişlemesi)

1. Kapalı bar ve açık panel **AYNI DOM düğümüdür** (`AiCommandCard.Root`). Genişleme
   bir `data-state` değişimidir; asla yeni bir yüzeyin mount edilmesi değildir.
2. Bu bileşenin İÇİNDE **Portal, Modal, Popover, Drawer, Dialog kullanımı yasaktır**;
   `document.body`'ye veya başka bir konteynere düğüm eklenmez.
3. İçerik her iki durumda da AYNI ağaçta durur; kapalı durumda görsel olarak gizlidir
   (`visibility` + boyut), unmount edilmez (durum makinesi bilinçli lazy-mount'a izin
   verebilir: NavCardGrid ilk genişlemede mount olur, sonra kalır).
4. Sayfa akışında yer tutan öğe `AiCommandCardAnchor`'dur (kapalı ayak izi); Root,
   genişlerken anchor içinde konumunu korur ve overlay katman token'ıyla yüzer —
   düğüm değişmez, yalnız CSS pozisyon/boyut değişir.

**Makine denetimi (play testi — bu testler kırmızıysa iş kabul edilmez):**

```text
MORPH-1: const root = getByRole('region', {name: 'AI komut kartı'});  // Root'un rolü, §4
         click → morph-end BEKLENİR (data-state 'expanding' → 'expanded') → root
         aynı referans VE root.dataset.state === 'expanded'
MORPH-2: [data-scope^="ai-command-card"] seçicisiyle taranan TÜM düğümler
         root.contains(...) === true — Root dışında bu bileşene ait DOM yok
         (bileşen-dışı portallardan — toast/tooltip — etkilenmez)
MORPH-3: açık paneldeki arama alanı root.contains(...) === true
MORPH-4: anchor.getBoundingClientRect() genişleme sırasında sabit (sayfa reflow yok)
```

Not: Yasa 1'in Portal yasağı BU BİLEŞEN KAPSAMINDADIR; R5'in genel overlay/Portal
altyapısını (Menu, Dialog vb. için) yasaklamaz.

### Yasa 2 — Mikro-Animasyon Sözleşmesi

Her mikro-bileşen, §7'deki sözleşme tablosunda kendi satırına sahiptir ve o satır
dışında animasyon içeremez. "Animasyon eksik" sorunu, her satırın bir story +
play/görsel test karşılığı olmasıyla kapanır: satırı boş bırakan worker'ın PR'ı
ChoreographyMatrix story'sinde görünür şekilde eksiktir.

## 3. Anatomi, adlandırma ve dosya haritası

Adlandırma ilkesi: isimler rolü + sahipliği söyler; kısaltma yok; her mikro-bileşen
kendi dosyasında; her dosya kökü `data-scope` attribute'u taşır — farklı LLM/araçlarla
(Codex, Claude, Cursor, Windsurf…) sonradan yapılacak düzenlemeler bu scope'a
hapsolur.

```text
packages/renderer-aep/src/patterns/ai-command-card/
  AiCommandCard.tsx              # Root: durum makinesi sahibi; LAYOUT/STİL İÇERMEZ
  useAiCommandCardState.ts       # davranış hook'u (R5): state machine + focus yönetimi
  AiCommandCardAnchor.tsx        # sayfa akışındaki ayak izi (reflow kalkanı)
  CollapsedBar.tsx               # data-scope="ai-command-card/collapsed-bar"
  ExpandedPanel.tsx              # data-scope="ai-command-card/expanded-panel"
  LogoMark.tsx                   # shared-element
  BreadcrumbCurrentCrumb.tsx     # kapalı: yalnız son uç (shared-element)
  BreadcrumbTrail.tsx            # açık: tam iz
  AiPresenceOrb.tsx              # nabız + partiküller; kendi kendine yeterli
  NotificationsControl.tsx       # kapalı=glyph, açık=buton — AYNI bileşen, data-state'ten türer
  ProfileControl.tsx             # yalnız açık durumda
  AiSearchField.tsx              # arama + AI sorgu girişi (tek alan)
  QuerySuggestionPillRow.tsx     # öneri pill'leri satırı
  QuerySuggestionPill.tsx
  NavCardGrid.tsx                # 12'li grid (1–12 öğeye uyarlanır)
  NavCard.tsx
  ai-command-card.motion.css     # TÜM koreografi tek dosyada (token tüketir)
  ai-command-card.stories.tsx    # kompozit story'ler
  <part>.stories.tsx             # HER mikro-bileşenin kendi story dosyası
```

Kurallar (10 §4 ile birebir): bileşen varyant-kör (`data-variant` token'ı tüketir),
hex/px hardcode yok, `antd` / router / query importu yok; navigasyon ve AI sorgusu
callback olarak dışarıdan gelir — bu sayede admin panel VE frontpage aynı bileşeni
props'la kullanır.

### Public API (kendini dokümante eden adlar)

```ts
interface AiCommandCardProps {
  navigationItems: AiCommandNavigationItem[];   // 12 hedeflenir; grid 1-12'ye uyarlanır
  breadcrumbTrail: BreadcrumbCrumb[];           // son öğe = current page
  aiQuerySuggestions: string[];
  notificationCount?: number;
  onNavigationSelect(item: AiCommandNavigationItem): void;
  onAiQuerySubmit(query: string): void;
  onNotificationsOpen(): void;
  onProfileOpen(): void;
  defaultExpanded?: boolean;
  onExpandedChange?(expanded: boolean): void;
}
```

## 4. Durum makinesi ve X5 eşlemesi

```text
collapsed ──EXPAND(click|Enter|Space)──▶ expanding ──morph-end──▶ expanded
expanded ──COLLAPSE(Esc|kapat|dış tık)─▶ collapsing ──morph-end──▶ collapsed
```

- `data-state="collapsed | expanding | expanded | collapsing"` Root üzerinde; tüm
  CSS koreografisi YALNIZ bu attribute'tan sürülür (X5: kanonik kanca R5'in yazdığı
  attribute'tur).
- Roller: **Root** = `role="region"` + `aria-label="AI komut kartı"` +
  `data-scope="ai-command-card/root"` (her iki durumda a11y ağacında; MORPH-1 bunu
  hedefler). CollapsedBar tetikleyicidir: `role="button"`, `aria-expanded`,
  `aria-controls`. ExpandedPanel düz konteynerdır (rol taşımaz — bölge zaten Root'tur).
  Modal DEĞİLDİR: focus trap yok; dış tıklama daraltır (mobilde ek olarak görünür
  44px kapatma butonu).
- Davranış sahipliği kaydı (MK-2): "same-node morph/disclosure" ailesinin sahibi
  `useAiCommandCardState` custom hook'udur — React Aria bu pattern'i sağlamadığı için
  karar kapısından geçirilmiş YENİ sahiptir ve [10](./10-frontend-katman-mimarisi.md)
  R5 sahiplik tablosuna işlenmiştir; iç odak yardımcıları (FocusScope) React Aria'dan
  alınabilir.
- Odak akışı: genişleme bitince `AiSearchField`'a; daralınca tetikleyiciye döner.
- Geçiş kilidi: `expanding/collapsing` sırasında yeni EXPAND/COLLAPSE olayları
  kuyruklanmaz, mevcut yön tersine çevrilir (yarıda kesilebilir morph).

## 5. Morph tekniği

- Boyut animasyonu kompozitör-dostu kurulur: birincil teknik **FLIP** (First-Last-
  Invert-Play; transform ile) veya `grid-template-rows: 0fr→1fr` (yükseklik) +
  ölçülmüş genişlik geçişi; `interpolate-size: allow-keywords` destekleyen
  tarayıcılarda saf CSS `height/width: auto` geçişi kullanılabilir. Hangisi seçilirse
  seçilsin **Yasa 1 ihlal edilemez** ve 60fps hedeflenir.
- Genişleme sırasında `overflow: hidden` — içerik kutunun büyümesiyle ORTAYA ÇIKAR
  ("kutu içindeki gizem"); içerik girişleri §7 zaman çizelgesine bağlıdır.
- Açık panel yüzer: `surface/overlay` token'ı + overlay z-index token'ı; scrim YOK
  (bu bir modal değil); [01] gereği blur/glass YOK.
- Boyut hedefleri (token'lardan):

| Viewport | Kapalı | Açık |
|---|---|---|
| 320–767 (mobile-first taban) | genişlik %100 (gutter içinde), yükseklik `density/control` (44–52) | AYNI düğüm viewport'u kaplar (safe-area payıyla); dikey ~10x |
| ≥1024 (desktop) | ~700px genişlik, yükseklik `density/control` (44–52) | genişlik +%50 (~1050px), yükseklik içerik kadar (maks. viewport − 2×`space/6` = 64px kompozisyonu) |

## 6. Motion token ekleri (01'in motion ölçeğine işlenen kategoriler)

[01]'deki 120–240ms işlevsel ölçek korunur; bu bileşen iki YENİ kategori (morph,
ambient) ve işlevsel banda bağlı bir entrance token'ı tanımlatır (sahip talebi — bu
dosya kaynaktır, [01]'e işlendi):

| Token | Değer | Kapsam |
|---|---|---|
| `motion/morph/expand` | 400ms, ease-out-quint benzeri (hafif esneme: `cubic-bezier(0.34, 1.16, 0.64, 1)`) | YALNIZ aynı-düğüm konteyner morph'ları |
| `motion/morph/collapse` | 300ms, ease-out | Aynı |
| `motion/ambient/ai-pulse` | 2000ms büyüme + 2000ms küçülme (4s periyot), scale 1→1.06, sonsuz | YALNIZ AiPresenceOrb (AI varlık işareti) |
| `motion/ambient/ai-particles` | 3000ms döngü, CSS-only ≤8 partikül, yalnız opacity/transform | YALNIZ AiPresenceOrb |
| `motion/entrance/stagger` | 30ms aralık, öğe başına 150ms fade + 8px translate; **kümülatif stagger gecikmesi tavanı 120ms** (öğeler/satırlar en fazla 4 giriş grubuna toplanır) | Genişleme içerik girişleri |

Reduced-motion eşlemesi: morph → 150ms crossfade (boyut animasyonu yok);
ai-pulse → kapalı (statik ikon + renk vurgusu); stagger → anlık. Ambient nabız ayrıca
`document.hidden` veya viewport dışındayken durdurulur (CPU/pil).

## 7. Koreografi zaman çizelgesi ve Mikro-Animasyon Sözleşme Tablosu

```text
t=0      tetik → data-state="expanding"
0–400    konteyner morph (Yasa 1 tekniğiyle)
0–400    shared-element FLIP: LogoMark, CurrentCrumb→Trail son öğesi,
         AiPresenceOrb, NotificationsControl yeni konumlarına süzülür
160→     içerik girişleri (morph ~%40), EN FAZLA 4 giriş grubu halinde
         (kümülatif stagger tavanı 120ms): G1 AiSearchField → G2 pill satırı →
         G3 NavCardGrid (satırlar gruplanır) → G4 alt kontroller
≤560     desktop bütçesi: son giriş 160+120+150 ≤ 430ms'te biter, morph 400ms —
         koreografi ≤560ms'te kapanır → data-state="expanded"; odak AiSearchField
≤700     mobil bütçesi (viewport-kaplama morph'u daha uzun yol alır)
```

| Mikro-bileşen | Idle/ambient | Hover (yalnız fine-pointer) | Press | Focus-visible | Giriş (expand) |
|---|---|---|---|---|---|
| CollapsedBar | — | Eksen-4 varyant kuralı | Eksen-4 press türevi | X5 ring | — (kendisi morph olur) |
| LogoMark | — | — | — | — | FLIP taşıma (kaybolmaz) |
| BreadcrumbCurrentCrumb→Trail | — | Crumb linkleri: eksen-4 varyant kuralı | — | Ring | Son öğe FLIP; öncekiler stagger grubunda fade+slide-start |
| AiPresenceOrb | `motion/ambient/ai-pulse` + `motion/ambient/ai-particles` — AI kimliğinin taşıyıcısı; accent sarı token'ı (`color/primary`) TÜM varyantlarda ([01] eksen-11 istisnası) | Halo tint; nabız HIZLANMAZ | Scale yok; ring | Ring | FLIP taşıma |
| NotificationsControl | Badge GELDİĞİNDE tek sefer 150ms pop; döngü yasak | Eksen-4 varyant kuralı | Eksen-4 press türevi | Ring | FLIP taşıma (glyph→buton aynı düğüm) |
| ProfileControl | — | Eksen-4 varyant kuralı | Eksen-4 press türevi | Ring | G4 grubunda fade |
| AiSearchField | — | — | — | X5 ring (odak otomatik gelir) | G1: morph %40'ında fade + 8px yukarıdan |
| QuerySuggestionPill | — | Eksen-4 varyant kuralı | Eksen-4 press türevi (scale YOK) | Ring | G2 grubunda fade + 8px |
| NavCard ×12 | — | Varyant hover kuralı ([02] kart tablosu) | Eksen-4 press türevi | Ring | G3: satırlar gruplanmış stagger, fade + 8px |

Sözleşme kuralları: hover'da/press'te scale-translate YOK ([01] değişmezi — morph ve
FLIP bu yasağın dışıdır çünkü hover değil durum geçişidir); her satır ChoreographyMatrix
story'sinde görünür; satırı uygulanmamış bileşen PR'ı eksik sayılır.

## 8. Storybook story listesi ve play testleri

```text
Patterns/AiCommandCard/
  Collapsed320            Collapsed Desktop
  ExpandJourney           (play: S1–S5 + MORPH-1..4 assert'leri)
  ReducedMotion           (play: S6 — matchMedia mock)
  Mobile320FullJourney    (play: S7 — yatay taşma assert'i)
  NoReflow                (play: S8 — anchor rect sabitliği)
  ChoreographyMatrix      (tüm mikro-bileşenler × idle/hover/press/focus/entrance)
  StateMatrix             (data-state × variant × theme × density — budanmış küme)
Patterns/AiCommandCard/Parts/
  AiPresenceOrb (+ReducedMotion)  NavCard  QuerySuggestionPill  AiSearchField
  NotificationsControl  BreadcrumbTrail  ...  (her parça MockAiCommandCardState
  provider'ı ile İZOLE render edilir — parça bazlı LLM özelleştirmesinin zemini)
```

Play testleri MORPH-1..4'ü, odak akışını, Esc/dış-tık daralmasını, reduced-motion
yolunu ve 320px taşmasını assert eder; axe her story'de fail-blocking.

## 9. Çift kullanım (admin + frontpage)

- Paket: `packages/renderer-aep` patterns katmanı (R8). `apps/admin-demo` ve
  frontpage uygulaması AYNI bileşeni yalnız props'la besler.
- İçerik farkı props'tadır (admin: modül navigasyonu; front: site navigasyonu);
  bileşen içinde bağlam dallanması (`if admin`) YASAK.
- AI sorgu gönderimi `onAiQuerySubmit` callback'idir; App Core (TanStack Query)
  bağlantısı uygulama tarafında kurulur — bileşen veri katmanı bilmez.
- [05]'teki CommandPalette'ten AYRIDIR: CommandPalette klavye-öncelikli global bir
  overlay'dir (glass opsiyoneli oradadır); AiCommandCard sayfa içi morph kartıdır —
  glass/blur bu bileşende YASAKTIR (opak `surface/overlay`). Mobil birincillik:
  320–767 bantta birincil AI/arama yüzeyi AiCommandCard'dır; CommandPalette mobilde
  varsayılan olarak sunulmaz (yalnız güç-kullanıcı kısayolu).
- [10] E1 ilişkisi: App Shell bu pattern'i header olarak KOMPOZE eder; shell'de
  ikinci bir breadcrumb/bildirim yüzeyi kurulmaz (tek sahip bu bileşendir).

## Kabul kriterleri

- [ ] MORPH-1..4 play testleri yeşil; bileşen içinde Portal/Modal/Popover/Drawer importu yok (CI import taraması).
- [ ] Shared-element sürekliliği: kapalı durumun 4 öğesi expand'de unmount OLMUYOR (test: aynı düğüm referansları).
- [ ] §7 tablosundaki her satır ChoreographyMatrix'te görünür ve uygulanmış.
- [ ] Motion yalnız token'lardan; `ai-command-card.motion.css` dışında animasyon tanımı yok.
- [ ] Reduced-motion: morph crossfade, nabız kapalı, işlev tam.
- [ ] 320px: kapalı %100 genişlik, açık viewport-kaplama, yatay taşma yok; dokunma hedefleri ≥44px.
- [ ] Odak akışı: expand→AiSearchField, collapse→tetikleyici; Esc çalışır; axe yeşil.
- [ ] Varyant-körlük: bileşen dosyalarında harf/hex yok; StateMatrix 6 varyantta render.
- [ ] Admin ve frontpage kullanımı yalnız props farkıyla; bileşende bağlam dallanması yok.
