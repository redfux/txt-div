# Changelog

Alle wesentlichen Änderungen an txt-div werden hier dokumentiert.
Format nach [Keep a Changelog](https://keepachangelog.com/de/1.0.0/), Versionierung nach [SemVer](https://semver.org/lang/de/).

---

## [0.11.1] – 2026-07-30

### Fixed
- **Eigene Schwelle für den Zeichenvergleich.** `charDiff` nutzte dieselbe Grenze wie der Zeilenvergleich (64 Mio. Zellen), läuft aber einmal *pro geänderter Zeile*. Eine Zeile mit 4.000 Zeichen kostete dadurch allein 55 ms — eine Datei mit 1.310 solcher Zeilen (5 MB, also innerhalb aller Limits) hätte hochgerechnet **77 Sekunden** gebraucht
  - Neue Grenze `MAX_CHAR_LCS_CELLS` = 250.000 Zellen, also Zeilen bis 500 Zeichen. Das deckt Konfigurations- und Quelldateien ab; die längste Zeile der Referenz-Switch-Configs hat 208 Zeichen
  - Oberhalb der Grenze wird die Zeichen-Hervorhebung **weggelassen statt genähert**: einzelne Zeichen wiederholen sich so stark, dass die greedy Variante nur ~1 % des Optimums findet und praktisch zufällig markieren würde. Die Zeile bleibt als geändert gekennzeichnet, nur ohne Detail innerhalb der Zeile
  - Gemessen: derselbe Fall fällt von hochgerechnet 77 s auf **1,5 s**. Der verbleibende ungünstigste Fall (5 MB aus Zeilen à 300–500 Zeichen, alle geändert) liegt bei ~14 s, wovon etwa ein Drittel auf das DOM-Rendering entfällt

### Changed
- **Verständlichere Meldungen.** Die Texte nannten Schwellwerte und technische Fehlernamen, aber nicht die Konsequenz:
  - Der Hinweis zum schnelleren Verfahren nannte „wiederkehrende Zeilen" als Ursache. Das galt vor der Reparatur in 0.11.0; seither sind **gelöschte oder verschobene Abschnitte** der Auslöser. Der Text sagt jetzt zusätzlich, was zuverlässig funktioniert (hinzugefügte und geänderte Zeilen)
  - Technische Fehlernamen wie `NotReadableError` erscheinen nicht mehr im Fenster, sondern nur in der Konsole
  - Dateimeldungen sind zweistufig: ein kurzer Text, der neben den Dateinamen passt (z. B. „zu groß: 6.0 MB (max. 5.0 MB)"), und die vollständige Erklärung im Tooltip. Zuvor wurden die längeren Meldungen im engen Feld abgeschnitten — ausgerechnet am erlaubten Höchstwert

### Notes
- Schwellwerte geprüft und unverändert gelassen: 5 MB, 20.000 Zeilen, 64 Mio. Zellen für den Zeilenvergleich. Die Referenz-Configs (4.628 Zeilen, längste Zeile 208 Zeichen) bleiben unberührt — 553 ms, alle 178 Hervorhebungen intakt

---

## [0.11.0] – 2026-07-30

Validiert an zwei echten Cisco-Switch-Konfigurationen mit je 4.628 Zeilen, von denen nur 12 % eindeutig sind (282× `!`, 178× identische Interface-Zeilen). Referenz für alle Angaben ist GNU `diff`, das 178 Änderungen findet.

### Fixed
- **Näherungsverfahren brach bei wiederkehrenden Zeilen zusammen.** Es merkte sich pro Zeileninhalt nur das *erste* Vorkommen in Datei B; sobald der Suchzeiger daran vorbei war, wurde für jede weitere gleichlautende Zeile nie mehr ein Treffer gefunden. Auf den Testkonfigurationen bedeutete das **529 statt 4.450** gemeinsamer Zeilen — praktisch die gesamte Datei erschien geändert.
  - Jetzt werden **alle** Vorkommen je Zeileninhalt vorgehalten und der erste Treffer ab dem Suchzeiger per Binärsuche bestimmt. Ergebnis auf denselben Dateien: **4.450**, also das exakte Optimum, bei 3–4 ms Laufzeit
  - Auch oberhalb der Schwelle korrekt: bei künstlich auf 8.618 und 11.978 Zeilen erweiterten Konfigurationen weiterhin genau 178 erkannte Änderungen
  - **Weiterhin eine Näherung:** Da stets der früheste Treffer genommen wird, bleibt sie bei Löschungen und verschobenen Blöcken schwächer als die exakte Berechnung — gemessen etwa 50 % des Optimums (vorher 11 %). Bei Einfügungen und geänderten Zeilen erreicht sie 100 %

### Changed
- Schwelle für den exakten LCS von 25 auf **64 Mio. Zellen** erhöht (≈ 8.000 × 8.000 Zeilen). Die Testkonfigurationen benötigen 21,4 Mio. Zellen — die bisherige Grenze ließ also nur 7 % Reserve, und 623 zusätzliche Zeilen hätten das Ergebnis von 178 auf 4.678 gemeldete Änderungen kippen lassen
  - Kosten am obereren Rand: 7.988 Zeilen = 63,8 Mio. Zellen, 390 ms Diff-Berechnung, ~122 MB für die Matrix
  - Der maximale DP-Wert bleibt mit 7.988 weit unter der `Uint16Array`-Grenze von 65.535

### Notes
- Messwerte mit den Originaldateien: 21,4 Mio. Zellen, exakte Berechnung, 131 ms für den Diff, **533 ms** für den vollständigen Vergleich inklusive Rendering, 10 MB Heap
- Ergebnis stimmt mit GNU `diff` überein: 178 Änderungsblöcke, 4.450 gleiche Zeilen
- Myers-Diff bleibt zurückgestellt. Er wäre nur noch für Dateien über 8.000 Zeilen mit Löschungen oder verschobenen Blöcken ein Gewinn; siehe `features.md`

---

## [0.10.0] – 2026-07-29

### Added
- **Harte Obergrenzen für Eingabedateien.** Größere Dateien werden abgelehnt, nicht nur mit einer Warnung geladen:
  - **5 MB** — geprüft vor dem Lesen, die Datei wird gar nicht erst geladen
  - **20.000 Zeilen** — geprüft nach dem Lesen, da die Zeilenzahl vorher nicht bekannt ist
  - Beide Grenzen sind nötig, weil Größe und Zeilenzahl unabhängig voneinander sind: 200 Zeilen à 2.000 Zeichen (0,4 MB) kosten mehr Zeit als 50.000 kurze Zeilen (2,9 MB)
  - Die Meldung nennt den gemessenen und den erlaubten Wert; der betroffene Slot wird geleert und „Vergleichen" deaktiviert
- **Hinweisleiste, wenn das Näherungsverfahren greift.** Oberhalb der Diff-Ansicht erscheint ein Hinweis mit der Zeilenzahl beider Dateien. Ohne ihn sieht ein Ergebnis von „fast alles geändert" wie ein Defekt aus, statt wie die bekannte Grenze des Verfahrens

### Changed
- Schwelle für den exakten LCS von **4 Mio. auf 25 Mio. Zellen** erhöht (≈ 5.000 × 5.000 Zeilen). Gemessen: ~185 ms Rechenzeit und ~50 MB für die Matrix, ein vollständiger Vergleich mit 4.900 Zeilen dauert 638 ms bei 26 MB Heap
  - Wirkung bei Dateien mit wiederkehrenden Zeilen (Klammern, Leerzeilen) und einem eingefügten Block: bei 3.000 und 4.900 Zeilen werden jetzt **4 %** Unterschiede erkannt statt vorher **100 %**
- Schwelle und Limits liegen als benannte Konstanten (`MAX_LCS_CELLS`, `MAX_FILE_BYTES`, `MAX_FILE_LINES`) am Anfang des Skripts statt als Zahl im Code

### Notes
- Zwischen 5.000 und 20.000 Zeilen bleibt das Näherungsverfahren aktiv — bei 20.000 × 20.000 wären es 400 Mio. Zellen, was mit ~800 MB Matrixspeicher nicht umsetzbar ist. Daher die Hinweisleiste. Bei Dateien mit eindeutigen Zeilen liefert die Näherung dort weiterhin korrekte Ergebnisse; problematisch sind nur viele identische Zeilen
- Die eigentliche Lösung wäre ein Myers-Diff (O((m+n)·d) Zeit, exakt). Bewusst zurückgestellt: der Algorithmus ist echte Rechenarbeit mit hohem Testbedarf, während das Heraufsetzen der Schwelle den Großteil realer Dateien abdeckt

---

## [0.9.0] – 2026-07-28

Abgleich mit dem Coding-Masterprompt.

### Added
- Fehlerbehandlung beim Dateilesen: `reader.onerror`, `reader.onabort` und ein `try/catch` um `readAsText()`. Die Meldung erscheint rot an der betroffenen Datei, der Slot wird geleert und „Vergleichen" deaktiviert — bisher schlug ein Lesefehler stillschweigend fehl
- `runCompare()` kapselt den Vergleich: bei einem unerwarteten Fehler (etwa zu großen Dateien) erscheint ein Hinweis statt einer halb aufgebauten Ansicht, Details gehen in die Konsole
- Rückmeldung beim Kopieren: der Button quittiert „Kopiert" bzw. „Fehlgeschlagen", falls die Zwischenablage nicht verfügbar ist

### Changed
- Versionsnummer liegt nur noch in der Konstante `APP_VERSION`; der Footer wird daraus gefüllt. Zuvor war sie im Markup hart kodiert und lief bei 0.3.4 und 0.3.5 versehentlich hinterher
- Alle Inline-Kommentare in `index.html` von Deutsch auf **Englisch** umgestellt (Masterprompt: Code englisch, `.md`-Dateien deutsch)
- `bugs.md` von Stand 0.1.0 auf aktuell gebracht — 15 seither behobene Fehler ergänzt. Der Eintrag „Spalten scrollten nicht synchron" ist als überholt markiert, da die dort genannte Lösung (`syncLocked`) sich in 0.3.6 als wirkungslos erwies
- `architecture.md` von Stand 0.1.0 auf aktuell gebracht: Merge-Modell, Scroll-Owner-Prinzip, Zeilenausrichtung, Navigationsziele, aktuelle DOM-Struktur, Fehlerbehandlung und bekannte Grenzen
- `README.md` um die Merge-Funktion erweitert, die bisher gar nicht beschrieben war — dazu Tausch-Button, Auswahlmodus, Drag & Drop, Lücken-Marker, Kopieren/Speichern und ein Abschnitt zu den Grenzen

### Notes
- Der im Abgleich gemeldete **Kontrastfehler im Dark Mode bestand nicht.** Die Nachmessung ergab 4,85:1 (nötig 4,5:1); der erste Wert von 2,0:1 entstand, weil im Messskript nach dem Theme-Wechsel ohne erzwungenen Reflow gelesen wurde und so die helle Textfarbe gegen den dunklen Hintergrund verglichen wurde. Dokumentiert in `bugs.md` unter „Nicht reproduzierbar / verworfen"
- Neu eingeführt wurde lediglich die Variable `--error`; im Dark-Theme auf `#ff8f8f` gesetzt, was 5,7:1 ergibt

---

## [0.8.1] – 2026-07-28

### Added
- Marker für Lücken im Merge-Ergebnis: weggelassene Zeilen hinterließen bisher keine Spur — sie fehlten einfach. An ihrer Stelle steht jetzt eine flache, rot schraffierte Zeile mit `⋯` in der Nummernspalte und der Anzahl
  - Die Beschriftung unterscheidet die beiden Ursachen: „N Zeilen entfernt" (eine A-Zeile fällt weg, weil die B-Version sie nicht hat) und „N Zeilen nicht übernommen" (eine B-Zeile wurde nicht gewählt). Gemischte Lücken heißen „N Zeilen ausgelassen" — „entfernt" wäre im zweiten Fall sachlich falsch
  - Die Marker sind rein visuell und gehen **nicht** in Kopieren/Speichern ein; die Zeilennummerierung zählt sie nicht mit

### Changed
- `computeResult()` liefert neben dem Text jetzt Render-Elemente (`items`) statt nur Zeilen. Die Zuordnungen `opToLine`/`lineToOp` zeigen entsprechend auf Positionen in `items`, da sich der Scroll-Sync nach den DOM-Kindern richtet — sonst hätten die eingefügten Marker die Zuordnung verschoben

---

## [0.8.0] – 2026-07-28

### Added
- Dateiangaben im Auswahlfeld: hinter dem Dateinamen stehen Größe und Änderungsdatum in grauer, kleinerer Schrift. Bei Platzmangel geben zuerst die grauen Angaben nach (hohes `flex-shrink`) und werden mit Ellipse gekürzt; der Dateiname bleibt vollständig, solange er ins Feld passt. Der volle Text steht jeweils im Tooltip
  - **Kein Dateipfad möglich:** Browser geben den lokalen Pfad nicht heraus — das `File`-Objekt kennt keinen, `input.value` liefert bewusst `C:\fakepath\<name>`, und `webkitRelativePath` ist nur bei Ordner-Auswahl gefüllt. Ist ein solcher Pfad vorhanden, wird sein Ordner zusätzlich angezeigt

### Changed
- Beschriftung des Auswahlfeldes lautet „Datei wählen oder hierher ziehen…" — Ablegen per Drag & Drop funktionierte bereits (native Funktion des Datei-Feldes, das die gesamte Fläche abdeckt), war aber nicht erkennbar und in `features.md` noch als offener Punkt geführt
- Platzhaltertext liegt nur noch in der Konstante `NO_FILE`; er war beim Speichern-Button zusätzlich hart kodiert

---

## [0.7.1] – 2026-07-27

### Fixed
- Die eingeblendete Scrollbar verdeckte in Pane A die Merge-Pfeile, solange sie nach dem Scrollen sichtbar war. Overlay-Scrollbars (macOS) legen sich über den Inhalt statt Platz einzunehmen; beide Panes halten rechts nun 14 px frei. Der Streifen wird in *beiden* Panes reserviert, damit die Textspalten gleich breit bleiben und die Zeilen weiter 1:1 fluchten
  - `scrollbar-gutter: stable` wäre hier wirkungslos, da es auf Overlay-Scrollbars nicht greift

---

## [0.7.0] – 2026-07-27

### Added
- Höhe des Merge-Bereichs ist durch Ziehen an seiner Oberkante verstellbar. Der Griff erscheint nur im ausgeklappten Zustand und färbt sich beim Ziehen in der Akzentfarbe
  - Grenzen: der Merge-Bereich behält mindestens 90 px, der Diff-Bereich mindestens 120 px
  - Nach dem Ziehen wird das Ergebnis wieder auf die Diff-Panes ausgerichtet

### Changed
- Der `ResizeObserver` der Diff-Panes löst den Zeilenhöhen-Ausgleich nur noch bei geänderter **Breite** aus. Nur sie verschiebt die Umbruchpunkte; ohne diese Prüfung wäre der Ausgleich bei jedem Frame des Ziehens gelaufen
- Die Zeiger-Listener beim Ziehen hängen an `window` statt am Griff — der Zeiger verlässt den 6 px schmalen Streifen sofort, `setPointerCapture` entfällt damit

---

## [0.6.0] – 2026-07-27

### Changed
- Navigation folgt jetzt dem Auswahlmodus: blockweise wird von Block zu Block gesprungen, zeilenweise von Diff-Zeile zu Diff-Zeile. Der Zähler zeigt entsprechend die Anzahl der Ziele im aktiven Modus
- Markierung umfasst das ganze Navigationsziel statt nur der ersten Zeile — im Blockmodus also alle Zeilen des Blocks, im Zeilenmodus die einzelne Zeile
- Beim Umschalten des Modus bleibt die Position erhalten: es wird das Ziel gewählt, das die bisher markierte erste Zeile enthält

### Fixed
- Der Rahmen des Navigationsziels ist über mehrere Zeilen durchgehend. Das bisherige `outline` hätte jede Zeile einzeln eingekästelt; die Kanten werden nun je Zeilenposition (`nav-first`/`nav-mid`/`nav-last`/`nav-single`) über `inset`-Schatten gesetzt

### Internal
- `diffGroups` führt pro Block die Liste seiner Zeilen (`rows`); die Navigationsziele liegen in `navList`, `currentDiff` wurde zu `currentNav`

---

## [0.5.3] – 2026-07-27

### Changed
- Modus-Switch beidseitig beschriftet: „Auswahlmodus: Zeilenweise [Switch] Blockweise". Der Thumb wandert zur aktiven Seite, das aktive Label wird in der Akzentfarbe `#23a96a` und halbfett hervorgehoben
- Die Zustandsklasse `.on` steht dadurch für *Blockweise* (Label rechts), nicht mehr für `lineMode` — Voreinstellung Blockweise zeigt den Thumb also rechts

---

## [0.5.2] – 2026-07-27

### Changed
- Thumb der Toggle-Switches (Theme und Auswahl-Modus) ist jetzt grün `#23a96a` statt in der Akzentfarbe; als Variable `--switch` hinterlegt und in beiden Themes identisch

---

## [0.5.1] – 2026-07-27

### Changed
- Umschalter zwischen Block- und Zeilenauswahl ist jetzt ein Toggle-Switch in derselben Optik wie der Theme-Umschalter, statt eines Buttons
- Die Switch-Styles (`.toggle-track`, `.toggle-thumb`) werden von beiden Schaltern gemeinsam genutzt; die Thumb-Verschiebung des Theme-Switches ist dabei auf `.theme-toggle` eingeschränkt worden — die Regel galt vorher für jeden `.toggle-thumb`, wodurch der Modus-Switch im Light-Theme mitgewandert wäre
- Verwaiste CSS-Regel `.nav-btn.active-mode` entfernt

---

## [0.5.0] – 2026-07-27

### Changed
- Merge-Bedienung von A/B-Buchstaben auf Pfeile umgestellt, je einer auf der eigenen Seite statt beide links:
  - Pane A: `→` hinter dem Text, Pane B: `←` vor dem Text — beide direkt an der Mittellinie, einander gegenüber
  - Die Pfeile zeigen zur Gegenseite und bedeuten „diese Version gilt"
  - Auswahl bleibt exklusiv (A oder B oder offen); erneutes Klicken hebt sie auf
  - Einheitliche blaue Aktivfarbe — welche Seite gewählt ist, ergibt sich aus der Position
  - Klicks werden jetzt in beiden Panes ausgewertet, vorher nur in Pane A
  - Bei umgebrochenen Zeilen sitzt der Pfeil oben auf Höhe der ersten Textzeile statt vertikal zentriert
- Die merge-cell ist in beiden Panes gleich breit (26 px), nur spiegelbildlich angeordnet — die Textspalten bleiben damit gleich breit und die Zeilen fluchten weiterhin 1:1

---

## [0.4.1] – 2026-07-27

### Fixed
- Tausch-Button saß nicht mittig, sondern direkt links neben Datei B: `max-width: 300px` begrenzte das Dateinamen-Feld, während die umgebende `.upload-group` per `flex: 1` auf ~512 px wuchs — der Leerraum entstand innerhalb der Gruppe A, rechts vom Feld. Feld ist jetzt `display: block` ohne `max-width` und füllt seine Gruppe; die Abstände links und rechts des Buttons betragen damit beide 12 px. Lange Dateinamen werden weiterhin per `text-overflow: ellipsis` abgeschnitten

---

## [0.4.0] – 2026-07-27

### Added
- Doppelpfeil-Button (⇄) zwischen den beiden Dateiauswahlen tauscht Original und Vergleichsdatei; ist bereits ein Vergleich aktiv, wird er sofort neu berechnet
  - Bereits getroffene A/B-Entscheidungen werden dabei verworfen: der Tie-Break im LCS ist nicht symmetrisch, `diffLines(B,A)` kann also anders gruppieren als `diffLines(A,B)` — eine Übernahme per Zeilenindex würde Entscheidungen an falsche Stellen hängen
  - Der Button ist deaktiviert, solange keine Datei geladen ist

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
