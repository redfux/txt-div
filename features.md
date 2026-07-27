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
- Merge-Funktion: A/B-Buttons pro Diff-Zeile — explizite Drei-Zustands-Wahl (null/A/B), Basis immer A
- Merge-Ergebnis-Bereich: aus-/einblendbar, eine Datei über volle Breite, B-Übernahmen hellblau hervorgehoben
- Auswahl-Modus: Blockweise (ganze Diff-Gruppe) oder Zeilenweise (einzelne Zeile), umschaltbar in der Toolbar
- Zeilentreue Darstellung: umgebrochene Zeilen halten beide Spalten auf gleicher Höhe; Umbruch an Wortgrenzen
- Tausch-Button (⇄) zwischen den Dateiauswahlen: vertauscht Original und Vergleichsdatei und vergleicht sofort neu
- Kopieren-Button und Speichern-Button für jedes Merge-Ergebnis

## Offen / Ideen

- Zeilenweise Verlinkung zwischen den Spalten (Verbindungslinien bei Unterschieden)
- Export des Diff-Ergebnisses als HTML oder PDF
- Unterstützung für Drag & Drop beim Datei-Upload
- Einstellbarer Kontext (wie viele gleiche Zeilen um einen Diff herum angezeigt werden)
