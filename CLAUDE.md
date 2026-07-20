# CLAUDE.md — PaperlessDemo

Klikací prototyp („demo") pro EUC PLS – digitalizace pracovnělékařských prohlídek
(pracovnělékařský posudek, elektronický podpis). Slouží k prezentacím klientům
z pohledu různých rolí (HR / lékař / operátor / zaměstnanec).

## Getting started (nový vývojář / nový chat)

Repo je **veřejné**: <https://github.com/SulcJakub88/DEMO-Paperless>

```bash
git clone https://github.com/SulcJakub88/DEMO-Paperless.git
cd DEMO-Paperless
# spuštění náhledu (potřebuje jen Node.js / npx — žádný npm install):
npx serve -p 5500 .
# → otevři http://localhost:5500/index.html
```

Alternativně stačí `index.html` otevřít přímo v prohlížeči (funguje i bez serveru,
protože assety jsou relativní PNG). V Claude Code se preview spouští configem
`PaperlessDemo` z `.claude/launch.json` (statický server na portu 5500).

Jediné závislosti: **Node.js** (kvůli `npx serve`) a prohlížeč. Nic víc.

## Jak s tímto souborem pracovat (pro Claude)

- **NEČTI celý `index.html` ani neprocházej celý repozitář.** Prototyp je jeden
  soubor (~6000 řádků, vanilla HTML+CSS+JS, žádný build). Tento CLAUDE.md je mapa,
  díky které sáhneš rovnou na konkrétní místo a jen tam uděláš změnu. Postupuj takto:
  1. V tomto CLAUDE.md najdi relevantní stránku / modal / funkci a její kotvu
     (název funkce, `id="…"`, nebo komentář `<!-- PAGE … -->`).
  2. `grep -n` v `index.html` na tu kotvu → zjistíš přesný řádek.
  3. Přečti jen ten úsek (Read s offset/limit) a edituj.
  Toto platí i pro jiný Claude Code / jiný účet — díky CLAUDE.md máš plný kontext,
  aniž bys musel číst celý soubor. Celý soubor čti jen tehdy, když opravdu nevíš,
  kde prvek je (a pak kotvu doplň do CLAUDE.md, ať to příště víš).
- Assety jsou v `assets/` (lokální PNG). Nikdy neodkazuj na `figma.com/api/mcp/asset/…`
  URL — expirují (404). Nové obrázky z Figmy stáhni přes `download_assets` a ulož lokálně.
- **Preview:** dev server běží na `http://localhost:5500` (statický server nad složkou).
  Spusť/obnov přes preview nástroj s configem `PaperlessDemo` (v `.claude/launch.json`).
  Po editaci vždy `window.location.reload()` a ověř screenshotem/evalem.

## Verzování (důležité — uživatelská konvence)

- Repo: <https://github.com/SulcJakub88/DEMO-Paperless> (větev `main`).
- Když uživatel řekne **„commit"**:
  1. **Nejdřív aktualizuj tento `CLAUDE.md`**, pokud změna přidala/přesunula
     stránku, modal, funkci, stavovou proměnnou, asset nebo změnila flow —
     uprav příslušnou kotvu/tabulku, ať mapa zůstane přesná.
  2. Pak `git add -A && git commit && git push`.
  Commit message ve stylu `Verze N — <stručný popis>`, česky, končí
  `Co-Authored-By: Claude Code <noreply@anthropic.com>`.
- **Proč:** CLAUDE.md se verzuje spolu s kódem. Když je po každém commitu aktuální,
  má kterýkoli další Claude Code (i pod jiným účtem, na jiném stroji) po `git pull`
  vždy platnou mapu a může sáhnout rovnou na konkrétní místo v `index.html`, aniž by
  četl celý soubor nebo repozitář. Zastaralý CLAUDE.md = ztráta kontextu → udržuj ho.
- Rollback = `git checkout <hash> -- .` nebo `git reset --hard <hash>` (po potvrzení).
- Historie: V1 základ, V2 potvrzovací modal, V3 SMS PIN flow, V4 konzistence dat,
  V5 role picker + odmítnutí flow, V6 profil zaměstnance + zobrazit posudek.

## Technologie & konvence

- Vanilla HTML + `<style>` + `<script>` v jednom souboru. Bez frameworků, bez npm.
- **Fluidní layout** (od V10): `.app` (FZ) a `.lz-app` (LZ) mají `width:100%;
  min-width:1024px` — canvas se roztahuje na celou šířku okna prohlížeče,
  pod 1024px se aktivuje horizontální scroll (`body{overflow-x:auto}`).
  Dřív byl canvas natvrdo `width:1440px` podle Figmy; vnitřní komponenty
  (sidebar 220px fixed, tabulky/karty width:100% svého kontejneru) se
  přizpůsobí automaticky. KL-CRM (page3/4) bylo fluidní odjakživa.
  Fonty a barvy inline / v CSS třídách.
- **Škála písma** (od V11): všechny `font-size` v souboru (v `<style>` i inline)
  škálovány nahoru kvůli čitelnosti na běžných monitorech (originál byl
  navržen na 7–9,5px dle Figmy, na reálné obrazovce nečitelné). Mapování:
  hodnoty ≤14px dostaly +2px, hodnoty 15–18px +1px, nad 18px beze změny
  (nadpisy/čísla už byly dost velké). `.data-table` (FZ) sloupce mají fixní
  `<col>` šířky (`table-layout:fixed`) — ty byly úměrně (~1,2×) rozšířeny,
  jinak by větší písmo víc ořezávalo text (`text-overflow:ellipsis`).
  Když přidáváš nový `font-size`, drž se už zvětšené škály (viz aktuální
  hodnoty v CSS), needěl ji zase zmenšovat na "figma" originál.
  KL-CRM (page3/4, `.crm-*` třídy) do V11 škálování spadalo jen částečně —
  dodatečně zvětšeno (`.crm-wrap` 726→860px, `.crm-title` 19→22px, `.crm-card`/
  `.crm-choice`/`.crm-pref`/`.crm-btn-*`/`.crm-result*` texty a paddingy +1–2px
  resp. ~20 %), ať se prostor na page3/4 využije lépe a obsah je čitelnější.
- Barvy: primární modrá `#1459bf` / `#0d3380`, růžová (aktivní nav / CTA) `#d92673`,
  zelená (způsobilý / úspěch) `#1a8026`, oranžová (varování) `#d97706`.
- Jazyk UI: čeština. Kód (funkce/proměnné): camelCase, česko-anglicky.
- Texty a rozměry se dělají **1:1 podle Figma** (file key `YjZdOTEj614bbcj8fjxdKK`).
  Uživatel posílá odkazy s `node-id`; přes Figma MCP `get_metadata` / `download_assets`.

### Layout & scrolling (celé demo, V7)
- **Fixní chrome, scrolluje jen obsah:** `body`/`.app`/`.lz-app` mají `height:100vh`
  (`body{overflow-y:hidden}`). Na každé stránce zůstává fixní **sidebar**, **záhlaví**
  a **submenu**; scrolluje jen hlavní obsahová oblast. Scrollovací kontejnery:
  `.p1-wrap` (seznamy), `#page2` (ŽÁDOST), `#page-zam-detail` (profil), `.lz-content`
  (nástěnka/rezervace), `.lz-det-wrap` (LZ detail 10–13), `.lz-hist-wrap` (historie 14).
  Záhlaví/submenu drží přes `position:sticky` (submenu s `top:` offsetem pod záhlavím).
- **Scroll odkudkoli:** globální `wheel` handler (konec `<script>`, `SCROLL_SELECTORS`)
  přeposílá kolečko na hlavní obsah i když je kurzor nad sidebarem/záhlavím/submenu;
  respektuje vnořené scrollovatelné prvky a modály (`z-index ≥ 300`).
- **Standardní FZ vzor stránky** (page2 ŽÁDOST, page-zam-detail profil): plné záhlaví
  nahoře (`padding:20px 24px 24px`, `sticky top:0`), pod ním řádek submenu (200px,
  `sticky`) + obsah. Barevné badge ve sloupcích (status/výsledek/závěr) mají jednotnou
  šířku dle nejširšího v sekci.

## Architektura navigace

Multi-page SPA řízená JS. Klíč: `const allPages = [...]` (řádek ~4461).

- `hideAll()` sundá `.active` ze všech stránek + nav + sidebar-wrap.
- `showPageN()` / `showPageZam()` / `showPageZamDetail()` → `hideAll()` + `.active`
  na cílovou stránku + `setPageIndicator(short, full)`.
- **Dvě kategorie stránek:**
  - Uvnitř `.app` divu (sdílí `aside.sidebar` FZ): `page1, page2, page5, page7,
    page-zam, page-zam-detail`.
  - Mimo `.app` (fixed-position, vlastní layout): `page3, page4` (KL-CRM),
    `page6, page8` (LZ nástěnka), `page9`–`page14` (LZ).
- **Sidebar sekce** se přepínají třídou `.visible` na `.sidebar-lz-wrap` /
  `.sidebar-zam-wrap` (default skryté, toggle v `showPage*` a `hideAll`).
- `#page-indicator` (vpravo dole) ukazuje aktuální stránku + prefix role.
  `#role-chip` vedle něj otevírá role picker.
- **LZ menu navigace (`.lz-sidebar`, všech 8 LZ stránek 6,8,9–14):** položky
  NÁSTĚNKA / REZERVACE / HISTORIE PROHLÍDEK jsou funkční na každé LZ stránce
  (`onclick` vede na výchozí stránku sekce — Nástěnka → `showPage6()`/`showPage8()`
  dle `_lzFromFZ`, Rezervace → `showPage9(_lzFromFZ)`, Historie → `showPage14()`).
  `_lzFromFZ` (5=digitální/7=fyzická cesta) nastavují i `showPage6()`/`showPage8()`,
  ne jen `showPage9()`, takže zůstává platné odkudkoli v LZ. Ostatní položky
  (KALENDÁŘE, PODKLADY PRO FAKTURACI, DOKUMENTY A FAQ, MŮJ PROFIL, ODHLÁSIT SE)
  nemají cílovou stránku a zůstávají neaktivní.
- `.lz-sidebar` má `padding-bottom:46px` (stejně jako FZ `.sidebar`), aby
  pomocná tlačítka (`.lz-helper-bar2`, např. „Zpět do FZ") nekolidovala
  s fixním `#bottom-left-bar` (tlačítko „Vše (režisér)" vlevo dole).

## Mapa stránek (id → význam)

| id | zóna | význam |
|----|------|--------|
| page1 | FZ | Rezervace (seznam zaměstnanců, výchozí) |
| page2 | FZ | Vytvořit žádost (→ digitální/fyzická) |
| page3 | KL-CRM | Přiřazení lékaře — online/digitální žádost |
| page4 | KL-CRM | Přiřazení lékaře — offline/fyzická žádost |
| page5 | FZ | Rezervace po **digitálním** odeslání (status Motla) |
| page6 | LZ | Nástěnka (digital) |
| page7 | FZ | Rezervace po **fyzickém** odeslání |
| page8 | LZ | Nástěnka (physical) |
| page9 | LZ | Rezervace (klik na řádek → detail) |
| page10 | LZ | Detail prohlídky — online cesta |
| page11 | LZ | Detail prohlídky — fyzická cesta |
| page12 | LZ | Ověření zaměstnance (radio: způsob podpisu) — vstup jen z **page10** (online cesta) |
| page13 | LZ | Rozpracovaná prohlídka + podpis lékaře + podpis zaměstnance — vstup jen z **page12** |
| page13-fyz | LZ | Rozpracovaná prohlídka — **fyzická žádost**, vstup jen z **page11** (viz níže) |
| page14 | LZ | Historie prohlídek (dynamický 1. řádek) |
| page-zam | FZ | Zaměstnanci (seznam) |
| page-zam-detail | FZ | Profil zaměstnance (Lukáš Motl) |

## Konzistentní demo data (napříč celou aplikací)

Aktivní záznam = **Lukáš Motl**, Vstupní prohlídka, **Zenit Banka, a.s.**:
- nar. 25.03.1996, Muž, RČ 9603250367, e-mail `lmotl@zenitbanka.cz`, os. č. 513343
- pozice „specialista Klientského centra plus", pracoviště
  „Staré nám. 28, Rychnov nad Kněžnou, 516 01", bydliště „Křemenec 69, Konice, 79852"
- IČO 24681357, sídlo „Vinohradská 1877/25, Praha 3"
- **Lékař = MUDr. David Janko** (podpis, certifikát `MUDr.David-Janko.pfx`).
- Ostatní řádky v tabulkách (Hanuš, Vydrová, Nováková…) jsou „pozadí" — needituj je.
- Pozor: text v **obrázcích posudku** (`assets/*posudek*.png`, `assets/zam-posudek*.png`)
  je zapečený v PNG, nejde editovat přes HTML/CSS. Zaměstnanec/žadatel/e-mail (dřív
  Kohoutová Michaela / Státní veterinární správa / marie.vopat10@gmail.com) byly
  přepsány přímo v pixelech (bílý obdélník + dokreslený text stejným fontem/barvou)
  na **Motl Lukáš / Zenit Banka, a.s. / lmotl@zenitbanka.cz**, ať sedí se zbytkem
  dema. Lékař (David Janko) a HR kontakt (Jitka Vacovská) v obrázcích zůstávají.
  Při přidání nového posudek-obrázku z Figmy dej pozor, ať znovu neobsahuje reálné
  jméno/firmu — over stejným postupem (bílý rect + PIL/ImageDraw dokreslení textu).

## Klíčové stavové proměnné (JS)

- `_role` (`fz|lz|kl|zam|all`) + `_roleMeta` — role picker. `setRole(role)` nastaví
  `document.body.className = 'role-'+role`, chip, indikátor a naviguje na výchozí
  stránku role. Viditelnost prvků: CSS `body.role-xx [data-hide~="xx"]{display:none}`
  a `[data-show~="kl"]`. 20+ tlačítek má `data-hide` / `data-show`.
- `_p12RadioSel` (`1` online / `2` SMS PIN / `3` fyzicky) — volba na page12,
  řídí flow na page13. `_p12Phone` — telefon z page12 → propíše se do SMS PIN modalu.
- `_empSignMethod` (`zona|sms|fyzicky|odmitl`) — jak zaměstnanec podepsal; řídí
  badge + tlačítka na page14 (`showPage14()`) a který sloupec ukáže `pdf-view-modal`.
- `_motlDeliveryMethod` (`elektronicky|fyzicky`) — řídí text na profilu zaměstnance
  (`showPageZamDetail()` / `applyMotlDeliveryMethodToDetail()`: "Fyzicky podepsáno"
  vs "Elektronicky podepsáno" + tlačítka Zobrazit/Stáhnout záznam/posudek).
  Zástupci `showPageZamOnlineEmail()/OnlineSms()/Fyzicky()` nastavují **oba**
  `_empSignMethod` i `_motlDeliveryMethod` zároveň, aby se seznam Zaměstnanci
  i detail profilu neshodly (dřív nastavovaly jen `_empSignMethod` → klik na
  řádek Motla v „FZ Zaměstnanci (fyzicky)" ukazoval chybně elektronicky podepsaný profil).
  `showPageZam()` navíc: (1) zapisuje aktuální variantu i do krátkého labelu
  `#page-indicator` (ne jen do hover tooltipu) — např. „Page ZAM (fyzicky)"; (2)
  zvýrazní `.active` odpovídající tlačítko v „Pomocné akce" flyoutu
  (`#zamflyout-zona`/`-sms`/`-fyzicky`, sidebar `#sidebar-zam-wrap`), ať je na
  první pohled vidět, na jaké variantě stránky ZAMĚSTNANCI se právě nacházíme;
  (3) schová `#zamflyout-potvrzeno`/`-automaticky` (náhledy e-mailových
  notifikací) na variantě "fyzicky" — posudek se tam nedoručuje e-mailem.
  Flyout je vizuálně rozdělen do 2 skupin popiskem `.helper-flyout-label`
  + `.helper-flyout-divider`: „ZOBRAZIT VARIANTU" (3 přepínače, vždy) a
  „NÁHLED E-MAILU" (2 náhledy, schované celé na "fyzicky" spolu s dividerem).
  **`#sidebar-zam-wrap` je sdílený mezi page-zam (seznam) i page-zam-detail
  (profil)** — obsah „Pomocné akce" flyoutu se proto při každém vstupu
  přestavuje přes `renderZamFlyoutList()` (viz výše) nebo
  `renderZamFlyoutDetail()` (všechny 3 varianty fyzicky/online-email/
  online-sms, `.active` na té aktuální — stejný vizuální vzor jako u
  `renderZamFlyoutList()`), onclick `showPageZamDetailOnlineEmail()/OnlineSms()/
  Fyzicky()`, které nastaví `_empSignMethod`+`_motlDeliveryMethod` a zůstanou
  na profilu.
- **„Pomocné akce" flyouty se skupinovým popiskem** (`.helper-flyout-label` +
  `.helper-flyout-divider`, stejný vzor napříč FZ i LZ) — používej vždy, když
  flyout obsahuje ≥2 logicky odlišné kategorie akcí; pro flyouty s jen 1
  homogenní skupinou (LZ page10–13/13-fyz mají jediné tlačítko „← Zpět do FZ")
  se popisky nepřidávají. Page9 (LZ Rezervace) nemá „Pomocné akce" vůbec —
  odstraněno. Page6/8 (LZ Nástěnka) mají tlačítko „Přejít do FZ" →
  `showPage1()` (FZ Rezervace) — dřív „← Zpět do FZ (Rezervace)" vedlo zpět
  na page5/7 (konkrétní žádost), teď jde vždy na page1 (seznam).
  - `#sidebar-lz-wrap` (FZ page1/2/5/7): „Navigace" (přepnout žádost, Přejít
    do KL/LZ) → „Simulace podpisu" (3 tlačítka) → „Náhled e-mailu" (2 tlačítka).
    Statické skupiny — na page5/7 je vždy aspoň 1 položka viditelná v každé
    skupině, takže se nikdy nemusí schovávat celý label/divider.
  - `#page14` helper-flyout: „Stav prohlídky" (5 přepínačů demo-stavu,
    `updatePage14HelperFlyout()` zvýrazní aktuální) → „Přejít do FZ
    Zaměstnanci" (4 kontextové zástupce, vždy právě 1 viditelný dle
    `_page14State`) → „Náhled posudku" (`#p14flyout-posudek-zona`/`-sms` +
    label/divider, celá skupina viditelná ve stavech `email`/`sms`, uvnitř
    vždy jen 1 tlačítko dle stavu — „...via zaměstnanecká zóna" nebo
    „...via SMS" otevře `zam-posudek-modal` přes `openZamPosudekModalZona()`/
    `Sms()`, které natvrdo zobrazí příslušný sloupec bez ohledu na
    `_empSignMethod`).

## Hlavní flow (sekvenční přehled)

```
FZ: page1 → (klik řádek) → page2 → [Odeslat digitálně] → page5
                                  → [Odeslat fyzicky]   → page7
page1 „Pomocné akce" (`#sidebar-p1-wrap`, vlastní/samostatný — nesdílí obsah
            s `#sidebar-lz-wrap`): jediné tlačítko [🏥 Přejít do LZ →] → showPage6()
            (LZ Nástěnka, digitální žádost). Viditelnost řídí showPage1()/hideAll(),
            stejný vzor jako `#sidebar-zam-wrap`.
page5/7 sidebar (režisér): [Přepnout na fyzickou/digitální žádost] přepíná page5↔page7
            (mění status Motla; řádky `#p5-motl-row`/`#p7-motl-row`, hover + zebra bez
            trvalého podbarvení); [Náhled e-mailu — online/tisk] → showEmailModal(digital|physical)
            (page5 obě tlačítka, page7 jen tisk); [Přejít do KL] → page3/4 ; [Přejít do LZ] → page6/8
KL page3/4: [Hledat lékaře]→výsledek→ [Přejít do FZ]→page5/7 (status Motla se změní)
            karty typu žádosti přepínají page3↔page4
            `.crm-kl-helper-bar` (page3/4): fixní stack nad `#bottom-left-bar`, vždy
            [← Zpět do FZ] (showPage5()/showPage7()); pro role kl/all navíc nad ním
            [Přepnout na fyzickou/digitální žádost] (`data-show="kl all"`). Pozici
            (`bottom`) dopočítává JS `positionCrmHelperBar()` (voláno ze
            `setPageIndicator()`) podle reálné výšky role-chipu (1 vs 2 řádky textu),
            aby se stack nikdy nepřekryl s tlačítkem role vlevo dole.
            `showResult3()/showResult4()` po nalezení lékaře volají
            `collapseCrmPreResultSections('p3'|'p4')` — sbalí karty „TYP PŘÍCHOZÍ
            ŽÁDOSTI"/„PREFERENCE KLIENTA" (`#p3-typ-bar/-card`, `#p3-pref-bar/-card`,
            obdoba pro p4) a schová druhé AKCE tlačítko (`#btn-hledat3-alt`/`-4-alt`),
            ať se celý výsledek (`SOUHRN DOPADU` v `.crm-ir-grid`, 3 sloupce místo
            stackování) vejde na výšku obrazovky bez scrollu (cíl: MacBook 13",
            ~1280×800/720). `showPage3Digital()/Physical()` volají opak —
            `restoreCrmPreResultSections()` — při novém vstupu na stránku.

LZ: page9 → (klik řádek) → page10 (online cesta) NEBO page11 (fyzická cesta)

page10 → [Zahájit prohlídku] → page12
page12: vyplň ID+e-mail, vyber radio (1/2/3), (radio2 → editovatelný telefon)
        → [Pokračovat] → page13
page13: vyplň výsledek → [Uzavřít prohlídku] → zobrazí „Podpis lékaře"
        → [Digitálně podepsat] → p13-confirm-modal → p13-sign-modal → podpis
        → sekce „Podpis zaměstnance" (4 volby):
          • Zaslat na e-mail  → emp-email-modal → (emp-mobile-overlay) → _empSignMethod='zona'
          • Zaslat PIN na telefon → sms-pin-modal (3 stavy: telefon→PIN→úspěch) → 'sms'
          • Vytisknout → p13-print-modal → 'fyzicky'
          • Odmítnutí → odmitnuti-modal → svedek-modal(3 stavy) → 'odmitl'
        každý flow končí → showPage14() (badge dle _empSignMethod)

page11 → [Zahájit prohlídku] → page13-fyz (PŘESKAKUJE page12/page13 — žádná volba
        způsobu podpisu, jde rovnou o fyzickou žádost)
page13-fyz: kopie page13 (bez sekce podpis lékaře/zaměstnance, levé submenu bez
        tlačítka „Uzavřít prohlídku"; „Zpět na rezervace" → showPage11())
        vyplň výsledek → [Uzavřít prohlídku] → p13fyz-confirm-modal
        („⚠️ Zkontrolujte zadané údaje" + Upravit prohlídku / Pokračovat k podpisu)
        → [Pokračovat k podpisu] → showPage14() (rovnou, žádný podpis lékaře/zaměstnance)

Odmítnutí (detail): odmitnuti-modal(poznámka) → [Potvrdit] → svedek-modal
  stav1 formulář (jméno/role/podpis myší canvas) → [Podepsat jako svědek]
  stav2 potvrzení (jméno svědka) → [Dokončit a uložit] → stav3 „Záznam uložen" → [Zavřít]→page14

„Simulovat kompletní online podpis" (sidebar page5 / page14): → page-zam + sim-helper-overlay
  → Notifikace(email-notif) / Eskalace(email-eskal) / Přijetí závěru(prijeti-zaveru-modal)
```

## Přehled modálů/overlayů (id → účel)

- `email-modal-digital` / `email-modal-physical` — CRM e-mail náhled (page2/3/4).
- `p13-confirm-modal` — „Zkontrolujte údaje" před podpisem lékaře (page13).
- `p13fyz-confirm-modal` — obdoba `p13-confirm-modal` pro **page13-fyz**; tlačítko
  „Pokračovat k podpisu" ale vede rovnou na `showPage14()` (žádný podpis lékaře).
- `p13-sign-modal` — podpis lékaře digitálním ID (výběr certifikátu → heslo → loading → success).
- `p13-print-modal` — simulace tisku posudku.
- `emp-email-modal` + `emp-modal-footer` — odeslání posudku do zaměstnanecké zóny.
- `emp-mobile-overlay` — zaměstnanecká zóna (mobil): e-mail → náhled posudku → podpis.
- `sms-pin-modal` — asistovaný SMS podpis, 3 stavy (state1 telefon / state2 PIN / state3 úspěch).
- `odmitnuti-modal` — „Zaznamenat odmítnutí podpisu" (poznámka).
- `svedek-modal` — podpis svědka, 3 stavy (formulář / potvrzení / záznam uložen).
- `pdf-view-modal` — 1–2 sloupce (zóna vs SMS), sloupce se zobrazují/skrývají
  dle `_empSignMethod` (`odmitl`/`fyzicky` → oba viditelné). `openPdfViewModal()`
  před mountem resetuje inline width/flex sloupců a dialogu (jinak by `mountPdoc()`
  počítal scale proti zúženému sloupci z minulého otevření), pak zavolá
  `mountPdoc(..., maxHeight)` (`pdfViewModalMaxHeight()` dopočítá dostupnou výšku
  z `window.innerHeight`, ať se posudek vejde bez scrollu) a nakonec
  `fitPdfViewDialogToContent()` — zúží dialog + viditelné sloupce přesně na
  šířku vykresleného posudku, ať kolem dokumentu nezůstává prázdný prostor.
- `zam-posudek-modal` — „Zobrazit posudek" na profilu zaměstnance (2 podepsané posudky).
- `posudek-elec-overlay` — fullscreen náhled elektronicky podepsaného posudku.
- `sim-helper-overlay` — spouštěč simulací: notifikace (digital/fyzická), eskalace,
  „Potvrzení seznámení se závěrem po přihlášení do FZ" (dřív „Přijetí závěru ve FZ").
- `email-notif/eskal/potvrzeno/automaticky-modal`, `prijeti-zaveru-modal` — e-mailové šablony.
  `prijeti-zaveru-modal` (=„Důležité upozornění") má 1. řádek Motl Lukáš. Sloupec
  „Způsob doručení posudku" → tag „Elektronicky podepsáno"
  (`onclick="openZamPosudekModalFromPrijeti()"`) otevře `zam-posudek-modal`
  (viz PDOC výše, varianta `zam-zona-blank`/`zam-sms-blank` dle `_empSignMethod`
  — poslední řádek DATUM PŘEVZETÍ/JMÉNO A PŘÍJMENÍ/PODPIS je zde záměrně prázdný,
  na rozdíl od `zam-zona`/`zam-sms` použitých v `openZamPosudekModal()` (FZ profil
  zaměstnance „Zobrazit posudek") a `openZamPosudekModalZona()`/`Sms()` (LZ
  page14 „Náhled posudku"), kde je vyplněný Jitka Vacovská + zelený podpisový
  text) — musí mít `z-index:970` (spolu s `#zam-zaznam-modal`), jinak
  zůstane schovaný za `prijeti-zaveru-modal` (z-index 955).
- `email-notif-physical-modal` — notifikace o **fyzickém** doručení posudku (layout dle 421-2,
  předmět „Fyzické doručení posudku", data Lukáš Motl / Zenit Banka); open/closeEmailNotifPhysicalModal().
- `role-picker-overlay` — výběr role (5 karet, zobrazí se při startu).

## Posudek jako HTML (PDOC) — náhrada za assets/*posudek*.png

Od V12 je "LÉKAŘSKÝ POSUDEK" dokument (dřív 9 samostatných PNG screenshotů)
vykreslován jako živé HTML/CSS (ostrý text, žádná pixelace při zoomu).
Kód: CSS `.pdoc-*` třídy (~řádek 2017+, hned za `.posudek-elec-img`),
JS `PDOC_VARIANTS` / `pdocHtml()` / `mountPdoc()` (~řádek 5410+, před
`_empSignMethod`). Obsah/texty (Motl Lukáš, Zenit Banka a.s., …) i vizuál
(tabulka, checkboxy, čárový kód, zelený podpisový text) jsou 1:1 podle
původních obrázků — jen ostřejší.

**Jak to funguje:** `mountPdoc(containerId, variantKey, keepFixedHeight?, maxHeight?)`
vloží do `#containerId` vykreslený dokument v přirozené šířce 860px
(`PDOC_NATURAL_WIDTH`) a přeškáluje ho přes `zoom` na šířku kontejneru
(`el.clientWidth / 860`). Kontejner musí mít třídu `pdoc-mount` (zajišťuje
`overflow:hidden` + `min-width:0` + `flex-shrink:0`, nutné uvnitř flex
rodičů typu `.pdf-view-col` / `.p13-sign-preview`, jinak by se obsah
nezmenšil pod svou přirozenou šířku). Volitelný 3. parametr
`keepFixedHeight=true` řekne `mountPdoc()`, ať nepřepisuje inline výšku
kontejneru (pro kontejnery s vlastní pevnou výškou z CSS). Volitelný 4.
parametr `maxHeight` (px) dodatečně omezí scale (vezme se `min(šířkový
scale, maxHeight/přirozená výška)`), takže se dokument zmenší tak, aby
se vešel na výšku bez scrollování — používá `openPdfViewModal()` (viz níže).
⚠️ Výšku kontejneru čti přes `inner.getBoundingClientRect().height`, ne
`scrollHeight` — `scrollHeight` zoomovaného elementu neodráží jeho vlastní
`zoom` (zůstává v "lokálních" jednotkách), takže by dal špatnou hodnotu
při scale výrazně odlišném od 1.

**7 variant** (`PDOC_VARIANTS` klíč → dřívější PNG → kde se používá):
| klíč | dřívější PNG | stav | kde se používá |
|---|---|---|---|
| `unsigned` | `posudek-doc.png` | nepodepsáno, žádná pole vyplněná | `p13-print-modal` (`#pdoc-mount-print`), `p13-sign-modal` (`#pdoc-mount-sign`) |
| `lekar` | `posudek-lekar.png` | podepsáno lékařem, PŘEVZETÍ prázdné (bez e-mail pole) | `sms-pin-modal` (`#pdoc-mount-sms-pin`) |
| `emp-zona` | `posudek-emp-zona.png` | podepsáno lékařem, e-mail vyplněn, PODPIS zaměstnance čeká (prázdný) | `emp-mobile-overlay` sign view (`#pdoc-mount-empzona`) |
| `signed-zona` | `posudek-elec.png` / `posudek-signed.png` | podepsáno zaměstnancem via zóna (klikyhák podpis), ŽADATEL řádek prázdný | `posudek-elec-overlay` (`#posudek-elec-img`, zona), `pdf-view-modal` (`#pdoc-mount-pdfview-zona`) |
| `signed-sms` | `posudek-elec-sms.png` / `posudek-sms.png` | podepsáno přes SMS (jen zelený text, bez klikyháku), ŽADATEL řádek prázdný | `posudek-elec-overlay` (sms), `pdf-view-modal` (`#pdoc-mount-pdfview-sms`) |
| `zam-zona` | `zam-posudek1.png` | plně uzavřeno: podpis zaměstnance (klikyhák) + druhý řádek vyplněný (Jitka Vacovská, bez ŽADATEL/ANO-NE děliče) | `zam-posudek-modal` (`#pdoc-mount-zam-zona`) |
| `zam-sms` | `zam-posudek2.png` | jako `zam-zona`, ale SMS podpis (bez klikyháku) | `zam-posudek-modal` (`#pdoc-mount-zam-sms`) |

Pokud potřebuješ v budoucnu upravit **jen jednu konkrétní variantu** (např.
"u lekar chci jiný text"), uprav `pdocHtml()`/`PDOC_VARIANTS` podmíněně dle
`key` — je to jediné místo, kde se markup generuje pro všech 7 variant
(sdílí stejnou hlavičku/tabulku rizik/posudkovou část, liší se jen
sekce PŘEVZETÍ + stav podpisu lékaře).

⚠️ **Neověřeno vizuálně:** `posudek-elec-overlay` (`signed-zona`/`signed-sms`
přes `openPosudekElecOverlay()`) — v testovacím prostředí se u tohoto
jednoho místa (fixed overlay, flex-centrovaný, bez zanoření do dialogu)
nepodařilo spolehlivě ověřit screenshotem, i když `getBoundingClientRect()`
potvrzuje správnou 468px šířku a obsah je stejný kód jako ve zbylých 8
funkčních místech. Při příští práci na této obrazovce zkontroluj vizuálně
v reálném prohlížeči.

Staré PNG (`assets/*posudek*.png`, `assets/zam-posudek*.png`) už se nikde
neodkazují — ponechány v repozitáři pro referenci/rollback, ale nejsou
součástí žádné stránky.

## Assety (`assets/`)

- `logo.png` — EUC PLS logo (sidebar všech stránek).
- (posudek obrázky nahrazeny HTML — viz "Posudek jako HTML (PDOC)" výše)

## Známé zvláštnosti / pozor

- Preview proces se ukončí při restartu harnessu (např. přepnutí modelu) —
  znovu ho spusť přes preview `PaperlessDemo`.
- `setPageIndicator` má guard `if(!el)return` (indikátor je na konci DOMu).
- Role picker se otevírá v `DOMContentLoaded` (na konci scriptu).
- Duplicitní `closeEmailModal` v minulosti přepisoval jiný — CRM verze je
  `closeCrmEmailModal`, zaměstnanecká `closeEmailModal`. Nepřejmenovávat zpět.
