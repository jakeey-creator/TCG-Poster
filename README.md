# Kartenposter 10×15

Poster-Generator für Pokémon-Karten im 10×15-cm-Bilderrahmen. Eine einzelne HTML-Datei, kein Build, keine Installation.

![Format](https://img.shields.io/badge/Format-100×150_mm-informational) ![Export](https://img.shields.io/badge/Export-PNG_·_PDF-success)

## Was es kann

- **Poster 100 × 150 mm** mit Header (Pokémon-Name), freiem Platz für die echte Karte (Standard 64 × 90 mm) und Infoblock darunter
- **Infoblock:** #Nummer, Name, Collection · Edition, Kartennummer, Special Edition, Seltenheit, Sprache
- **Bis zu 4 Poster** gleichzeitig (Reiter), „Stil auf alle übertragen“ für einheitliche Sets
- **Hintergründe:** Nacht, Galerie, Typ-Verlauf plus 8 generierte Muster (Holo-Folie, Strahlen, Waben, Kosmos, Energie-Wellen, Halbton, Topografie, Kristall) mit „Neu würfeln“ / „Überrasch mich“, oder ein eigenes Bild (Weichzeichnen, Ausschnitt)
- **Akzentfarbe** nach Pokémon-Typ oder frei wählbar, Schriftfarbe passt sich automatisch dem Hintergrund an
- **Export:** PNG in 300 oder 600 dpi, PDF mit allen Postern in A4 quer (2 pro Seite), A3 (4 auf einer Seite) oder 10×15 (1 pro Seite), optional mit Schnittmarken

## Benutzen

`index.html` im Browser öffnen – fertig. Eingaben werden lokal im Browser gespeichert.

Online über GitHub Pages: *Settings → Pages → Branch `main` / root* aktivieren.

## Drucken

Immer **„Tatsächliche Größe / 100 %“** wählen, nicht „An Seite anpassen“ – sonst passt die Karte nicht mehr ins Feld.
4 Poster à 10×15 ergeben 20 × 30 cm und passen deshalb nicht in Originalgröße auf A4 (29,7 cm hoch) – dafür gibt es A4 quer mit 2 Seiten oder A3.

## Technik

- Zeichnen per Canvas, alle Maße in Millimetern (auflösungsunabhängig)
- Schriften: Barlow, Barlow Condensed, IBM Plex Mono (Google Fonts)
- PDF: [jsPDF](https://github.com/parallax/jsPDF) 2.5.1 via cdnjs
