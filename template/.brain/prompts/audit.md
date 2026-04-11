# Brain Audit

Dieser Prompt wird ausgeführt wenn der User `brain audit` aufruft oder die KI auf einen fälligen Audit hinweist.
Ziel: Wissensbasis auf Aktualität, Vollständigkeit, Konsistenz und Nützlichkeit prüfen.

## Ablauf

### Schritt 1: Status prüfen

`.brain/state.json` und `.brain/config.json` lesen. Letzten Audit, Intervall und Review-Defaults ausgeben.

### Schritt 2: Alle Wissensdateien scannen

Dateien: `CLAUDE.md`, `.brain/glossary.md`, alle `.brain/domains/*.md`, `PRIVATE.md` (falls vorhanden).

### Schritt 3: Basis-System prüfen

Bevor Inhalte bewertet werden, prüfen:
- Stimmen `_template.md` und `brain init` strukturell überein?
- Verweist `.brain/config.json > domains` auf die tatsächlich vorhandenen Domain-Dateien?
- Ist `.brain/glossary.md` vorhanden wenn `CLAUDE.md` darauf verweist?
- Wurden `init.lastRun`, `capture.lastRun`, `audit.lastRun` sinnvoll gepflegt oder ist der State offensichtlich veraltet?
- Enthalten `PROTOCOL.md`, `capture.md` und `audit.md` widersprüchliche Regeln?

### Schritt 4: Vier Dimensionen prüfen

**Coverage – Fehlen zentrale Domänen?**
- Gibt es aktive Features im Code ohne Domain-Datei?
- Gibt es Begriffe in CLAUDE.md oder Domain-Dateien die nicht in glossary.md erklärt sind?
- Hat CLAUDE.md Einträge die eigentlich Domain-Dateien sein sollten?

**Freshness – Sind Dateien veraltet?**
- `review_after`-Datum überschritten?
- `status: geplant` obwohl Feature implementiert?
- `status: aktiv` obwohl Feature abgelöst oder entfernt?
- `modified`-Datum alt trotz erkennbar aktiver Code-Bereiche?

**Consistency – Gibt es Widersprüche?**
- Gleiche Aussage in zwei Dateien unterschiedlich formuliert?
- Domain-Datei widerspricht CLAUDE.md?
- Wiki-Link verweist auf nicht existierende Datei oder falschen Abschnitt?
- Zwei Domains beschreiben dasselbe Konzept?

**Usefulness – Sind Aussagen konkret genug?**
- Einträge die sich direkt aus dem Code ergeben (kein Mehrwert)?
- Zu vage formulierte Constraints ("Fehler vermeiden")?
- Abschnitte die leer oder nur mit Platzhaltern gefüllt sind?
- Fehlannahmen oder Stolpersteine die offensichtlich sind?

### Schritt 5: Befunde zusammenfassen

```
## Audit-Befunde

### Kritisch
- .brain/prompts/init.md verwendet ein anderes Domain-Schema als .brain/domains/_template.md [Basissystem]
- .brain/domains/indeed.md: status "geplant", Feature ist implementiert [Freshness]
- .brain/domains/imap-inbox.md widerspricht CLAUDE.md Zeile 8 [Consistency]

### Verbesserung
- .brain/state.json: init.lastRun fehlt trotz vorhandener init-Artefakte [Basissystem]
- CLAUDE.md: Eintrag zu IMAP-Trash gehört in .brain/domains/imap-inbox.md [Coverage]
- .brain/domains/applicant-docs.md: Abschnitt "Typische Fehlannahmen" leer [Usefulness]

### Minor
- .brain/domains/applicant-docs.md: Wiki-Link [[user-settings]] ohne Zieldatei [Consistency]
- .brain/domains/kats-settings.md: review_after überschritten [Freshness]

Ergebnis: warn
```

Gesamtergebnis: `ok` | `warn` | `broken`
- `ok`: keine kritischen Befunde
- `warn`: Verbesserungen vorhanden, aber keine Widersprüche oder falschen Angaben
- `broken`: kritische Befunde – veraltete oder widersprüchliche Informationen die KI-Fehler verursachen

### Schritt 6: Priorisierte Maßnahmen

3–5 konkrete Maßnahmen ausgeben, wichtigste zuerst:

```
1. brain init und _template.md synchronisieren [confirm]
2. .brain/state.json: init.lastRun prüfen/setzen [confirm]
3. Widerspruch IMAP klären: Domain oder CLAUDE.md korrigieren [confirm]
4. .brain/domains/applicant-docs.md > Typische Fehlannahmen befüllen [confirm]
5. Wiki-Link in applicant-docs.md reparieren [auto-safe]
```

### Schritt 7: Schreiben und State aktualisieren

Auto-safe-Korrekturen direkt ausführen. Confirm-Änderungen zur Bestätigung vorlegen.

Danach `.brain/state.json` aktualisieren:

```json
{
  "audit": {
    "lastRun": "<ISO-Datum>",
    "lastResult": "ok|warn|broken"
  }
}
```

## Qualitätsregeln

- Nichts löschen ohne Bestätigung
- Domain-Dateien nicht zusammenführen ohne explizite Zustimmung
- Widersprüche nie stillschweigend auflösen – immer eskalieren
- Auto-safe nur für: Pfade, Links, Tippfehler, Status-Updates die eindeutig sind
- Ein Audit ist auch dann `warn`, wenn Inhalte gut sind, aber das Basis-System sich selbst widerspricht
