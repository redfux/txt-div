# Bugs

Bekannte Fehler, gemeldete Probleme und deren Behebung.

---

## Behoben

### Navigation sprang zu falschen Positionen
- **Version:** 0.1.0
- **Problem:** Die Weiter/Zurück-Buttons sprangen nicht zu den tatsächlichen Unterschieden, sondern zu zufälligen Stellen im Dokument.
- **Ursache:** Die Scrollposition wurde über `rowIndex × lineHeight` berechnet, was bei variablen Zeilenhöhen falsche Werte ergab. Zusätzlich verhinderte `scroll-behavior: smooth` die sofortige Synchronisation beider Spalten.
- **Lösung:** Scrollposition wird nun über `element.offsetTop` des tatsächlichen DOM-Elements bestimmt. `scroll-behavior: smooth` wurde entfernt. Beide Spalten werden beim Sprung gleichzeitig gesetzt, während der Sync-Lock aktiv ist.

### Spalten scrollten nicht synchron
- **Version:** 0.1.0
- **Problem:** Nach der Navigation per Buttons liefen die beiden Spalten auseinander.
- **Ursache:** Die Sync-Logik verwendete eine lokale `syncing`-Variable, die beim programmatischen Scrollen (Navigation) nicht gesetzt wurde.
- **Lösung:** Gemeinsame `syncLocked`-Variable, die sowohl vom Scroll-Event-Handler als auch von der Navigationsfunktion genutzt wird.
