# Changelog

Alle wesentlichen Änderungen an txt-div werden hier dokumentiert.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.0.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

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
