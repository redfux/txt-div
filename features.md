# Features

Anforderungen und geplante Funktionen für txt-div.

---

## Umgesetzt

- Zwei-Spalten-Diff-Ansicht (Datei A links, Datei B rechts)
- LCS-Algorithmus für zeilen- und zeichengenauen Vergleich
- Farbkodierung: grün = hinzugefügt, orange = geändert, rot = entfernt
- Navigation zwischen Unterschieden (Vor/Zurück + Zähler)
- Minimap mit Viewport-Anzeige
- Synchrones Scrollen beider Spalten
- Dark Mode / Light Mode Umschalter
- Lokale Verarbeitung – keine Daten verlassen den Browser
- Unterstützte Formate: alle Textdateien (`text/*`), u. a. `.txt`, `.md`, `.json`, `.csv`, `.log`, `.xml`, `.cfg`, `.config`, `.js`, `.ts`, `.css`, `.py`
- Merge-Funktion: Übernahme einzelner Änderungen in beide Richtungen per ← / → Buttons
- Merge-Ergebnis-Bereich: aus-/einblendbar, zeigt beide Ergebnis-Texte mit hellblauer Hervorhebung übernommener Zeilen
- Kopieren-Button und Speichern-Button für jedes Merge-Ergebnis

## Offen / Ideen

- Zeilenweise Verlinkung zwischen den Spalten (Verbindungslinien bei Unterschieden)
- Export des Diff-Ergebnisses als HTML oder PDF
- Unterstützung für Drag & Drop beim Datei-Upload
- Einstellbarer Kontext (wie viele gleiche Zeilen um einen Diff herum angezeigt werden)
