# Brain Audit

Dieser Prompt wird ausgeführt wenn der User `brain audit` aufruft oder die KI auf einen fälligen Audit hinweist.
Ziel: Wissensbasis auf Aktualität, Vollständigkeit, Konsistenz und Nützlichkeit prüfen – **aktivitätsbasiert per Git**, nicht nur kalenderbasiert.

## Ablauf

### Schritt 1: Status prüfen

`.brain/state.json` und `.brain/config.json` lesen. Letzten Audit, Intervall und Review-Defaults ausgeben.

### Schritt 2: Alle Wissensdateien scannen

Dateien: `CLAUDE.md`, `.brain/glossary.md`, alle `.brain/domains/*.md`, `PRIVATE.md` (falls vorhanden).

Frontmatter jeder Domain einlesen, insbesondere `modified`, `review_after`, `status`, `watched_paths`.

### Schritt 3: Basis-System prüfen

Bevor Inhalte bewertet werden, prüfen:
- Stimmen `_template.md` und `brain init` strukturell überein?
- Verweist `.brain/config.json > domains` auf die tatsächlich vorhandenen Domain-Dateien?
- Ist `.brain/glossary.md` vorhanden wenn `CLAUDE.md` darauf verweist?
- Wurden `init.lastRun`, `capture.lastRun`, `audit.lastRun` sinnvoll gepflegt oder ist der State offensichtlich veraltet?
- Enthalten `PROTOCOL.md`, `capture.md` und `audit.md` widersprüchliche Regeln?
- Fehlt `watched_paths` in Domains, obwohl Code-Pfade eindeutig zuordenbar wären?

### Schritt 4: Git-Aktivität pro Domain (aktivitätsbasiertes Review)

Für jede Domain mit nicht-leerem `watched_paths`:

```
git log --since=<modified> --name-only --pretty=format: -- <watched_paths ...>
```

- Gibt es Commits an diesen Pfaden nach `modified`? → Domain ist **aktivitätsbasiert review-fällig**.
- Commit-Count und jüngstes Commit-Datum notieren.
- Wenn `watched_paths` leer ist und Domain-Frontmatter-Kommentar nicht begründet, warum → als Befund melden.

### Schritt 5: Reverse-Index generieren

Schreibe `.brain/_index.json`:

```json
{
  "generated": "<ISO-Datum>",
  "byPath": {
    "<pfad-oder-glob>": ["<domain1>", "<domain2>"]
  }
}
```

Zweck: Bei Code-Arbeit kann die KI vor Glob/Grep prüfen, ob ein Pfad zu einer Domain gehört. Datei wird nicht manuell gepflegt, sondern von `audit` (und optional von `capture`) neu geschrieben.

### Schritt 6: Coverage-Report (aktivitätsbasiert)

Laufe `git log --since=<audit.lastRun || 90 Tage> --name-only --pretty=format:` und sammle geänderte Pfade. Für jeden Pfad: matcht irgendein `watched_paths`-Eintrag? Wenn nein → Kandidat für Coverage-Lücke.

Cluster die unzugeordneten Pfade nach Verzeichnis/Modul und zeige die Top-5 als „eventuell fehlende Domain" an. Nicht automatisch anlegen.

### Schritt 7: Vier Dimensionen prüfen

**Coverage – Fehlen zentrale Domänen?**
- Unzugeordnete Pfade mit auffälliger Aktivität (aus Schritt 6)?
- Begriffe in CLAUDE.md oder Domain-Dateien die nicht in glossary.md erklärt sind?
- Hat CLAUDE.md Einträge die eigentlich Domain-Dateien sein sollten?
- Sind alle `.brain/domains/*.md` (außer `_template.md`) im Domain-Index von `CLAUDE.md` gelistet? Fehlende Einträge als Coverage-Lücke melden und als auto-safe ergänzen.

**Freshness – Sind Dateien veraltet?**
- `review_after`-Datum überschritten?
- Git-Aktivität auf `watched_paths` seit `modified` (Schritt 4)?
- `status: geplant` obwohl Feature implementiert?
- `status: aktiv` obwohl Feature abgelöst oder entfernt?
- `status: stub`/`stale` obwohl Domain nachweislich gepflegt/weiterentwickelt wird?

**Consistency – Gibt es Widersprüche?**
- Gleiche Aussage in zwei Dateien unterschiedlich formuliert?
- Domain-Datei widerspricht CLAUDE.md?
- Wiki-Link verweist auf nicht existierende Datei oder falschen Abschnitt?
- Zwei Domains beschreiben dasselbe Konzept?
- `watched_paths` einer Domain überlappen mit einer anderen, obwohl sie unterschiedliche Aussagen treffen?
- Zusammenhänge-Einträge unidirektional? (Domain A verweist auf B, B verweist nicht zurück auf A) → als Minor-Befund melden, Gegeneintrag als auto-safe ergänzen.

**Usefulness – Sind Aussagen konkret genug?**
- Einträge die sich direkt aus dem Code ergeben (kein Mehrwert)?
- Zu vage formulierte Constraints („Fehler vermeiden")?
- Abschnitte die leer oder nur mit Platzhaltern gefüllt sind?
- Fehlannahmen oder Stolpersteine die offensichtlich sind?

### Schritt 8: Befunde zusammenfassen

```
## Audit-Befunde

### Kritisch
- .brain/prompts/init.md verwendet ein anderes Domain-Schema als .brain/domains/_template.md [Basissystem]
- .brain/domains/indeed.md: status „geplant", Feature ist implementiert [Freshness]
- .brain/domains/imap-inbox.md widerspricht CLAUDE.md Zeile 8 [Consistency]

### Verbesserung
- .brain/domains/user-settings.md: 12 Commits an watched_paths seit modified=2026-02-14 – review fällig [Freshness/Git]
- .brain/domains/applicant-docs.md: watched_paths leer, obwohl src/composables/useApplicantDocumentOpener.ts klar zuordenbar [Coverage]
- CLAUDE.md: Eintrag zu IMAP-Trash gehört in .brain/domains/imap-inbox.md [Coverage]

### Coverage-Kandidaten (aktivitätsbasiert)
- backend/app/Services/NotificationService.php (8 Commits, keiner Domain zugeordnet)
- frontend/src/pages/ReportsPage.vue (5 Commits, keiner Domain zugeordnet)

### Minor
- .brain/domains/applicant-docs.md: Wiki-Link [[user-settings]] ohne Zieldatei [Consistency]
- .brain/domains/kats-settings.md: review_after überschritten [Freshness]

Ergebnis: warn
```

Gesamtergebnis: `ok` | `warn` | `broken`
- `ok`: keine kritischen Befunde, keine aktivitätsbasierten Freshness-Flags
- `warn`: Verbesserungen vorhanden, aber keine Widersprüche oder falschen Angaben
- `broken`: kritische Befunde – veraltete oder widersprüchliche Informationen die KI-Fehler verursachen

### Schritt 9: Priorisierte Maßnahmen

3–5 konkrete Maßnahmen ausgeben, wichtigste zuerst:

```
1. brain init und _template.md synchronisieren [confirm]
2. user-settings.md überarbeiten, watched_paths-Aktivität einarbeiten [confirm]
3. Widerspruch IMAP klären: Domain oder CLAUDE.md korrigieren [confirm]
4. watched_paths in applicant-docs.md ergänzen [auto-safe falls eindeutig]
5. Wiki-Link in applicant-docs.md reparieren [auto-safe]
```

### Schritt 10: Schreiben und State aktualisieren

Auto-safe-Korrekturen direkt ausführen. Confirm-Änderungen zur Bestätigung vorlegen.

`.brain/_index.json` schreiben (Ergebnis aus Schritt 5).

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
- Auto-safe nur für: Pfade, Links, Tippfehler, eindeutige Status-Updates, Reverse-Index-Regeneration
- Ein Audit ist auch dann `warn`, wenn Inhalte gut sind, aber das Basis-System sich selbst widerspricht
- Git-basierte Befunde nie ohne Commit-Evidenz (Anzahl + jüngstes Datum) melden
