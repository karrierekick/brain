# Brain Init

Dieser Prompt wird ausgeführt wenn der User `brain init` aufruft.
Ziel: Wissensbasis für ein Projekt von Grund auf aufbauen.

---

## Schritt 1: Vollscan

Analysiere das Projekt automatisch:
- Repo-Struktur und Hauptverzeichnisse
- Vorhandene Doku-Dateien (CLAUDE.md, AGENTS.md, README, PLANNING.md)
- Tech-Stack (Frameworks, Sprachen, Datenbank, Queue-System, externe Dienste)
- Erkennbare Hauptmodule (Routes, Controllers, Services, Stores, Pages)
- Nicht-offensichtliche Verkopplungen zwischen Modulen
- Vorhandene Domain-Dateien in `.brain/domains/`
- Vorhandenes Glossar `.brain/glossary.md`
- Vorhandene Zustände in `.brain/config.json` und `.brain/state.json`

Fasse die Erkenntnisse kompakt zusammen: „Ich habe folgendes erkannt: …"

---

## Schritt 2: Fragebogen generieren

Baue aus dem Scan-Ergebnis ein **Ausfüll-Formular** für den User.

### Universelle Felder (immer enthalten)

```
PROJEKTZIEL:
  → (1–2 Sätze: Kernnutzen für den Anwender, nicht die Technik)

DOMÄNEN:
  → (aus Scan: bestätigen ✓, streichen ✗, ergänzen)

DATENLÖSCHUNG:
  → [ ] hart löschen  [ ] soft delete  [ ] gemischt  [ ] unklar

BEWUSST VERMIEDEN:
  → Bibliotheken/Dienste: (z.B. "kein Redis", "kein Elasticsearch")
  → Architektur-Patterns: (z.B. "keine God-Classes")
  → Scope-Grenzen: (z.B. "keine interne Stellenverwaltung")
```

### Scan-getriebene Felder (nur hinzufügen wenn Signal erkannt)

Füge je Treffer einen passenden Block hinzu:

| Signal im Scan                              | Block hinzufügen                                                                                   |
|---------------------------------------------|----------------------------------------------------------------------------------------------------|
| Queue / Jobs / Scheduler                    | `ASYNC-VERARBEITUNG: → [ ] alles async  [ ] nur bestimmte Operationen  [ ] sync, kein Queue`      |
| Externe API-Clients (HTTP, SDK)             | `EXTERNE DIENSTE: → Rate-Limit-Handling? Retry-Strategie?`                                        |
| Inbound-Daten (E-Mail, Webhook, Feed)       | `DATENEINGANG <Quelle>: → [ ] direkt in DB  [ ] Staging/Queue  [ ] unklar`                        |
| Auth + mehrere User-Typen / Rollen          | `BERECHTIGUNGEN: → Rollen: … / Wer darf was NICHT sehen:`                                         |
| SaaS / Tenant-Felder in DB                  | `MANDANTENTRENNUNG: → Strategie: …`                                                                |
| Komplexes Statusmodell (state/status-Felder)| `STATUSÜBERGÄNGE <Entität>: → Welche sind businesskritisch / haben Seiteneffekte?`                 |
| FE + BE mit State-Management                | `BUSINESS-LOGIK: → [ ] immer im Backend  [ ] FE darf Regeln berechnen  [ ] gemischt`              |
| Automatisierungen / Trigger / Cron          | `AUTOMATIONEN: → [ ] ereignis-getrieben  [ ] zeitgesteuert  [ ] beides`                           |
| DSGVO-relevante Personendaten               | `DATENSCHUTZ: → Löschfristen? Einwilligungs-Tracking? Datenexport?`                               |
| Multi-Repo / mehrere Services               | `SERVICE-GRENZEN: → Welche Services kommunizieren wie? Sync oder async?`                          |

### Constraints-Block (immer ans Ende)

```
WEITERE CONSTRAINTS:
  → (harte Grenzen die die KI kennen muss, nicht oben abgedeckt)
```

---

## Schritt 3: Formular vorlegen

Präsentiere das fertige Formular als **einen Codeblock**, den der User direkt bearbeitet und zurückschickt:

```
Ich habe das Projekt gescannt. Bitte fülle die Felder aus und schick den Block zurück.
Felder die ich bereits erkannt habe, habe ich vorausgefüllt – einfach bestätigen oder korrigieren.

---
PROJEKTZIEL:
  →

DOMÄNEN (✓ bestätigen, ✗ entfernen, ergänzen):
  → Modul A ?, Modul B ?, ...

[...scan-getriebene Felder...]

BEWUSST VERMIEDEN:
  → Bibliotheken/Dienste:
  → Architektur-Patterns:
  → Scope-Grenzen:

WEITERE CONSTRAINTS:
  →
---
```

Erkannte Vorschläge im Formular vorausfüllen, mit `?` markieren wenn unsicher.
**Warte auf die ausgefüllte Antwort des Users. Kein Kommentar, keine Rückfragen davor.**

---

## Schritt 4: Strukturvorschlag

Schlage nach dem Ausfüllen vor:
- Inhalt für `CLAUDE.md` (Projektüberblick, Design-Prinzipien, Domain-Index)
- Welche Domain-Dateien angelegt oder erweitert werden
- Welche Glossar-Begriffe direkt seedbar sind
- Welches `review_after` pro Domain sinnvoll ist (Status + Änderungsrisiko beachten)

Kennzeichne klar:
- ✅ Sicher bekannt
- ⚠️ Annahme – bitte bestätigen
- ❓ Unklar – wird als TODO markiert

Warte auf Bestätigung.

---

## Schritt 5: Dateien anlegen

- `CLAUDE.md` mit Projektüberblick, Design-Prinzipien, Domain-Index-Tabelle
- `.brain/glossary.md` anlegen oder ergänzen
- Domain-Dateien in `.brain/domains/` nach Template
- `.brain/config.json` → `project`-Feld und erkannte Domänen aktualisieren
- `.brain/state.json` → `init.lastRun` setzen

Beim Anlegen von Domain-Dateien:
- `review_after` immer setzen
- Wert standardmäßig aus `.brain/config.json > review.defaultDaysByStatus` ableiten
- Wenn Status unklar ist: konservativ `aktiv` + kürzerer Review-Zeitraum oder `<!-- TODO: Status unklar -->`

---

## Domain-Datei-Template

```markdown
---
domain: <bezeichner>
status: aktiv
modified: <YYYY-MM-DD>
review_after: <YYYY-MM-DD>
tags: [<tag1>, <tag2>]
watched_paths:
  - <relativer Pfad oder Glob>
---

## Ziel
Ein Satz: Was soll dieses Feature erreichen?

## Review-Auslöser
Freitext-Ergänzung zu `watched_paths`: fachliche Auslöser, die per Glob nicht erkennbar sind (externe API-Vertragsänderung, Produktentscheidung, Rechtsgrundlage).

## Einstiegspunkte
- Frontend: `src/...`
- Backend: `app/...`

## Nicht verwechseln mit
Abgrenzung zu ähnlichen Konzepten, Dateien oder Domänen die leicht verwechselt werden.

## Zusammenhänge
Querverweise auf andere Domänen mit [[wiki-link]].

## Design-Entscheidungen
Bewusste Entscheidungen mit Begründung, die nicht aus dem Code erkennbar sind.

## Harte Constraints
Was gilt explizit und darf nicht gebrochen werden. Was wird bewusst vermieden.

## Typische Fehlannahmen
Konzeptuelle Missverständnisse die immer wieder zu falschen Lösungen führen – nicht das gleiche wie Stolpersteine.

## Stolpersteine
Nicht-offensichtliche technische Fallstricke bei der Implementierung.
```

**status-Werte:** `aktiv`, `beta`, `stub` (Domain bekannt, Inhalt unvollständig), `stale` (bewusst nicht weiterentwickelt, nicht als Entscheidungsgrundlage nutzen), `eingestellt`.

**watched_paths:** Glob-Liste der Code-Pfade, die diese Domain betreffen. `brain audit` prüft per Git auf Commits seit `modified` und flaggt review-fällige Domains aktivitätsbasiert. Leere Liste nur mit Begründung im Dateikommentar (z. B. reine Business-Domain ohne direkten Code-Bezug).

---

## Qualitätsregeln

- `CLAUDE.md` bleibt kurz: Projektüberblick max. 3 Sätze, Domain-Index als Tabelle
- Domain-Dateien beschreiben das **Warum**, nicht das Was (Was steht im Code)
- Domain-Dateien müssen mit `_template.md` konsistent bleiben; kein eigenes Schatten-Template pflegen
- Keine Datei-Listings, keine Framework-Selbstverständlichkeiten
- Bestehende Doku verdichten, nicht kopieren
- Offene Lücken mit `<!-- TODO: unklar -->` markieren statt raten
- `init` ist erst vollständig, wenn `.brain/state.json > init.lastRun` aktualisiert wurde
