# Changelog

Versionshistorie des Brain-Frameworks. Architektur-Rationale siehe [DESIGN.md](./DESIGN.md).

## [2.3] – 2026-04-21

### Added
- **Projekt-spezifische brain-Befehle** – Jede Datei `.brain/prompts/<befehl>.md` definiert automatisch `brain <befehl>`. Damit Cursor den Trigger zuverlässig erkennt, muss der Befehl zusätzlich in der Liste in `PROTOCOL.md` und in der `description` der Rule `.cursor/rules/brain-protocol.mdc` eintragen werden.
- **Rule-Template generischer formuliert** – `description` nennt jetzt explizit alle brain-Befehle (init/capture/audit + Platzhalter für projekt-spezifische), damit Cursors Rule-Selector bei jedem `brain …`-Prompt zieht. `alwaysApply: true` bleibt.
- **`SKILL.md` abgegrenzt** – klare Trennung: Skill behandelt nur `brain install` / `brain upgrade`; Knowledge-Operationen laufen über die projekteigene Rule. Zusätzlich: Beim Upgrade werden nur die vier Framework-Prompts überschrieben; projekt-spezifische Prompts bleiben erhalten.

## [2.2] – 2026-04-20

### Changed
- **`brain capture` läuft autonom** – `capture.confirmDefault` defaulted auf `false`. Kein Vorschlags-/Bestätigungs-Dialog mehr am Session-Ende; die KI schreibt direkt nach strenger Qualitätsprüfung. Nur ungeklärte Konflikte (Prioritätshierarchie ohne eindeutigen Sieger) pausieren den Lauf für Rückfrage.
- **Schärfere Aufnahmekriterien** – dritte Aufnahme-Bedingung fordert eine explizit benennbare Mehrwert-Kategorie: Recherche-Ersparnis, Token-Reduktion, besserer Kontext, höhere Codequalität durch Verständnis, oder Design-Entscheidung. Wer die Kategorie nicht in einem Satz benennen kann: `reject`. Ersetzt die weichere frühere Formulierung „verhindert Fehler oder Rückfragen".
- **Harte Ausschlussregel ergänzt** – Erkenntnisse ohne klar benennbare Mehrwert-Kategorie werden nie aufgenommen.
- **Session-End-Output wird knapper** – einzeilige Statusmeldung statt mehrzeiligem Vorschlags-Dialog. Leere Durchläufe enden mit `lastResult: "skipped"`.
- **Capture-Stufe `review` entfällt im Auto-Mode** – unsichere Kandidaten werden `reject`. Wer sie diskutieren will, setzt `capture.confirmDefault: true` zurück.

### Added
- **`PROTOCOL.md` – Abschnitt „Wann Brain-Lektüre übersprungen werden darf"** – Skip-Regeln für einfache Aufgaben: Ein-Stellen-Änderungen ohne fachliche Entscheidung, vom User explizit benannte Dateien, explorative Such-Aufgaben, reine Code-Zustands-Fragen, projekt-agnostische Snippet-Requests. Brain ist Werkzeug gegen falsche Annahmen, nicht Ritual.
- **`state.json > capture.lastResult: "skipped"`** – semantisch eigener Erfolgszustand, wenn keine `accept`-Kandidaten gefunden wurden.
- **`DESIGN.md` – zwei neue Rationale-Abschnitte:** „Auto-Capture mit strengem Mehrwert-Filter (v2.2)" und „Skip-Regeln für einfache Aufgaben (v2.2)".

### Migration 2.1 → 2.2

1. `brain upgrade` (Framework-Dateien werden aktualisiert, Domain-Inhalte bleiben).
2. In `.brain/config.json`:
   - `version` auf `"2.2"` setzen.
   - `capture.confirmDefault` auf `false` setzen (oder Feld weglassen – Default ist nun `false`).
3. Keine weiteren Eingriffe nötig. Der nächste Capture-Lauf läuft autonom unter dem verschärften Filter. Wer den alten Bestätigungs-Modus behalten will, setzt `capture.confirmDefault: true`.

Keine Breaking Changes bei bestehenden Domain-Dateien oder Befehlsoberfläche.

## [2.1] – 2026-04-20

### Added
- **`watched_paths` im Domain-Frontmatter** – Glob-Liste der Code-Pfade, die die Domain betreffen. Basis für aktivitätsbasiertes Review.
- **Git-basierter Review-Check in `brain audit`** – prüft per `git log --since=<modified> -- <watched_paths>` auf Commits seit letzter Pflege und flaggt Domains aktivitätsbasiert als review-fällig.
- **Reverse-Index `.brain/_index.json`** – wird von `brain audit` generiert; Map `<pfad> → <domain>`. PROTOCOL.md verweist darauf als Discovery-Hilfe vor Glob/Grep.
- **Coverage-Report in `brain audit`** – listet Code-Pfade mit Git-Aktivität, die keiner Domain zugeordnet sind.
- **Status-Werte `stub` und `stale`** – jetzt fünf Stufen: `aktiv | beta | stub | stale | eingestellt`. `stale`-Domains dürfen Kontext liefern, aber nicht Grundlage für neue Entscheidungen sein.
- **`config.audit.gitActivityWindowDays`** – konfiguriert das Zeitfenster für Coverage-Report-Aktivität (Default 90).
- **`DESIGN.md` und `CHANGELOG.md`** im brain-Repo (User-Doku vs. Architektur-Rationale vs. Versionshistorie getrennt).

### Changed
- **PROTOCOL.md Trigger-Hinweise verschärft:**
  - `watched_paths`-Match macht Domain zur Pflichtlektüre (vorher: Trigger-Hinweis abhängig von Begriffs-Match).
  - `brain capture` nach Code-Änderung an `watched_paths`-Pfad ist Pflicht-Vorschlag (vorher: optional bei „substantiellen Erkenntnissen").
- **`review.defaultDaysByStatus`** – Defaults für `stub` (30) und `stale` (180) ergänzt.
- **`config.json` Version auf `2.1`** – bestehende Installationen brauchen `brain upgrade` + `brain audit` zum Retrofit der `watched_paths`.

### Migration 2.0 → 2.1

1. `brain upgrade` ausführen (Framework-Dateien werden aktualisiert, Domain-Inhalte bleiben).
2. `brain audit` ausführen – Audit zeigt, welche Domains `watched_paths` noch fehlen.
3. Pro Domain `watched_paths` im Frontmatter ergänzen. Ableitung meist direkt aus dem Abschnitt „Einstiegspunkte" möglich.
4. Optional: Status `aktiv` bei ungepflegten Features auf `stale`/`stub` umstellen.
5. Erneut `brain audit` – jetzt läuft die aktivitätsbasierte Prüfung vollständig.

Keine Breaking Changes bei bestehenden Befehlen. `watched_paths` ist optional; fehlende Angabe erzeugt nur einen Audit-Befund, keinen Fehler.

## [2.0] – 2026-04-11

### Added
- Ersterscheinung als portables Skill-System.
- Kommandos: `brain install`, `brain init`, `brain capture`, `brain audit`, `brain upgrade`.
- Domain-Schema mit Frontmatter (`domain`, `status`, `modified`, `review_after`, `tags`).
- Protokoll mit kanonischer Wahrheitshierarchie und State-Semantik.
