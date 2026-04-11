# Brain Protocol

Dieses Projekt hat eine Wissensbasis unter `.brain/`. Sie enthält ausschließlich Wissen, das nicht sauber aus dem Code ableitbar ist und das Fehlannahmen, Rückfragen oder Fehländerungen reduziert: Design-Entscheidungen, Feature-Ziele, bewusste Constraints, Modul-Zusammenhänge, Begriffsdefinitionen.

Was direkt und sicher aus dem Code ablesbar ist, gehört nicht hier.

## Lesestrategie

1. `CLAUDE.md` immer lesen – Projektüberblick, Design-Prinzipien, Domain-Index
2. Domain-Index in `CLAUDE.md` prüfen – passende Domain-Datei aus `.brain/domains/` laden wenn Trigger zutrifft
3. `.brain/glossary.md` laden wenn Begriffe unklar sind oder Fachvokabular auftaucht
4. Maximal 2–3 Zusatzdateien laden, bevor in den Code gegangen wird
5. Erst bei Lücken Code lesen

Nicht alle Domain-Dateien laden. Nur die eindeutig relevante.

## Kanonische Wahrheit

Wenn Aussagen kollidieren, gilt diese Reihenfolge:

1. Explizite User-Aussage in der aktuellen Session
2. Verifiziertes Verhalten aus Code oder Laufzeit
3. Bestehender bestätigter Brain-Eintrag
4. README / AGENTS / andere Projektdoku
5. Inferenz aus Kontext

Konflikte nie stillschweigend auflösen.

## Knowledge-Operationen

Drei Operationen – ausgelöst durch User-Befehl oder Vorschlag:

- `brain init` – Projekt initial erschließen (`.brain/prompts/init.md`)
- `brain capture` – Erkenntnisse aus dieser Session festhalten (`.brain/prompts/capture.md`)
- `brain audit` – Wissensbasis prüfen und bereinigen (`.brain/prompts/audit.md`)

**Trigger-Hinweise:**
- Session mit substantiellen Erkenntnissen → am Ende vorschlagen: "Soll ich `brain capture` ausführen?"
- `audit.lastRun` + `audit.intervalDays` überschritten → hinweisen: "Ein Audit steht an."
- Geänderter oder untersuchter Code passt zu keiner Domain-Datei → Coverage-Lücke benennen und Capture/Audit vorschlagen
- `brain init` nie automatisch vorschlagen – nur auf expliziten Wunsch.

## Pflege-Loop

- `brain init` erzeugt oder aktualisiert `CLAUDE.md`, `.brain/glossary.md`, Domain-Dateien, `.brain/config.json` und `.brain/state.json`.
- `brain capture` aktualisiert betroffene Wissensdateien, setzt `modified` / `review_after` und schreibt den Capture-Status nach `.brain/state.json`.
- `brain audit` prüft Inhalte und Basis-System gemeinsam: Domains, Glossar, Template-/Prompt-Konsistenz, State-Pflege.
- Wenn `review_after` überschritten ist oder Einstiegspunkte einer Domain sichtbar geändert wurden, ist die Domain review-fällig.

## State-Semantik

- `lastRun` ist der Zeitstempel des **erfolgreich abgeschlossenen** Durchlaufs einer Knowledge-Operation.
- `lastRun` wird **am Ende** geschrieben, nachdem alle Wissensdateien erfolgreich aktualisiert wurden.
- Abgebrochene oder nicht bestätigte Durchläufe schreiben keinen neuen `lastRun`.
- Wenn ein historischer `lastRun` unbekannt ist, darf er bei einer bestätigten Re-Konsolidierung auf den aktuellen Stand gesetzt werden.

## Grundregeln

- Kein Code schreiben während einer Knowledge-Operation.
- Unsicherheit nie stillschweigend auflösen – kennzeichnen oder nachfragen.
- Das System ist semi-selbstpflegend: die KI erkennt Pflegebedarf und schlägt Updates vor. Änderungen werden bestätigt. Nur offensichtliche Strukturkorrekturen (Pfade, Links, Tippfehler) werden direkt ausgeführt.
