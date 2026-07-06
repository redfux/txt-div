# Architektur

Technische Entscheidungen und Aufbau von txt-div.

---

## Überblick

txt-div ist eine **Single-File Web-App** – der gesamte Code (HTML, CSS, JavaScript) befindet sich in einer einzigen Datei (`txt-div.html`). Es gibt kein Build-System, keine Abhängigkeiten, keinen Server. Die Datei kann direkt im Browser geöffnet werden.

## Technologie-Entscheidungen

### Kein Framework
Bewusst kein React, Vue o. ä. – der Umfang der App rechtfertigt keinen Framework-Overhead. Vanilla JS reicht vollständig aus und hält die Datei schlank.

### Kein CDN / keine externen Ressourcen
Alle Ressourcen sind inline. Eine Content-Security-Policy verhindert technisch externe Nachladungen. Ziel: vollständige Offline-Fähigkeit.

### Diff-Algorithmus: LCS (Longest Common Subsequence)
- **Zeilenebene:** LCS über das Array der Zeilen beider Dateien. Liefert exakte Zuordnung gleicher Zeilen.
- **Zeichenebene:** Für jede als "geändert" erkannte Zeile wird erneut ein LCS über die einzelnen Zeichen berechnet, um die exakten Zeichenunterschiede hervorzuheben.
- **Fallback für große Dateien:** Bei mehr als 4.000.000 Zellen (m × n) in der DP-Matrix wird auf einen greedy-basierten Näherungsalgorithmus (`fastLcs`) umgeschaltet, um Speicherprobleme bei sehr großen Dateien zu vermeiden.

## Datenfluss

```
Datei A (FileReader) ──┐
                        ├─► diffLines() ──► ops[] ──► DOM rendern
Datei B (FileReader) ──┘                         └──► Minimap aufbauen
                                                  └──► diffGroups[] für Navigation
```

## DOM-Struktur (nach Vergleich)

```
.diff-wrapper
  .diff-container
    .diff-pane          (Spalte A)
      .pane-header
      .pane-content     (scrollbar, id="scrollA")
        .line[.added|.removed|.changed|.empty]
          .line-num
          .line-text
    .diff-pane          (Spalte B, analog)
  .minimap
    .minimap-track      (farbige Segmente)
    .minimap-viewport   (aktuelle Position)
```

## Theming

Umgesetzt über CSS Custom Properties (`--bg`, `--text`, `--accent` etc.) auf dem `<html>`-Element. Das Umschalten zwischen Dark und Light Mode setzt nur `data-theme="light"` bzw. `data-theme="dark"` – kein JS-seitiges Neu-Rendern nötig.
