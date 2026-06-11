# txt-div

Ein einfaches, browserbasiertees Werkzeug zum visuellen Vergleich zweier Textdateien — ohne Installation, ohne Server, direkt im Browser.

## Funktionsweise

txt-div lädt beide Dateien lokal im Browser und berechnet die Unterschiede mithilfe eines **LCS-Algorithmus** (Longest Common Subsequence). Der Vergleich findet auf zwei Ebenen statt:

- **Zeilenebene** — welche Zeilen wurden hinzugefügt, entfernt oder verändert
- **Zeichenebene** — innerhalb veränderter Zeilen werden die exakten Unterschiede markiert

Die Dateien verlassen den Browser nicht. Es werden keine Daten an einen Server übertragen.

## Nutzung

1. `txt-div.html` im Browser öffnen (Doppelklick genügt)
2. Unter **Datei A** die Originaldatei auswählen
3. Unter **Datei B** die Vergleichsdatei auswählen
4. Auf **Vergleichen** klicken

Die beiden Dateien werden nebeneinander angezeigt. Unterschiede sind farblich hervorgehoben:

| Farbe | Bedeutung |
|---|---|
| 🟢 Grün | Zeile wurde in Datei B hinzugefügt |
| 🟠 Orange | Zeile wurde verändert |
| 🔴 Rot | Zeile wurde aus Datei A entfernt |

## Navigation

- Mit den Buttons **↑ Zurück** und **Weiter ↓** kann direkt zwischen den einzelnen Unterschieden gesprungen werden
- Der Zähler zeigt die aktuelle Position (z. B. `3 / 12`)
- Die **Minimap** am rechten Rand gibt einen Überblick über alle Unterschiede im Dokument — ein Klick springt direkt zur entsprechenden Stelle
- Beide Spalten scrollen synchron

## Unterstützte Dateiformate

Alle plaintext-basierten Formate, u. a.: `.txt`, `.md`, `.json`, `.csv`, `.log`, `.xml`, `.html`, `.js`, `.ts`, `.css`, `.py`

## Darstellung

Über den Schalter oben rechts kann zwischen **Dark Mode** und **Light Mode** gewechselt werden.
