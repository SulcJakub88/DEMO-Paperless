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

- **Celý prototyp je JEDEN soubor: `index.html`** (~6000 řádků, vanilla HTML+CSS+JS,
  žádný build, žádné závislosti). Nemusíš číst celý soubor. Postupuj takto:
  1. V tomto CLAUDE.md najdi relevantní stránku / modal / funkci a její kotvu
     (název funkce, `id="…"`, nebo komentář `<!-- PAGE … -->`).
  2. `grep -n` v `index.html` na tu kotvu → zjistíš přesný řádek.
  3. Přečti jen ten úsek (Read s offset/limit) a edituj.
- Assety jsou v `assets/` (lokální PNG). Nikdy neodkazuj na `figma.com/api/mcp/asset/…`
  URL — expirují (404). Nové obrázky z Figmy stáhni přes `download_assets` a ulož lokálně.
- **Preview:** dev server běží na `http://localhost:5500` (statický server nad složkou).
  Spusť/obnov přes preview nástroj s configem `PaperlessDemo` (v `.claude/launch.json`).
  Po editaci vždy `window.location.reload()` a ověř screenshotem/evalem.

## Verzování (důležité — uživatelská konvence)

- Repo: <https://github.com/SulcJakub88/DEMO-Paperless> (větev `main`).
- Když uživatel řekne **„commit"** → `git add -A && git commit && git push`.
  Commit message ve stylu `Verze N — <stručný popis>`, česky, končí
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Rollback = `git checkout <hash> -- .` nebo `git reset --hard <hash>` (po potvrzení).
- Historie: V1 základ, V2 potvrzovací modal, V3 SMS PIN flow, V4 konzistence dat,
  V5 role picker + odmítnutí flow, V6 profil zaměstnance + zobrazit posudek.

## Technologie & konvence

- Vanilla HTML + `<style>` + `<script>` v jednom souboru. Bez frameworků, bez npm.
- **1440px fixed-width** layout. Fonty a barvy inline / v CSS třídách.
- Barvy: primární modrá `#1459bf` / `#0d3380`, růžová (aktivní nav / CTA) `#d92673`,
  zelená (způsobilý / úspěch) `#1a8026`, oranžová (varování) `#d97706`.
- Jazyk UI: čeština. Kód (funkce/proměnné): camelCase, česko-anglicky.
- Texty a rozměry se dělají **1:1 podle Figma** (file key `YjZdOTEj614bbcj8fjxdKK`).
  Uživatel posílá odkazy s `node-id`; přes Figma MCP `get_metadata` / `download_assets`.

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
| page12 | LZ | Ověření zaměstnance (radio: způsob podpisu) |
| page13 | LZ | Rozpracovaná prohlídka + podpis lékaře + podpis zaměstnance |
| page14 | LZ | Historie prohlídek (dynamický 1. řádek) |
| page-zam | FZ | Zaměstnanci (seznam) |
| page-zam-detail | FZ | Profil zaměstnance (Lukáš Motl) |

## Konzistentní demo data (napříč celou aplikací)

Aktivní záznam = **Lukáš Motl**, Vstupní prohlídka, **Česká spořitelna, a.s.**:
- nar. 25.03.1996, Muž, RČ 9603250367, e-mail `lmotl@csas.cz`, os. č. 513343
- pozice „specialista Klientského centra plus", pracoviště
  „Staré nám. 28, Rychnov nad Kněžnou, 516 01", bydliště „Křemenec 69, Konice, 79852"
- IČO ČS 45244782, sídlo „Olbrachtova 1929/62, Praha 4"
- **Lékař = MUDr. Tomáš Závodský** (podpis, certifikát `MUDr_Zavodsky.pfx`).
- Ostatní řádky v tabulkách (Hanuš, Vydrová, Nováková…) jsou „pozadí" — needituj je.
- Pozor: jména zapečená v **obrázcích posudku** (Kohoutová Michaela, David Janko,
  Jitka Vacovská) nelze změnit z kódu — jen výměnou assetu z Figmy.

## Klíčové stavové proměnné (JS)

- `_role` (`fz|lz|kl|zam|all`) + `_roleMeta` — role picker. `setRole(role)` nastaví
  `document.body.className = 'role-'+role`, chip, indikátor a naviguje na výchozí
  stránku role. Viditelnost prvků: CSS `body.role-xx [data-hide~="xx"]{display:none}`
  a `[data-show~="kl"]`. 20+ tlačítek má `data-hide` / `data-show`.
- `_p12RadioSel` (`1` online / `2` SMS PIN / `3` fyzicky) — volba na page12,
  řídí flow na page13. `_p12Phone` — telefon z page12 → propíše se do SMS PIN modalu.
- `_empSignMethod` (`zona|sms|fyzicky|odmitl`) — jak zaměstnanec podepsal; řídí
  badge + tlačítka na page14 (`showPage14()`) a který sloupec ukáže `pdf-view-modal`.

## Hlavní flow (sekvenční přehled)

```
FZ: page1 → (klik řádek) → page2 → [Odeslat digitálně] → page5
                                  → [Odeslat fyzicky]   → page7
page5/7 sidebar: [Přejít do KL] → page3/4 ; [Přejít do LZ] → page6/8
KL page3/4: [Hledat lékaře]→výsledek→ [Přejít do FZ]→page5/7 (status Motla se změní)
            karty typu žádosti přepínají page3↔page4

LZ: page9 → (klik řádek) → page10/11 → [Zahájit prohlídku] → page12
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

Odmítnutí (detail): odmitnuti-modal(poznámka) → [Potvrdit] → svedek-modal
  stav1 formulář (jméno/role/podpis myší canvas) → [Podepsat jako svědek]
  stav2 potvrzení (jméno svědka) → [Dokončit a uložit] → stav3 „Záznam uložen" → [Zavřít]→page14

„Simulovat kompletní online podpis" (sidebar page5 / page14): → page-zam + sim-helper-overlay
  → Notifikace(email-notif) / Eskalace(email-eskal) / Přijetí závěru(prijeti-zaveru-modal)
```

## Přehled modálů/overlayů (id → účel)

- `email-modal-digital` / `email-modal-physical` — CRM e-mail náhled (page2/3/4).
- `p13-confirm-modal` — „Zkontrolujte údaje" před podpisem lékaře.
- `p13-sign-modal` — podpis lékaře digitálním ID (výběr certifikátu → heslo → loading → success).
- `p13-print-modal` — simulace tisku posudku.
- `emp-email-modal` + `emp-modal-footer` — odeslání posudku do zaměstnanecké zóny.
- `emp-mobile-overlay` — zaměstnanecká zóna (mobil): e-mail → náhled posudku → podpis.
- `sms-pin-modal` — asistovaný SMS podpis, 3 stavy (state1 telefon / state2 PIN / state3 úspěch).
- `odmitnuti-modal` — „Zaznamenat odmítnutí podpisu" (poznámka).
- `svedek-modal` — podpis svědka, 3 stavy (formulář / potvrzení / záznam uložen).
- `pdf-view-modal` — 2 sloupce (zóna vs SMS), sloupce se přepínají dle `_empSignMethod`.
- `zam-posudek-modal` — „Zobrazit posudek" na profilu zaměstnance (2 podepsané posudky).
- `posudek-elec-overlay` — fullscreen náhled elektronicky podepsaného posudku.
- `sim-helper-overlay` — spouštěč simulací (notifikace/eskalace/přijetí závěru).
- `email-notif/eskal/potvrzeno/automaticky-modal`, `prijeti-zaveru-modal` — e-mailové šablony.
- `role-picker-overlay` — výběr role (5 karet, zobrazí se při startu).

## Assety (`assets/`)

- `logo.png` — EUC PLS logo (sidebar všech stránek).
- `posudek-doc.png` — nepodepsaný dokument (thumbnail tisku).
- `posudek-lekar.png` / `posudek-signed.png` — posudek podepsaný lékařem.
- `posudek-sms.png` — posudek podepsaný přes SMS.
- `posudek-emp-zona.png` — posudek pro zaměstnaneckou zónu (lékař podepsán, zaměstnanec ne).
- `posudek-elec.png` — elektronicky podepsaný (overlay).
- `zam-posudek1.png` / `zam-posudek2.png` — plně podepsané posudky (profil zaměstnance).

## Známé zvláštnosti / pozor

- Preview proces se ukončí při restartu harnessu (např. přepnutí modelu) —
  znovu ho spusť přes preview `PaperlessDemo`.
- `setPageIndicator` má guard `if(!el)return` (indikátor je na konci DOMu).
- Role picker se otevírá v `DOMContentLoaded` (na konci scriptu).
- Duplicitní `closeEmailModal` v minulosti přepisoval jiný — CRM verze je
  `closeCrmEmailModal`, zaměstnanecká `closeEmailModal`. Nepřejmenovávat zpět.
