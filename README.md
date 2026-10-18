# Artifact Glitch Studio

Realtime datamosh, tracking HUD a glitch studio pro filmaře a editory. Běží celé v prohlížeči na WebGL (Three.js `EffectComposer` + vlastní shader passy), v jednom souboru [`index.html`](index.html), bez buildu.

## Spuštění

Otevři `index.html` v Chromu, nebo ho pusť přes lokální server (kvůli kameře a ukládání souborů):

```bash
python3 -m http.server 8000
```

Pak otevři <http://localhost:8000>. Po zapnutí GitHub Pages pro tohle repo poběží i na `https://antoninfeldek-bot.github.io/glitch/`.

## Moduly (signální řetězec)

Každý modul jde zapnout, vypnout a přesunout v řetězci.

| Kód | Modul | Co dělá |
| --- | --- | --- |
| M3 | Datamosh | Skutečné vektory pohybu z block matchingu na GPU (jako encoder), akumulace P-framů, bloom, intra refresh, turbulence, melt, malování šmouh, překryv vektorů jako v analyzéru kodeku |
| M2 | Pixel Sort / Blocks | Poškozené makrobloky (plochý blok, posunutý blok, prohozené kanály), mřížka, třídění pixelů podle jasu |
| M1 | RGB Split | Posun kanálů, radiální chromatická aberace, spektrální disperze, fázová vlna, trhání řádků |
| M5 | Color Grade / Crush | Expozice, kontrast, saturace, gamma, rotace odstínu, black crush, bit-crush s ditheringem, posterizace |
| M4 | Scanlines & Noise | CRT řádkování, analogový šum, fázové blikání, chyba chroma fáze, tracking tear, rolování, vinětace, zakřivení |
| M6 | Tracking HUD | Blob/point tracking (pohyb, světla, hrany, stíny), ID, popisky, spojnice, efekty uvnitř boxů, mosh jen na sledovaných objektech |

## Performance

| Ovládání | Akce |
| --- | --- |
| `M` (držet) | Mosh hold: zahodí I-framy, po puštění se obraz rozpouští blok po bloku |
| `N` (držet) | Bloom: zmrazí vektory a opakuje je |
| `I` | Vloží čistý I-frame |
| `Space` (držet) | Flash glitch |
| tah po obraze | Šmouha ve směru tahu |
| `B` | A/B porovnání se zdrojem |
| `R` | Nahrávání |

- **Chaos** spouští náhodné události v náhodných intervalech, nic neběží ve smyčce.
- **Motion react** spustí mosh při náhlém pohybu.
- **On cut** pozná střih ve videu a buď přes něj moshuje, nebo obraz obnoví.

## Vstupy a výstupy

- **Vstupy:** drag-and-drop MP4, WebM, PNG nebo JPG; webkamera; vestavěné demo s náhodnými střihy.
- **Výstup:** nahrávání plátna v reálném čase (`MediaRecorder`, MP4/H.264 nebo WebM) a export snímku do PNG.
- **DaVinci Resolve DCTL:** vygeneruje `.dctl` z aktuálního nastavení (bloky, RGB split, grade, scanlines), všechny parametry jsou v Resolve jako posuvníky. Datamosh, tracking a pixel sort pracují přes víc snímků, takže do DCTL převést nejdou.

Instalace DCTL: soubor dej do `/Library/Application Support/Blackmagic Design/DaVinci Resolve/LUT/` (macOS) nebo `C:\ProgramData\Blackmagic Design\DaVinci Resolve\Support\LUT\` (Windows), pak **Project Settings → Color Management → Update Lists** a použij OpenFX **DCTL** na nodu.

## Výkon

Block matching je nejnáročnější část. Když se náhled na 1080p seká, přepni ho na 720p, nebo zvětši *Macroblock* a zmenši *Search range*.
