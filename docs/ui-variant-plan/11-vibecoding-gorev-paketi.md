# 11 — Vibecoding Görev Paketi (Codex Orkestrasyon + Claude Worker Dalgaları)

Bu doküman, custom AEP renderer'ın Storybook'ta headless olarak inşasını çok-ajanlı
(pane'lerde paralel Claude worker) yürütecek orkestratör için bağlayıcı görev paketidir:
temel yapı sırası, dalga planı, worker görev şablonu ve birleştirme kuralları.
Spesifikasyon kaynağı bu dosya DEĞİLDİR — worker'lar değerleri
[01](./01-varyant-cercevesi.md), [03](./03-form-varyantlari.md),
[04](./04-table-varyantlari.md), [05](./05-bilesen-varyantlari.md) ve
[10](./10-frontend-katman-mimarisi.md)'dan okur.

Bağlantılı dosyalar: [00-genel-plan.md](./00-genel-plan.md) ·
[07-storybook-mcp-promptlari.md](./07-storybook-mcp-promptlari.md) ·
[10-frontend-katman-mimarisi.md](./10-frontend-katman-mimarisi.md)

## 1. Kapsam düzeltmesi (yanlış anlaşılmasın)

- Hedef **AntD'yi yeniden yazmak değildir**. AntD, default CRUD renderer olarak kalır
  (MK-4). İnşa edilen şey **custom AEP renderer'ın bileşen kütüphanesidir**: davranış
  R5 sahibinden (React Aria / TanStack), görünüm R1 token'larından gelir (MK-5).
- "Temel bileşenler" listesindeki shadows / radius / color scheme **bileşen değil
  token'dır** (R1); layout / 320px / adaptive-fluid **bileşen değil R3 katmanıdır**.
  Bunlar butondan ÖNCE biter — aksi halde her bileşen iki kez yazılır.

## 1.5 W−1 — Foundation Contract kapısı (HER ŞEYDEN ÖNCE)

Gerçek durum: bu repo şu an yalnız spesifikasyon içerir — `package.json`, `src/`,
`.storybook` YOKTUR. Var olmayan workspace'e writer açılmaz. W0'dan önce W−1 kapısı
tamamlanır; W−1 **read-only**'dir, tek satır ürün kodu yazılmaz.

**Tek bloklayıcı sahip kararı — implementation workspace:**

- **Öneri (A):** bu repo (`karaca`) monorepo'ya genişletilir — spec ile kod aynı
  repoda kalır, CI kapıları spec'e karşı çalışır, senkron sorunu doğmaz:

  ```text
  docs/ui-variant-plan/   (mevcut spec — dokunulmaz)
  apps/storybook/         apps/admin-demo/
  packages/tokens/  foundations/  layout/  behaviors/
  packages/renderer-antd/  renderer-aep/  charts/  testing/
  ```

- Alternatif (B): ayrı implementation reposu + spec senkronu (ek koordinasyon maliyeti).

Karar İsmail'indir; verilmeden hiçbir writer kod yazmaz (fail-closed).

**W−1 read-only analiz worker'ları (5 pane):**

| Analist | Görev |
|---|---|
| repo-analyst | Yerel `frontend/claudeui` dahil mevcut kod varlığını envanterle (package.json, React/Vite/TS/Storybook sürümleri, package manager); workspace kararı için A/B kanıtı topla |
| legacy-analyst | [12-legacy-figma-gecis-haritasi.md](./12-legacy-figma-gecis-haritasi.md) taslağını eski Figma dosyasına karşı doğrula/tamamla; UNKNOWN kalanları sahibe listele |
| token-analyst | DTCG-biçimli tokens JSON'u **tek doğruluk kaynağı** olarak kurgula; akış TEK YÖN: JSON → Style Dictionary → CSS vars / TS tipleri / AntD ThemeConfig / ECharts theme / Storybook docs / Figma Variables. Figma'da yapılan değişiklik JSON'a PR olarak döner. Koleksiyon ADLARI SABİTTİR: primitive / semantic / density / variant-overlay ([01]/[06]); `comp.*` component-token seviyesi yalnız gerçekten gerektiğinde açılır. Henüz değer dosyası yazma |
| contract-analyst | Capability manifest: AntD ↔ AEP bileşen eşlemesi **kabiliyet paritesi** olarak (AntD props API'si KOPYALANMAZ); davranış sahipleri; ilk pilot kümesi |
| qa-analyst | Kabul kapıları: 320 min viewport sözleşmesi, container-query, RTL, klavye, **axe yeşil + klavye-tam play + manuel screen-reader/reflow turu** (axe tek başına a11y kanıtı değildir), görsel regresyon, performans |

**Çıktı:** tek Foundation Contract dokümanı (workspace, package sınırları, CSS @layer
sırası + scope politikası, token akışı, legacy matrisi, Storybook bilgi mimarisi,
capability manifest, pilot sırası, kabul kriterleri) — donar, hash'lenir. STOP GATE:
bu donmadan W0 açılmaz.

## 2. Temel yapı sırası (bağımlılık sıralı)

| Sıra | İş | Katman | Neden bu sırada |
|---|---|---|---|
| 0.1 | Token seti: renk (primitive + semantic light/dark), radius ölçeği, shadow token'ları, spacing, tipografi, motion, density, variant-overlay; Style Dictionary → CSS vars + AntD ThemeConfig adapter | R1 | Her şeyin girdisi; hex/px başka hiçbir yerde yazılmaz |
| 0.2 | CSS temeli: reset, Roboto self-host + Noto fallback, logical properties (RTL), X5 taban kuralları (`:where()` focus ring, hover-guard, `:user-invalid`), reduced-motion | R2 | Bileşenlerin üzerine oturduğu zemin |
| 0.3 | Layout sistemi: 320px-first, container-query, Stack/Flex/Grid/Split, breakpoint bantları | R3 | Adaptive-fluid davranış bileşenden önce kanıtlanır |
| 0.4 | Storybook altyapısı: globalTypes decorator (theme × density × variant), viewport preset'leri, MSW, a11y addon, test-runner | — | Worker'ların ortak test zemini ([07] Prompt 1) |
| 1.1 | Görsel primitive'ler: Box, Text, Icon (Phosphor), VisuallyHidden, Portal | R4 | Ürün anlamı taşımayan yapı taşları |
| 1.2 | Davranış substratı: React Aria kurulumu (form/overlay aileleri), TanStack Table+Virtual (grid); sahiplik tablosu | R5 | MK-2: aile başına tek sahip; kopya davranış yasak |
| 2.1 | **Button + IconButton** | R6 | En az bağımlılık; X5 StateMatrix ilk burada kanıtlanır |
| 2.2 | **Field çatısı + TextField** (Label/Hint/Error anatomisi) | R6 | Form ailesinin çekirdeği ([03] validation modeli) |
| 2.3 | Checkbox / Radio / Switch | R6 | Basit seçim kontrolleri |
| 2.4 | Badge / Tag / Status (kapsül) | R6 | Tablo hücrelerinin ön koşulu |
| 2.5 | **Select / Dropdown / Menu** — bilinçli olarak SONRA | R6 | Portal + positioning + dismiss + typeahead ister; overlay makinesi 1.2'de hazır olmalı |
| 2.6 | Tabs, Toolbar | R6 | Kompozisyona hazırlık |
| KAPI | **MK-16 golden slice**: liste/DataGrid + URL state + form/drawer + 5 durum + tr/ar-RTL + 3 density, aynı SurfaceDefinition ile AntD + AEP | — | Bu kapı geçilmeden geniş üretime (04/05'in kalanı, dashboard) girilmez |

Dropdown'ın "temel bileşen" değil 2.5 olduğuna dikkat: overlay altyapısı olmadan erken
yazılan Select, en sık baştan yazılan bileşendir.

## 3. Dalga planı (pane paralelliği)

Dalga içi worker'lar paralel; dalgalar arası sıralıdır. Dosya sınırları çakışmaz.

| Dalga | Worker | İş | Dosya sınırı |
|---|---|---|---|
| **W−1** | 5 analist (§1.5) | Foundation Contract — read-only | Dosya yazmaz; rapor üretir |
| W0 | w0-tokens | 0.1 | `packages/tokens/**`, `style-dictionary.config.*` |
| W0 | w0-cssbase | 0.2 (0.1'in token ADLARINA karşı yazar, değerleri beklemez) | `src/styles/**` |
| W0 | w0-layout | 0.3 | `src/layout/**` |
| W0 | w0-storybook | 0.4 | `.storybook/**`, `src/testing/**` |
| W1 | w1-primitives | 1.1 | `src/primitives/**` |
| W1 | w1-behavior | 1.2 | `src/behavior/**` + sahiplik tablosu PR'ı |
| W2 | w2-button | 2.1 | `src/components/button/**` |
| W2 | w2-field | 2.2 | `src/components/field/**` |
| W2 | w2-selection | 2.3–2.4 | `src/components/{checkbox,radio,switch,badge}/**` |
| W3 | w3-overlay | 2.5 | `src/components/{select,menu}/**` |
| W3 | w3-nav | 2.6 | `src/components/{tabs,toolbar}/**` |
| W4 | w4-goldenslice | MK-16 dilimi | `src/surfaces/**`, `src/renderers/**` |
| W5 | w5-ai-command-card | [13](./13-ai-command-card-spec.md) spesifikasyonu (post-golden-slice) | `packages/renderer-aep/src/patterns/ai-command-card/**` |

Birleştirme kuralı: her worker kendi branch'inde çalışır, PR açar; orkestratör dalga
sonunda sırayla merge eder (w0-tokens her zaman ilk). Merge kapısı = story + play +
axe yeşil + token drift temiz.

## 3.5 Storybook bilgi mimarisi (ilk kümeler — Button İLK DEĞİLDİR)

| Küme | İçerik | Kapı |
|---|---|---|
| **00 Foundations** | Token docs sayfaları (JSON kaynağından otomatik): renk primitive'leri, semantic renkler + kontrast oranları, tipografi, spacing, sizing/density, border, radius, elevation, motion, ikonografi, focus ring, tema'lar, legacy geçiş haritası ([12]) | Light/dark değerleri ve primitive→semantic alias ilişkisi görünür; hardcode değer yok |
| **01 Layout Lab** | Container, Box, Stack, Inline, Grid, AutoGrid, Split, SidebarLayout, StickyRegion, ScrollArea, SafeArea | Her story 320→1440 kesintisiz resize + LTR/RTL + uzun içerik + overflow + zoom/reflow |
| **02 State & A11y Lab** | X5 gramerinin görsel sözleşmesi: HoverPointerGuard, FocusVisible, DisabledVsReadOnly, CheckedIndeterminate, SelectedPressedCurrent, ExpandedCollapsed, ValidInvalidUserInvalid, LoadingBusy, ReducedMotion | Native pseudo / ARIA state / headless `data-*` / ürün durumu ayrımı görünür ve test edilebilir |
| **03 Primitives** | Text, Heading, Icon, Divider, Surface, VisuallyHidden, Skeleton, Spinner. **Portal burada DEĞİL** — görsel primitive değil, R5 overlay iç altyapısıdır | Story + axe |
| **04 Pilotlar** | Button, IconButton, Link, FormField, TextField, Checkbox, Switch, Badge/Status — **A–F pilotu yalnız burada başlar** | StateMatrix + form anatomisi + theme/density + 320px dört sözleşmeyi kanıtlar |
| **05 Overlay** | Tooltip, Popover, Menu, Select, Combobox, Dialog, Drawer | Overlay makinesi (Portal/positioning/dismiss) W1'den |
| **06 Veri + master | Card, Table, DataGrid, Pagination, Toolbar, FilterBar, ChartBlock, AppShell, ListPage/FormPage/DetailPage/Dashboard master'ları | MK-16 golden slice sonrası |

Kural: 00–02 kümeleri görünür ve test edilebilir olmadan 04 açılmaz.

**A–F rollout kuralı (netleştirme):** A–F eksen DEĞERLERİ zaten [01]'de spesifiye
edilmiştir — worker değer İCAT ETMEZ. Erken yapılmayacak olan şey implementasyon
kapsamıdır: variant-overlay CSS'i önce 04 pilot bileşenlerinde (Button/Field/Surface)
kanıtlanır, sonra genişletilir. Tam varyant×tema×density×state matrisi nightly koşulur;
PR'da budanmış/pairwise küme ([07] §4).

**CSS mimarisi (W0'da, ilk bileşenden önce):** `@layer reset, tokens, base, layout,
components, utilities, overrides` sırası; `:where()` taban specificity politikası;
scope attribute'ları `[data-theme] [data-density] [data-variant] [data-renderer]`;
AntD reset izolasyonu; portal container + z-index politikası; logical properties
zorunluluğu; global selector yasağı.

## 4. Worker görev şablonu (her pane'e verilen prompt iskeleti)

```text
ROLE: Component engineer for the AEP custom renderer (headless + token-driven).

BINDING DOCS (read before writing any code, in this order):
docs/ui-variant-plan/10-frontend-katman-mimarisi.md  (layers, MK decisions, X5)
docs/ui-variant-plan/01-varyant-cercevesi.md         (invariants, A-F axes)
<component-specific file: 03 forms / 04 table / 05 supporting>
docs/ui-variant-plan/07-storybook-mcp-promptlari.md  (story/test format)

TASK: <one wave item, e.g. "Build Button + IconButton (2.1)">
FILE BOUNDARY: <paths> — do not touch files outside this boundary.

HARD RULES:
- No hardcoded hex/px: consume CSS variables from src/tokens only.
- Component is variant-blind: reads data-variant tokens, never branches on a letter.
- Behavior comes ONLY from the family owner (React Aria / TanStack); do not
  reimplement keyboard/focus/dismiss logic.
- X5 grammar: state styling targets the owner's data-*/aria-* attributes;
  hover only under @media (hover:hover) and (pointer:fine);
  validation via :user-invalid/touched; focus-visible is never suppressed.
- Typography >= 1rem, weights 400/500/700; touch targets >= 44px;
- 320px-first: the component must work at 320px before any wider layout.
- Turkish UI text via i18n keys, never literals.

DEFINITION OF DONE:
- Stories: Default + all states + StateMatrix (variant x theme x density x state).
- play tests: keyboard-complete interaction; axe: zero violations (fail-blocking).
- RTL story renders correctly; long-content (de) story does not break layout.
- No imports from antd, react-router, @tanstack/react-query inside the component.
Report: what you built, what you could not verify, any spec ambiguity found
(do NOT silently resolve spec conflicts — 01/10 win; list them in the PR).
```

## 5. Orkestratör (Codex) kuralları

0. **W−1 tamamlanıp Foundation Contract donmadan writer açma (fail-closed).**
   Implementation workspace kararı (§1.5 A/B) sahibden gelmeden kod yazılmaz.
1. Önce yerel ağacı kanonik repo ile senkronla (`karacaismail/karaca`,
   branch `claude/ea-platform-ui-design-u6ew78`); spesifikasyonu yerel kopyadan değil
   repodan oku. Yerel-repo farkı görürsen ÜRETİME BAŞLAMADAN raporla.
2. Kod yazma; görev paketle, worker'lara dağıt, PR'ları dalga sırasıyla merge et.
3. Worker çıktısındaki "spec ambiguity" raporlarını topla; 01/10 ile çöz,
   çözülemeyeni sahibine (İsmail) tek listede sor — worker'a sordurma.
4. Dalga atlama yasak: W2, W0+W1 merge edilmeden açılmaz; MK-16 kapısı geçilmeden
   W4 sonrası geniş üretim yok.
5. Her dalga sonunda Storybook build + test-runner + axe koş; kırık ana dal bırakma.

## Kabul kriterleri

- [ ] W0 tamam: token'lar tek kaynak, Storybook decorator üçlüsü çalışıyor, 320px viewport preset'leri hazır.
- [ ] W1 tamam: sahiplik tablosu PR'ı merge; primitive'lerde ürün anlamı yok.
- [ ] W2'deki her bileşen: StateMatrix + play + axe yeşil; varyant-kör; AntD/Router/Query importu yok.
- [ ] Select/Menu, overlay makinesi (Portal + positioning + dismiss) W1'den gelmeden merge edilmedi.
- [ ] MK-16 golden slice iki renderer'da yeşil olmadan W4 sonrası iş açılmadı.
- [ ] Worker'ların bildirdiği spec çelişkileri tek listede toplandı ve 01/10 öncelik kuralıyla kapatıldı.
