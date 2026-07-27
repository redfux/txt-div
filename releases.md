# Changelog

Alle wesentlichen Änderungen an txt-div werden hier dokumentiert.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.0.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

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
