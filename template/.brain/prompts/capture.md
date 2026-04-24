# Brain Capture

Dieser Prompt wird ausgeführt wenn der User `brain capture` aufruft oder die KI es am Session-Ende automatisch startet.
Ziel: Dauerhaft nützliche Erkenntnisse aus dieser Session in die Wissensbasis einordnen.

## Ausführungsmodus

**Auto-Mode (Default):** Wenn `capture.confirmDefault` in `.brain/config.json` auf `false` steht (Default ab v2.2), läuft Capture autonom. Keine Nachfrage, keine Bestätigung – die KI prüft jeden Kandidaten gegen die harten Aufnahmekriterien und schreibt direkt.

Voraussetzung für Auto-Mode: Die Aufnahmekriterien werden **konsequent streng** angewendet. Lieber zehnmal nichts schreiben als einmal schwache Doku.

**Explizite Ausnahme – Rückfrage trotz Auto-Mode:** Nur bei `conflict`-Kandidaten (siehe Schritt 5), bei denen die Prioritätshierarchie keinen eindeutigen Sieger ergibt. In allen anderen Fällen autonom entscheiden.

## Aufnahmekriterien

Ein Kandidat wird nur aufgenommen wenn **alle drei** zutreffen:

1. **Nicht aus Code ableitbar** – grep, Glob oder eine Handvoll Datei-Reads liefern die Antwort nicht.
2. **Wiederverwendbar** – Nutzen für realistische zukünftige Aufgaben, nicht nur diese Session.
3. **Liefert konkreten Mehrwert** – mindestens einer dieser fünf Nutzen muss deutlich erkennbar sein:
   - **Recherche-Ersparnis** – spart der KI in künftigen Sessions Such-/Leseaufwand, weil sie sonst grep/read wiederholen müsste.
   - **Token-/Kosten-Reduktion** – kuratiertes Wissen ersetzt mehrere Tool-Aufrufe.
   - **Besserer Kontext** – liefert Hintergrund, den die KI sonst erraten oder fehlinterpretieren würde.
   - **Höhere Codequalität durch besseres Verständnis** – macht Zusammenhänge explizit, die zu saubereren Änderungen führen.
   - **Design-Entscheidung explizit** – das Warum hinter Code, gewählte vs. verworfene Alternative, bewusste Constraints.

Wenn auch nur eine der drei Bedingungen nicht zutrifft: `reject`.

Lies vor dem Schreiben `.brain/config.json`, um mindestens diese Werte zu kennen:
- `capture.confirmDefault`
- `capture.minFactsForNewDomain`
- `review.defaultDaysByStatus`
- `files.glossary`

## Harte Ausschlussregeln

Niemals aufnehmen, ohne Ausnahme:
- Task-spezifische Details ohne Wiederverwendungswert ("in dieser Session haben wir X gemacht")
- Framework-Standardverhalten (Laravel-Resource-Routes, Vue-Lifecycle, etc.)
- Code-Zustände ohne Abstraktion ("Datei X hat 150 Zeilen", "Funktion Y ruft Z auf")
- Einmalige Debug-Erkenntnisse (konkrete Fix-Rezepte gehören in Commit-Messages)
- Dateiaufzählungen ohne semantischen Mehrwert
- Vermutungen ohne explizite Kennzeichnung als `<!-- unsicher -->`
- Umformulierungen bestehender Einträge ohne neuen Inhalt
- Erkenntnisse, deren Mehrwert-Kategorie (siehe oben) nicht klar benennbar ist

**Faustregel:** Wenn die KI beim Kandidaten nicht in einem Satz sagen kann _welchen der fünf Mehrwerte er liefert_, ist es `reject`.

## Ablauf

### Schritt 1: Erkenntnisse sammeln

Gehe die aktuelle Session durch und identifiziere:
- Getroffene Design-Entscheidungen mit Begründung
- Aufgedeckte Zusammenhänge zwischen Modulen
- Erkannte Constraints oder Ausnahmen
- Korrigierte Fehlannahmen (eigene oder in bestehender Wissensbasis)
- Feature-Ziele oder Business-Kontext der erläutert wurde
- Typische Fehlannahmen die aufgedeckt wurden
- Stolpersteine die aufgetreten sind
- Themenbereiche die im Code oder in der Session sichtbar waren, aber noch keine passende Domain-Datei haben

### Schritt 2: Kategorisieren

Pro Kandidat eine Kategorie vergeben:

- `accept` – klar relevant, sicher korrekt, passt in bestehende Struktur, Mehrwert-Kategorie eindeutig benennbar
- `reject` – erfüllt Aufnahmekriterien nicht (Default bei Unsicherheit)
- `conflict` – widerspricht einem bestehenden Eintrag in der Wissensbasis

Nur `accept` und `conflict` werden weiterverarbeitet. `reject` still verwerfen.

Es gibt in v2.2 keine eigene `review`-Stufe mehr im Auto-Mode: unsichere Kandidaten werden `reject`. Wer sie trotzdem haben will, kann `brain capture` bei `confirmDefault: true` im Review-Modus ausführen.

### Schritt 3: Ziel bestimmen

Pro `accept`-Kandidat entscheiden:
- `CLAUDE.md` – betrifft das gesamte Projekt (Design-Prinzip, globaler Constraint)
- `.brain/domains/<name>.md` – betrifft eine bestehende Domäne
- `.brain/glossary.md` – neuer Begriff oder Korrektur eines bestehenden
- Neue Domain-Datei – neues Feature das noch nicht erfasst ist → **CLAUDE.md Domain-Index gleichzeitig ergänzen** (Domain-Name, Kernaussage, „Datei laden wenn..."-Spalte)
- `PRIVATE.md` – persönlich, nicht team-relevant
- `none` – kein echter Mehrwert nach nochmaliger Prüfung → `reject`

Wenn ein relevanter Kandidat keiner bestehenden Domain sauber zugeordnet werden kann:
- Coverage-Lücke als Audit-Maßnahme notieren
- Neue Domain-Datei nur anlegen wenn mindestens `capture.minFactsForNewDomain` eigenständige, `accept`-taugliche Fakten vorliegen
- Sonst `reject`, nicht schwache Mini-Domain bauen

### Schritt 4: Deduplizieren

Betroffene Datei lesen und prüfen:
- Bereits enthalten (gleich oder ähnlich)? → `reject`
- Ergänzt bestehenden Eintrag? → einordnen, nicht neu anlegen
- Widerspricht bestehendem Eintrag? → zu `conflict` hochstufen

### Schritt 5: Konflikte auflösen

Für jeden `conflict`-Kandidaten die Prioritätshierarchie anwenden:

1. Explizite User-Aussage in dieser Session
2. Verifikation durch Code-Inspektion
3. Bestehender Domain-Eintrag (verified)
4. Inferenz aus Kontext

Ist der Sieger eindeutig: **autonom** den unterlegenen Eintrag ersetzen. Im Commit-Log der Domain-Datei hinterlässt git die Spur.

Ist der Sieger **nicht eindeutig** (z. B. zwei gleich starke Quellen widersprechen): dem User gebündelt zeigen und fragen. Nur hier darf der Auto-Mode pausieren.

### Schritt 6: Schreiben

Direkt nach bestandener Qualitätsprüfung.

Beim Schreiben:
- `modified` immer auf heute setzen
- `review_after` auf Basis von `.brain/config.json > review.defaultDaysByStatus` aktualisieren, wenn sich die Aussage einer Domain **materiell** geändert hat (nicht bei reinen Formulierungs-Updates)
- **Neue Zusammenhänge sind bidirektional:** Wenn ein neuer Zusammenhang zwischen Domain A und Domain B erfasst wird, prüfen ob Domain B bereits zurückverweist. Fehlt der Gegeneintrag, beide Dateien aktualisieren. Stil: aufgabenorientiert — „wenn Du an X arbeitest, auch [[Y]] laden", nicht nur ein abstrakter Link.
- `.brain/state.json > capture.lastRun` und `capture.lastResult` setzen:
  - `ok` – Einträge geschrieben
  - `skipped` – keine `accept`-Kandidaten, keine Änderungen an Dateien
  - `conflict` – Durchlauf wegen ungeklärtem Konflikt pausiert

### Schritt 7: Kurzbericht

Eine knappe Zeile pro Änderung ausgeben (nicht mehr):

```
Brain capture: 2 Einträge (email-templates-communication, glossary), 5 verworfen, state → ok.
```

Wenn nichts geschrieben wurde:

```
Brain capture: keine Einträge mit Mehrwert gefunden, state → skipped.
```

Kein langer Bestätigungs-Dialog, keine Vorschau, keine Nachfragen – außer bei echtem Konflikt.

## Qualitätsregeln

- **Im Zweifel reject.** Der Auto-Mode funktioniert nur mit strengem Filter.
- Eine Zeile pro Fakt, kein erklärender Fließtext
- Jeder `accept`-Kandidat kann in einem Satz seine Mehrwert-Kategorie benennen
- Ziel-Datei immer begründet wählen
- Neue Domain-Datei nur anlegen wenn mindestens `capture.minFactsForNewDomain` `accept`-taugliche Fakten vorhanden
- Wenn eine Coverage-Lücke erkannt wird, aber noch nicht genug Fakten vorliegen: für `brain audit` vormerken, nicht improvisieren
- Capture darf kein stilles Rewriting bestehender Einträge sein – nur Konflikt-Auflösung mit klarer Prioritäts-Begründung
