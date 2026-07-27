# Changelog

Alle wesentlichen Änderungen an txt-div werden hier dokumentiert.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.0.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

---

## [0.3.7] – 2026-07-27

### Fixed
- **Zeilenausrichtung bei Umbrüchen:** Brach eine Zeile auf mehrere Bildschirmzeilen um, behielt die Gegenzeile ihre einfache Höhe — Leerräume waren zu klein und die Spalten liefen auseinander. Die Höhen beider Panes werden jetzt paarweise angeglichen (`alignRowHeights`), gemessen und geschrieben in getrennten Durchgängen, damit es bei zwei Reflows bleibt. Ein `ResizeObserver` richtet nach Breitenänderungen neu aus, da sich dabei die Umbruchpunkte verschieben. Betrifft Leerzeilen ohne Gegenstück ebenso wie geänderte Zeilen unterschiedlicher Länge
- **Umbruch mitten im Wort:** `word-break: break-all` trennte bedingungslos an jeder Stelle. Ersetzt durch `overflow-wrap: anywhere` mit `word-break: normal` — Umbruch erfolgt an Leerzeichen, ein einzelnes zu langes Wort wird weiterhin getrennt. `min-width: 0` stellt sicher, dass das Flex-Kind dafür schrumpfen darf

---

## [0.3.6] – 2026-07-27

### Fixed
- **Scroll-Positionen waren systematisch verschoben:** `offsetTop` liefert den Abstand zum `offsetParent`, nicht zum Scroll-Container. Da weder `.pane-content` noch `.result-content` positioniert waren, enthielten alle Werte den Abstand vom Seitenanfang — je Bereich ein anderer Sockel (Diff-Pane 186 px, Ergebnis 712 px). Beide Container sind jetzt `position: relative`, womit `offsetTop` direkt als `scrollTop` verwendbar ist. Betraf auch die Vor/Zurück-Navigation, die dadurch am Ziel vorbeisprang
- **Scroll-Sync erstarrte bei geöffnetem Merge-Bereich:** Das `syncLocked`-Boolean wurde synchron wieder auf `false` gesetzt, Scroll-Events feuern aber asynchron — das Flag war beim Eintreffen des Echo-Events längst zurückgesetzt. Ersetzt durch ein Owner-Prinzip: wer zuletzt gescrollt hat, behält die Kontrolle für 150 ms
- **Sync ist jetzt zeilengenau** statt proportional — Zuordnung über `opToLine`/`lineToOp`, sodass Diff-Panes und Ergebnis bei abweichender Zeilenzahl dieselbe Textstelle zeigen
- **Merge-Ergebnis sprang beim A/B-Klick nach oben:** Der Neuaufbau per `innerHTML` setzt `scrollTop` auf 0; die Position wird nun aus den Diff-Panes wiederhergestellt

---

## [0.3.5] – 2026-07-27

### Changed
- Theme folgt beim Laden der System-/Browser-Einstellung (`prefers-color-scheme`) statt fest auf Dark zu starten; gesetzt im `<head>`, daher kein Aufblitzen beim Laden
- Theme-Umschalter zeigt das aktive Theme an, nicht mehr das Wechselziel

---

## [0.3.4] – 2026-07-27

### Fixed
- Scroll-Sync funktioniert jetzt in beide Richtungen: Scrollen im Merge-Ergebnis bewegt auch die Diff-Panes

---

## [0.3.3] – 2026-07-27

### Fixed
- Merge-Ergebnis: alle Zeilen aus Diff-Positionen hellblau hinterlegt, unabhängig von A/B-Wahl
- Merge-Ergebnis scrollt proportional mit den Diff-Panes (auch beim Navigieren per Vor/Zurück)
- Beide Diff-Panes haben jetzt identische Textbreite durch gleichen 48px-Spacer rechts — Zeilenumbrüche entstehen an denselben Stellen, Zeilen fluchten 1:1

---

## [0.3.2] – 2026-07-27

### Changed
- Merge-Buttons: zwei explizite Buttons pro Diff-Zeile — `A` (behalten, rot hervorgehoben) und `B` (übernehmen, blau hervorgehoben); nochmaliges Klicken hebt die Wahl auf; Status null/A/B je Zeile
- Auswahl-Modus: neuer Toggle-Button in der Toolbar wechselt zwischen **Blockweise** (Klick setzt alle Zeilen einer Gruppe) und **Zeilenweise** (Klick setzt nur die einzelne Zeile)
- Statuszeile im Ergebnis-Bereich zeigt Anzahl B-Übernahmen, A-Beibehaltungen und offene Entscheidungen

---

## [0.3.1] – 2026-07-27

### Fixed
- Merge-Buttons sitzen jetzt direkt in der jeweiligen Diff-Zeile (Pane A) statt in einer separaten Spalte — garantiert pixelgenaue Ausrichtung auch bei umgebrochenen Zeilen
- Merge-Ergebnis zeigt nur noch eine Datei (Basis: A), statt zwei — B-Änderungen werden selektiv per ← Button in das Ergebnis übernommen

---

## [0.3.0] – 2026-07-27

### Added
- Merge-Spalte zwischen den Diff-Ansichten mit ← / → Buttons pro Diff-Block
  - `←` übernimmt B-Version in Ergebnis A; `→` übernimmt A-Version in Ergebnis B
  - Beide Richtungen unabhängig voneinander wählbar, Toggle durch erneutes Klicken
  - Aktiver Button bleibt hervorgehoben (blau), Quellzeile erhält gestrichelten Rahmen
- Merge-Ergebnis-Bereich unterhalb der Diff-Ansicht (ein-/ausklappbar)
  - Zeigt „Ergebnis A" und „Ergebnis B" nebeneinander
  - Übernommene Zeilen sind hellblau hinterlegt
  - Statuszeile zeigt Anzahl aktiver Übernahmen
- Kopieren-Button: übergibt den jeweiligen Ergebnis-Text an die Zwischenablage
- Speichern-Button: lädt den Ergebnis-Text als Datei herunter (Dateiname: `merged_<original>`)

---

## [0.2.0] – 2026-07-27

### Changed
- Datei-Inputs verwenden jetzt `accept="text/*"` statt einer festen Endungsliste — alle Textdateien werden akzeptiert, inkl. `.cfg`, `.config` und beliebige weitere Formate; behebt das Problem in Safari, das keine „Alle Dateien"-Option anbietet
- Hauptdatei von `txt-div.html` in `index.html` umbenannt für GitHub Pages Kompatibilität

---

## [0.1.0] – 2025-07-06

### Added
- Zwei-Spalten-Ansicht zum Vergleich zweier Textdateien nebeneinander
- LCS-basierter Diff-Algorithmus auf Zeilen- und Zeichenebene
- Farbliche Hervorhebung: grün (hinzugefügt), orange (geändert), rot (entfernt)
- Navigation zwischen Unterschieden per Vor/Zurück-Buttons mit Zähler
- Minimap am rechten Rand mit Überblick aller Diffs und Viewport-Anzeige
- Synchrones Scrollen beider Spalten
- Umschalter zwischen Dark Mode und Light Mode
- Content-Security-Policy Meta-Tag
- Footer mit Versionsnummer und Hinweistext
