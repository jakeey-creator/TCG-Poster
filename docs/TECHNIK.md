# Technik

Wie Kartenposter 10×15 aufgebaut ist und wie man es erweitert.

> Privates Fanprojekt ohne Verbindung zu Nintendo, Creatures Inc., GAME FREAK oder The Pokémon Company. Ohne Gewähr, siehe [Rechtliches](../README.md#rechtliches).

---

## Überblick

| | |
|---|---|
| **Aufbau** | Eine einzelne Datei `index.html` mit HTML, CSS und JavaScript, ohne Build-Schritt und ohne Framework |
| **Rendering** | HTML5 Canvas 2D |
| **PDF** | [jsPDF 2.5.1](https://github.com/parallax/jsPDF) (UMD-Build von cdnjs) |
| **Schriften** | Barlow Condensed (Header, Labels), Barlow (Werte), IBM Plex Mono (Nummern), über Google Fonts |
| **Speicher** | `localStorage` im Browser, kein Server |
| **Abhängigkeiten zur Laufzeit** | `fonts.googleapis.com`, `fonts.gstatic.com`, `cdnjs.cloudflare.com` |

```
index.html
├── <style>        Oberfläche (hell/dunkel über prefers-color-scheme)
├── <form>         Reiter, Eingabefelder, Hintergrund-Galerie
├── <aside>        Vorschau-Canvas, PNG-Export, PDF-Export
└── <script>
    ├── TYPES / BGS             Typ-Farben und Hintergrund-Liste
    ├── getState / setState     Zustand eines Posters ↔ Formular
    ├── drawBg()                alle Hintergründe (mm-basiert, seeded)
    ├── lum() / palette()       Helligkeit messen → helle/dunkle Schrift
    ├── drawText()              Text mit Sperrung + automatischem Verkleinern
    ├── draw()                  komplettes Poster in beliebiger dpi
    ├── renderTabs / Thumbs     Vorschaubilder in Reitern und Galerie
    └── Export                  PNG (toBlob) und PDF (jsPDF)
```

---

## Koordinatensystem: alles in Millimetern

Das ganze Poster wird in **Millimetern** beschrieben und erst beim Zeichnen in Pixel umgerechnet:

```js
const MM = dpi / 25.4;          // Pixel pro mm
const m  = v => v * MM;         // mm → px
const W  = Math.round(100 * MM); // 100 mm Breite
const H  = Math.round(150 * MM); // 150 mm Höhe
```

Dieselbe Funktion `draw(canvas, dpi, forExport)` erzeugt damit:

| Verwendung | dpi | Pixel |
|---|---|---|
| Reiter-Vorschaubild | 15,24 | 60 × 90 |
| Live-Vorschau | 150 | 591 × 886 |
| PNG / PDF Standard | 300 | 1181 × 1772 |
| PNG hoch | 600 | 2362 × 3543 |

Weil auch die Zufallsmuster in mm rechnen (Anzahl der Linien, Waben, Sterne …), sieht jede Auflösung exakt gleich aus. Die Vorschau entspricht also dem Druck.

---

## Layout (Maße in mm)

| Element | Position / Größe |
|---|---|
| Poster | 100 × 150 |
| Zeile darüber | Grundlinie y = 10,8 · Schriftgröße 2,5 · Sperrung 0,32 em |
| Header | Grundlinie y = 20,6 (ohne Zeile darüber: 18,5) · Schriftgröße 10,5 · max. Breite 86 |
| Kartenfeld | x = (100 − Breite) / 2 · y = 24 · Standard 64 × 90 · Eckradius 3,2 |
| Holo-Schimmer | 12 Lagen à 0,42 mm nach außen, nur außerhalb des Kartenfelds (Clip mit even-odd) |
| Infoblock oben (`top`) | y = 24 + Kartenhöhe + 5 → Standard 119 |
| Farbstrich | x 7–17, Trennlinie x 7–93 bei y = `top` |
| #Nummer | x 7 · y = top + 8,2 · Mono 6,2 |
| Name | x 7 · y = top + 14,6 · Condensed 5,4 |
| Collection | x 7 · y = top + 19,8 · 3,0 |
| Senkrechte Trennlinie | x 48,5 · y top + 3 bis top + 23 |
| Rechte Spalte | x 52 (2. Spalte x 76) · Zeilen bei top + 5,6 / 12,5 / 19,4 · Label 1,85 · Wert 2,75 |

`drawText()` verkleinert einen Text schrittweise um 4 % (bis minimal 40 %), bis er in `maxW` passt. So wird nichts abgeschnitten.

---

## Hintergrund-Generator

Alle Muster entstehen in `drawBg(ctx, id, W, H, m, accent, seed)`.

### Reproduzierbarer Zufall

```js
function mulberry(a){ return () => { /* mulberry32 */ }; }
const R = mulberry(seed);  // gleiche seed → gleiche Zahlenfolge
```

„Neu würfeln“ setzt nur eine neue `seed` (1–99 999). Deshalb lässt sich jede Variante über ihre Nummer exakt wiederherstellen.

### Farben

Die Akzentfarbe wird in HSL zerlegt (`hexToHsl`). Die Muster arbeiten mit Farbton `h` und Sättigung `S = max(s, 35)` und variieren nur die Helligkeit. So passt jedes Muster automatisch zu jeder Typ-Farbe.

### Die Muster

| id | Technik |
|---|---|
| `nacht`, `galerie`, `typ` | Linearer Verlauf + radialer Farbschein hinter der Karte |
| `holo` | 3 Regenbogen-Verläufe in zufälligen Winkeln mit `screen`-Überblendung, feine Diagonalstreifen, Glitzerpunkte und 4-Strahl-Sterne, Vignette |
| `strahlen` | Radialer Verlauf, 24–42 Tortenstücke abwechselnd aufgehellt, Mittelpunkt in der Kartenmitte |
| `waben` | Sechseckraster (Radius 3,2–5,6 mm), Deckkraft fällt mit Abstand zu einem Fokuspunkt, einzelne Waben gefüllt |
| `kosmos` | 8 Nebel-Glows (Akzent, +40°, −60°, Komplementär) mit `screen`, 420 Sterne, 10 helle Sterne mit Glow |
| `wellen` | 34–47 überlagerte Sinuskurven, Amplitude zur Mitte hin am größten |
| `halbton` | Punktraster im Dreiecksgitter, Punktgröße nimmt mit Abstand zum Fokus ab |
| `topo` | 2–3 Zentren mit je 33 verzerrten Ringen (3 Sinus-Harmonische), jede 5. Linie kräftiger |
| `kristall` | 7 × 10 verrücktes Punktgitter → Dreiecke (Low-Poly), Helligkeit nach Abstand zur Karte |
| `eigenes` | Bild „cover“ eingepasst, vertikal verschiebbar. Weichzeichnen durch Verkleinern und Vergrößern (läuft in jedem Browser, auch Safari ohne `ctx.filter`) |

### Neues Muster hinzufügen

1. In `BGS` einen Eintrag ergänzen: `{id:"meinmuster", name:"Mein Muster", gen:true}`
2. In `drawBg()` einen `case "meinmuster": { … break; }` schreiben:
   - Größen immer über `m(...)` in mm angeben
   - Zufall nur über `R()`, nie über `Math.random()`
   - Farben über `hsl(h, S, L, alpha)` vom Akzent ableiten
   - optional `vignette(ctx, W, H, 0.4)` am Ende
3. Fertig. Galerie, Vorschaubilder, „Überrasch mich“ und der Export nutzen es automatisch.

---

## Automatische Schriftfarbe

`lum(canvas, x, y, w, h)` skaliert einen Bereich des bereits gezeichneten Hintergrunds auf 8 × 8 Pixel und berechnet die mittlere relative Helligkeit (Rec. 709: `0.2126 R + 0.7152 G + 0.0722 B`). Gemessen wird getrennt für:

- Kopfbereich (x 8–92, y 4–22)
- Kartenfeld (für Markierungslinien)
- Infoblock

Liegt der Wert **unter 0,55**, gibt es helle Schrift, sonst dunkle. `palette(light)` liefert dazu passende Werte für Schrift, Grau, Linien, Akzent, Textschatten und Infofeld. Bei den generierten Mustern und eigenen Bildern bekommt Text zusätzlich einen weichen Schatten.

---

## Zustand und Speicherung

Jedes Poster ist ein Objekt:

```js
{
  vals:   { header, eyebrow, slotW, slotH, guide, num, name, coll,
            edition, cardno, special, rarity, lang, dim, ink, bgBlur, bgPos },
  checks: { holo, imgExport, panelOn },
  accent: "#E9B31B", bgId: "kosmos", seed: 202,
  cardImg, bgImg, bgImgData            // nur im Speicher
}
```

Das Formular zeigt immer das aktive Poster. `getState()` liest es aus, `setState()` schreibt es zurück. Für Reiter-Vorschau und Export werden die anderen Poster kurz über `withPoster(i, fn)` geladen, gezeichnet und danach wiederhergestellt.

| localStorage-Schlüssel | Inhalt |
|---|---|
| `kartenposter-v3` | `{ v4, posters[], active, count, dpi, layout }` ohne Bilder |
| `kartenposter-v3-bg-0` … `-3` | Eigenes Hintergrundbild je Poster als JPEG-Data-URL (auf max. 2400 px verkleinert, Qualität 0,9) |

Kartenfotos werden nicht gespeichert, sie dienen nur der Vorschau. Alle Zugriffe auf `localStorage` laufen in `try/catch`, damit das Tool auch im privaten Modus funktioniert.

**Beispieldaten ändern:** Das Array `SAMPLES` im Script enthält die 4 Start-Poster. Wer eigene Vorlagen möchte, ändert es dort und erhöht `KEY` (z. B. auf `kartenposter-v4`), damit alte gespeicherte Stände nicht darüberliegen.

---

## Export

### PNG

```js
const out = document.createElement("canvas");
draw(out, dpi, true);                  // forExport = true
out.toBlob(blob => …, "image/png");
```

Wenn die Seite als Claude-Artifact läuft, wird über die `downloads`-Schnittstelle gespeichert, sonst über einen normalen Download-Link.

### PDF

Jedes Poster wird mit 300 dpi gerendert und als JPEG (Qualität 0,95) in jsPDF eingebettet, immer mit exakt 100 × 150 mm.

| Layout | Seite | Startpunkt (mm) | Abstand |
|---|---|---|---|
| `a4` | 297 × 210 quer | x = (297 − 206) / 2 = 45,5 · y = 30 | 6 mm |
| `a3` | 297 × 420 hoch | x = 45,5 · y = (420 − 306) / 2 = 57 | 6 mm |
| `foto` | 100 × 150 | 0 · 0 | – |

Die Schnittmarken sitzen 1 mm außerhalb jeder Ecke, sind 2 mm lang und 0,15 mm stark. Dadurch berühren sich die Marken benachbarter Poster im 6-mm-Abstand nicht.

---

## Lokal entwickeln

Es gibt nichts zu bauen: `index.html` im Editor öffnen, speichern, Browser neu laden.

Für einen lokalen Server (z. B. zum Testen auf dem Handy im selben WLAN):

```bash
python3 -m http.server 8000
# → http://<deine-ip>:8000
```

---

## Browser-Unterstützung

Getestet mit aktuellen Versionen von Chrome, Edge, Firefox und Safari. Voraussetzungen: Canvas 2D, `createRadialGradient`, `globalCompositeOperation = "screen"`, `FileReader`, `localStorage`, `document.fonts`.

---

← Zurück zur [Übersicht](../README.md) · [Anleitung](ANLEITUNG.md)
