# Brain Capture

Dieser Prompt wird ausgeführt wenn der User `brain capture` aufruft oder die KI es am Session-Ende vorschlägt.
Ziel: Dauerhaft nützliche Erkenntnisse aus dieser Session in die Wissensbasis einordnen.

## Aufnahmekriterien

Ein Kandidat wird nur aufgenommen wenn alle drei zutreffen:
1. Wiederverwendbar – nützlich über diese Session hinaus
2. Nicht sauber aus dem Code ableitbar – kein offensichtliches Framework-Verhalten, keine trivialen Ableitungen
3. Verhindert künftige Fehler oder Rückfragen

Wenn auch nur eine Bedingung nicht erfüllt ist: `reject`.

Lies vor dem Schreiben `.brain/config.json`, um mindestens diese Werte zu kennen:
- `capture.minFactsForNewDomain`
- `review.defaultDaysByStatus`
- `files.glossary`

## Harte Ausschlussregeln

Niemals aufnehmen, ohne Ausnahme:
- Task-spezifische Details ohne Wiederverwendungswert
- Framework-Standardverhalten
- Code-Zustände ohne Abstraktion (z.B. "Datei X hat 150 Zeilen")
- Einmalige Debug-Erkenntnisse
- Dateiaufzählungen ohne semantischen Mehrwert
- Vermutungen ohne explizite Kennzeichnung als `<!-- unsicher -->`

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

- `accept` – klar relevant, sicher korrekt, passt in bestehende Struktur
- `review` – relevant aber unsicher, widersprüchlich oder schwer einzuordnen
- `reject` – erfüllt Aufnahmekriterien nicht
- `conflict` – widerspricht einem bestehenden Eintrag in der Wissensbasis

Nur `accept` und `conflict` werden weiterverarbeitet. `review` dem User zeigen mit Hinweis warum unsicher. `reject` still verwerfen.

### Schritt 3: Ziel bestimmen

Pro `accept`-Kandidat entscheiden:
- `CLAUDE.md` – betrifft das gesamte Projekt (Design-Prinzip, globaler Constraint)
- `.brain/domains/<name>.md` – betrifft eine bestehende Domäne
- `.brain/glossary.md` – neuer Begriff oder Korrektur eines bestehenden
- Neue Domain-Datei – neues Feature das noch nicht erfasst ist
- `PRIVATE.md` – persönlich, nicht team-relevant
- `none` – kein echter Mehrwert nach nochmaliger Prüfung

Wenn ein relevanter Kandidat keiner bestehenden Domain sauber zugeordnet werden kann:
- Coverage-Lücke markieren
- Neue Domain-Datei nur anlegen wenn mindestens `capture.minFactsForNewDomain` eigenständige Fakten vorliegen
- Sonst als Audit-Maßnahme notieren statt schwache Mini-Domain anzulegen

### Schritt 4: Deduplizieren

Betroffene Datei lesen und prüfen:
- Bereits enthalten (gleich oder ähnlich)? → verwerfen
- Ergänzt bestehenden Eintrag? → einordnen, nicht neu anlegen
- Widerspricht bestehendem Eintrag? → zu `conflict` hochstufen

### Schritt 5: Konflikte behandeln

Für jeden `conflict`-Kandidaten:

```
KONFLIKT: .brain/domains/kats-settings.md > Harte Constraints

Bestehend: "PUT /api/user-settings/kats ist Partial-Patch, null entfernt Override"
Neu (aus Session): "null setzt auf leeren String, nicht auf Config-Default zurück"

Priorität prüfen:
  User-Aussage in dieser Session schlägt Domain-Datei
  Code-Verhalten schlägt Inferenz
  Jüngere bestätigte Aussage schlägt ältere

Empfehlung: bestehenden Eintrag ersetzen [j] / beide behalten mit Konflikt-Marker [k] / verwerfen [n]
```

**Prioritätshierarchie bei Konflikten:**
1. Explizite User-Aussage in dieser Session
2. Verifikation durch Code-Inspektion
3. Bestehender Domain-Eintrag (verified)
4. Inferenz aus Kontext

Konflikte nie stillschweigend auflösen.

### Schritt 6: Vorschläge zeigen

Alle `accept`-Kandidaten gebündelt zeigen:

```
Capture-Vorschläge (3 Einträge):

[1] accept → .brain/domains/indeed.md > Design-Entscheidungen
    "Bewerbungen werden über Queue-Job verarbeitet um Indeed-API Rate-Limits einzuhalten."

[2] accept → .brain/glossary.md
    "Primärer Admin: User mit kleinster id und role = admin; für Admin-Overrides verwendet."

[3] review → .brain/domains/imap-inbox.md > Stolpersteine
    "Trash-Service löscht nach 30 Tagen – Basis unklar, aus Kontext erschlossen."
    → Unsicher: nicht aus Code bestätigt. Aufnehmen?

Alle bestätigen [j] | Einzeln [e] | Abbrechen [n]
```

### Schritt 7: Schreiben

Nur nach Bestätigung.

Beim Schreiben:
- `modified` immer auf heute setzen
- `review_after` auf Basis von `.brain/config.json > review.defaultDaysByStatus` aktualisieren, wenn sich die Aussage einer Domain materiell geändert hat
- `.brain/state.json > capture.lastRun` und `capture.lastResult` setzen (`ok` | `warn` | `conflict`)

## Qualitätsregeln

- Lieber `reject` als schwache Doku
- Eine Zeile pro Fakt, kein erklärender Fließtext
- Ziel-Datei immer begründen wenn nicht offensichtlich
- Neue Domain-Datei nur anlegen wenn mindestens `capture.minFactsForNewDomain` eigenständige Fakten vorhanden
- Wenn eine Coverage-Lücke erkannt wird, aber noch nicht genug Fakten vorliegen: nicht improvisieren, sondern für `brain audit` vormerken
