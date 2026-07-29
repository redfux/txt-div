# Architektur

Technische Entscheidungen und Aufbau von txt-div.

---

## Überblick

txt-div ist eine **Single-File Web-App** — der gesamte Code (HTML, CSS, JavaScript) liegt in `index.html`. Es gibt kein Build-System, keine Abhängigkeiten, keinen Server. Die Datei kann direkt im Browser geöffnet werden.

Der Dateiname `index.html` ist bewusst gewählt: GitHub Pages sucht standardmäßig genau diese Datei, sodass die App ohne zusätzliche Konfiguration auch live läuft.

## Technologie-Entscheidungen

### Kein Framework
Bewusst kein React, Vue o. ä. — der Umfang rechtfertigt keinen Framework-Overhead. Vanilla JS reicht aus und hält die Datei schlank.

### Kein CDN, keine externen Ressourcen
Alle Ressourcen sind inline. Eine Content-Security-Policy (`connect-src 'none'`) verhindert externe Nachladungen technisch. Es gibt keinen `localStorage`, keine Cookies, kein Tracking — die App speichert nichts und sendet nichts.

### Versionsnummer an einer Stelle
Die Version steht ausschließlich in der Konstante `APP_VERSION` im Skript; der Footer wird daraus gefüllt. Vorher war sie im Markup hart kodiert und lief bei zwei Releases (0.3.4, 0.3.5) versehentlich hinterher.

### Diff-Algorithmus: LCS (Longest Common Subsequence)
- **Zeilenebene:** LCS über die Zeilen-Arrays beider Dateien, liefert die Zuordnung gleicher Zeilen.
- **Zeichenebene:** Für jede „geänderte" Zeile ein zweiter LCS über die Zeichen, um die exakten Unterschiede hervorzuheben.
- **Nicht symmetrisch:** Der Tie-Break `dp[i+1][j] >= dp[i][j+1]` bevorzugt bei Gleichstand eine Richtung. `diffLines(B,A)` kann daher anders gruppieren als `diffLines(A,B)` — weshalb Merge-Entscheidungen beim Dateitausch verworfen und nicht per Index übertragen werden.

### Grenzen des Verfahrens und Näherung

Die exakte DP-Matrix kostet O(m × n) an Speicher und Zeit. Ab `MAX_LCS_CELLS` (64 Mio. Zellen, ≈ 8.000 × 8.000 Zeilen) übernimmt daher die greedy Näherung `fastLcs`.

Messwerte für die exakte Berechnung:

| Zeilen | Zellen | Matrix | Zeit (nur LCS) |
|---|---|---|---|
| 2.000 | 4 Mio. | 8 MB | 37 ms |
| 4.000 | 16 Mio. | 31 MB | 130 ms |
| 6.000 | 36 Mio. | 69 MB | 267 ms |
| 7.988 | 63,8 Mio. | ~122 MB | 390 ms |

Die Schwelle ist am Hauptanwendungsfall bemessen: Switch- und WLC-Konfigurationen mit einigen tausend Zeilen. Zwei echte Configs mit je 4.628 Zeilen benötigen 21,4 Mio. Zellen — bei der früheren Grenze von 25 Mio. blieben also nur 7 % Reserve.

Der maximale DP-Wert entspricht der Länge der gemeinsamen Teilfolge und bleibt bei 8.000 Zeilen weit unter der `Uint16Array`-Grenze von 65.535.

### Verhalten der Näherung

`fastLcs` läuft einmal durch A und nimmt jeweils den frühesten noch verfügbaren Treffer in B. Dazu hält es **alle** Positionen je Zeileninhalt vor und bestimmt den ersten Treffer ab dem Suchzeiger per Binärsuche.

Eine frühere Fassung speicherte nur das *erste* Vorkommen. Das brach bei Dateien mit vielen wiederkehrenden Zeilen zusammen: sobald der Suchzeiger diese einzige Position passiert hatte, wurde für jede weitere gleichlautende Zeile nie mehr ein Treffer gefunden. Auf zwei echten Switch-Configs (4.628 Zeilen, nur 12 % davon eindeutig) fand sie **529** statt 4.450 gemeinsamer Zeilen.

Güte der aktuellen Fassung, gemessen an denselben Dateien gegen die exakte Berechnung:

| Änderungsart | Anteil des Optimums |
|---|---|
| Einfügung, geänderte Zeilen | **100 %** |
| Löschung | ~50 % |
| verschobener Block | ~50 % |

Sie bleibt also eine Näherung: weil stets der früheste Treffer genommen wird, verliert sie bei Löschungen und Verschiebungen die Synchronisation. Deshalb zeigt die Oberfläche eine Hinweisleiste, sobald sie greift.

Die saubere Lösung wäre ein Myers-Diff (O((m+n)·d) Zeit, O(m+n) Speicher, exakt). Bewusst zurückgestellt, siehe `features.md`.

### Obergrenzen für Eingabedateien

| Grenze | Wert | Geprüft |
|---|---|---|
| `MAX_FILE_BYTES` | 5 MB | vor dem Lesen — die Datei wird gar nicht geladen |
| `MAX_FILE_LINES` | 20.000 | im `onload`, da die Zeilenzahl vorher unbekannt ist |

Beide Kriterien sind nötig, weil sie unabhängig voneinander greifen. Gemessen: 200 Zeilen à 2.000 Zeichen (0,4 MB) brauchen 4.816 ms, 50.000 kurze Zeilen (2,9 MB) nur 5.811 ms — die Dateigröße allein ist also kein verlässlicher Indikator. Maßgeblich sind Zeilenanzahl, Zeilenlänge und der Anteil geänderter Zeilen.

Gezählt wird über `countLines()` mit einer Schleife über `charCodeAt`, nicht über `split('\n')` — letzteres würde bei einer 5-MB-Datei alle Zeilen als Array materialisieren.

Der Engpass oberhalb der Grenze ist nicht der Algorithmus, sondern das DOM: pro Zeile entstehen 11 Knoten, bei 50.000 Zeilen also 550.000.

## Datenfluss

```
Datei A (FileReader) ──┐
                       ├─► diffLines() ──► allOps[] ──┬─► Diff-Panes rendern
Datei B (FileReader) ──┘                              ├─► diffGroups[] (Blöcke)
                                                      ├─► navList[] (Navigationsziele)
                                                      ├─► Minimap
                                                      └─► computeResult() ─► Merge-Ergebnis
```

`allOps` ist die zentrale Struktur. Jeder Eintrag beschreibt eine Zeilenposition:

```js
{ type: 'equal' | 'added' | 'removed' | 'changed',
  textA: string | null,     // null = in A nicht vorhanden
  textB: string | null,     // null = in B nicht vorhanden
  choice: null | 'A' | 'B', // Merge-Entscheidung
  groupIdx: number }        // Index des Diff-Blocks, -1 bei 'equal'
```

## Merge-Modell

Basis ist immer Datei A. Pro Diff-Zeile gibt es drei Zustände: offen (`null`), explizit A oder B. Offen verhält sich wie A — die Basis bleibt unverändert.

`computeResult()` liefert zwei Dinge getrennt:
- **`lines`** — nur echte Textzeilen, das ist der Export (Kopieren/Speichern)
- **`items`** — Zeilen *und* Lücken-Marker, das ist die Anzeige

Die Trennung ist notwendig, weil weggelassene Zeilen sichtbar markiert werden sollen, die Marker aber nicht in der Datei landen dürfen.

**Zwei Ursachen für eine Lücke** werden unterschieden, da „entfernt" im zweiten Fall sachlich falsch wäre:

| Ursache | Beschriftung |
|---|---|
| A-Zeile fällt weg (gewählte B-Version hat sie nicht) | „N Zeilen entfernt" |
| B-Zeile wurde nicht gewählt | „N Zeilen nicht übernommen" |
| beides in derselben Lücke | „N Zeilen ausgelassen" |

## Scroll-Synchronisation

Drei Bereiche werden synchron gehalten: Pane A, Pane B und das Merge-Ergebnis.

### Owner-Prinzip statt Sperr-Flag
Ein Boolean nach dem Muster `locked = true; el.scrollTop = x; locked = false;` funktioniert **nicht**: Scroll-Events feuern asynchron, das Flag ist beim Eintreffen des Echos längst zurückgesetzt. Stattdessen behält über `claimScroll(name)` derjenige Bereich die Kontrolle, der zuletzt gescrollt hat — für `OWNER_HOLD_MS` (150 ms). Echos anderer Bereiche werden in diesem Fenster verworfen.

### Zeilengenau statt proportional
Diff-Panes und Ergebnis haben unterschiedlich viele Zeilen. Eine proportionale Umrechnung driftet, daher gibt es zwei Zuordnungstabellen:

- `opToLine[opIndex]` → Index in `items` (Diff → Ergebnis)
- `lineToOp[itemIndex]` → op-Index (Ergebnis → Diff)

Beide zeigen auf Positionen in `items`, nicht in `lines`, weil der Sync über die DOM-Kinder arbeitet und die Marker dort mitzählen.

### offsetTop-Bezugsrahmen
`.pane-content` und `.result-content` sind `position: relative`. Nur dadurch sind sie `offsetParent` ihrer Zeilen und `row.offsetTop` ist direkt als `scrollTop` verwendbar. Ohne das enthält jeder Wert den Abstand vom Seitenanfang — je Bereich ein anderer Sockel.

## Zeilenausrichtung

Beide Panes enthalten dieselben drei Bausteine pro Zeile, nur spiegelbildlich:

```
Pane A:  line-num | line-text | merge-cell
Pane B:  line-num | merge-cell | line-text
```

Da die `merge-cell` in beiden Panes gleich breit ist (26 px), bleibt die Textspalte identisch breit — Zeilen brechen an denselben Stellen um.

Für den Fall, dass eine Zeile trotzdem höher wird als ihre Gegenzeile (Umbruch, unterschiedliche Textlänge), gleicht `alignRowHeights()` jedes Zeilenpaar auf die größere Höhe an. Messen und Schreiben laufen in getrennten Durchgängen, damit es bei zwei Reflows bleibt statt zwei pro Zeile. Ein `ResizeObserver` löst den Ausgleich **nur bei Breitenänderung** aus — die Höhe verschiebt keine Umbruchpunkte, und beim Ziehen des Merge-Bereichs würde er sonst in jedem Frame laufen.

## Navigation

`navList` enthält die Navigationsziele des aktuellen Auswahlmodus. Jedes Ziel ist eine Liste zusammenhängender op-Indizes:

- **Blockweise:** ein Ziel pro Diff-Block (`diffGroups[].rows`)
- **Zeilenweise:** ein Ziel pro Diff-Zeile

Beim Moduswechsel bleibt die Position erhalten: gewählt wird das Ziel, das die bisher markierte erste Zeile enthält.

Der Rahmen um das Ziel entsteht über `inset`-Schatten pro Kante (`nav-first` / `nav-mid` / `nav-last` / `nav-single`). Ein `outline` würde jede Zeile einzeln einkästeln statt einen durchgehenden Rahmen zu ziehen.

## DOM-Struktur (nach dem Vergleich)

```
.diff-wrapper
  .diff-container
    .diff-pane                    (Spalte A)
      .pane-header
      .pane-content               (scrollbar, id="scrollA", position: relative)
        .line[.added|.removed|.changed|.empty][.current-diff .nav-*][.chosen]
          .line-num
          .line-text
          .merge-cell             → button.merge-btn (→)
    .diff-pane                    (Spalte B, merge-cell vor line-text, ←)
  .minimap
    .minimap-track                (farbige Segmente)
    .minimap-viewport             (aktuelle Position)

.result-area[.open]
  .result-resizer                 (Ziehgriff, nur sichtbar wenn offen)
  .result-toggle
  .result-panels
    .result-pane
      .result-pane-header         → Kopieren / Speichern
      .result-content             (id="resultContent", position: relative)
        .line[.diff-line]         Ergebniszeile
        .line.gap-line            Lücken-Marker (nicht im Export)
```

## Theming

Umgesetzt über CSS Custom Properties auf dem `<html>`-Element. Umgeschaltet wird nur `data-theme="light"` bzw. `"dark"` — kein Neu-Rendern nötig. Ein Inline-Skript im `<head>` setzt beim Laden das System-Theme (`prefers-color-scheme`), bevor der erste Frame gezeichnet wird; dadurch entsteht kein Aufblitzen.

Die Verschiebung des Theme-Switch-Thumbs ist auf `.theme-toggle` eingeschränkt. Ohne diese Einschränkung würde die Regel jeden `.toggle-thumb` treffen und der Modus-Switch im Light-Theme mitwandern.

## Fehlerbehandlung

- **Dateilesen:** `reader.onerror`, `reader.onabort` und ein `try/catch` um `readAsText()`. Die Meldung erscheint an der betroffenen Datei, der Slot wird geleert und „Vergleichen" deaktiviert.
- **Vergleich:** `runCompare()` kapselt `compare()`. Bei einem unerwarteten Fehler erscheint ein Hinweis statt einer halb aufgebauten Ansicht; Details gehen in die Konsole.
- **Zwischenablage:** `navigator.clipboard.writeText()` kann bei fehlender Berechtigung oder unsicherem Kontext scheitern; der Button quittiert Erfolg bzw. Fehlschlag kurz.

## Bekannte Grenzen

- **Kein Dateipfad:** Browser geben ihn nicht heraus. Das `File`-Objekt kennt keinen, `input.value` liefert bewusst `C:\fakepath\<name>`, `webkitRelativePath` ist nur bei Ordner-Auswahl gefüllt. Angezeigt werden daher Größe und Änderungsdatum.
- **Näherung zwischen 8.000 und 20.000 Zeilen:** In diesem Bereich ist die exakte Matrix nicht mehr tragfähig (bei 20.000 × 20.000 wären es 400 Mio. Zellen und ~800 MB). Die Oberfläche weist darauf hin. Bei Einfügungen und Änderungen liefert die Näherung dort das exakte Ergebnis, bei Löschungen und verschobenen Blöcken etwa die Hälfte des Optimums.
- **Bedienelemente ohne Tastaturzugang:** Theme-Switch, Modus-Switch, Ergebnis-Aufklappen, Ziehgriff und Minimap sind noch nicht fokussierbar.
- **Mobil eng:** Bei 375 px Breite bleiben pro Spalte rund 93 px Textbreite. Technisch responsive, praktisch kaum nutzbar.
- **Abweichung vom Masterprompt:** Google Material Design ist nicht als gestalterische Grundlage verwendet; das Projekt nutzt ein eigenes Farbschema. Ebenso ist das GitHub-Repo öffentlich statt privat — das ist Voraussetzung für GitHub Pages im kostenlosen Plan.
