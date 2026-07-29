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
- **Fallback für große Dateien:** Ab mehr als 4.000.000 Zellen (m × n) in der DP-Matrix übernimmt eine greedy Näherung (`fastLcs`), um Speicherprobleme zu vermeiden.
- **Nicht symmetrisch:** Der Tie-Break `dp[i+1][j] >= dp[i][j+1]` bevorzugt bei Gleichstand eine Richtung. `diffLines(B,A)` kann daher anders gruppieren als `diffLines(A,B)` — weshalb Merge-Entscheidungen beim Dateitausch verworfen und nicht per Index übertragen werden.

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
- **Keine Größenbegrenzung:** Sehr große Dateien können den Browser blockieren, da LCS O(m × n) ist und das Rendering alle Zeilen erzeugt. `runCompare()` fängt den Fehlerfall ab, verhindert ihn aber nicht.
- **Bedienelemente ohne Tastaturzugang:** Theme-Switch, Modus-Switch, Ergebnis-Aufklappen, Ziehgriff und Minimap sind noch nicht fokussierbar.
- **Mobil eng:** Bei 375 px Breite bleiben pro Spalte rund 93 px Textbreite. Technisch responsive, praktisch kaum nutzbar.
- **Abweichung vom Masterprompt:** Google Material Design ist nicht als gestalterische Grundlage verwendet; das Projekt nutzt ein eigenes Farbschema. Ebenso ist das GitHub-Repo öffentlich statt privat — das ist Voraussetzung für GitHub Pages im kostenlosen Plan.
