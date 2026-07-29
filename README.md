# txt-div

Ein einfaches, browserbasiertes Werkzeug zum visuellen Vergleich zweier Textdateien — und zum Zusammenführen der Unterschiede. Ohne Installation, ohne Server, direkt im Browser.

**Live:** https://redfux.github.io/txt-div/

## Funktionsweise

txt-div lädt beide Dateien lokal im Browser und berechnet die Unterschiede mithilfe eines **LCS-Algorithmus** (Longest Common Subsequence). Der Vergleich findet auf zwei Ebenen statt:

- **Zeilenebene** — welche Zeilen wurden hinzugefügt, entfernt oder verändert
- **Zeichenebene** — innerhalb veränderter Zeilen werden die exakten Unterschiede markiert

Die Dateien verlassen den Browser nicht. Es werden keine Daten übertragen und nichts gespeichert.

## Setup

Keines. Entweder die Live-Seite öffnen oder `index.html` herunterladen und per Doppelklick starten — die Datei funktioniert auch offline.

## Nutzung

1. Unter **Datei A** die Originaldatei wählen — per Klick oder indem die Datei auf das Feld gezogen wird
2. Unter **Datei B** die Vergleichsdatei wählen
3. Auf **Vergleichen** klicken

Neben dem Dateinamen stehen Größe und Änderungsdatum in grauer Schrift. Mit dem Doppelpfeil **⇄** zwischen den Feldern lassen sich Original und Vergleichsdatei tauschen; der Vergleich wird sofort neu berechnet.

Die Dateien werden nebeneinander angezeigt, Unterschiede sind farblich hervorgehoben:

| Farbe | Bedeutung |
|---|---|
| 🟢 Grün | Zeile wurde in Datei B hinzugefügt |
| 🟠 Orange | Zeile wurde verändert |
| 🔴 Rot | Zeile wurde aus Datei A entfernt |

## Navigation

- **↑ Zurück** und **Weiter ↓** springen zwischen den Unterschieden, der Zähler zeigt die Position (z. B. `3 / 12`)
- Die Navigation folgt dem gewählten Auswahlmodus: blockweise von Block zu Block, zeilenweise von Zeile zu Zeile
- Das aktuelle Ziel ist grün umrahmt — im Blockmodus der ganze Block
- Die **Minimap** am rechten Rand zeigt alle Unterschiede im Dokument; ein Klick springt zur entsprechenden Stelle
- Beide Spalten und das Merge-Ergebnis scrollen synchron

## Änderungen zusammenführen (Merge)

Grundlage des Ergebnisses ist immer **Datei A**. Pro Unterschied entscheidet man, welche Version gilt:

- **→** in Spalte A: die A-Version übernehmen
- **←** in Spalte B: die B-Version übernehmen
- Erneutes Klicken hebt die Wahl wieder auf. Ohne Entscheidung bleibt die A-Version stehen.

Über **Auswahlmodus** wird umgeschaltet, ob ein Klick den ganzen zusammenhängenden Block oder nur die einzelne Zeile setzt.

Unten öffnet **Merge-Ergebnis** den zusammengeführten Text. Der Bereich lässt sich an seiner Oberkante in der Höhe ziehen. Darin gilt:

- Zeilen an Diff-Positionen sind hellblau hinterlegt
- Wo Zeilen weggelassen wurden, steht ein rot schraffierter Marker mit der Anzahl — unterschieden nach „entfernt" und „nicht übernommen". Diese Marker sind nur eine Anzeige und landen nicht im Export.

Mit **Kopieren** geht das Ergebnis in die Zwischenablage, mit **Speichern** wird es als Datei abgelegt (Dateiname `merged_<Original>`).

## Unterstützte Dateiformate

Alle textbasierten Formate — der Dateidialog akzeptiert `text/*`, u. a. `.txt`, `.md`, `.json`, `.csv`, `.log`, `.xml`, `.cfg`, `.config`, `.js`, `.ts`, `.css`, `.py`. Lässt ein Browser eine Datei nicht auswählen, kann sie meist per Drag & Drop trotzdem geladen werden.

## Darstellung

Beim Start folgt die App der System-Einstellung für Hell/Dunkel. Über den Schalter oben rechts lässt sich manuell umschalten.

## Grenzen

- Sehr große Dateien können den Browser ausbremsen; der Vergleich ist rechenintensiv. Schlägt er fehl, erscheint ein Hinweis.
- Der vollständige Dateipfad kann nicht angezeigt werden — Browser geben ihn aus Sicherheitsgründen nicht heraus.
- Auf schmalen Displays ist die Zwei-Spalten-Ansicht sehr eng.

## Dokumentation

| Datei | Inhalt |
|---|---|
| `releases.md` | Änderungsverlauf (Keep a Changelog, SemVer) |
| `features.md` | Anforderungen und Ideen |
| `bugs.md` | Bekannte Fehler und deren Behebung |
| `architecture.md` | Technische Entscheidungen und Aufbau |
