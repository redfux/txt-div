# Features

Anforderungen und geplante Funktionen für txt-div.

---

## Umgesetzt

- Zwei-Spalten-Diff-Ansicht (Datei A links, Datei B rechts)
- LCS-Algorithmus für zeilen- und zeichengenauen Vergleich
- Farbkodierung: grün = hinzugefügt, orange = geändert, rot = entfernt
- Navigation zwischen Unterschieden (Vor/Zurück + Zähler), folgt dem Auswahlmodus: block- oder zeilenweise; das ganze Ziel wird umrandet
- Minimap mit Viewport-Anzeige
- Synchrones Scrollen beider Spalten
- Dark Mode / Light Mode Umschalter
- Lokale Verarbeitung – keine Daten verlassen den Browser
- Unterstützte Formate: alle Textdateien (`text/*`), u. a. `.txt`, `.md`, `.json`, `.csv`, `.log`, `.xml`, `.cfg`, `.config`, `.js`, `.ts`, `.css`, `.py`
- Merge-Funktion: Pfeil pro Diff-Zeile auf der jeweiligen Seite (`→` in A, `←` in B), exklusive Wahl (offen/A/B), Basis immer A
- Merge-Ergebnis-Bereich: aus-/einblendbar, eine Datei über volle Breite, Diff-Zeilen hellblau hervorgehoben, Höhe per Ziehgriff verstellbar
- Lücken im Merge-Ergebnis werden markiert (rot schraffierte Zeile mit Anzahl), unterschieden nach „entfernt" und „nicht übernommen"; die Marker gehen nicht in den Export ein
- Auswahl-Modus: Blockweise (ganze Diff-Gruppe) oder Zeilenweise (einzelne Zeile), umschaltbar in der Toolbar
- Zeilentreue Darstellung: umgebrochene Zeilen halten beide Spalten auf gleicher Höhe; Umbruch an Wortgrenzen
- Tausch-Button (⇄) zwischen den Dateiauswahlen: vertauscht Original und Vergleichsdatei und vergleicht sofort neu
- Kopieren-Button und Speichern-Button für jedes Merge-Ergebnis
- Datei per Drag & Drop auf das Auswahlfeld ablegen (native Funktion des Datei-Feldes, das die gesamte Fläche abdeckt); die Beschriftung weist darauf hin
- Dateiangaben im Auswahlfeld: Dateiname hervorgehoben, dahinter Größe und Änderungsdatum in grauer Schrift. Bei Platzmangel werden zuerst die grauen Angaben gekürzt, der Dateiname bleibt vollständig, solange er ins Feld passt; der volle Text steht im Tooltip

- Obergrenzen für Eingabedateien: **5 MB** und **20.000 Zeilen**; größere Dateien werden mit Meldung abgelehnt
- Hinweisleiste, wenn oberhalb von ~5.000 × 5.000 Zeilen das Näherungsverfahren für den Vergleich greift

## Offen / Ideen

- Zeilenweise Verlinkung zwischen den Spalten (Verbindungslinien bei Unterschieden)
- Export des Diff-Ergebnisses als HTML oder PDF
- Einstellbarer Kontext (wie viele gleiche Zeilen um einen Diff herum angezeigt werden)
- **Myers-Diff statt der Näherung:** würde den exakten Vergleich auch oberhalb von 5.000 Zeilen ermöglichen (O((m+n)·d) Zeit, O(m+n) Speicher statt O(m×n)). Aufwand: eigene Algorithmus-Implementierung inkl. Pfad-Rekonstruktion, Präfix-/Suffix-Trimming und Testabdeckung; `charDiff` müsste separat bewertet werden, da dort die DP-Matrix oft günstiger ist
- **Virtuelles Scrollen:** oberhalb von ~20.000 Zeilen ist das DOM der Engpass (11 Knoten je Zeile). Würde tief in `alignRowHeights`, `findRowAt` und den Scroll-Sync eingreifen, da diese auf real existierenden Zeilen aufbauen

### Nicht umsetzbar

- **Vollständiger Dateipfad im Auswahlfeld:** Browser geben den lokalen Pfad nicht heraus. Das `File`-Objekt kennt nur `name`, `size`, `lastModified` und `type`; `input.value` liefert bewusst `C:\fakepath\<name>`, und `webkitRelativePath` ist ausschließlich bei Ordner-Auswahl (`webkitdirectory`) gefüllt. Statt des Pfades werden daher Größe und Änderungsdatum angezeigt — ist ein `webkitRelativePath` vorhanden, wird dessen Ordner zusätzlich genutzt.
