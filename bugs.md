# Bugs

Bekannte Fehler, gemeldete Probleme und deren Behebung.

---

## Behoben

### Navigation sprang zu falschen Positionen
- **Version:** 0.1.0
- **Problem:** Die Weiter/Zurück-Buttons sprangen nicht zu den tatsächlichen Unterschieden, sondern zu zufälligen Stellen im Dokument.
- **Ursache:** Die Scrollposition wurde über `rowIndex × lineHeight` berechnet, was bei variablen Zeilenhöhen falsche Werte ergab. Zusätzlich verhinderte `scroll-behavior: smooth` die sofortige Synchronisation beider Spalten.
- **Lösung:** Scrollposition wird über `element.offsetTop` des tatsächlichen DOM-Elements bestimmt. `scroll-behavior: smooth` wurde entfernt.
- **Nachtrag:** Der Sprung blieb dennoch um einen konstanten Betrag verschoben — siehe „Scrollpositionen systematisch verschoben" (0.3.6).

### Spalten scrollten nicht synchron
- **Version:** 0.1.0
- **Problem:** Nach der Navigation per Buttons liefen die beiden Spalten auseinander.
- **Ursache:** Die Sync-Logik verwendete eine lokale `syncing`-Variable, die beim programmatischen Scrollen nicht gesetzt wurde.
- **Lösung:** Gemeinsame `syncLocked`-Variable für Scroll-Handler und Navigation.
- **⚠️ Diese Lösung war unzureichend** und wurde in 0.3.6 ersetzt — siehe „Sperr-Flag gegen Scroll-Echos wirkungslos".

### Merge-Buttons nicht bündig zu den Zeilen
- **Version:** 0.3.1
- **Problem:** Die Merge-Buttons saßen neben den Zeilen, aber nicht auf deren Höhe.
- **Ursache:** Sie lagen in einer eigenen Spalte, die per JS mitscrollte. Bei umgebrochenen Zeilen stimmten die Höhen nicht mehr überein.
- **Lösung:** Die Buttons wurden direkt in die Diff-Zeile integriert und bewegen sich damit zwangsläufig mit ihr.

### Ergebnis zeigte zwei Dateien statt einer
- **Version:** 0.3.1
- **Problem:** Der Merge-Bereich zeigte zwei Ergebnis-Spalten, obwohl es nur ein Ergebnis gibt.
- **Lösung:** Auf ein Ergebnis über die volle Breite reduziert; Basis ist immer Datei A.

### Nur übernommene Zeilen waren im Ergebnis markiert
- **Version:** 0.3.3
- **Problem:** Im Ergebnis waren nur B-Übernahmen hervorgehoben, behaltene A-Zeilen an Diff-Positionen nicht — man sah nicht, wo überhaupt ein Unterschied bestand.
- **Lösung:** Alle Zeilen aus Diff-Positionen werden hellblau hinterlegt, unabhängig von der Wahl.

### Textspalten der beiden Panes unterschiedlich breit
- **Version:** 0.3.3
- **Problem:** Zeilen brachen in Pane A und B an verschiedenen Stellen um und liefen dadurch auseinander.
- **Ursache:** Nur Pane A hatte rechts eine Zelle für die Merge-Buttons; die Textspalte war dort schmaler.
- **Lösung:** Beide Panes erhalten eine gleich breite Zelle, sodass die Textbreite identisch ist.

### Scroll-Sync funktionierte nur in eine Richtung
- **Version:** 0.3.4
- **Problem:** Scrollen in den Diff-Panes bewegte das Merge-Ergebnis, umgekehrt nicht.
- **Lösung:** Eigener Scroll-Handler auf dem Ergebnis-Bereich, der auf die Diff-Panes zurücksynchronisiert.

### Scroll-Rückkopplung bei geöffnetem Merge-Bereich
- **Version:** 0.3.5
- **Problem:** Bei geöffnetem Ergebnis-Bereich sprang die Ansicht an den Rand und ließ sich nicht mehr scrollen.
- **Ursache:** Die Synchronisation in beide Richtungen schaukelte sich gegenseitig auf.
- **Lösung:** Erster Anlauf über das Sperr-Flag — die eigentliche Ursache wurde erst in 0.3.6 gefunden.

### Theme-Umschalter zeigte den falschen Zustand
- **Version:** 0.3.5
- **Problem:** Der Schalter beschriftete das Wechselziel statt des aktiven Themes — bei „Dark" war das helle Theme aktiv.
- **Lösung:** Beschriftung zeigt das aktive Theme.

### Scrollpositionen systematisch verschoben
- **Version:** 0.3.6
- **Problem:** Der Merge-Bereich sprang beim Scrollen an den Anfang oder das Ende und ließ sich nicht mehr bewegen. Auch die Vor/Zurück-Navigation landete neben dem Ziel.
- **Ursache:** `offsetTop` liefert den Abstand zum **`offsetParent`**, nicht zum Scroll-Container. Da weder `.pane-content` noch `.result-content` positioniert waren, enthielt jeder Wert den Abstand vom Seitenanfang — je Bereich ein anderer Sockel (Diff-Pane 186 px, Ergebnis 712 px). Zeile 17 meldete `577` in Pane A, aber `1103` im Ergebnis, bei identischer Zeilenhöhe von 23 px (korrekt wären 383).
- **Lösung:** `position: relative` auf beide Scroll-Container. Damit ist `offsetTop` relativ zum Container und direkt als `scrollTop` verwendbar.

### Sperr-Flag gegen Scroll-Echos wirkungslos
- **Version:** 0.3.6
- **Problem:** Die Scroll-Synchronisation driftete und lief an die Ränder.
- **Ursache:** Das Muster `syncLocked = true; el.scrollTop = x; syncLocked = false;` schützt nicht, weil Scroll-Events **asynchron** feuern — beim Eintreffen des Echo-Events war das Flag längst zurückgesetzt. Bei zwei gleich hohen Panes fiel das nicht auf (identischer Wert, idempotent); bei der Umrechnung auf das Ergebnis driftete der Wert.
- **Lösung:** Owner-Prinzip (`claimScroll`): wer zuletzt gescrollt hat, behält die Kontrolle für 150 ms. Zusätzlich wurde der Sync von proportional auf zeilengenau umgestellt.

### Merge-Ergebnis sprang beim A/B-Klick nach oben
- **Version:** 0.3.6
- **Problem:** Jede Merge-Entscheidung riss die Ansicht an den Dateianfang.
- **Ursache:** `renderResult()` baut den Bereich per `innerHTML` neu auf, wodurch `scrollTop` auf 0 fällt.
- **Lösung:** Nach dem Neuaufbau wird die Position aus den Diff-Panes wiederhergestellt.

### Umgebrochene Zeilen liefen auseinander
- **Version:** 0.3.7
- **Problem:** Brach eine Zeile auf mehrere Bildschirmzeilen um, behielt die Gegenzeile ihre einfache Höhe — die Leerräume waren zu klein und die Spalten verschoben sich.
- **Ursache:** Kein Höhenausgleich zwischen den Zeilenpaaren. Betraf Leerzeilen ohne Gegenstück ebenso wie geänderte Zeilen unterschiedlicher Länge.
- **Lösung:** `alignRowHeights()` gleicht jedes Paar auf die größere Höhe an; ein `ResizeObserver` richtet nach Breitenänderungen neu aus.

### Zeilenumbruch mitten im Wort
- **Version:** 0.3.7
- **Problem:** Lange Zeilen wurden an beliebiger Stelle getrennt, auch innerhalb von Wörtern.
- **Ursache:** `word-break: break-all` trennt bedingungslos.
- **Lösung:** `overflow-wrap: anywhere` mit `word-break: normal` und `min-width: 0` — Umbruch an Leerzeichen, nur ein einzelnes zu langes Wort wird noch getrennt.

### Tausch-Button saß nicht mittig
- **Version:** 0.4.1
- **Problem:** Der Doppelpfeil klebte an Datei B statt mittig zwischen den Feldern zu stehen.
- **Ursache:** Nicht die Position des Buttons, sondern `max-width: 300px` am Dateinamen-Feld: die `.upload-group` wuchs per `flex: 1` auf ~512 px, der Leerraum entstand innerhalb von Gruppe A.
- **Lösung:** Feld auf `display: block` ohne `max-width`; beide Abstände betragen nun 12 px.

### Scrollbar verdeckte die Merge-Pfeile
- **Version:** 0.7.1
- **Problem:** Nach dem Scrollen lag die eingeblendete Scrollbar über den Pfeilen in Pane A.
- **Ursache:** Overlay-Scrollbars (macOS) legen sich über den Inhalt, statt Platz einzunehmen. `scrollbar-gutter: stable` greift auf sie nicht.
- **Lösung:** Beide Panes halten rechts 14 px frei — in beiden, damit die Textspalten gleich breit bleiben.

### Dateilesefehler blieben unbemerkt
- **Version:** 0.9.0
- **Problem:** Schlug das Lesen einer Datei fehl, passierte nichts sichtbares; die App wirkte, als sei nichts geschehen.
- **Ursache:** Es gab weder `reader.onerror` noch eine Absicherung um `readAsText()`.
- **Lösung:** `onerror`, `onabort` und ein `try/catch` melden den Fehler an der betroffenen Datei; der Slot wird geleert und „Vergleichen" deaktiviert. Zusätzlich fängt `runCompare()` unerwartete Fehler im Vergleich ab und zeigt einen Hinweis statt einer halb aufgebauten Ansicht.

---

## Nicht reproduzierbar / verworfen

### Vermeintlich zu geringer Kontrast der Dateiangaben
- **Gemeldet:** 0.8.1 (Masterprompt-Abgleich)
- **Befund:** Eine erste Messung ergab 2,0:1 für die grauen Dateiangaben im Dark Mode. Die Nachmessung ergab **4,85:1** — die Anforderung ist erfüllt.
- **Ursache des Fehlalarms:** Im Messskript wurde das Theme umgeschaltet und unmittelbar danach gelesen. Die Style-Neuberechnung war noch nicht durch, sodass die *helle* Textfarbe (`#555577`) gegen den *dunklen* Hintergrund (`#0f3460`) gemessen wurde.
- **Merke:** Nach `dataset.theme`-Wechsel einen Reflow erzwingen oder in einem separaten Durchlauf messen.
